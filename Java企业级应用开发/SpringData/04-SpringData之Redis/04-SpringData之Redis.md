# Spring Data 之 Redis

Redis是一个基于Key-Value形式的非关系型内存数据库，主要用作高性能缓存和存储一些临时信息。有关Redis的使用，请参考Redis相关章节，这里就不具体介绍了。

Java操作Redis数据库，可以使用`jedis`这个包，使用Spring框架时，SpringData项目提供了`RedisTemplate`，为我们进一步对Redis的操作进行了封装。

下面例子中，我们编写了一个最简单的Java程序，用于演示如何使用RedisTemplate。

## 使用RedisTemplate的XML配置

applicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- redis连接池配置 -->
    <bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
        <property name="maxIdle" value="400"/>
        <property name="maxTotal" value="6000"/>
        <property name="maxWaitMillis" value="-1"/>
    </bean>

    <!--Jedis连接工厂-->
    <bean id="jedisConnFactory"
          class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
        <property name="hostName" value="127.0.0.1"/>
        <property name="port" value="6379"/>
        <property name="database" value="0"/>
        <!--<property name="password" value="" />-->
        <property name="timeout" value="1000"/>
        <property name="poolConfig" ref="poolConfig"/>
        <property name="usePool" value="true"/>
    </bean>

    <!--JedisTemplate配置-->
    <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
        <property name="connectionFactory" ref="jedisConnFactory"/>
    </bean>
</beans>
```

App.java
```java
package com.ciyaz;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;

public class App
{
	public static void main(String[] args)
	{
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
		RedisTemplate<String, String> redisTemplate = (RedisTemplate<String, String>) applicationContext.getBean("redisTemplate");
		ValueOperations<String, String> vops = redisTemplate.opsForValue();
		vops.set("str", "hello, world!");
		String s = vops.get("str");
		System.out.println(s);
	}
}
```

代码非常简单，我们从Spring的ApplicationContext中获得我们之前XML中配置好的bean，`RedisTemplate`中有一组方法`opsFor*`专门用于操作Redis的各种数据结构，我们获取`*Operations`对象调用其中的方法即可。

## 复杂类型的Redis存储

上面代码中，我们发现`RedisTemplate`是支持泛型的，下面代码中演示实例化一个自定义的Java对象，存入Redis再取出。

App.java
```java
package com.ciyaz;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;

import java.io.Serializable;
import java.util.Date;

class User implements Serializable
{
	private String username;
	private Date birthday;

	public User(String username, Date birthday)
	{
		this.username = username;
		this.birthday = birthday;
	}

	public String getUsername()
	{
		return username;
	}

	public void setUsername(String username)
	{
		this.username = username;
	}

	public Date getBirthday()
	{
		return birthday;
	}

	public void setBirthday(Date birthday)
	{
		this.birthday = birthday;
	}

	@Override
	public String toString()
	{
		return "User{" +
				"username='" + username + '\'' +
				", birthday=" + birthday +
				'}';
	}
}

public class App
{
	public static void main(String[] args)
	{
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("applicationContext.xml");
		RedisTemplate<String, User> redisTemplate = (RedisTemplate<String, User>) applicationContext.getBean("redisTemplate");
		ValueOperations<String, User> vops = redisTemplate.opsForValue();
		vops.set("user", new User("root", new Date()));
		User user = vops.get("user");
		System.out.println(user);
	}
}
```

这里要注意的是，我们必须为要存储的类实现`Serializable`接口，这样该对象才能够成功进行序列化并存入Redis数据库中，在Redis中序列化后的对象使用`string`类型进行存储。

除此之外，一个对象引用其他多个对象，也是能够成功存入Redis中的，总之只要能够成功进行Java对象序列化，就能存入Redis中。

## Spring Data Redis的序列化机制

Spring Data Redis默认使用Java自带的对象序列化机制，如果我们存储的只是一个简单的String类型，但是实际上存入Redis中的内容可能和我们想象的不太相同。

```java
vops.set("str", "hello, world!");
```

Redis中实际的存储结果并不是键值对`"str"`和`"hello, world!"`，我们可以在`redis-cli`中使用`keys *`进行查询，得到的键是这样的：

```
"\xac\xed\x00\x05t\x00\x03str"
```

再查询键对应的值：

```
127.0.0.1:6379> get "\xac\xed\x00\x05t\x00\x03str"
"\xac\xed\x00\x05t\x00\rhello, world!"
```

这是因为Spring Data Redis为了支持泛型，使用了Java序列化机制对所有Java对象进行了序列化，包括String类型。如果我们仅仅希望Redis操作字符串，不要使用序列化机制支持泛型，我们可以为`RedisTemplate`手动指定字符串类型的序列化器。

applicationContext.xml
```xml
<!--JedisTemplate配置-->
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
  <property name="connectionFactory" ref="jedisConnFactory"/>
  <property name="keySerializer">
    <bean class="org.springframework.data.redis.serializer.StringRedisSerializer"/>
  </property>
  <property name="valueSerializer">
    <bean class="org.springframework.data.redis.serializer.StringRedisSerializer"/>
  </property>
  <property name="hashKeySerializer">
    <bean class="org.springframework.data.redis.serializer.StringRedisSerializer"/>
  </property>
  <property name="hashValueSerializer">
    <bean class="org.springframework.data.redis.serializer.StringRedisSerializer"/>
  </property>
</bean>
```

这样我们存取的就直接是字符串类型了。

### 应该使用默认序列化机制还是只是用字符串？

使用默认的序列化机制，Spring Data Redis能够支持泛型，但是每一个存入Redis的键值对即使是简单的字符串也会多出一些字节的内容，如果配置字符串序列化机制，Spring Data Redis就不能支持泛型了，向Redis插入自定义的Java类型，会因为该对象不能转换成String类型而报错，除此之外，我们还可以针对业务需求，实现我们自己的序列化机制，但是这就比较复杂，而且写出的代码肯定不通用。具体该选用哪种方式，就应该我们自己权衡了。

## 有关其他Redis数据类型的操作

Redis提供了5种灵活的数据结构供我们使用，在`RedisTemplate`中都能够找到对应的操作方法，都比较简单，这里就不赘述了。
