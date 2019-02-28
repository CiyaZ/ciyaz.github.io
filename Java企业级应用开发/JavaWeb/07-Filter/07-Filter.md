# Filter

filter可以对用户的请求进行预处理和后处理，也就是说，filter可以拦截发给servlet的request，可以拦截发给客户端的response，多个filter可以形成处理链。filter主要用于用户权限认证，生成访问日志，编解码等。

## filter例子

下面例子中，在请求传递给`TestServlet`之前和之后，会分别打印两条信息。从Filter和Servlet的设计上看，`FilterChain`体现的是责任链设计模式。

```java
@WebFilter(filterName = "TestFilter", urlPatterns = "/TestServlet")
public class TestFilter implements Filter
{
	public void destroy()
	{
	}

	public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException
	{
		System.out.println("Before---");
		chain.doFilter(req, resp);
		System.out.println("After---");
	}

	public void init(FilterConfig config) throws ServletException
	{
	}
}
```

Filter实际上在功能上完全可以替代Servlet（struts2控制器实际上就是基于Filter的），在MVC中也可以作为控制器。

## Filter的生命周期

和Servlet相同。默认情况下，第一次收到请求后初始化，直到Web应用关闭时销毁。

## filter配置

和servlet一样，filter也可以通过注解和xml两种方式进行配置，配置的参数和servlet基本相同。这里需要注意，使用Filter时，经常会一个Filter拦截多个请求，这里特别介绍一下urlpatterns中通配符`*`的使用。假设项目的context path是`/test`：

* `urlPatterns = "/aaa"`：只匹配对url为`/test/aaa`的请求
* `urlPatterns = "/*"` ：匹配所有对该项目的请求
* `urlPatterns = "*.do"`：匹配对该项目所有带有`.do`后缀的请求，例如`/test/a.do`，`/test/aaa/a.do`

实际上，Filter匹配请求的方式分为两类，路径匹配和后缀匹配。上面前两种是路径匹配，后两种是后缀匹配。

注意，以下写法都是错误的：

* `urlPatterns = "/"`：匹配不到任何请求
* `urlPatterns = "/*.do"`：错误写法，启动直接报错

Servlet中，`/`表示该Servlet是项目的默认Servlet，但是Filter是不能匹配`/`这个请求的。

## 获取请求URL

有时我们对匹配的URL要求的比较特殊，上述匹配方式无法满足，这种情况下实际上不需要过于复杂的匹配表达式，我们可以通过编程的方式，对请求URL进行判断。

假设请求地址是`http://localhost:8080/test/a.do`，HttpServletRequest中，有这样几个方法：

* `request.getRequestURI()`：返回值是String类型，`/test/a.do`
* `request.getRequestURL()`：返回值是StringBuffer类型，`http://localhost:8080/test/a.do`
