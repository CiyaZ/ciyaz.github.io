# Java 内置的HttpServer

Java（JDK6）内置了一个简单的HttpServer，能够读取HTTP请求体，设置响应头、响应体等，用起来很方便，一些简单的HTTP服务可以直接用它来做，不需要上Tomcat。

下面是使用HttpServer的例子

```java
import com.sun.net.httpserver.HttpServer;

import java.io.*;
import java.net.InetSocketAddress;

public class Main
{
	public static void main(String[] args) throws IOException
	{
		HttpServer server = HttpServer.create(new InetSocketAddress("localhost", 8000), 0);
		server.createContext("/", (he) ->
		{
			try
			{
				//输出请求URI
				System.out.println(he.getRequestURI());

				InputStream in = he.getRequestBody();
				OutputStream out = he.getResponseBody();

				//把请求体原样输出到响应体
				byte[] buffer = new byte[in.available() == 0 ? 1024 : in.available()];
				System.out.println("buffer size=" + buffer.length);

				ByteArrayOutputStream baos = new ByteArrayOutputStream(buffer.length);
				int length;
				while ((length = in.read(buffer, 0, buffer.length)) >= 0)
				{
					baos.write(buffer, 0, length);
				}
				he.sendResponseHeaders(200, baos.size());
				baos.writeTo(out);

			}
			finally
			{
				he.close();
			}
		});
		server.start();
	}
}
```

代码很简单，启动HttpServer，监听8000端口，当有HTTP请求时，读取请求URL，并把请求体原样写到响应体。

下面是测试页面
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="http://127.0.0.1:8000" method="post">
        <input type="text" name="aa" value="aaa" />
        <input type="submit" value="submit">
    </form>
</body>
</html>
```

使用浏览器打开上面HTML，发起POST请求，控制台输出结果为：
```
/
buffer size=6
```

注：请求URI包括GET参数，比如请求URL为`localhost/myapp?a=1`，context path为`/myapp`，那么请求URI字符串就是`/myapp?1=a`。
