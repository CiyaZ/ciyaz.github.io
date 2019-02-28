# Socket

这篇笔记记录Java中Socket相关的知识，本片笔记不会涉及基础知识，仅仅讲解Java中Socket的使用，有关Socket的基础知识请参考`Unix环境C语言编程`和`计算机网络原理和协议`相关章节。

## 建立基于TCP协议的Socket服务器和客户端

### 服务器部分

```java
package com.ciyaz;

import java.io.IOException;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * @author CiyaZ
 */
public class Server
{
	public static void main(String[] args)
	{
		// 线程池
		ExecutorService threadPool = Executors.newCachedThreadPool();

		// 这里使用JDK7的自动释放资源的try...catch...写法
		try (ServerSocket serverSocket = new ServerSocket(8080))
		{
			while (true)
			{
				Socket inSocket = serverSocket.accept();
				threadPool.execute(() -> {
					try (OutputStream outputStream = inSocket.getOutputStream())
					{
						outputStream.write("hello\n".getBytes());
					}
					catch (IOException e)
					{
						e.printStackTrace();
					}
					finally
					{
						if (inSocket != null && !inSocket.isClosed())
						{
							try
							{
								inSocket.close();
							}
							catch (IOException e)
							{
								e.printStackTrace();
							}
						}
					}
				});
			}
		}
		catch (IOException e)
		{
			e.printStackTrace();
		}
	}
}
```

上面代码很简单，当一个新的TCP连接建立时，服务端会从线程池中获取一个线程处理Socket，向客户端发送`hello\n`后关闭Socket连接，我们约定`\n`代表服务器已经发送完数据了。

注：上面我们使用的是线程池，而不是每次连接都新建一个线程，使用线程池具有更好的性能。

### 客户端部分

```java
package com.ciyaz;

import java.io.IOException;
import java.io.InputStream;
import java.net.Socket;

/**
 * @author CiyaZ
 */
public class Client
{

	public static void main(String[] args)
	{
		try (Socket socket = new Socket("127.0.0.1", 8080))
		{
			InputStream inputStream = socket.getInputStream();
			StringBuilder stringBuilder = new StringBuilder();
			while (true)
			{
				char c = (char) inputStream.read();
				stringBuilder.append(c);

				if (c == '\n')
				{
					System.out.println(stringBuilder.toString());
					break;
				}
			}
		}
		catch (IOException e)
		{
			e.printStackTrace();
		}
	}
}
```

上述代码中，我们建立一个Socket连接服务器，并使用输入流接收服务器发过来的数据，当接收到`\n`时接收结束（这是我们认为约定的），关闭Socket连接，程序结束。

## 基于UDP协议收发数据报

接收端：
```java
package com.ciyaz.udp;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;

public class Reciever
{
	public static void main(String[] args)
	{
		try(DatagramSocket datagramSocket = new DatagramSocket(8080))
		{
			DatagramPacket datagramPacket = new DatagramPacket(new byte[6], 0, 6);
			datagramSocket.receive(datagramPacket);

			System.out.println(new String(datagramPacket.getData()));
		}
		catch (IOException e)
		{
			e.printStackTrace();
		}
	}
}
```

发送端：
```java
package com.ciyaz.udp;

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

public class Sender
{
	public static void main(String[] args)
	{
		try(DatagramSocket datagramSocket = new DatagramSocket())
		{
			byte[] data = "hello\n".getBytes();
			InetAddress address = InetAddress.getByName("127.0.0.1");
			DatagramPacket datagramPacket = new DatagramPacket(data, 0, data.length, address, 8080);
			datagramSocket.send(datagramPacket);
		}
		catch (IOException e)
		{
			e.printStackTrace();
		}
	}
}
```
