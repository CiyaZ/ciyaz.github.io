# RabbitMQ的编程接口

RabbitMQ实现了大多数主流语言的客户端，这里以Java为例进行介绍。

## 添加Maven依赖

使用RabbitMQ，需要`amqp-client`这个依赖，除此之外，还需要日志模块`logback-classic`。

```xml
<dependencies>
  <dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.5.3</version>
  </dependency>
  <dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
  </dependency>
</dependencies>
```

## 生产消息

```java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.MessageProperties;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class RabbitProducer
{
	private static final String EXCHANGE_NAME = "exchange1";
	private static final String ROUTING_KEY = "route_key_1";
	private static final String QUEUE_NAME = "queue1";

	private static final String HOST = "192.168.0.152";
	private static final int PORT = 5672;

	public static void main(String[] args)
	{
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost(HOST);
		connectionFactory.setPort(PORT);
		connectionFactory.setUsername("admin");
		connectionFactory.setPassword("123456");

		try
		{
			Connection connection = connectionFactory.newConnection();
			// 声明一个信道
			Channel channel = connection.createChannel();
			// 声明一个Exchange
			channel.exchangeDeclare(EXCHANGE_NAME, "direct", true, false, null);
			// 声明一个队列
			channel.queueDeclare(QUEUE_NAME, true, false, false, null);
			// 通过路由键将Exchange和队列绑定
			channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);

			// 通过信道发送一条纯文本消息
			String msg = "Hello, world!";
			channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, MessageProperties.TEXT_PLAIN, msg.getBytes());

			channel.close();
			connection.close();
		}
		catch (IOException | TimeoutException e)
		{
			e.printStackTrace();
		}
	}
}
```

注意`MessageProperties.TEXT_PLAIN`这个参数，我们点进去源码发现它是一个`AMQP.BasicProperties`类型，这个工具用于表示一个消息配置。`MessageProperties.TEXT_PLAIN`实际上是一个预定义的消息配置，我们也可以手动创建这样的配置，并调用`basicPublish()`的另一个重载传入：
```java
AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder();
builder.contentType("text/plain");
AMQP.BasicProperties properties = builder.build();
channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, properties, msg.getBytes());
```

配置类中还有其它选项可用，比如超时时间，消息优先级等。这里就不多做介绍了。

## 消费消息

### 推模式消费消息

下面这种接收消息的方式称为`推模式`，消费者端一直监听着消息队列，如果有消息就立即消费，这也是最常见的消费模式。

```java
import com.rabbitmq.client.*;

import java.io.IOException;
import java.util.Scanner;
import java.util.concurrent.TimeoutException;

public class RabbitConsumer
{
	private static final String QUEUE_NAME = "queue1";

	private static final String HOST = "192.168.0.152";
	private static final int PORT = 5672;

	public static void main(String[] args)
	{
		ConnectionFactory connectionFactory = new ConnectionFactory();
		connectionFactory.setHost(HOST);
		connectionFactory.setPort(PORT);
		connectionFactory.setUsername("admin");
		connectionFactory.setPassword("123456");
		try
		{
			Connection connection = connectionFactory.newConnection();
			Channel channel = connection.createChannel();
			// 设置当前最大同时处理的消息数，以免大量消息造成过大的线程开销
			channel.basicQos(1);

			// 创建一个消费者对象，定义处理函数
			Consumer consumer = new DefaultConsumer(channel)
			{
				@Override
				public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException
				{
					// 打印字符串消息
					System.out.println(new String(body));
					// 确认消息
					channel.basicAck(envelope.getDeliveryTag(), false);
				}
			};
			channel.basicConsume(QUEUE_NAME, consumer);

			// 收到新消息后，Consumer中的处理函数会以新线程方式执行，这里是为了防止主线程退出的代码
			System.out.println("输入任意字符退出程序");
			Scanner scanner = new Scanner(System.in);
			scanner.next();

			channel.close();
			connection.close();

		}
		catch (IOException | TimeoutException e)
		{
			e.printStackTrace();
		}
	}
}
```

上面消费者代码中，我们主要关注消息接收部分，我们新创建了一个`Consumer`实例，其中的`handleDelivery()`会根据我们指定的线程数进行多线程的消息处理。上面代码中处理过程比较简单，只是打印了一下消息，然后调用`basicAck()`进行确认。如果消息未确认，RabbitMQ会一直为我们保留着消息，不用担心消费者程序卡死而造成消息丢失。

### 拉模式消费消息

拉模式中，消费者不会一直监听消息队列，而是在需要时，去消息队列主动拉取一条消息。基于之前的代码，这里展示如何实现拉模式。

```java
GetResponse response = channel.basicGet(QUEUE_NAME, false);
if(response != null)
{
  System.out.println(new String(response.getBody()));
  channel.basicAck(response.getEnvelope().getDeliveryTag(), false);
}
else
{
  System.out.println("现在还没有消息");
}
```

注：`channel.basicGet()`并不会把线程阻塞掉，如果现在没有消息，会立即返回`null`。

### 拒绝一条消息

消费端如果发现一条消息处理不了，这时可以明确的拒绝一条消息，拒绝消息时，我们可以指定这条消息删除还是重新放入消息队列。

例子代码：
```java
channel.basicNack(envelope.getDeliveryTag(), false, false);
```

其中第三个布尔类型参数`requeue`用于指定是否将消息重新放回队列，这里要注意，重新放回队列后，如果没有能够正确处理这条消息的机制，该消费者还是可能收到该消息，这就形成了死循环，会耗尽消息队列的性能，使用`nack`时这一点要尤其注意。
