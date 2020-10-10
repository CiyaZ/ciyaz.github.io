# WebSocket服务器

WebSocket是一个基于消息的应用层协议（传输层基于TCP），而且浏览器中JavaScript脚本对于处理Json和XML是非常擅长的，所以WebSocket经常用于传输文本格式的Json或XML数据。因此，我们定义自己的协议报文其实非常简单，也不用像TCP一样考虑变长的报文拆装等问题了，直接传一个Json字符串即可。

实际上，WebSocket因为简单易用，相关框架多，在游戏、即时通讯等领域应用非常广泛。

这篇笔记中，我们介绍如何使用Netty实现服务端和浏览器端的WebSocket通信。

## 例子代码

这里我们实现一个和前篇一模一样的IM功能，支持消息和心跳两种Json报文格式，只不过客户端是浏览器。在服务端，我们使用`com.fasterxml.jackson`这个库作为Json报文的序列化和反序列化库。

## 服务端

WsImServer.java
```java
package com.ciyaz.demo.netty.ws.server;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * @author CiyaZ
 */
public class WsImServer {

    private static Logger logger = LoggerFactory.getLogger(WsImServer.class);

    public static void main(String[] args) {
        EventLoopGroup parentGroup = new NioEventLoopGroup();
        EventLoopGroup childGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap
                    .group(parentGroup, childGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new WsImServerInitializer());
            serverBootstrap
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

WsServerInitializer.java
```java
package com.ciyaz.demo.netty.ws.server;

import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.socket.SocketChannel;
import io.netty.handler.codec.http.HttpObjectAggregator;
import io.netty.handler.codec.http.HttpServerCodec;
import io.netty.handler.codec.http.websocketx.WebSocketServerProtocolHandler;
import io.netty.handler.stream.ChunkedWriteHandler;
import io.netty.handler.timeout.IdleStateHandler;

import java.util.concurrent.TimeUnit;

/**
 * @author CiyaZ
 */
public class WsImServerInitializer extends ChannelInitializer<SocketChannel> {

    private WsImServerHandler wsImServerHandler = new WsImServerHandler();

    @Override
    protected void initChannel(SocketChannel ch) throws Exception {
        ChannelPipeline pipeline = ch.pipeline();
        // HTTP协议编解码器
        pipeline.addLast(new HttpServerCodec());
        // HTTP协议组装
        pipeline.addLast(new HttpObjectAggregator(65535));
        // 分块写入
        pipeline.addLast(new ChunkedWriteHandler());
        // WebSocket协议处理器
        pipeline.addLast(new WebSocketServerProtocolHandler("/", null, true));
        // 心跳超时处理器
        pipeline.addLast(new IdleStateHandler(5, 7, 10, TimeUnit.SECONDS));
        pipeline.addLast(new WsIdleEventHandler());
        // 业务逻辑处理器
        pipeline.addLast(wsImServerHandler);
    }
}
```

这里注意`pipeline`的配置，了解WebSocket协议就知道，WebSocket建立连接时握手还是基于HTTP，所以首先配置了`HttpServerCodec`、`HttpObjectAggregator`这两个组件。

`ChunkedWriteHandler`能够实现分块写入，防止数据包太大导致JVM内存溢出。

`WebSocketServerProtocolHandler`用于处理WebSocket协议，第一个参数是服务端运行的地址。

关于心跳超时处理，和前一篇是一模一样的，这里就不再赘述了。

WsImServerHandler.java
```java
package com.ciyaz.demo.netty.ws.server;

import com.ciyaz.demo.netty.ws.model.Sms;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import io.netty.channel.Channel;
import io.netty.channel.ChannelHandler;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.SimpleChannelInboundHandler;
import io.netty.channel.group.ChannelGroup;
import io.netty.channel.group.DefaultChannelGroup;
import io.netty.handler.codec.http.websocketx.TextWebSocketFrame;
import io.netty.util.concurrent.GlobalEventExecutor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * @author CiyaZ
 */
@ChannelHandler.Sharable
public class WsImServerHandler extends SimpleChannelInboundHandler<TextWebSocketFrame> {
    /**
     * ChannelGroup其实可以看成包含很多Channel的线程安全Set
     */
    private ChannelGroup channels = new DefaultChannelGroup(GlobalEventExecutor.INSTANCE);

    private Logger logger = LoggerFactory.getLogger(this.getClass());
    private ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public void channelRegistered(ChannelHandlerContext ctx) throws Exception {
        broadcast("[" + ctx.channel().remoteAddress() + "] " + " 加入聊天");
        logger.info(ctx.channel().remoteAddress() + " 加入聊天");
        channels.add(ctx.channel());
    }

    @Override
    public void channelUnregistered(ChannelHandlerContext ctx) throws Exception {
        broadcast("[" + ctx.channel().remoteAddress() + "] " + " 退出聊天");
        logger.info(ctx.channel().remoteAddress() + " 退出聊天");
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        super.exceptionCaught(ctx, cause);
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, TextWebSocketFrame frame) throws Exception {
        Sms sms = objectMapper.readValue(frame.text(), Sms.class);
        if (sms != null) {
            if ("TYPE_SMS".equals(sms.getSmsType())) {
                broadcast("[" + ctx.channel().remoteAddress() + "] " + sms.getContent());
                logger.info(ctx.channel().remoteAddress() + " 收到消息");
            } else if ("TYPE_HEART_BEAT".equals(sms.getSmsType())) {
                echoHeartBeat(ctx.channel());
                logger.info(ctx.channel().remoteAddress() + " 收到心跳");
            }
        }
    }

    /**
     * 向一个Channel回复一个心跳包
     *
     * @param channel 通道
     */
    private void echoHeartBeat(Channel channel) {
        Sms sms = new Sms();
        sms.setId(1);
        sms.setContent("Heart beat echo!");
        sms.setSmsType("TYPE_HEART_BEAT");
        try {
            String smsStr = objectMapper.writeValueAsString(sms);
            TextWebSocketFrame frame = new TextWebSocketFrame(smsStr);
            channel.writeAndFlush(frame);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
    }

    /**
     * 向所有Channel广播消息
     *
     * @param message 消息字符串
     */
    private void broadcast(String message) {
        Sms sms = new Sms();
        sms.setId(1);
        sms.setContent(message);
        sms.setSmsType("TYPE_SMS");
        try {
            String smsStr = objectMapper.writeValueAsString(sms);
            TextWebSocketFrame frame = new TextWebSocketFrame(smsStr);
            channels.writeAndFlush(frame);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }
    }
}
```

对于WebSocket协议报文，我们需要处理的其实是Netty为我们包装的`WebSocketFrame`对象，而这里我们使用的是Json文本报文，因此直接使用其子类`TextWebSocketFrame`即可。具体逻辑非常简单，和之前其实是一样的，只不过把`protobuf`给换成了Json的序列化和反序列化器。

## 浏览器客户端

index.html
```html
<!doctype html>
<html lang="zh-cmn-Hans">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">

    <script src="index.js"></script>
    <title>ws测试</title>
</head>
<body>
    <div id="info-block"></div>
    <div>
        <label for="reply">回复：</label>
        <textarea id="reply"></textarea>
        <button id="reply-btn">发送消息</button>
    </div>
</body>
</html>
```

index.js
```javascript
window.onload = function () {
    var infoBlock = document.getElementById('info-block');
    var reply = document.getElementById('reply');
    var replyBtn = document.getElementById('reply-btn');

    var socket = new WebSocket('ws://127.0.0.1:8080/');
    // 收到消息时，解析json并显示
    socket.onmessage = function (msg) {
        var sms = JSON.parse(msg.data);
        if (sms.smsType === 'TYPE_SMS') {
            infoBlock.innerText += sms.content;
        }
    };
    // 发送消息时，使用序列化为json并写入socket
    replyBtn.onclick = function () {
        if (socket.readyState === WebSocket.OPEN) {
            var wsMsg = JSON.stringify({ id: 1, content: reply.value, smsType: 'TYPE_SMS' });
            socket.send(wsMsg);
        }
    };
    // 发送心跳包
    setInterval(function () {
        if (socket.readyState === WebSocket.OPEN) {
            var wsMsg = JSON.stringify({ id: 1, content: '', smsType: 'TYPE_HEART_BEAT' });
            socket.send(wsMsg);
        }
    }, 1000);
};
```

浏览器端就非常简单了，JavaScript天生支持Json，只需要调用WebSocket相关接口，读取消息和写入消息即可。

## 关于protobuf

实际上，我们也可以用WebSocket来传输protobuf序列化的二进制数据，protobuf比文本的json更加高效一些，但大多数情况Json已经足够使用了，而且文本格式方便调试，这里就不多介绍了。
