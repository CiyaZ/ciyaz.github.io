# 使用Cookie

其实Express一般都是做接口开发的，Node生态大家都在玩前后端分离，Cookie和后端模板用的真的很少，我们这里还是简单介绍一下Cookie的使用。

## 依赖

默认Express创建项目时，已经自动帮我们添加Cookie处理相关的依赖了。

```json
"cookie-parser": "~1.4.3"
```

## 使用Cookie

下面例子中，我们把用户名和密码放进了Cookie（这只是个例子，不要在真实开发场景中这么做）。

```javascript
router.post('/doLogin', function(req, res, next){
    if(req.body['username'] === 'root' && req.body['password'] === '123456') {
        res.cookie('user', {username: 'root', password: '123456'});
        res.redirect('/users');
    } else {
        res.render('login', {errMsg: '用户名或密码错误'});
    }
});
```

取出Cookie也非常简单。

```javascript
let cookieUser = req.cookies.user;
```

我们直接访问`req.cookies`属性就行了。

## 使用Session

Session其实完全可以用Cookie+Redis实现，Redis或Redis集群集中存储Session信息，也便于搭建多实例集群，这里就不多做介绍了。
