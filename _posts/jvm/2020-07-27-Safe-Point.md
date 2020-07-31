---
layout: post
title: Safepoints in HotSpot JVM
categories: Jvm
description: GC安全点浅谈
keywords: JVM ，Safepoints 
---

GC安全点浅谈, stop-the-world时java线程是如何暂停的？然后又是如何恢复？

**目录**

* TOC
{:toc}

## GC时java线程是如何暂停的？然后又是如何恢复？

我们一直都知道当发生gc时，正在执行的java code线程需要全部停下来(运行本机代码的线程可以继续运行，只要它们不与JVM交互)，才可以进行垃圾回收，也就是我们知道的stop-the-world。

那么线程是如何暂停的？然后又是如何恢复？

本文章源码基于OpenJdk8，希望通过本次学习，我们能对GC时java线程的暂停和恢复机制有大概了解。

## 什么是safepoint？

```sh
1.safepoint安全点顾名思义是指一些特定的位置，当线程运行到这些位置时，
线程的一些状态可以被确定(the thread's representation of it's Java machine state is well described)。

2.程序执行期间所有GC根已知且所有堆对象内容一致的点。

3.当线程执行到这些位置的时候，说明虚拟机当前的状态是安全的，如果有需要，可以在这个位置暂停。

4.在safepoint会生成polling操作去检查全局的一个poling page是否可读，从而决定java线程是否需要挂起。
```

```sh
But many other safepoint based VM operations exist, for example: biased locking revocation, thread stack dumps, thread suspension or stopping (i.e. The java.lang.Thread.stop() method) and numerous inspection/modification operations requested through JVMTI.

官方介绍safepoint除了用于GC外还可以用在不同地方，例如：取消偏向锁、线程堆栈转储、线程的暂停或终止...

这里我们只研究 GC safepoint。
```

## GC整体过程大体浏览（openjdk-8）

![](/images/posts/jvm/safepoint/1.jpg)

为了搞明白线程是如何被挂起? 以及如何恢复？有必要了解下GC整体过程。

```sh
大体流程:
1.VMThread在创建VMThread对象的同时会创建一个储存VM操作的队列VMOperationQueue。
2.启动方法run()里会调用loop（）方法。
3.GC操作会添加到VMOperationQueue队列。
4.loop（）方法执行如下：
  .不停的从VMOperationQueue队列取出操作。
  .假设取出的是GC操作，那么调用SafepointSynchronize::begin()进入安全点,将线程挂起。
  .线程被挂起后，会执行evaluate_opesration开始gc。
  .gc完毕调用SafepointSynchronize::end()将线程唤醒。
```

## 源码初探

### 1.VMThread创建

vmThread.cpp:

![](/images/posts/jvm/safepoint/2.png)

Vmthread负责调度执行虚拟机内部的VM线程操作，如GC操作等，在JVM实例创建时进行初始化。

这里除了创建VMThread对象，还会伴随着创建一个VMOperationQueue队列（线程操作队列，例如GC操作）。

### 2.VMThread启动方法

```java
void VMThread::run() {
  ...
  ...
  // Wait for VM_Operations until termination
  this->loop();

}
```
VMThread启动方法run（），我们看到它会调用loop()方法。

### 3.添加GC操作到VMOperationQueue队列

collectorPolicy.cpp：

![](/images/posts/jvm/safepoint/7.png)

触发gc操作时，会调用VMThread::execute()方法

![](/images/posts/jvm/safepoint/8.png)

_vm_queue->add(op)将当前线程操作加入队列（例如：GC操作）

### 4.VMThread::loop()

![](/images/posts/jvm/safepoint/3.png)

```sh
while（true）循环里，会不停地从VMOperationQueue队列取线程操作（例如：GC操作）。
remove_next()会对VM_operation优先级进行重新排序，并返回队列头部的VM_operation，如果没有操作的话会一直等待。
```

![](/images/posts/jvm/safepoint/5.png)

```sh
假设从VMOperationQueue队列取出来的是gc操作，那么需要在安全点操作：

  .调用SafepointSynchronize::begin()进入safepoint.cpp，最终会使所有的java线程挂起。

  .调用evaluate_operation(_cur_vm_operation)执行当前vmOperation操作，也就是GC操作。
```

```java
// Complete safepoint synchronization
SafepointSynchronize::end();
```

GC完毕调用SafepointSynchronize::end()将线程唤醒。

## 如何将线程挂起？

SafepointSynchronize::begin()方法会将线程挂起，这里我们只关注重点，如何将java线程挂起？

![](/images/posts/jvm/safepoint/9.png)

将java线程挂起时，java线程可能处于不同状态，所以挂起的机制也不同，大致就是如上图5种状态。

### 1.Running interpreted 

SafepointSynchronize::begin()->Interpreter::notice_safepoints()：

```java
void TemplateInterpreter::notice_safepoints() {
  if (!_notice_safepoints) {
    // switch to safepoint dispatch table
    _notice_safepoints = true;
    copy_table((address*)&_safept_table, (address*)&_active_table, sizeof(_active_table) / sizeof(address));
  }
}
```

```sh
需要先解释一个概念，线程处于不同状态下，DispatchTable会设置不同的值：
._active_table 正在解释运行的线程
._normal_table 正常运行的
._safept_table 处于安全点

DispatchTable可以理解为调度表的意思，会记录方法地址跳转。
当线程在解释java字节码的时候，想让线程进入safepoint，只需通知解释器将DispatchTable替换为_safept_table，解释器就会把指令跳转到safepoint，
然后检查状态，比如检查某个内存页位置，从而让线程阻塞。这里检查内存页操作，后面会讲。
```

### 1.

## 一些概念辅助了解

### 1.Poling page

```sh
Poling page是在jvm初始化启动的时候会初始化的一个单独的内存页面，这个页面是让运行的编译过的代码的线程进入停止状态的关键，
它是一个全局的safepoint polling page。 
```

### 2.jvm 解释器

```sh
JVM内部的一个数据结构，用来保存字节码和机器码。
```

### 3.JIT编译器

```sh
Java程序最初是仅仅通过解释器解释执行的，即对字节码逐条解释执行，这种方式的执行速度相对会比较慢，
尤其当某个方法或代码块运行的特别频繁时，这种方式的执行效率就显得很低。

于是后来在虚拟机中引入了JIT编译器（即时编译器），当虚拟机发现某个方法或代码块运行特别频繁时，就会把这些代码认定为“Hot Spot Code”（热点代码），
为了提高热点代码的执行效率，在运行时，虚拟机将会把这些代码编译成与本地平台相关的机器码，并进行各层次的优化，完成这项任务的正是JIT编译器。

java的JIT会直接编译一些热门的源码到机器码直接执行而不需要在解释执行从而提高效率，在编译的代码中，当函数或者方法块返回的时候会去访问一个内存poling页面。

通过JIT编译的代码里，会在所有方法的返回之前，以及所有非counted loop的循环（无界循环）回跳之前放置一个safepoint，为了防止发生GC需要STW时，该线程一直不能暂停。

另外，JIT编译器在生成机器码的同时会为每个safepoint生成一些“调试符号信息”，为GC生成的符号信息是OopMap，指出栈上和寄存器里哪里有GC管理的指针。

守护线程是一种特殊的线程，就和它的名字一样，它是系统的守护者，在后台默默地完成一些系统性的服务，比如垃圾回收线程、JIT线程就可以理解为守护线程。
```

### 4.抢先式中断

```sh
抢先式中断不需要线程的执行代码主动去配合，在GC发生时，首先把所有线程全部中断，如果发现有线程中断的地方不在安全点上，就恢复线程，让它跑到安全点上。

现在几乎没有虚拟机实现采用抢先式中断来暂停线程响应GC事件。 
```

### 5.主动式中断

```sh
主动式中断的思想是当GC需要中断线程的时候，不直接对线程操作，仅仅简单地设置一个标志，

各个线程执行时主动去轮询这个标志，发现中断标志为真时就自己中断挂起。轮询标志的地方和安全点是重合的。
```

### 6._active_table

```sh
解释运行的线程一直都在使用_active_table,关键处就是在进入saftpoint 的时候，用_safept_table替换_active_table,
在退出saftpoint 的时候，使用_normal_table来替换_active_table。
```

### 7.JVM有两种执行方式

```sh
解释型和编译型(JIT)，JVM要保证这两种执行方式下safepoint都能工作。
```

### 8.mprotect

设置内存映像保护

### 9.safepoint的使用场景

```sh
.垃圾回收(这是最常见的场景)
.取消偏向锁(JVM会使用偏向锁来优化锁的获取过程)
.Class重定义(比如常见的hotswap和instrumentation)
.Code Cache Flushing(JDK1.8在CodeCache满的情况下就可能出现)
.线程堆栈转储(jstack命令)
```

### 10.UseCompilerSafepoints 和 DeferPollingPageLoopCount 默认值

10.在默认的情况下这2个参数分别是true和-1
