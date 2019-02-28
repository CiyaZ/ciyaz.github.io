# SpringBoot部署

这篇笔记记录如何在生产环境中部署SpringBoot程序。实际上，我们编写完一个SpringBoot应用后，直接打包成可执行jar包，部署到生产环境是完全可以的。这里再介绍一下如何将SpringBoot程序打包成war包，并部署到应用服务器（如Tomcat）。

## 打包成war包

首先我们需要修改一下Maven的打包方式：

pom.xml
```xml
<packaging>war</packaging>
```

然后，我们需要一些Servlet配置。

DemoServletInitializer.java
```java
package com.example.demo;

import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

public class DemoServletInitializer extends SpringBootServletInitializer
{
	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder builder)
	{
		//载入配置类
		return builder.sources(DemoApplication.class);
	}
}
```

这个Java类的作用相当于`web.xml`，它引入了包含许多配置信息的`DemoApplication.class`（实际上许多配置信息都是默认配置或是间接引入的）。

然后我们执行Maven的打包命令：
```
mvn package
```

这样我们就得到了一个war包。

## 使用生成的war包的两种方式

### 方式一：部署到应用服务器

直接将war包复制到tomcat的webapps目录即可，但是我们要注意修改下context相关配置。

### 方式二：直接运行

```
java -jar demo-0.0.1-SNAPSHOT.war
```

实际上我们得到的war包中还是有主函数的，打包后`META-INF`中也指定了`MAIN-CLASS`等信息，因此也是能够直接运行的。
