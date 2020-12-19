---
layout: post
title: Netty基础概览
categories: Netty
description: netty基础概念掌握
keywords: netty,base
---

netty基础概念掌握

**目录**

* TOC
{:toc}

## netty简介

## 核心组件概念

### 1.Channel

### 2.EventLoop

### 3.ChannelFuture

### 4.ChannelHandler

### 5.ChannelPipeline

## 遗留问题

1.ByteBuf的池化技术ByteBufAllocator

2.当某个 ChannelInboundHandler 的实现重写 channelRead()方法时，

必须要显式地释放与池化的 ByteBuf 实例相关的内存吗？不释放会导致内存泄露吗？


### 6.ChannelHandlerContext

概念:
```sh
1.代表了ChannelHandler 和 ChannelPipeline 之间的关联，每当有ChannelHandler添加到ChannelPipeline
中时，都会创建ChannelHandlerContext。

2.管理所关联的ChannerHandler和在同一个ChannelPipeline中的其他ChannelHandler之间的交互。

3.每一个ChannerHandler都有一个绑定的ChannelHandlerContext。
```

### 7.处理入站异常

```java
protected void initChannel(SocketChannel socketChannel) throws Exception
{
    //责任链，按顺序处理handler
    ChannelPipeline p = socketChannel.pipeline();

    // 超时处理
    p.addLast(new IdleStateHandler(600, 0, 600, TimeUnit.SECONDS));
    //解码
    p.addLast("decoder", new ClientDecode());
    //编码
    p.addLast("encoder", new ClientEncode());
//        p.addLast("encoder", new ClientEncodePerformance());
    //handler一般放最后，因为handler有处理异常，需要保证所有的异常能被处理
    p.addLast("handler", new ClientHandler());

}
```

```sh
异常将会继续按照入站方向流动（就像所有的入站事件一样），所以实现了前面所示逻
辑的 ChannelInboundHandler 通常位于 ChannelPipeline 的最后。这确保了所有的入站
异常都总是会被处理，无论它们可能会发生在 ChannelPipeline 中的什么位置。
```
