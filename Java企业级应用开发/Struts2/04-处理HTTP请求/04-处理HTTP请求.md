# 处理HTTP请求

作为MVC框架，struts2的核心功能之一就是处理HTTP请求，解析请求参数，封装进Model里，传递给Action。struts2框架中，实际上是框架的拦截器自动将请求参数的值传入Action或相应的Model。

struts2的Action和数据结合有三种写法，下面以登录表单为例，一一进行介绍。

## 直接将Action作为Model

LoginAction.java
```java
package com.ciyaz.action;

import com.opensymphony.xwork2.ActionSupport;

public class LoginAction extends ActionSupport
{

	private String username;
	private String password;

	public String getUsername()
	{
		return username;
	}

	public void setUsername(String username)
	{
		this.username = username;
	}

	public String getPassword()
	{
		return password;
	}

	public void setPassword(String password)
	{
		this.password = password;
	}

	public String login()
	{
		if(this.username.equals("admin") && this.password.equals("123"))
		{
			return "success";
		}
		else
		{
			return "error";
		}
	}
}
```

login.jsp
```html
<form action="${pageContext.request.contextPath}/login.action" method="post">
  用户名<input type="text" name="username" /><br />
  密码<input type="password" name="password" /><br />
  <input type="submit" value="注册" />
</form>
```

将Action直接作为Model，或者可以理解为框架将请求参数直接传入Action的属性中。实际开发中，我们都会定义实体类来封装数据，但是有时候我们需要单独读取某个HTTP请求参数，或是表单结构和实体类有区别，需要一个FormBean，此时可以把Action作为FormBean使用，省去再定义一个类的麻烦。

注意，表单的name字段（也就是请求参数的键）和Action的属性名相同。

## Action和Model分离

LoginAction.java
```java
package com.ciyaz.action;

import com.ciyaz.domain.User;
import com.opensymphony.xwork2.ActionSupport;

public class LoginAction extends ActionSupport
{
	private User user;

	public User getUser()
	{
		return user;
	}

	public void setUser(User user)
	{
		this.user = user;
	}

	public String login()
	{
		if(user.getUsername().equals("admin") && user.getPassword().equals("123"))
		{
			return "success";
		}
		else
		{
			return "error";
		}
	}
}
```

login.jsp
```html
<form action="${pageContext.request.contextPath}/login.action" method="post">
  用户名<input type="text" name="user.username" /><br />
  密码<input type="password" name="user.password" /><br />
  <input type="submit" value="注册" />
</form>
```

这种写法将数据封装进了Model（User类），注意表单`name`属性的写法。这种情况下，框架通过`name`属性，得知应该传入参数的对象是Action的user属性。

## 模型驱动写法

```java
package com.ciyaz.action;

import com.ciyaz.domain.User;
import com.opensymphony.xwork2.ActionSupport;
import com.opensymphony.xwork2.ModelDriven;

public class LoginAction extends ActionSupport implements ModelDriven<User>
{
	private User user = new User();

	public User getUser()
	{
		return user;
	}

	public void setUser(User user)
	{
		this.user = user;
	}

	public String login()
	{
		if(user.getUsername().equals("admin") && user.getPassword().equals("123"))
		{
			return "success";
		}
		else
		{
			return "error";
		}
	}

	@Override
	public User getModel()
	{
		return user;
	}
}
```

```html
<form action="${pageContext.request.contextPath}/login.action" method="post">
  用户名<input type="text" name="username" /><br />
  密码<input type="password" name="password" /><br />
  <input type="submit" value="注册" />
</form>
```

这种写法Action实现了ModelDriven接口，通过泛型传入了模型类，同时实现了一个`getModel()`方法，返回Model对象的实例。

## 静态参数注入和动态参数注入

上面例子代码中，我们通过提交表单发起带参数HTTP请求，struts2自动解析请求并封装数据，这个过程称为动态参数注入。除此之外，struts2还支持静态参数注入，简单地说就是通过配置文件把参数值传递给Action。下面是静态参数注入的例子。

struts.xml
```xml
<action name="test" class="com.ciyaz.action.TestAction" method="printName">
  <param name="name">aaa</param>
</action>
```

TestAction.java
```java
package com.ciyaz.action;

public class TestAction
{
	private String name;

	public String getName()
	{
		return name;
	}

	public void setName(String name)
	{
		this.name = name;
	}

	public String printName()
	{
		System.out.println(this.name);
		return null;
	}
}
```

用浏览器访问`http://localhost:8080/struts2_demo1/test.action`，注意URL中没有包含参数，此时配置文件中的参数会赋予Action的name属性，程序输出`aaa`。但是，一旦HTTP请求的URL包含了参数，例如`http://localhost:8080/struts2_demo1/test.action?name=bbb`，动态参数注入就会覆盖掉静态参数注入，此时会输出`bbb`。
