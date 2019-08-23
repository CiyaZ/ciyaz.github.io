# Netty入门简介

![](res/1.png)

Netty是一个基于NIO的异步事件驱动的网络编程框架，它基本对所有常见的网络协议都进行了封装，而且性能较好，我们一般基于Netty来开发TCP、UDP、HTTP、WebSocket、RTSP等协议的高性能服务端程序。

## Netty的适用场景

如果学习过Java的Socket，或Unix环境C语言的Socket网络编程，我们就会意识到，直接使用套接字接口进行服务端网络编程是比较繁琐的。例如编写一个基于TCP的服务端应用程序，我们要处理的不仅仅是从流中读数据、写数据那么简单，并发处理、线程调度、协议解析、报文编解码都要手写，即使是老手也要花不少精力去处理这些和业务逻辑不太相关的代码，而且还很容易出bug！

而Netty的意义就在于，框架对很多常用的代码逻辑进行了封装，我们按照一个“模板”把各个组件、工具类配置好，它就能良好的工作了。

Netty几乎适用于除Web外（毕竟基于Servlet的技术还是远比Netty方便的），所有和网络相关的编程工作，包括游戏服务端，流媒体服务端等等。

## 引入Netty依赖

这里我们使用Maven进行依赖和工程构建配置管理，Netty各个功能都封装了不同的包目录，我们可以直接引用一个`netty-all`依赖把所有功能引入：

```xml
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.38.Final</version>
</dependency>
```

注：关于Netty版本，Netty有`3`，`4`，`4.1`这三个版本分支，目前大部分都是使用的`4.1`版本，曾经出现过`Netty5`，但是该版本因为种种原因很快就被废弃了（不确定未来是否还会有Netty5、Netty6）。这些版本的API都不一样，因此网上的代码千奇百怪，确实给我们的学习增加了不小的难度。

## Hello World

这里我们使用Netty编写一个基于TCP的服务端程序，可以使用`telnet`进行访问，服务器将原样返回我们输入的一行内容，我们可以和基于Socket（NIO）的代码进行对比学习。创建的3个Java文件如下：

![](res/2.png)

EchoServer.java
```java
package com.ciyaz.demo.netty.hello;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;

/**
 * 启动服务器
 *
 * @author CiyaZ
 */
public class EchoServer {
    public static void main(String[] args) {

        // 主事件循环组，负责接受请求并分派给从事件循环组
        EventLoopGroup parentGroup = new NioEventLoopGroup();
        // 从事件循环组
        EventLoopGroup childGroup = new NioEventLoopGroup();

        try {
            // 服务器启动
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap
                    // 配置两个事件循环组
                    .group(parentGroup, childGroup)
                    // 配置NioServerSocketChannel
                    .channel(NioServerSocketChannel.class)
                    // 注册自定义处理器
                    .childHandler(new EchoServerInitializer());
            serverBootstrap
                    // 尝试获取异步通知并阻塞等待
                    .bind(8080).sync()
                    .channel().closeFuture().sync();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            parentGroup.shutdownGracefully();
            childGroup.shutdownGracefully();
        }
    }
}
```

上面代码包含我们程序的入口函数，其功能就是初始化服务端程序，具体每一步的作用参考代码注释和API文档。虽然看似很复杂，但是基本上所有基于Netty的应用程序，其写法思路都和上面类似。

EchoServerInitializer.java
```java
package com.ciyaz.demo.netty.hello;

import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.codec.LineBasedFrameDecoder;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.codec.string.StringEncoder;
import io.netty.util.CharsetUtil;

/**
 * 初始化服务器
 *
 * @author CiyaZ
 */
public class EchoServerInitializer extends ChannelInitializer<SocketChannel> {

    private static final int MAX_FRAME_SIZE = 8192;

    @Override
    protected void initChannel(SocketChannel socketChannel) throws Exception {
        // 处理管道
        ChannelPipeline pipeline = socketChannel.pipeline();
        pipeline.addLast(new LineBasedFrameDecoder(MAX_FRAME_SIZE));
        pipeline.addLast(new StringEncoder(CharsetUtil.UTF_8));
        pipeline.addLast(new StringDecoder(CharsetUtil.UTF_8));
        pipeline.addLast(new EchoServerHandler());
    }
}
```

`ChannelInitializer`实现了`ChannelHandler`，代码中我们其实做的就是配置了一个处理管道（pipeline）。这里简单介绍一下为处理管道配置的几个`Handler`组件：

* `LineBasedFrameDecoder`：我们知道TCP是一个流式协议，但我们的应用一般是基于消息的，因此为了把每条消息区分开来，这里设定了一个基于文本行的消息分割机制。该组件是Netty帮我们实现好的，客户端发来的数据会自动以换行符进行分割。
* `StringEncoder`和`StringDecoder`：字符编解码器，Socket传输的数据是二进制的，该组件用于在文本和二进制数据之间做编解码转换，默认字符编码为`UTF-8`。
* `EchoServerHandler`：我们自己定义的处理器。

EchoServerHandler.java
```java
package com.ciyaz.demo.netty.hello;

import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;

/**
 * 自定义处理器
 *
 * @author CiyaZ
 */
public class EchoServerHandler extends SimpleChannelInboundHandler<String> {
    /**
     * 该方法会在收到请求时回调
     */
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
        ctx.writeAndFlush(msg);
        System.out.println("已处理消息：" + msg);
    }

    @Override
    public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
        super.channelRegistered(ctx);
        System.out.println("channel注册");
    }

    @Override
    public void channelUnregistered(ChannelHandlerContext ctx) throws Exception {
        super.channelUnregistered(ctx);
        System.out.println("channel取消注册");
    }

    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        super.channelActive(ctx);
        System.out.println("channel激活");
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        super.channelInactive(ctx);
        System.out.println("channel取消激活");
    }
}
```

上面代码就是我们真正自定义的处理器了，注意类继承的`SimpleChannelInboundHandler`带有一个泛型`String`，这样配置，前面基于`LineBasedFrameDecoder`分割出的文本消息，会作为`msg`参数回调传入`channelRead0`。

至于`channelRead0`方法（尽管它的名字比较奇怪），我们自定义的处理逻辑很简单，无论请求是什么，都原样返回给客户端。

我们可以用`telnet`作为TCP客户端进行测试：

```
telnet 127.0.0.1 8080
```

## 总结

上面代码乍一看非常复杂，但是每个组件的存在都是有意义的，Netty程序的大致框架基本都遵循上面三步，了解每个组件都是干什么用的之后，就能体会到它们的作用。

Netty不仅封装了从Socket读取、写入的代码，也封装了事件循环、消息报文分割、编解码器等功能，这些功能全靠手写真的很繁琐（尤其是多线程编程难调试，且很容易出bug）。

随着后续学习，就能深刻体会到这一点。
