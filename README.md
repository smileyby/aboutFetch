Fetch
=====

传统Ajax指的是XMLHttpRequest(XHR)，未来现在已被[Fetch](https://fetch.spec.whatwg.org/)替代。

## Why Fetch

XMLHttpRequest是一个世纪粗糙的API，不符合关注分离（Separation of Concerns）的原则，配置和调用方式非常混乱，而且基于时间的一部模型写起来也没有现代的Promise，generator/yield，async/await友好。

Fetch的出现就是为了解决XHR的问题，拿例子来说明：

使用XHR发送一个json请求一般是这样：

```javascript

	var xhr = new XMLHttpRequest();
	xhr.open('GET', url);
	xhr.responseType = 'json';
	
	xhr.onload = function() {
	  console.log(xhr.response);
	};
	
	xhr.onerror = function() {
	  console.log("Oops, error");
	};
	
	xhr.send();

```

使用Fetch后，顿时看起来好一点：

```javascript

	fetch(url).then(function(response) {
	  return response.json();
	}).then(function(data) {
	  console.log(data);
	}).catch(function(e) {
	  console.log("Oops, error");
	});

```

使用ES6的[箭头函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/Arrow_functions)后：

```javascript

	fetch(url).then(response => response.json())
			  .then(data => console.log(data))
			  .catch(e => console.log("Oops, error", e))

```

现在看起来好很多了，但这种Promise的写法还是有Callback的影子，而且promise使用catch方法进行错误处理的方式有点奇怪。不用急，下面使用async/await来做最终优化：

> 注：async/await 是非常新的 API，属于 ES7，目前尚在 Stage 1(提议) 阶段，这是它的[完整规范](https://github.com/tc39/ecmascript-asyncawait)。使用 Babel 开启 runtime 模式后可以把 async/await 无痛编译成 ES5 代码。也可以直接使用 regenerator 来编译到 ES5。

```javascrit

	try {
	  let response = await fetch(url);
	  let data = await response.json();
	  console.log(data);
	} catch(e) {
	  console.log("Oops, error", e);
	}
	// 注：这段代码如果想运行，外面需要包一个 async function

```

使用`await`后，写一部代码就想写同步代码一样爽。`await`后面可以跟Promise对象，表示等待 Promise `resolve()`才会继续向下执行，如果Proomise`reject()`或抛出异常则会被外面的`try...catch`捕获。

另外，Fetch 也很适合做现在流行的同构应用，有人基于 Fetch 的语法，在 Node 端基于 http 库实现了 node-fetch，又有人封装了用于同构应用的 isomorphic-fetch。

> 注：同构(isomorphic/universal)就是使前后端运行同一套代码的意思，后端一般是指 NodeJS 环境。

总结一下，Fetch 优点主要有：

1.	语法简洁，更加语义化
2.	基于标准 Promise 实现，支持 async/await
3.	同构方便，使用 isomorphic-fetch

## Fetch 启用方法

先看一下Fetch原生支持率：

![](img/1.png)

原生支持率并不高，幸运的是，引入下面这些 polyfill 后可以完美支持 IE8+ ：

1.	由于 IE8 是 ES3，需要引入 ES5 的 polyfill: es5-shim, es5-sham
2.	引入 Promise 的 polyfill: es6-promise
3.	引入 fetch 探测库：fetch-detector
4.	引入 fetch 的 polyfill: fetch-ie8
5.	可选：如果你还使用了 jsonp，引入 fetch-jsonp
6.	可选：开启 Babel 的 runtime 模式，现在就使用 async/await


Fetch polyfill 的基本原理是探测是否存在 window.fetch 方法，如果没有则用 XHR 实现。这也是 github/fetch 的做法，但是有些浏览器（Chrome 45）原生支持 Fetch，但响应中有中文时会乱码，老外又不太关心这种问题，所以我自己才封装了 fetch-detector 和 fetch-ie8 只在浏览器稳定支持 Fetch 情况下才使用原生 Fetch。这些库现在 每天有几千万个请求都在使用，绝对靠谱 ！

终于，引用了这一堆 polyfill 后，可以愉快地使用 Fetch 了。但要小心，下面有坑：

## Fetch常见坑

*	Fetch 请求默认是不带 cookie 的，需要设置 fetch(url, {credentials: 'include'})
*	服务器返回 400，500 错误码时并不会 reject，只有网络错误这些导致请求不能完成时，fetch 才会被 reject。

## IE使用策略

所有版本的 IE 均不支持原生 Fetch，fetch-ie8 会自动使用 XHR 做 polyfill。但在跨域时有个问题需要处理。

IE8, 9 的 XHR 不支持 CORS 跨域，虽然提供 XDomainRequest，但这个东西就是玩具，不支持传 Cookie！如果接口需要权限验证，还是乖乖地使用 jsonp 吧，推荐使用 fetch-jsonp。如果有问题直接提 issue，我会第一时间解决。

## Fetch和标准Promise的不足

由于 Fetch 是典型的异步场景，所以大部分遇到的问题不是 Fetch 的，其实是 Promise 的。ES6 的 Promise 是基于 Promises/A+ 标准，为了保持 简单简洁 ，只提供极简的几个 API。如果你用过一些牛 X 的异步库，如 jQuery(不要笑) 、Q.js 或者 RSVP.js，可能会感觉 Promise 功能太少了。

## 没有 Deferred

Deferred 可以在创建 Promise 时可以减少一层嵌套，还有就是跨方法使用时很方便。
ECMAScript 11 年就有过 Deferred 提案，但后来没被接受。其实用 Promise 不到十行代码就能实现 Deferred：es6-deferred。现在有了 async/await，generator/yield 后，deferred 就没有使用价值了。

## 没有获取状态方法：isRejected，isResolved

标准 Promise 没有提供获取当前状态 rejected 或者 resolved 的方法。只允许外部传入成功或失败后的回调。我认为这其实是优点，这是一种声明式的接口，更简单

## 缺少其它一些方法：always，progress，finally

always 可以通过在 then 和 catch 里重复调用方法实现。finally 也类似。progress 这种进度通知的功能还没有用过，暂不知道如何替代。

## 不能中断，没有 abort、terminate、onTimeout 或 cancel 方法

Fetch 和 Promise 一样，一旦发起，不能中断，也不会超时，只能等待被 resolve 或 reject。幸运的是，whatwg 目前正在尝试解决这个问题 [whatwg/fetch#27](https://github.com/whatwg/fetch/issues/27)

## 资料

*	[WHATWG Fetch 规范](https://fetch.spec.whatwg.org/)
*	[Fetch API 简介](http://bubkoo.com/2015/05/08/introduction-to-fetch/)
*	[教你驯服 async](https://pouchdb.com/2015/03/05/taming-the-async-beast-with-es7.html)
*	[阮一峰介绍 async](http://www.ruanyifeng.com/blog/2015/05/async.html)

## 最后

Fetch 替换 XHR 只是时间问题，现在看到国外很多新的库都默认使用了 Fetch。

最后再做一个大胆预测：由于 async/await 这类新异步语法的出现，第三方的 Promise 类库会逐渐被标准 Promise 替代，使用 polyfill 是现在比较明智的做法。

一直想了解下fetch是什么，看了这篇文章还是懵懵懂懂。不过既然阿里这个千万量级的都在使用fetch，且运行良好。我们也应该尝试着用起来。

转自[https://github.com/camsong/blog/issues/2](https://github.com/camsong/blog/issues/2)









