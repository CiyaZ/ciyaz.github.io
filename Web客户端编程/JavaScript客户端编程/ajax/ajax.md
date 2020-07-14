# ajax

ajax即Asynchronous Javascript And XML，异步JavaScript和XML的HTTP网络请求。

异步HTTP请求应用十分广泛，传统网页是同步式的，一次HTTP请求响应一个HTML页面，而ajax则是异步发起HTTP请求，根据返回结果进行dom操作或其他操作，整个HTML页面不用刷新即可动态更新展示数据，因此能够实现极其复杂的交互体验。

注：随着技术发展，现在我们更倾向于使用Json作为数据交换格式，而非之前曾风靡一时的XML格式。

## XMLHttpRequest（XHR）

XMLHttpRequest对象用于发起异步请求。

### 方法

* `open(method, url, async)`：参数为请求类型，资源URL，同步还是异步（实际上只能取`true`）
* `send(str)`：异步发送消息，参数为发送的请求体字符串，参数仅用于POST，GET时send无参数即可
* `setRequestHeader(header, value)`：设置请求头信息，例如：提交表单需要设置请求头为`xhr.setRequestHeader("Content-Type","application/x-www-form-urlencoded")`

### 属性

* `responseText`：服务器的响应字符串
* `responseXML`：服务器响应的XML，和`responseText`的区别是`responseXML`返回的是XMLDocument对象，直接使用DOM解析即可。
* `onreadystatechange`：每次`readystate`改变都回调对应的函数
* `readystate`参数：
  * 0：请求未初始化
  * 1：服务器已建立连接
  * 2：请求已接收
  * 3：请求处理中
  * 4：请求已完成
* `status`：状态码，如200表示请求成功，正常完成

### 示例代码

ajax发起GET请求
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Demo AJAX</title>
</head>
<body>
	<button onclick="ajax()">click</button>
	<div id="div"></div>
	<script>
		function ajax()
		{
			var xhr = new XMLHttpRequest();
			xhr.open("GET", "test.txt");
			xhr.send();
			xhr.onreadystatechange = function()
			{
				if(xhr.readyState == 4 && xhr.status == 200)
				{
					document.querySelector("#div").innerHTML = xhr.responseText;
				}
			};
		}
	</script>
</body>
</html>
```

上述代码中，点击按钮，请求了test.txt的内容，将其以文本形式显示到div中。

## FetchAPI

在最新的浏览器中，引入了一个独立于XHR的全新ajax接口Fetch。下面是一段例子代码，实现了上方代码完全同样的功能：

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Demo AJAX</title>
</head>
<body>
<button onclick="ajax()">click</button>
<div id="div"></div>
<script>
	function ajax()
	{
		fetch('test.txt').then(
			rsp => rsp.text().then(
				txt => document.querySelector("#div").innerHTML = txt
			)
		);
	}
</script>
</body>
</html>
```

代码非常简单，就不多解释了。

当然，FetchAPI其实并没有什么特别的优势，仅仅是写法上更贴合新的JavaScript规范，对于个人项目来说倒无所谓，对于商业项目，还是不建议使用。

## ajax相关库

* JQuery：JQuery库可以说是相当强大的一个库，包含了前端开发必备的各种功能，其对XHR有一套良好的封装，具体参考JQuery相关章节
* Axios：这个库封装了冗长的XHR，而写法类似fetch，非常轻巧好用而且还保证了兼容性，在基于MVVM各种框架的项目中，是没有JQuery时的一个不错的选择，在常用库相关章节有做介绍
* Polyfill：一个能让旧浏览器用兼容性方式实现新功能（如Fetch）的库，功能非常强大也非常实用，但仅仅为了使用Fetch而引入Polyfill就有点舞文弄墨的意思了，这种做法不推荐

## 常见问题

### URL参数路径

无论XHR还是Fetch，这个URL参数的写法和页面中的资源路径是完全一致的，例如当前URL是`http://a.com/b/`：如果请求`http://xxx.com/xxx`，这是绝对路径，浏览器直接请求该路径；如果是`/xxx.txt`，`/`代表网站根目录，因此请求`http://a.com/xxx.txt`；如果是`xxx.txt`，则请求同级目录下的xxx.txt，即`http://a.com/b/xxx.txt`。

### ajax跨域

浏览器中ajax是不允许跨域请求的，这是出于安全的考虑。JavaScript能够读取`Cookie`，`Web Storage`, `indexedDB`等应用数据，如果能够跨域请求，当页面被xss攻击时，这些信息就会泄露。

但有时候确实有ajax跨域的需求，这时可以考虑服务器端加上CORS响应头、jsonp等手段解决。

### ajax上传文件

HTML5后，ajax可以通过提交手动拼装的`FormData`对象实现异步的小文件上传，对于旧浏览器，则只能考虑构造一个隐藏的iframe来解决这个问题（写法非常不优雅）。JQuery有一个`ajaxFileUpload`插件，可以方便的解决文件上传的跨浏览器兼容问题。

对于较大文件，建议使用HTML5的ArrayBuffer等新功能来实现分块上传。
