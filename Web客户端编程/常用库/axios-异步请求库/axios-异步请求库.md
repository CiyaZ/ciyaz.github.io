# axios 基于Promise模型的异步请求库

axios是一个基于Promise异步模型的用于浏览器和Node的HTTP客户端。

## axios和JQuery对比

JQuery的`$.ajax()`也能实现同样的功能，但是使用JQuery有以下几个缺点：

1. 如果项目不依赖JQuery，引入JQuery比引入axios开销更大（JQuery有很多其他功能，库比较大）
2. 因为比较古老，JQuery不是基于Promise模型的，如果有大量嵌套回调，会影响代码可读性

除了JQuery，JavaScript自带的XMLHttpRequest和Fetch（ES6）也能实现同样的功能，但缺点就是代码比较复杂。因此axios通常是个不错的选择，这篇笔记我们简单介绍一下如何使用axios实现AJAX操作。

## 安装axios到项目

如果项目基于包管理工具构建，可以使用npm安装：
```
npm install --save axios
```

也可以直接引入CDN或是本地库：
```
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

## axios使用示例

axios使用起来比较简单，我们直接通过例子代码的形式进行讲解，但是使用axios前需要掌握ES6的Promise写法，可以参考笔记的相关章节：`/Web客户端编程/EcmaScript6/06-异步编程之Promise`。

因为需要和后台服务器交互，这里我们起一个SpringBoot工程进行演示，但不会展示后端代码。

我们实现了如下接口：

* POST /api/user 添加一个user
* DELETE /api/user/{username} 根据用户ID删除一个user
* PUT /api/user/{username} 根据username更新一个user信息，如果该username不存在，则创建一个
* GET /api/user 获取全部user
* GET /api/user/{username} 根据username查询一个user

## 异步GET请求

请求所有的用户信息：

```javascript
axios.get('/api/user').then((resp) => {
  console.log('执行成功');
  console.log(JSON.stringify(resp));
});
```

这里给出resp中的内容（JSON字符串）：
```json
{
  "data": {
	"users": [
	  {
		"username": "Tom",
		"password": "123"
	  },
	  {
		"username": "Jerry",
		"password": "456"
	  }
	]
  },
  "status": 200,
  "statusText": "",
  "headers": {
	"date": "Sun, 30 Sep 2018 03:12:48 GMT",
	"transfer-encoding": "chunked",
	"content-type": "application/json;charset=UTF-8"
  },
  "config": {
	"transformRequest": {},
	"transformResponse": {},
	"timeout": 0,
	"xsrfCookieName": "XSRF-TOKEN",
	"xsrfHeaderName": "X-XSRF-TOKEN",
	"maxContentLength": -1,
	"headers": {
	  "Accept": "application/json, text/plain, */*"
	},
	"method": "get",
	"url": "/api/user"
  },
  "request": {}
}
```

其中，`data`就是我们服务器接口返回的数据，除此之外，还有一些其他信息，比如`status`表示状态码之类的，这里就不多做介绍了。axios发出的所有类型的AJAX请求都会返回这样的一个json。

这里我们再尝试一个失败的AJAX请求`/api/aaa`，请求这个URL服务器会返回404：

```javascript
axios.get('/api/aaa')
  .then((resp) => {
    console.log('执行成功');
    console.log(JSON.stringify(resp));
  })
  .catch((err) => {
    console.log('失败捕获');
    console.log(JSON.stringify(err));
  });
```

这里我们使用`catch()`函数捕获执行失败的情况，实际上一般情况下，我们都会编写`then()`和`catch()`分别处理执行成功或失败的情况。

控制台输出的json如下：
```json
{
  "config": {
	"transformRequest": {},
	"transformResponse": {},
	"timeout": 0,
	"xsrfCookieName": "XSRF-TOKEN",
	"xsrfHeaderName": "X-XSRF-TOKEN",
	"maxContentLength": -1,
	"headers": {
	  "Accept": "application/json, text/plain, */*"
	},
	"method": "get",
	"url": "/api/aaa"
  },
  "request": {},
  "response": {
	"data": {
	  "timestamp": "2018-09-30T03:17:59.160+0000",
	  "status": 404,
	  "error": "Not Found",
	  "message": "No message available",
	  "path": "/api/aaa"
	},
	"status": 404,
	"statusText": "",
	"headers": {
	  "date": "Sun, 30 Sep 2018 03:17:58 GMT",
	  "transfer-encoding": "chunked",
	  "content-type": "application/json;charset=UTF-8"
	},
	"config": {
	  "transformRequest": {},
	  "transformResponse": {},
	  "timeout": 0,
	  "xsrfCookieName": "XSRF-TOKEN",
	  "xsrfHeaderName": "X-XSRF-TOKEN",
	  "maxContentLength": -1,
	  "headers": {
		"Accept": "application/json, text/plain, */*"
	  },
	  "method": "get",
	  "url": "/api/aaa"
	},
	"request": {}
  }
}
```

## 异步POST请求

这里我们调用一下POST接口，上传一个对象：

```javascript
axios.post('/api/user', u1).then((resp) => {
	console.log('执行成功');
	console.log(resp);
});
```

代码中，u1对象是如下的形式：
```javascript
let u1 = {
	username: 'Tom',
	password: '123'
};
```

## 异步PUT和DELETE

用法和GET和POST都差不多，这里就不多介绍了，使用时要注意服务器是否支持PUT和DELETE。

## 手动构建请求

axios能够通过一个配置对象手动构建请求，例子：

```javascript
axios({
  method:'get',
  url:'/api/user',
  responseType:'json'
}).then((resp) => {
      console.log('执行成功');
      console.log(JSON.stringify(resp));
});
```

## 使用await

`axios()`返回的是一个Promise对象，其实它可以作为一个`async`函数，如果你对此比较熟悉，完全可以用`await`写法取代Promise的`then()`等函数，让代码看上去更加直观（见`Web客户端编程/EcmaScript6/异步编程之async函数`），这里就不多介绍了。

## 文档

axios还有一些其他的配置，可以参考这里：[https://www.npmjs.com/package/axios](https://www.npmjs.com/package/axios)。这里就不多做介绍了。
