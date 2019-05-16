# SpringBoot集成H2

像个人博客这样的小规模应用不适合用MySQL，资源占用比较高，而且我们我们也不需要太多企业级的特性，此时H2数据库就非常适合。

这里我们在SpringBoot中集成嵌入式的H2数据库。

## 起步依赖配置

加入H2数据库依赖。

```xml
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <scope>runtime</scope>
</dependency>
```

如果我们使用Spring Initializr，可以直接在创建项目时勾选H2。当然，除了数据库，我们还需要SpringDataJPA进行ORM，这里就不多介绍了。

## SpringBoot项目配置

application.properties
```
spring.datasource.url=jdbc:h2:~/h2/test
spring.jpa.hibernate.ddl-auto=none
```

这里我们配置一下数据库文件的地址，以嵌入式模式进行访问。

要注意的是`spring.jpa.hibernate.ddl-auto=none`这个配置，SpringBoot默认情况下，使用嵌入式数据库时默认值为`create-drop`，这会在数据库重启时清空数据，而使用MySQL等数据库服务器默认值为`none`，不会清空数据。我们这里不希望清空数据，因此改为`none`。

## 配置H2控制台

由于我们配置的是嵌入式模式，因此一个应用程序访问数据库文件时就会将其锁定，我们SpringBoot程序运行中是无法通过其它客户端连接H2数据库文件的，但我们可以在SpringBoot项目内部配置启动H2控制台。

application.properties
```
spring.h2.console.path=/h2-console
spring.h2.console.enabled=true
```

这样我们调试时就可以访问H2控制台了，非常方便。
