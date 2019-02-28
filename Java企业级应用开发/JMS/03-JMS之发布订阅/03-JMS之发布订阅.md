# JMS之发布订阅

除了单纯的通过消息队列收发消息，消息的`发布订阅`模式也是一个常用的功能。下面例子代码演示发布消息的Producer，和订阅消息的Consumer。Producer发送消息后，所有的Consumer都会收到“广播”。

## Producer

Producer.java
```java
package com.ciyaz.example2;

import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

public class Producer
{
	public static void main(String[] args)
	{
		//连接工厂
		ConnectionFactory connectionFactory;
		//连接对象
		Connection connection = null;
		//会话，代表一个生成消息的线程
		Session session;
		//消息目的地
		Destination destination;
		//消息生产者
		MessageProducer messageProducer;

		connectionFactory = new ActiveMQConnectionFactory("admin", "admin", ActiveMQConnection.DEFAULT_BROKER_URL);
		try
		{
			//创建连接，创建Session，获取消息生产者对象
			connection = connectionFactory.createConnection();
			connection.start();
			session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);
			destination = session.createTopic("topic1");
			messageProducer = session.createProducer(destination);

			//发送十条消息
			for(int i = 0; i < 10; i++)
			{
				TextMessage textMessage = session.createTextMessage("msg" + i);
				messageProducer.send(textMessage);
			}
			//提交事务
			session.commit();

		}
		catch (JMSException e)
		{
			e.printStackTrace();
		}
		finally
		{
			if(connection != null)
			{
				try
				{
					connection.close();
				}
				catch (JMSException e)
				{
					e.printStackTrace();
				}
			}
		}
	}
}
```

## Consumer

Consumer.java
```java
package com.ciyaz.example2;

import org.apache.activemq.ActiveMQConnection;
import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

public class Consumer
{
	public static void main(String[] args)
	{
		System.out.println(Thread.currentThread().getId());
		ConnectionFactory connectionFactory;
		Connection connection = null;
		Session session;
		Destination destination;
		MessageConsumer messageConsumer;
		connectionFactory = new ActiveMQConnectionFactory("admin", "admin", ActiveMQConnection.DEFAULT_BROKER_URL);
		try
		{
			connection = connectionFactory.createConnection();
			connection.start();
			session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
			destination = session.createTopic("topic1");
			messageConsumer = session.createConsumer(destination);
			//收到消息的回调函数
			messageConsumer.setMessageListener(message -> {
				try
				{
					System.out.println(Thread.currentThread().getId());
					TextMessage textMessage = (TextMessage) message;
					System.out.println("收到消息：" + textMessage.getText());
				}
				catch (JMSException e)
				{
					e.printStackTrace();
				}
			});
		}
		catch (JMSException e)
		{
			e.printStackTrace();
		}
	}
}
```

实际上，代码和前面消息队列模式基本一样的，就是`Destination`对象的创建方式从啊`createQueue()`变成了`createTopic`了。

## Topic模式的用法

Topic模式和Queue模式正好相反，我们通常有一个Producer用来生成消息，多个Consumer获取消息，Topic模式主要解决的问题是软件系统内部通讯的问题。
