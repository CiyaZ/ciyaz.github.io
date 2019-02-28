# RabbitMQ基本概念

## RabbitMQ的架构

![](res/1.png)

### 生产者 Producer

「生产者」指投递消息的一方，消息一般包含标签（Label）和载荷（Payload）两部分：

* 标签描述这条消息，包括交换器（Exchange）的名称和路由键（Route Key），用于标识这个消息会被送到哪些消费者手中
* 载荷是消息携带的数据，通常是一个具有业务逻辑的数据结构，比如一个JSON字符串

### 消费者 Consumer

消费者用于接收并处理消息，在RabbitMQ中间件处理的过程中，消息会通过标签被送到正确的消费者手中，消费者处理的是消息的载荷。

### 中间人 Broker

Broker中文可以翻译为「中间人」，在各种消息中间件中，通常用英文单词Broker表示一个启动的中间件实例，也可以理解为一台跑着RabbitMQ服务的服务器。

### 队列 Queue

队列是一个存储消息的数据结构，多个消费者可以定义同一个队列，队列中的消息会被订阅的消费者分摊，大多数消息中间件都是这么设计的。

### 交换器 Exchange

生产者生成消息后，会将消息发送到交换器，由交换器发送到一个或多个队列。RabbitMQ中，Exchange常用的有三种类型：

* fanout：把所有发送到该Exchange的消息路由到所有与其绑定的Queue中
* direct：把发送到该Exchange的消息，通过Routing Key和Binding Key精确匹配，路由到所有与其绑定的Queue中
* topic：和direct的区别就是topic用通配符模糊匹配，具体见下文有关Routing Key和Binding Key的介绍

### 路由键 Routing Key 绑定键 Binding Key

Binding Key用于指定消息从Exchange流向哪些Queue，它对应的值是从Exchange到Queue的一对多映射关系，可以包含通配符。而Routing Key更像是一个标识，用于Binding Key去匹配，很多时候我们要求Binding Key精确匹配Routing Key，那么此时这两个键名就是相同的（Exchange的direct模式），有时Binding Key则是带有通配符的，例如`*.rabbitmq.*`，它可以匹配`com.rabbitmq.client`这个Routing Key（Exchange的topic模式）。

### 连接 Connection 信道 AMQP Channel

Connection就是从AMQP客户端到RabbitMQ的TCP连接，而Channel是TCP连接之上的一个抽象概念，这就像多层网络协议栈一样，Channel的实现是基于TCP连接的，我们使用AMQP客户端时，操作的是Channel。

## RabbitMQ例子

下面我们用Java编写一个使用RabbitMQ收发文本消息的例子。

### 添加Maven依赖

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

### 生产消息

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

### 消费消息

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

实际上，针对AMQP的各个抽象概念，`amqp-client`定义了一大堆可用的重载方法和参数，上面只是一个例子，具体使用时可以参考文档。
