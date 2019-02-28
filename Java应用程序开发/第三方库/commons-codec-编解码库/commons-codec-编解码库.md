# Apache Commons Codec

`commons-codec`这个库主要实现了文本和二进制数据之间的各种编码算法，常用的包括Base64编解码，MD、SHA系列散列摘要算法，HMAC散列消息认证算法，URL编码等。

## base64编解码

下面代码是将一个文本字符串进行base64编码和解码。

```java
Base64 base64 = new Base64();
//base64编码
String strBase64 = base64.encodeToString(str.getBytes("UTF-8"));
//base64解码
str = new String(base64.decode(strBase64), "UTF-8");
```

对于图片或是其它的二进制数据，也是同样的道理，下面是base64编解码图片的例子。

```java
import org.apache.commons.codec.binary.Base64;

import java.io.*;

public class Main
{
	public static void main(String[] args) throws Exception
	{
		//读取图片二进制数据，存入byte[]
		InputStream inputStream = Main.class.getClassLoader().getResourceAsStream("1.png");
		byte[] buffer = new byte[1024];
		ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
		int len;
		while ((len = inputStream.read(buffer)) != -1)
		{
			byteArrayOutputStream.write(buffer, 0, len);
		}
		byte[] imageBytes = byteArrayOutputStream.toByteArray();

		inputStream.close();
		byteArrayOutputStream.close();

		Base64 base64 = new Base64();
		//base64编码
		String imageBase64Str = base64.encodeToString(imageBytes);
		//base64解码
		imageBytes = base64.decode(imageBase64Str);

		//将解码后的二进制数据再写入文件
		File newImageFile = new File("/home/ciyaz/2.png");
		OutputStream outputStream = new FileOutputStream(newImageFile);
		outputStream.write(imageBytes);
		outputStream.close();

		System.out.println(imageBase64Str);
	}
}
```

实际上base64编解码的API和上面处理文本时是一样的。

注意：

1.如果需要在HTML中直接使用base64显示图片，把上面生成的base64编码直接写进`<img src=""`是无效的，必须在base64编码前加上头信息`data:image/png;base64,`，注明图片格式是png，使用的编码是base64，而解码时一定要记得去掉头信息。
2.关于图片格式，假如我们读取的图片是JPG格式，base64编码的仅仅是JPG格式的二进制数据，而不是位图数据，不要弄混了。无论是在网页上显示base64编码的图片，还是在javafx的ImageView控件里显示某段二进制数据，JPG格式的二进制数据与位图二进制数据之间转换都是由浏览器或是ImageView实现的，跟我们没关系。

## 散列摘要算法

下面例子是使用SHA256对一个字符串进行散列值计算。`commons-codec`中，对于SHA256算法，提供了`sha256()`和`sha256Hex()`两个函数，区别就是后者返回的是：算法获得的散列值二进制数据取每四位为一个十六进制字符连接而成的字符串。实际上散列算法返回的是二进制数据，但我们一般使用的都是这种处理后的字符串，因此这个函数十分方便。

```java
String str = "你好，世界！";
String sha256Str = DigestUtils.sha256Hex(str);
```

对于计算文件散列值，把文件全读入内存肯定是不可取的，`commons-codec`也提供了方便的函数对文件输入流进行处理。

```java
InputStream inputStream = Main.class.getClassLoader().getResourceAsStream("1.png");
String fileSha256 = DigestUtils.sha256Hex(inputStream);
```

## URL编码

下面例子是对字符串进行URL编码。

```java
URLCodec urlCodec = new URLCodec();
String encodedStr = urlCodec.encode(str);
String decodedStr = urlCodec.decode(encodedStr);
```
