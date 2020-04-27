# 使用Thymeleaf模板

SpringBoot2中，后端模板引擎仅支持Thymeleaf和Freemarker，这里简单介绍一下Thymeleaf。

SpringBoot中，引入Thymeleaf模板需要依赖：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

当然，我们一般都是在`Spring initializr`中自动配置的。

在SpringBoot中，无需任何配置，Thymeleaf使用起来和SpringMVC就没什么区别，这里就不多做介绍了。
