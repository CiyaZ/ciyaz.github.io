# JWT身份认证

JWT（Json Web Token）常用于基于HTTP协议的数据接口权限认证，是相对于Session的另一种可行方式。

## JWT和Session区别

基于Session的认证中，服务端返回一个`sessionId`作为`cookie`存储在客户端，用户相关的信息，以及是否登录等字段，都存储在服务端，存储的键为`sessionId`。

基于JWT的认证，则是将所有信息存储在`token`中，服务端对`token`进行了签名，因此客户端无法篡改其内容。`token`在用户名和密码认证成功后即下发到客户端，客户端请求时则一般是将其带在HTTP请求头中。

![](res/1.png)

## JWT数据格式

JWT约定了一个`token`数据格式，由三部分组成：

* 头部 Header
* 载荷 Payload
* 签名 Signature

三部分由点号`.`连接，形如`Base64(Header).Base64(Payload).Signature`。

下面是一个DjangoDRF框架`djangorestframework-jwt`自动生成的`token`例子：

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoyLCJ1c2VybmFtZSI6InRlc3QiLCJleHAiOjE1OTU4NjMyOTYsImVtYWlsIjoidGVzdEBnbWFpbC5jb20iLCJvcmlnX2lhdCI6MTU5NTgzNDQ5Nn0.SwLv5sj3Bg7I9cTYzFifSe6DiR-EzbbH4MNMzP_zOAE
```

### 头部 Header

头部描述JWT的基本信息，比如使用的算法等，例子：

```json
{
    "typ":"JWT",
    "alg":"HS256"
}
```

其中，`typ`标识该数据是一个JWT的`token`，`alg`标识签名算法。

格式为Json，在`token`中是使用Base64编码的明文数据。

### 载荷 Payload

```json
{
    "user_id":2,
    "username":"test",
    "exp":1595863296,
    "email":"test@gmail.com",
    "orig_iat":1595834496
}
```

和Header一样，格式为Json，在`token`中是使用Base64编码的明文数据。

### 签名 Signature

签名是对Header和Payload的签名，签名算法在Header中标识。签名密钥由服务端保管，签名的目的是防篡改（如果不签名，客户端可以篡改payload，就能够随意切换成其它用户了）。

## HTTP请求头

具体调用需要认证的接口时，如何将`token`传递给服务端，这个和服务端实现相关。一般来说，常见的方式是以如下格式的HTTP请求头来传递：

```
Authorization: JWT <token>
```

注：各种Web框架基本都有JWT的扩展，通常不需要我们手动实现。

## JWT的优缺点

最后，我们总结下JWT相比于Session方式的优缺点。

缺点：

* 载荷信息明文存储在客户端，网络被监听时有一定隐患
* 载荷信息可能会很大，造成不必要的网络开销
* `token`一旦下发，除非自动过期，不能手动使其失效
* `token`续期难以实现

优点：

* 不需要在服务端存储数据，每次请求完全是无状态的，勉强算个优点
