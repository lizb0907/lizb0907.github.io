---
layout: post
title: ConcurrentLinkedQueue源码
categories: Concurrent
description: ConcurrentLinkedQueue源码学习
keywords: ConcurrentLinkedQueue，源码，学习
---

ConcurrentLinkedQueue源码学习

**目录**

* TOC
{:toc}

## 顶部一些基本概念

```sh
1.An unbounded thread-safe {@linkplain Queue queue} based on linked nodes.
基于链表的无界线程安全队列

2.This queue orders elements FIFO (first-in-first-out)
有序，先进先出

3.Like most other concurrent collection implementations, this class does not permit the use of {@code null} elements.
类似绝大多数的并发集合，ConcurrentLinkedQueue不允许使用null元素


4. <p>Iterators are <i>weakly consistent</i>, returning elements reflecting the state of the queue at some point at or since the creation of the iterator.  
They do <em>not</em> throw {@link java.util.ConcurrentModificationException}, and may proceed concurrently with other operations.  
Elements contained in the queue since the creation of the iterator will be returned exactly once.
迭代器弱一致性，返回的元素是队列某个节点或迭代器创建时状态的反映。
当进行并发操作时，不会抛出java.util.ConcurrentModificationException异常。
能够保持创建迭代器后的元素一定被正确的返回。

```
