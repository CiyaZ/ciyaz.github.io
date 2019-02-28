# Config 分布式配置中心

Spring Cloud Config主要解决的是在分布式的微服务系统中，由于服务器数量很多引起的部署、配置复杂的问题。使用Spring Cloud Config能够实现配置文件的集中管理和实时更新。

## 配置中心的工作机制

Config Server是一个单独的SpringBoot工程，其它微服务工程会配置Config Server的访问地址，启动时会自动读取Config Server上的配置信息。

## 配置Config Server

Config Server工程只需要一个依赖：
```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

配置好依赖后，需要在SpringBoot启动类上，使用`@EnableConfigServer`进行标注，开启Config Server功能。

```java
@SpringBootApplication
@EnableConfigServer
public class CloudConfigApplication
{
	public static void main(String[] args)
	{
		SpringApplication.run(CloudConfigApplication.class, args);
	}
}
```

application.properties
```
spring.application.name=config-server
server.port=8085
spring.profiles.active=native
spring.cloud.config.server.native.search-locations=classpath:/shared
```

上面配置中，我们指定使用工程`CLASSPATH:/shared`目录下的配置文件作为Config Server提供的配置文件。例如，该目录下有`CLASSPATH:/shared/demo-dev.properties`，那么名字叫做`demo`的微服务会在`dev`模式下加载这个配置文件，使用其中的配置信息和自己的配置信息合并。

src/main/resources/shared/consumer-feign-service-dev.properties
```
eureka.client.serviceUrl.defaultZone=http://localhost:8080/eureka/
```

上面例子配置中，我们的配置文件名`consumer-feign-service-dev.properties`表示名字为`consumer-feign-service`的微服务将在`dev`模式下加载该配置文件。配置文件的内容比较简单，仅仅是配置了一个Eureka地址，实际情况下则复杂得多，我们一般是分为开发、测试、预生产、生产模式，每个微服务都会准备单独的对应这四个模式的配置文件，配置文件内容包括数据库连接、Redis地址等许多信息。

## 读取Config Server配置

这里我们在一个叫做`consumer-feign-service`的微服务上，读取Config Server的信息，Eureka地址配置在了Config Server上。

bootstrap.properties
```
spring.cloud.config.uri=http://localhost:8085
spring.cloud.config.fail-fast=true
spring.profiles.active=dev
```

上面配置中指定了Config Server的地址，以及使用的环境模式是`dev`。

application.properties
```
server.port=8083
spring.application.name=consumer-feign-service
```

上面配置指定了该微服务的运行端口和该微服务的名字，注意微服务的名字需要和Config Server中配置文件的名字对应上。

启动这个服务，我们会发现该微服务正确注册到了Eureka中，读取Config Server的配置文件成功。

注，bootstrap.properties和application.properties的区别：bootstrap是系统级的配置，加载后就不会被覆盖，application是应用级别的配置。因此，和Config Server相关的配置应该写在bootstrap.properties中，因为这些配置写好一次就不会再变更了。

## Config Server从Git仓库读取配置文件

将配置文件编写在Config Server工程的`CLASSPATH:shared/`下依然不是很方便，Config Server支持把配置文件放置在Git仓库中，这样我们甚至能用版本控制系统管理整个微服务体系的配置文件。

Config Server的配置文件：

application.properties
```
spring.application.name=config-server
server.port=8085
spring.cloud.config.server.git.uri=https://gitee.com/ciyaz/config-demo.git
spring.cloud.config.server.git.default-label=master
```

配置文件中，我们指定了Git服务器的地址，以及master分支。一般情况下，我们使用的是私有库，这时通过HTTPS访问Git地址需要用户名和密码，这些也都可以通过`spring.cloud.config.server.git.username`和`spring.cloud.config.server.git.password`进行配置。除此之外，如果我们的配置文件是分目录存储的，也可以通过`spring.cloud.config.server.git.search-paths`配置文件的路径。

## 将Config Server注册到Eureka

Config Server支持将自己作为微服务，配置到Eureka中。这样Config Server也可以多实例部署，实现高可用性。下面我们演示如何将Config Server注册到Eureka。

Config Server配置：

bootstrap.properties
```
spring.application.name=config-server
server.port=8085
spring.cloud.config.server.git.uri=https://gitee.com/ciyaz/config-demo.git
spring.cloud.config.server.git.default-label=master
eureka.client.serviceUrl.defaultZone=http://localhost:8080/eureka/
```

读取配置的微服务配置：

bootstrap.properties
```
spring.cloud.config.discovery.enabled=true
spring.cloud.config.discovery.service-id=config-server
spring.cloud.config.fail-fast=true
spring.profiles.active=dev
eureka.client.serviceUrl.defaultZone=http://localhost:8080/eureka/
```

注意：`spring.cloud.config.discovery.service-id`需要和Config Server微服务的名字对应，否则是找不到这个微服务的。

application.properties
```
server.port=8083
spring.application.name=consumer-feign-service
```

说明：前面例子中，我们把Eureka地址放进了Config Server，学到这里我发现如果把Config Server配置到Eureka，其他微服务也需要通过Eureka获取Config Server微服务，那么把Eureka地址的配置信息放进Config Server肯定是不行的。这里，我们Config Server中的配置信息为：

```
foo=foo version 1
```

微服务中，通过控制器读取这个配置信息的值：

```java
@RestController
public class MainController
{
  @Value("${foo}")
  private String demo;

  @RequestMapping(value = "/demo", method = RequestMethod.GET)
  public String demo()
  {
    return demo;
  }
}
```

额外补充：在调试中，我发现一个问题，Config Server和读取Config Server的微服务中，`eureka.client.serviceUrl.defaultZone`必须配置在`bootstrap.properties`中，写在`application.pproperties`中会报错`No instances found of configserver (config-server)`。这个错误比较莫名其妙，不知道是不是bug。我使用的SpringCloud版本是`Greenwich.M3`，SpringBoot版本`2.1.0`。
