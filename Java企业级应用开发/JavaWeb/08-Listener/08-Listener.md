# Listener

Listener组件可以监听web容器内的各种事件。定义Listener和Servlet、Filter相同，定义实现类并使用注解配置一下即可。我们编写的Listener主要用于应用启动时的一些初始化工作。

## 可以实现的接口

* ServletContextListener 监听web应用的启动和关闭
* ServletContextAttributeListener 监听web应用范围内的属性的改变
* ServletRequestListener 监听用户请求
* ServletRequestAttributeListener 监听request范围内属性的变化
* ServletSessionListener 监听session的开始和结束
* ServletSessionAttributeListener 监听session范围内属性的变化

## 例子

监听应用启动和停止

```java
@WebListener()
public class TestListener implements ServletContextListener
{

	@Override
	public void contextInitialized(ServletContextEvent servletContextEvent)
	{
		System.out.println("application start");
	}

	@Override
	public void contextDestroyed(ServletContextEvent servletContextEvent)
	{
		System.out.println("application terminated");
	}
}
```
