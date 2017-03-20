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
