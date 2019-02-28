# JMS和ActiveMQ简介

JMS（Java Message Service）是Java平台中，针对消息中间件制定的一组API规范，它的定位和和JDBC类似。和JDBC一样，JMS有多种实现可选，而其中比较常用的开源实现就是Apache ActiveMQ。（MQ：Message Queue，消息队列）

ActiveMQ是Apache出品的开源消息总线，支持JMS1.1规范，本系列笔记以ActiveMQ作为实现讲解JMS，有关ActiveMQ的内容将在专门章节记录。

## 消息队列的应用场景

消息队列主要应用在以下几个方面：

1. 异步处理：将同步的业务流程操作（尤其是因为IO阻塞而比较耗时间的操作）通过消息队列改为异步方式，能够提高CPU的利用率
2. 应用解耦：将一个大项目拆分后，各个子模块之间可以通过消息队列进行通讯
3. 流量削锋：如短时间内出现大量订单的情况，流量峰值可以通过消息队列实现排队，而不至于服务器崩溃
4. 日志处理：日志也可能有峰值的等问题存在，通过消息队列接受日志能够解决这个问题
5. 消息通讯：可以用消息队列实现类似聊天室的功能

## ActiveMQ下载和使用

下载地址：[http://activemq.apache.org/download.html](http://activemq.apache.org/download.html)

和Tomcat一样，下载后解压，运行`bin`目录中的脚本即可使用了。

```
activemq start
```

用浏览器可以访问ActiveMQ的web控制台：`http://127.0.0.1/admin`，默认情况下账号密码都是`admin`。

## Maven依赖

我们的项目中需要添加JMS和ActiveMQ的依赖。

```xml
<dependency>
    <groupId>javax.jms</groupId>
    <artifactId>jms</artifactId>
    <version>1.1</version>
</dependency>
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-all</artifactId>
    <version>5.15.5</version>
</dependency>
```
