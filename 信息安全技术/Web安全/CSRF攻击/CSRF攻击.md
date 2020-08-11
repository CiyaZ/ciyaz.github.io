# CSRF攻击

CSRF（Cross-site request forgery），跨站请求伪造，是一种危害性较大，且至今仍广泛存在的一种安全漏洞。

## 原理

CSRF原理非常简单。

我们知道，`<img>`、`<form>`等元素不受同源策略限制，那么攻击者就可以通过这类元素来伪造HTTP请求。

当用户登录目标系统后，浏览器保存其身份凭证，此时攻击者通过钓鱼邮件、钓鱼短信等方式诱导用户访问攻击者搭建的钓鱼网站，网站上通过`<img>`、`<form>`等方式伪造用户的HTTP请求，此时浏览器是具有目标系统的登录凭证的，因此攻击者的伪造请求能够成功发送。

GET、POST等伪造请求都是可以实现的，因此CSRF攻击能够造成很多意料不到的后果。

## 攻击演示

这里我们搭建了两个网站。

网站Site：

* GET `/index`：首页，必须登录才能访问，否则重定向到登录页
* POST `/reply`：发表评论，该操作必须登录
* GET `/login`：登录页面
* POST `/login`：登录

网站Evil：

* GET `/index`：攻击页面

这里我们主要看攻击页面：

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
<iframe src="/iframe" style="display: none;"></iframe>
<script>

</script>
</body>
</html>
```

主页面嵌套了一个隐藏的`<iframe>`页面，其目的是让当前页面不刷新，让受害者被攻击了还不自知。

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <form id="myform" action="http://localhost:8090/reply" method="post">
        <input type="text" name="reply" value="hahaha, fuck u!" />
    </form>
<script>
    document.getElementById('myform').submit();
</script>
</body>
</html>
```

`<iframe>`内部的页面，我们编写了一个表单，填入键值，并使用JavaScript自动提交。

这样一来，被攻击者被诱导访问我们的钓鱼网站后，就会在其毫不知情的情况下，让他的浏览器发送一个`POST /reply`请求给目标系统，达到攻击的目的。

实际情况下，攻击可能就不只是发表一条评论这么简单了，可能是修改密码、删除账号、转账等更危险的操作。

## 防御CSRF攻击

### 开发者

对于开发者来说，防御CSRF并不困难，因为现在主流的框架都有相关的安全组件。防御CSRF攻击主要有两个方面：

1. 检查请求`Referer`头信息，手动限制请求必须同源，实际上该限制还能同时实现防盗链。
2. 重要操作尽量采用POST请求，同时使用CsrfToken，将一个随机数放入用户填写的表单并存入服务端Session，用户发起请求时验证这个字段，该限制还能同时实现手抖造成的多次提交问题。（注：钓鱼网站是无法拿到CsrfToken的，同源策略限制浏览器不能读取跨域`iframe`中的DOM元素）

通常情况下，上述两种方式是同时使用的。

### 用户

由于系统开发者安全意识不足，造成的CSRF漏洞是防不胜防的。对于用户来说，只能是提高安全意识，不轻信钓鱼邮件等虚假信息，不轻易点击网页链接。使用浏览器的“隐私模式”等方式，以独立会话的形式，访问安全性未知的网站。
