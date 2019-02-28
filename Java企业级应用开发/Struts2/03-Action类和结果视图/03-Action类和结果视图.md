# Action类和结果视图

struts2中，一个http请求对应一个Action类的方法，功能相近的方法应该放在同一个Action类中。简单地说，对于一张数据表的增删改查，四个操作就可以放到一个Action类中，因为这四个方法可能调用同一个Service类，可能有相似的操作可以抽取出函数，而且这样做代码结构更加清晰。

## Action类的生命周期

每次新的请求都会实例化新的Action实例，完成请求处理后销毁。显然这种方法性能不高。

我们知道SpringMVC的Controller默认就是单例的，为什么struts2的Action默认是多例的呢？因为Action和数据进行了绑定，实体类模型或是请求参数都可以定义在Action的属性中，而SpringMVC请求对应的是Controller的方法，Controller的属性只有依赖注入的Service等。Action如果单例，一定会发生线程安全问题，而SpringMVC的Controller则不会。

## 定义Action类

普通的POJO就能作为一个Action类存在，在环境搭建章节中，已经给出例子了。这里介绍一下Action接口和ActionSupport类。

Action接口源码（删掉了注释）
```java
package com.opensymphony.xwork2;

public interface Action {

    public static final String SUCCESS = "success";

    public static final String NONE = "none";

    public static final String ERROR = "error";

    public static final String INPUT = "input";

    public static final String LOGIN = "login";

    public String execute() throws Exception;

}
```

我们定义的Action类可以继承这个Action接口，这个接口预定义了几个字符串常量，可以作为Action方法的返回值使用。当然，也完全可以不使用它，个人认为这个设计还是比较鸡肋的。

除此之外，我们还可以继承ActionSupport类。

ActionSupport部分源码
```java
public class ActionSupport implements Action, Validateable, ValidationAware, TextProvider, LocaleProvider, Serializable {
  ...
}
```

我们查看源码可以看到，ActionSupport实现了很多实用的接口，比如验证功能，国际化等。我们编写的Action类如果继承ActionSupport类，实现起来就比较方便了。因此，推荐继承ActionSupport。

## 动作方法

一个用户的http请求对应一个Action的动作方法，它由struts2框架通过反射调用。

```java
public String xxx(){}
```

动作方法不能带有参数，返回值字符串对应配置文件的视图模板。如果不想返回任何视图，可以返回`null`或字符串`"none"`。

## Action中获取ServletAPI

我们如果需要在Action类中访问Session，Cookie等，就需要访问ServletAPI，得到HttpServletRequest等对象。访问ServletAPI的示例代码如下：

HelloAction.java
```java
package com.ciyaz.action;

import javax.servlet.ServletContext;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.struts2.interceptor.ServletRequestAware;
import org.apache.struts2.interceptor.ServletResponseAware;
import org.apache.struts2.util.ServletContextAware;

public class HelloAction implements ServletRequestAware, ServletResponseAware, ServletContextAware
{

	private HttpServletRequest request;
	private HttpServletResponse response;
	private ServletContext servletContext;

	public String hello()
	{
		System.out.println(servletContext);
		System.out.println(response);
		System.out.println(request);
		return "success";
	}

	@Override
	public void setServletContext(ServletContext context)
	{
		this.servletContext = context;
	}

	@Override
	public void setServletResponse(HttpServletResponse response)
	{
		this.response = response;

	}

	@Override
	public void setServletRequest(HttpServletRequest request)
	{
		this.request = request;
	}

}
```

如果我们实现了几个XXAware接口，struts2框架会自动把我们需要的对象注入进来。这是通过`ServletConfigInterceptor`实现的，这个拦截器在默认的拦截器栈中，它通过判断我们自定义的Action是否实现了某个接口，例如实现了`ServletRequestAware`，那么`action instanceof RequestAware = true`，此时就可以通过`setServletRequest()`方法把需要的对象传递给自定义的Action类中。

除此之外，也可以通过struts2提供的`ServletActionContext`类的静态方法，获得ServletAPI对象，更加简便。

## 动作方法的结果视图配置

在配置文件中，有关结果视图的配置是result元素实现的，例子如下：

```xml
<result name="success" type="dispatcher">/success.jsp</result>
```

* name：对应Action类的返回值，默认值为`success`
* type：结果类型（转发还是重定向，或是其他类型），默认值是转发。
  * dispatcher 转发
  * redirect 重定向到另一个JSP页面（或其他视图模板）
  * redirectAction 重定向到另一个action
  * chain 转发到另一个action
  * freemarker 用于转发到freemarker模板
  * velocity 用于转发到velocity模板
  * httpheader 用于输出http协议的消息头
  * stream 用于文件下载
  * plainText 以纯文本形式展现页面（例如使用JSP时，会直接输出JSP源码，而不是编译执行JSP）
  * xslt 以扩展样式表转换语言展现页面

重定向的例子
```xml
<struts>
	<package name="p1" extends="struts-default">
		<action name="hello" class="com.ciyaz.action.HelloAction" method="hello">
			<result name="success" type="dispatcher">/success.jsp</result>
		</action>
		<action name="hello2" class="com.ciyaz.action.HelloAction" method="hello2">
			<result name="success" type="redirectAction">/hello.action</result>
		</action>
	</package>
</struts>
```

访问`hello2.action`结果如图

![](res/1.png)

## 局部视图和全局视图

为一个action服务的视图叫做局部视图，我们有时候需要为一个模块统一设置一个错误处理页面，或者为整个项目设置一个错误页面，这时候可以用到全局视图。

```xml
<package name="p1" extends="struts-default">
  <global-results>
    <result name="error">/error.jsp</result>
  </global-results>
  ...
</package>
```

全局视图的`global-results`元素必须放在`package`元素里，如果想让多个package共享这个配置，那么可以单独定义一个package继承`struts-default`，让其它package继承这个package。



## 自定义结果视图

举个例子，开发网站时有时需要输出验证码图片，使用MVC框架的情况下，验证码通常是请求控制器，然后动态输出的。那么struts2中，就可以把“输出验证码”这个动作封装成一个自定义结果视图。这样工程的结构更加清晰。下面是一个自定义验证码结果视图的例子：

CaptchaResult.java
```java
package com.ciyaz.results;

import javax.imageio.ImageIO;
import javax.servlet.ServletContext;
import javax.servlet.http.HttpServletResponse;

import org.apache.struts2.ServletActionContext;
import org.apache.struts2.dispatcher.StrutsResultSupport;

import com.opensymphony.xwork2.ActionInvocation;

public class CaptchaResult extends StrutsResultSupport
{
	@Override
	protected void doExecute(String finalLocation, ActionInvocation invocation) throws Exception
	{
		HttpServletResponse res = ServletActionContext.getResponse();
		BufferedImage image = this.generate();
		ImageIO.write(image, "png", res.getOutputStream());
	}

  ...验证码生成部分略
}
```

struts.xml
```xml
<struts>
	<package name="p1" extends="struts-default">
		<result-types>
			<result-type name="captcha" class="com.ciyaz.results.CaptchaResult"></result-type>
		</result-types>
		<action name="hello" class="com.ciyaz.action.HelloAction" method="hello">
			<result name="success" type="captcha"></result>
		</action>
	</package>
</struts>
```

上面代码中，我们定义了一个`CaptchaResult`类，继承`StrutsResultSupport`类，自定义结果视图都要继承这个类，并重写`doExecute()`方法，实际上这个方法里面的内容就是调用ServletAPI，输出自定义结果数据。在`struts.xml`中，我们配置了自定义结果视图的名字和执行的类全名，最后action将自定义结果视图作为结果返回。

除此之外，我们还能通过`param`向`CaptchaResult`类中传递参数，这里就不一一实验了。
