# SpringMVC 处理AJAX请求

AJAX通常通过JSON和后台交互数据，由于HTTP协议决定了页面的请求-响应模型是同步的，因此异步的AJAX出现了，弥补了用户体验上的不足。这篇笔记记录了SpringMVC如何处理AJAX数据。

## 处理AJAX请求

我们直接看一个使用SpringMVC处理AJAX请求的例子。

ajax.jsp
```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
	<title>AJAX Test</title>
	<script src="${pageContext.request.contextPath}/resources/jquery-2.1.4.min.js"></script>
</head>
<body>
<form onsubmit="return ajaxPost();">
	<label for="username">用户名</label>
	<input type="text" name="username" id="username">
	<label for="age">年龄</label>
	<input type="text" name="age" id="age">
	<input type="submit" value="提交">
</form>
<script>

	function ajaxPost()
	{
		var obj = {};
		obj.username = $("#username").val();
		obj.age = $("#age").val();

		$.ajax({
			type: "POST",
			async: true,
			url: "/ajax",
			data: JSON.stringify(obj),
			dataType: "json",
			contentType: "application/json; charset=utf-8",
			success: function(msg)
			{
				alert("success:" + msg);
			},
			error: function(err)
			{
				alert("error:" + msg);
			}
		});

		return false;
	}
</script>
</body>
</html>
```

页面中，我们使用AJAX，POST提交一个表单。为了简单起见，我们使用了JQuery。

控制器代码：
```java
package com.ciyaz.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class AjaxController
{
	@RequestMapping(value = "/ajax", method = RequestMethod.GET)
	public String getAjaxPage()
	{
		return "ajax";
	}

	@ResponseBody
	@RequestMapping(value = "/ajax", method = RequestMethod.POST)
	public String doAjaxPost(@RequestBody String jsonText)
	{
		System.out.println(jsonText);
		return jsonText;
	}
}
```

注意`@ResponseBody`，这个注解表示直接将返回的字符串作为响应体。`@RequestBody`则表示，`jsonText`这个参数直接读取请求体得到。这里我们实际没有解析JSON，只是在控制台上打印了请求的JSON字符串，然后原样返回。

## SpringMVC整合jackson

上面我们看到了如何使用SpringMVC处理json字符串，json字符串肯定是要被解析的，但是我们难道要自己使用json解析库，在控制器中解析吗？SpringMVC当然提供了相关的功能。

jackson是一个方便的json解析库，还能够直接将json映射成Java对象。SpringMVC可以直接整合jackson，整合后，AJAX控制器的方法和返回值可以直接使用实体类。下面我们看看如何在SpringMVC中整合jackson。

spring-mvc-demo-servlet.xml中，添加如下配置：
```xml
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
  <property name="messageConverters">
    <list>
      <ref bean="mappingJacksonHttpMessageConverter"/>
    </list>
  </property>
</bean>
<bean id="mappingJacksonHttpMessageConverter" class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
  <property name="supportedMediaTypes">
    <list>
      <value>application/json;charset=UTF-8</value>
    </list>
  </property>
</bean>
```

我们在Spring应用上下文中，设置了两个Bean，这两个Bean的具体作用根据他们的名字就能想到，就是将HTTP请求中的数据交换格式映射成对象的。注意千万别忘了这个配置，如果没有写这个配置，结果浏览器只会得到`HTTP 415 Unsupported Media Type`错误，后台没有任何报错，极难想到是缺少`RequestMappingHandler`造成的，网上相关问题也很少。

jackson的gradle依赖：
```java
compile group: 'com.fasterxml.jackson.core', name: 'jackson-core', version: '2.8.8'
compile group: 'com.fasterxml.jackson.core', name: 'jackson-databind', version: '2.8.8'
```

这个jackson解析库几乎是web应用中必备的。

后台代码中，添加了一个实体类：
```java
package com.ciyaz.domain;

public class User
{
	private String username;
	private String age;

	public String getUsername()
	{
		return username;
	}

	public void setUsername(String username)
	{
		this.username = username;
	}

	public String getAge()
	{
		return age;
	}

	public void setAge(String age)
	{
		this.age = age;
	}

	@Override
	public String toString()
	{
		return "User{" +
				"username='" + username + '\'' +
				", age='" + age + '\'' +
				'}';
	}
}
```

这个类没什么好说的，就是一个标准的JavaBean。

控制器AjaxController.java
```java
package com.ciyaz.controller;

import com.ciyaz.domain.User;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class AjaxController
{
	@RequestMapping(value = "/ajax", method = RequestMethod.GET)
	public String getAjaxPage()
	{
		return "ajax";
	}

	@ResponseBody
	@RequestMapping(value = "/ajax", method = RequestMethod.POST)
	public User doAjaxPost(@RequestBody User user)
	{
		System.out.println(user.toString());
		return user;
	}
}
```

这里要注意`doAjaxPost()`方法，我们这回得到的不是json字符串了，而是直接的实体类！这是因为SpingMVC帮我们把json字符串映射成了实体类，而我们的返回值也是一个实体类，SpringMVC也会自动帮我们把返回值进行json序列化，并返回给浏览器。这就是分方便了，我们只需要专注于我们的业务，而不用过多关注json解析之类的操作了。

## RestController

在新版本的SpringBoot中，我们通常使用`@RestController`注解代替`@Controller`注解实现接口服务的控制器，使用这个注解我们就能省去@ResponseBody了，但是@RequestBody依然不能省略。

```java
@RestController
public class MyController
{
	...
}
```

## ajax文件上传注意

有时我们会遇到文件需要异步上传的问题，具体实现可以参考`Web客户端编程`相关章节，这里就不重复了。

这里要注意，此时我们千万不要别出心裁的用Json（base64）的形式上传文件。二进制文件编码成base64后，数据会变大，此时经常发生请求体大小超过网关、中间件、框架的数据大小限制，进而触发一些莫名其妙的错误。

对于分布式应用，内部调用时，无论是使用Json还是二进制报文传递应用数据，也都尽量不要直接传文件体，而应该使用专门的OSS系统上传，应用数据中只传递URL或一个文件对象的主键，以免造成不必要的麻烦。
