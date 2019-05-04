# 创建项目

点击Create New Project，项目类型选择Maven项目，模板选择maven-archetype-webapp，填写groupId，artifactId。选择项目文件目录，创建即可。

注：

* 在Maven参数中添加archetypeCatalog=local，指定使用本地archetype-catalog.xml文件，否则因为网络原因会卡死（2019年补充：现已不需要指定该参数）
* 进入项目后可以点击maven的enable auto import，这样修改`pom.xml`后，改动会立即生效

![](res/1.png)

![](res/2.png)

![](res/3.png)

![](res/4.png)

![](res/5.png)

# 项目配置

点击project structure打开项目配置界面。

## JDK和编译等级

这里我们为集成开发环境指定JDK的编译等级。

![](res/6.png)

## 项目模块配置

### 模块结构

左侧是模块列表，右侧是该模块的结构目录树，由于是Maven的项目，这里在src/main/下创建一个java文件夹，并标记为source folder。

![](res/7.png)

### 输出路径

一般留作默认即可。

![](res/8.png)

### 模块依赖

一个模块可以依赖其他模块，依赖maven管理的类库，还有jdk和tomcat的类库。默认已经设置了jdk和maven依赖，这里因为要使用tomcat的servlet api，因此需要加入tomcat依赖。点击加号，点击Library，选择tomcat的类库（此操作前需要配置tomcat运行环境）。

![](res/9.png)

后期补充：上述做法是邪路！！不要这样添加`servlet-api`依赖，这样配置如果离开集成开发环境，将导致无法编译，应该使用Maven添加一个`scope`为`provided`的依赖。

xml
```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
```

# 运行配置

解压缩tomcat的压缩包后，先给catalina.sh和shutdown.sh添加执行权限。

idea中，点击运行的下拉三角，可以打开运行需要的tomcat服务器和运行配置信息的设置面板。

## tomcat设置

左侧点击加号添加一个tomcat服务器，右侧配置名字，tomcat路径，端口等。

![](res/10.png)

后期补充：如今Tomcat早已更新到9.x，研究学习时建议使用新版本。

## 部署设置

部署设置可以选择war包部署或者展开部署，还可以设置application context（就是请求你的应用的基础URL）。

![](res/11.png)

要注意的是，开发过程中，一定要选择war展开，否则是难以热部署的。

# 一些其他修改

## Maven依赖库版本

默认创建的maven项目，jdk版本低，使用的是servlet2.3，junit3.x，这些在新项目中都应该改成新版本了。

当然，项目Archetype也是会不断更新的，可能未来默认集成的就是新版本了。

## maven编译级别

在build中加入使用最新jdk8的编译插件。

pom.xml
```xml
<plugins>
  <!--设置编译级别和运行环境为Java8-->
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
      <source>1.8</source>
      <target>1.8</target>
    </configuration>
  </plugin>
</plugins>
```

后期补充：上述写法是过时的！现在建议通过`properties`配置编译JDK版本，顺便指定一下源码的编码。

```xml
<properties>
  <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  <maven.compiler.source>1.8</maven.compiler.source>
  <maven.compiler.target>1.8</maven.compiler.target>
</properties>
```

## 使用servlet3.0

web.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         id="WebApp_ID" version="3.0">
  <display-name>TestWebapp</display-name>

</web-app>
```

后期补充：上述写法是过时的！时至今日，Tomcat9和Servlet4.0都已经发布很长时间了，研究学习建议使用最新版本。

```xml
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
  version="4.0"
  metadata-complete="true">
</web-app>
```

参见Tomcat9项目目录`webapps`中，`examples`项目。

# 运行应用

点击绿色三角即可。

注：

* 看到hello world！是因为项目模板默认创建了一个index.jsp

![](res/12.png)
