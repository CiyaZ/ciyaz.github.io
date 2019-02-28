# javascript多线程

webworker出现之前，javascript只能单线程运行，耗时操作通过异步回调的方式完成，这是由javascript最开始的用途决定的，如DOM操作，多线程会带来复杂的同步问题。但是如果代码中有一个需要大量计算的耗时操作，单线程运行会导致浏览器弹出页面脚本无响应，用户的界面操作会被阻塞，用户体验很不好，因此没有webworker可能就没法实现了。

webworker可以创建一个子线程，在后台执行耗时操作。

# webworker

1. **创建子线程** var w = new Worker(url)，参数是子线程代码的url
2. **发送消息** postMessage(msg)，主线程可以发送消息给子线程，子线程使用onmessage=function(event){}接收；子线程也可以发送消息给主线程，主线程使用w.onmessage=function(event){}接收
3. **onmessage** 用于设置事件接收回调函数

例子

index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>web worker demo</title>
</head>
<body>
	<span id="count">0</span>
	<script>
		var w = new Worker("worker.js");
		w.onmessage = function(event)
		{
			document.querySelector("#count").innerHTML = event.data;
		}
	</script>
</body>
</html>
```

worker.js
```javascript
var i = 0;
function timedCount()
{
	i = i + 1;
	postMessage(i);
	setTimeout("timedCount()", 500);
}
timedCount();
```

如上述代码，直接new Worker()就能创建一个web worker对象，参数需要指定子线程js代码的url。

主线程使用onmessage回调消息处理，子线程使用postMessage()向主线程发送消息。
