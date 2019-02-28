# HttpURLConnection HTTP客户端

HTTP是应用最广泛的应用层协议，JavaSE内置了HttpURLConnection对象，它可以作为一个简单的HTTP客户端，在编写爬虫程序，GUI程序，和Android应用时，非常常用。这个类的方法很多，这里不可能一一介绍，这里只是记录几个使用HttpURLConnection的最佳实践，以备日后查阅。有关更详细的方法使用，查阅JDK文档即可。

## 发起GET请求并读取响应体

```java
import java.io.File;
import java.io.FileOutputStream;
import java.io.InputStream;
import java.net.HttpURLConnection;
import java.net.URL;

public class Main
{
	public static void main(String[] args) throws Exception
	{

		File file = new File("/home/ciyaz/test.html");
		FileOutputStream fileOutputStream = new FileOutputStream(file);

		//初始化请求URL
		URL url = new URL("http://www.baidu.com");
		//获取HttpURLConnection对象
		HttpURLConnection connection = (HttpURLConnection) url.openConnection();
		//发起HTTP请求
		connection.connect();

		//读取响应状态码，200表示OK
		if(connection.getResponseCode() == 200)
		{
			//获取响应体输入流
			InputStream inputStream = connection.getInputStream();

			//读取输入流写入文件
			byte[] buffer = new byte[1024];
			int n;
			while ((n = inputStream.read(buffer)) != -1)
			{
				fileOutputStream.write(buffer, 0, n);
			}

			inputStream.close();
			fileOutputStream.close();
		}
	}
}
```

这里我们向一个网页发起GET请求，显然响应体是一个HTML页面，我们获得了这个页面内容的输入流，并将其存入一个文件中。

注：为了代码美观，上面我们对HttpURLConnection并没有配置很多属性，因为它的很多属性默认值就是我们需要的。具体请参考文档。

## 使用POST方式提交表单

```java
import java.io.*;
import java.net.HttpURLConnection;
import java.net.URL;
import java.nio.charset.StandardCharsets;

public class Main
{
	public static void main(String[] args) throws Exception
	{

		URL url = new URL("http://localhost:8080/TestPost/TestServlet");
		HttpURLConnection connection = (HttpURLConnection) url.openConnection();
		connection.setRequestMethod("POST");
		//获取请求体输出流
		connection.setDoOutput(true);
		DataOutput dataOutput = new DataOutputStream(connection.getOutputStream());
		//向请求体写入数据
		dataOutput.write("param1=a&param2=b".getBytes(StandardCharsets.UTF_8));
		connection.connect();

		if(connection.getResponseCode() == 200)
		{
			System.out.println("POST请求成功");
		}
	}
}
```

上面代码演示了使用POST方式，向一个URL提交表单数据。实际上HttpURLConnection提供的接口比较底层，并没有区分POST的是表单还是二进制数据，统统使用OutputStream。这里POST的地址是一个Servlet编写的后台，后台可以正常收到POST的参数即：`param1 = a`和`param2 = b`。

## 设置请求头

编写爬虫时，可能待爬取的服务器对访问的HTTP请求头做了限制，以判断是爬虫还是正常的浏览器。此时我们可以让代码发的HTTP请求带上请求头，模拟浏览器的行为。

```java
connection.setRequestProperty("User-Agent", "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:53.0) Gecko/20100101 Firefox/53.0");
```

## 读取响应头

```java
Map<String, List<String>> headerFields = connection.getHeaderFields();
for(String headerKey : headerFields.keySet())
{
  System.out.println(headerKey);
  for(String headerValue : headerFields.get(headerKey))
  {
    System.out.println(headerValue);
  }
}
```

我们可以通过`connection.getHeaderFields()`拿到所有响应头信息的Map，然后读取即可。下面是Tomcat服务器默认的返回结果：

```
null
HTTP/1.1 200 OK
Server
Apache-Coyote/1.1
Content-Length
0
Date
Tue, 18 Jul 2017 14:11:06 GMT
```

## 使用Cookie

HttpURLConnection并没有提供和Cookie相关的API，但是这并不影响我们使用Cookie。当请求网页时，我们可以从响应头的`Set-Cookie`属性中，读取服务器向我们设置的所有Cookie。我们向服务器发起请求时，则可以在请求头中的`Cookie`属性中，带上需要发给服务器的Cookie。

我们可以自己使用Map存储Cookie，也可以使用JavaSE内置的`java.net.CookieManager`存储。
