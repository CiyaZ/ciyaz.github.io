# 处理异步请求和Json

异步请求中的「异步」，是指客户端发起的和主页面不同步的请求，俗称Ajax，通常传递的都是纯的数据，不会有页面渲染（HTML、CSS等）相关的信息，这个「异步」和我们后端没关系，我们的PHP该怎么处理就怎么处理，而且PHP最初的设计就是`处理请求->返回响应`，如果处理的完全就是纯数据，那简直不要太简单。

## 返回Json数据

我们直接看个例子，客户端发起一个GET请求，服务端返回一段Json加载到页面上。

user.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Ajax测试</title>
	<script src="https://cdn.bootcss.com/axios/0.18.0/axios.js"></script>
</head>
<body>
<table>
	<caption>您的用户信息</caption>
	<tr>
		<td>用户名</td>
		<td id="username"></td>
	</tr>
	<tr>
		<td>密码</td>
		<td id="password"></td>
	</tr>
</table>
<button onclick="loadData()">发起请求</button>
<script>
	function loadData() {
		axios({
			method:'get',
			url:'/user.php',
			responseType:'json'
		}).then((resp) => {
			let user = resp.data;
			document.getElementById('username').innerText = user.username;
			document.getElementById('password').innerText = user.password;
		});
	}
</script>
</body>
</html>
```

页面中，我们用到了`axios`库，这里出于简单考虑直接从CDN进行加载（因此这个例子需要联网运行），`axios`简化了恐怖的`XmlHttpRequest`，当然我们也可以用最新的`fetch`相关API。

user.php
```php
<?php
header('Content-type:application/json;charset=utf-8');
$result = array();
$result["username"] = "root";
$result["password"] = "123456";
echo json_encode($result);
```

这里我们的PHP的角色是纯的数据接口，因此没有任何和页面相关的内容。我们组织了一个关联数组，然后使用`json_encode()`函数将其序列化为Json，返回给页面。可以看到，PHP没用任何框架，5行就能实现的功能，如果用Java实现，你至少得准备200MB内存跑个SpringBoot，然后写一大串控制器的代码，如果没有项目生成模板和IDE给我们自动配置项目那就更惨了。

## 获取Json数据

我们光响应GET请求，返回Json数据还不够，我们还需要能读取POST提交的JSON数据。

index.html
```javascript
function postData() {
  axios({
    method:'post',
    url:'/user.php',
    data: {
      username: 'root',
      password : '123456'
    },
    responseType:'json'
  }).then((resp) => {
    console.log(resp.data);
  });
}
```

页面代码就略过了，上面JavaScript代码其实就是通过POST请求提交Json数据。

```php
<?php
$jsonReq = json_decode(file_get_contents('php://input'), true);
```

PHP中，我们使用`json_decode()`函数对请求内容进行Json解析。注意`file_get_contents('php://input')`，这个写法看起来比较奇怪，它的作用是以文件形式读取请求数据，而不是解析为表单。`json_decode`第二个参数为`true`表示解析为关联数组。
