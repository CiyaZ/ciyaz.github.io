# ajax之jsonp

## 跨域

只有当两个资源地址的协议、端口、主机全部相同时，才说这两个资源是同源的。在浏览器中，跨域请求一个图片、一个脚本是没有问题的，但是跨域发起ajax请求，浏览器是不允许的，这是因为ajax能够使用JavaScript代码读取跨域后的返回结果，浏览器最初设计时，就认为这具有安全问题，因此跨域ajax不被允许。

实际上，黑客有各种方法绕过浏览器的ajax跨域安全限制，而这个限制反而给应用开发者带来了麻烦，不得不说ajax跨域问题是历史原因造成的。

## 跨域ajax的解决方案

### HTTP协议头

```
Access-Control-Allow-Origin
```

该响应头信息允许特定路径的跨域访问，支持IE10以上版本的Internet Explorer，以及其他主流浏览器。

### jsonp

虽然给服务器设置一个`Access-Control-Allow-Origin`更加简单，但是如果为了兼容性考虑，还是需要用jsonp的形式实现跨域ajax请求。jsonp的原理也比较简单，下面举一个例子：

假设服务器jsonp接口的请求URL为
```
http://xxx/api?callback=showData
```

我们发起ajax请求的过程是这样的：

1. 给页面上添加一个`<script src="http://xxx/api?callback=showData"></script>`，浏览器就会按照这个地址加载一段JavaScript代码
2. 服务器返回字符串格式为`showData(真正的json数据)`
3. 我们在页面JavaScript代码里预先编写了一个函数`showData(result)`，result参数其实就是真正需要的json数据，这个函数就会恰好被服务端返回的`showData(真正的json数据)`调用，至此我们就跨域得到了真正的json数据
