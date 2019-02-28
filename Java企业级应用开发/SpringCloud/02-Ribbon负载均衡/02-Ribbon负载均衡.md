# Spring Cloud Ribbon 负载均衡

Netflix Ribbon是一个基于TCP/HTTP协议的负载均衡工具，Spring Cloud对Netflix Ribbon进行了封装，便于我们通过Spring提供的RestTemplate实现对服务的客户端负载均衡调用。

Ribbon实现的负载均衡属于客户端负载均衡，在以前无论是BS还是CS结构的web应用程序，大多都是服务端负载均衡的架构，虽然客户端负载均衡也能实现，但是没人去这么做。而微服务架构下，Ribbon这么做了，似乎也并没有什么问题。

* 基于服务端的负载均衡：服务端中有一个接收请求的组件，可以是硬件（F5）或软件（Nginx），该组件将用户的请求按照一定的算法转发给相同服务的不同实例，实现服务端负载均衡
* 基于客户端的负载均衡：实现负载均衡的软件模块处于客户端中，通常是客户端程序的一部分，它维护了一组服务列表，调用服务时，在客户端通过一定算法判断调用哪一个服务实例，实现负载均衡

在SpringCloud中，Ribbon可以结合RestTemplate使用，也可以结合Feign使用，有关Feign的内容将在其他章节介绍，这篇笔记主要介绍Ribbon结合RestTemplate实现服务调用。

## RestTemplate的使用

在Eureka相关章节中，我们已经简单演示了如何使用RestTemplate实现一个GET请求。Eureka的服务治理是通过Rest服务实现的，Spring Cloud中，Ribbon需要结合RestTemplate使用。下面简单介绍一下RestTemplate中几个常用的函数。

### 配置RestTemplate依赖注入

由于是SpringBoot项目，这里我们使用Java代码的方式在Application类中配置一个`RestTemplate`的Bean。并标注`@LoadBalanced`表示启动Ribbon实现负载均衡。

```java
@Bean
@LoadBalanced
RestTemplate restTemplate()
{
    return new RestTemplate();
}
```

### getForObject

```java
public <T> T getForObject(java.lang.String url,
                                    java.lang.Class<T> responseType,
                                    java.util.Map<java.lang.String,?> uriVariables)
                             throws RestClientException
```

* url：请求服务的地址
* responseType：返回数据的封装类型
* uriVariables：请求参数的Map结构

例子：这里我们假设调用的微服务名为`TEST-SERVICE`，REST服务的请求路径为`/users/{userId}`，请求方式为GET。

```java
@RequestMapping(value = "/consumer", method = RequestMethod.GET)
public User testConsumer()
{
  Map<String, Object> paramMap = new HashMap<>();
  paramMap.put("userId", 2);
  return restTemplate.getForObject("http://TEST-SERVICE/users/{userId}", User.class, paramMap);
}
```

### getForEntity

`getForEntity()`包含更多的响应信息，如HTTP响应码，响应头信息等。

```java
@RequestMapping(value = "/consumer", method = RequestMethod.GET)
public User testConsumer()
{
  Map<String, Object> paramMap = new HashMap<>();
  paramMap.put("userId", 2);
  ResponseEntity<User> entity = restTemplate.getForEntity("http://TEST-SERVICE/users/{userId}", User.class, paramMap);
  System.out.println(entity.getStatusCode());
  System.out.println(entity.getStatusCodeValue());
  System.out.println(entity.getHeaders());

  return entity.getBody();
}
```

相应的，REST接口还有POST、PUT、DELETE方式，参数都大同小异，参考文档即可。除此之外，getForObject和getForEntity还有很多其他重载，随意选用一种即可，这里就不多做介绍了。

### 有关返回值

如果服务提供端返回的数据是JSON数组，我们最好使用数组作为接收参数类型，而不是泛型列表List，因为JVM的泛型实现原因，Java语言不支持类似`List<User>.class`的形式，而使用`List.class`作为getForObject的接收参数类型，会报一个未检查类型转换警告，因此建议使用数组。

提供端例子代码：
```java
@RequestMapping(value = "/users", method = RequestMethod.GET)
public List<User> getUserList()
{
  return userService.getAllUser();
}
```

调用端例子代码：
```java
@RequestMapping(value = "/users", method = RequestMethod.GET)
public User[] getUserList()
{
  return restTemplate.getForObject("http://user-service/users", User[].class);
}
```

调用端如果有需要，将数组专为List也是很方便的。

## 负载均衡策略配置

默认情况下，Ribbon的负载均衡策略是轮询。除此之外，Ribbon自带了一些负载均衡策略，我们也可以实现自己的负载均衡策略。
