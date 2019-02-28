# REST风格

REST（全称Representational State Transfer），是`Roy Thomas Fielding`在他2000年的博士论文中提出的。RESTful是一种网络接口的设计风格，并不是某种标准。REST对HTTP协议的利用主要有资源定位和资源操作（其实就是资源增删改查）两种方式。REST服务主要利用起了HTTP协议的GET，POST，PUT，DELETE四种动作：

* GET 查询 对应数据库SELECT
* POST 插入 对应数据库INSERT
* PUT 更新 对应数据库UPDATE
* DELETE 删除 对应数据库DELETE

REST服务使用HTTP作为传输协议，传输的内容经常是XML，JSON，图片等。在某些web应用中，前台浏览器（JavaScript）和服务器就会使用REST服务接口进行通信，Android开发中，app也经常使用REST接口和服务器通信。

REST风格要求URL参数中尽量不要有动词，因为“动作”已经在HTTP协议的动作字段中指定了，举个例子：

获取id为1的用户的所有订单
```
GET /v1/users/1/orders?_type=xml
```

* `/v1`：我们设计的API可以携带一个版本号，以免API升级时给原来用户造成不便
* `/users/1/orders`：`/users`代表所有用户集合，`/1`表示在所有用户集合中取id为1的用户，`/orders`表示这个用户的所有订单集合，REST接口的资源定位描述大致就是按照这种形式构造的
* `?_type=xml`：指定返回的数据类型是xml。REST接口可以带有参数，但是参数的作用类似于形容词，而不应该是动词

新建一个用户
```
POST /v1/users
```

修改id为1的用户信息
```
PUT /v1/users/1
```

删除id为1的用户
```
DELETE /v1/users/1
```

## 当客户端或服务端不支持PUT和DELETE时

REST相对HTTP使用的历史来说，算是个新技术。加上历史原因，某些HTTP客户端或服务器不支持PUT和DELETE请求。此时，可以用POST代替，POST中表单加一个特殊参数例如：`method=put`，用来表示这个POST请求是否按照REST形式解析。这种方式用的十分广泛。

## RESTful风格接口的使用场景

RESTful风格接口是对HTTP协议很好的诠释。使用REST接口，对网络资源的定位更加简单，RESTful风格结构清晰，便于理解。但是当URL的层级过多时，REST接口的可读性就不好了。

当然上面说的这些都是理论上的，我个人认为REST接口用于参数较少，比较简单的资源定位，十分方便。比如就像前面举的例子，查个用户，查个订单，RESTful风格的接口可读性非常好，层次十分清晰。但是一旦复杂起来就不行了，复杂的查询条件会让REST接口编写起来十分困难（换句话说就是RESTful风格下没法写出太复杂的查询条件），强行RESTful结果就是不仅接口冗长，而且可读性几乎没有，给用户带来困难。好在大多数情况下，接口都是属于前者，因此RESTful风格一定程度上得以流行。

# JAX-RS

JAX-RS（全称Java API for RESTful Web Service），是JavaEE6引入的新规范，用于以Rest架构风格编写网络服务接口，和JAX-WS用途是大致相同的，但是SOAP协议更多的用于分布式系统的协作（接口复杂，重量级），REST接口更多用于互联网应用（接口简单，轻量级）。实现JAX-RS的框架有Jersey，RESTEasy，CXF等。除了JAX-RS，SpringMVC还自己有一套REST API，就不多介绍了。
