# ajax

异步http请求，应用十分广泛，传统网页是同步式的，一次http请求响应一个html页面，而ajax则是使用javascript异步发起http请求，根据返回结果进行dom操作或其他操作，整个html不用刷新，因此用户体验比较好。

## XMLHttpRequest

XMLHttpRequest用于在后台和服务器交换数据，创建该对象时，直接new即可。

### 方法

* open(method, url, async)：参数为请求类型，资源URL，同步还是异步（实际上只能取true）
* send(str)：异步发送消息，参数为发送的请求体字符串，参数仅用于POST，GET时send无参数即可
* setRequestHeader(header, value)：设置请求头信息，例如：提交表单需要设置为xhr.setRequestHeader("Content-Type","application/x-www-form-urlencoded")

### 属性

* responseText：服务器的响应字符串
* responseXML：服务器响应的XML，和responseText的区别是responseXML返回的是XMLDocument对象，直接使用DOM解析即可。但是实际上，现在最常用的数据交换格式是json
* onreadystatechange：每次readystate改变都回调对应的函数
* readystate：
  * 0：请求未初始化
  * 1：服务器已建立连接
  * 2：请求已接收
  * 3：请求处理中
  * 4：请求已完成
* status：状态码，如200表示请求正常完成

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

### 常见问题

#### xhr.open()的url参数路径

这个参数的写法和html中是完全一致的，例如当前URL是`http://a.com/b/`：如果请求`http://xxx.com/xxx`，这是绝对路径，浏览器直接请求该路径；如果是`/xxx.txt`，`/`代表网站根目录，因此请求`http://a.com/xxx.txt`；如果是`xxx.txt`，则请求同级目录下的xxx.txt，即`http://a.com/b/xxx.txt`

#### ajax跨域

ajax是不能跨域请求的，这是出于安全的考虑。javascript能够读取cookie，web storage, web sql，如果能够跨域请求，当页面被xss时，这些信息就会泄露。但有时候有ajax跨域的需求，如：请求一个天气接口，结果返回为json。这种情况建议将接口调用放在服务端。网上有一些jsonp，iframe等解决方案，但是问题都比较多，不建议使用。
