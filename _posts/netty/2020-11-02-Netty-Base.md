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


