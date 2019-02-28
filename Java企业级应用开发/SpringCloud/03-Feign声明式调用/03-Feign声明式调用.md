# Feign 声明式调用

在Ribbon章节中，我们学习了如何使用RestTemplate结合Ribbon实现具有负载均衡功能的服务调用，但是直接使用RestTemplate并不符合我们的习惯，这里我们学习使用Netflix Feign在SpringCloud中实现声明式服务调用。

## 引入Feign起步依赖

首先我们的项目要提供web访问功能，因此引入`spring-boot-starter-web`，我们的项目还是一个微服务，因此引入`spring-cloud-starter-netflix-eureka-client`，集成Feign客户端使用`spring-cloud-starter-openfeign`。

实际上这些起步依赖我都是通过Spring Initializr添加的。

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

## Feign的使用

首先我们编写一个远程服务，服务名为`user-service`，端口8081，URL为`/users/{id}`。

UserService.java
```java
@FeignClient("user-service")
public interface UserService
{
	@RequestMapping(value = "/users/{id}", method = RequestMethod.GET)
	User getUserById(@PathVariable("id") Integer id);
}
```

上面代码中我们定义了一个接口，这个接口就是我们声明的远程调用服务。接口使用`@FeignClient`标注，值为远程服务名，接口的方法和SpringMVC控制器的写法很类似，但是表达的意义不太一样，这里表达的意义是如何访问远程接口。

实际上，Feign的作用就是通过我们配置的注解，生成一个REST请求的模板，从而简化我们HTTP接口调用的开发，我们的远程调用还是通过REST请求实现的。

CloudUserFeignApplication.java
```java
@SpringBootApplication
@EnableEurekaClient
@EnableFeignClients(basePackages = {"com.ciyaz.demo.clouduserfeign.service"})
public class CloudUserFeignApplication
{
	public static void main(String[] args)
	{
		SpringApplication.run(CloudUserFeignApplication.class, args);
	}
}
```

SpringBoot入口类中，我们使用`@EnableEurekaClient`和`@EnableFeignClients`标注，注意需要在Feign相关注解中配置一下包扫描路径，否则Spring找不到Feign声明式远程接口，无法进行依赖注入。

MainController.java
```java
@RestController
public class MainController
{
	private final UserService userService;

	@Autowired
	public MainController(UserService userService)
	{
		this.userService = userService;
	}

	@RequestMapping(value = "/users/{id}", method = RequestMethod.GET)
	public User getUserById(@PathVariable Integer id)
	{
		return userService.getUserById(id);
	}
}
```

在控制器中，我们调用Feign接口，这个就和调用普通的方法一样了。

## Feign默认集成Ribbon和Hystrix

注意我们上面`pom.xml`中并没有引入Ribbon的起步依赖，但是上面我们编写的Feign客户端实际上已经具有远程服务调用的负载均衡能力了，这是因为Feign整合了Ribbon，Ribbon是Feign起步依赖的一个底层依赖，我们并不需要再手动引入一遍Ribbon的起步依赖了。

有关Hystrix将在后文介绍。
