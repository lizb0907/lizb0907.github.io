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

## 几个很重要的概念

### 1.Poling page

```sh
Poling page是在jvm初始化启动的时候会初始化的一个单独的内存页面，这个页面是让运行的编译过的代码的线程进入停止状态的关键，
它是一个全局的safepoint polling page。
```

```sh
抢先式中断不需要线程的执行代码主动去配合，在GC发生时，首先把所有线程全部中断，如果发现有线程中断的地方不在安全点上，就恢复线程，让它跑到安全点上。

现在几乎没有虚拟机实现采用抢先式中断来暂停线程响应GC事件。 
```

```sh
主动式中断的思想是当GC需要中断线程的时候，不直接对线程操作，仅仅简单地设置一个标志，

各个线程执行时主动去轮询这个标志，发现中断标志为真时就自己中断挂起。轮询标志的地方和安全点是重合的。
```

```sh
mprotect设置内存映像保护
```

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

## safepoint设置的主要位置

```sh
1. 循环的末尾 (防止大循环的时候一直不进入safepoint，而其他线程在等待它进safepoint）
2. 方法返回前
3. 调用方法的call之后
4. 抛出异常的位置
Safepoint的选定既不能太少以致于让GC等待时间太长，也不能过于频繁以致于过分增大运行时的负荷。所以，安全点的选定基本上是以程序“是否具有让程序长时间执行的特征”为标准进行选定的——因为每条指令执行的时间都非常短暂，程序不太可能因为指令流长度太长这个原因而过长时间运行，“长时间执行”的最明显特征就是指令序列复用，例如方法调用、循环跳转、异常跳转等，所以具有这些功能的指令才会产生Safepoint。
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

## 不同状态下，挂起操作

![](/images/posts/jvm/safepoint/9.png)

SafepointSynchronize::begin()方法会做各种操作，最终将线程挂起，这里我们只关注重点，如何将java线程挂起？

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

### 2.Runing in native code

我们前面也提到过，运行本机代码的线程可以继续运行。

如果VM thread发现一个Java thread正在执行native code，并不会等待该Java thread阻塞，当Java thread从native code返回时，必须检查safepoint状态，看是否需要进行阻塞。


### 3.Runing compiled code

```java
if (UseCompilerSafepoints && DeferPollingPageLoopCount < 0) {
    // Make polling safepoint aware
    guarantee (PageArmed == 0, "invariant") ;
    PageArmed = 1 ;
    os::make_polling_page_unreadable();
  }
```

```sh
java线程执行编译代码时,会调用make_polling_page_unreadable函数，make_polling_page_unreadable在不同的操作系统中实现不同，我们只看linux下的实现:

make_polling_page_unreadable()->guard_memory((char*)_polling_page, Linux::page_size())->linux_mprotect(addr, size, PROT_NONE)->
mprotect(bottom, size, prot)。

mprotect(bottom, size, prot)：
1.在Linux中，mprotect()函数可以用来修改一段指定内存区域（例如全局内存页）的保护属性。
2.mprotect()函数把自start开始的、长度为len的内存区的保护属性修改为prot指定的值。
3.prot可以取以下几个值：
  .PROT_READ：表示内存段内的内容可写。
  .PROT_WRITE：表示内存段内的内容可读。
  .PROT_EXEC：表示内存段中的内容可执行。
  .PROT_NONE：表示内存段中的内容根本没法访问。

我们看到最终是调用mprotect函数，并且prot传入的值是PROT_NONE（不可访问）。

hotspot采用的是主动式中断，当GC需要中断线程的时候，不直接对线程操作，仅仅简单地设置一个标志，各个线程执行时主动去轮询这个标志。

mprotect函数将全局的内存页设置为不可读，当线程访问到该内存地址，发现不可读，则该线程会被挂起等待。
```

这个检查内存页是否可读是一个轮询指令，它的插入是在JIT编译中，编译器会把很多的是否进入安全点的检查操作插入到机器码指令中。

比如下面的指令HotSpot生成的轮询指令：

![](/images/posts/jvm/safepoint/11.png)

```sh
这个内存页地址就是0x160100，当需要暂停线程时，虚拟机把0x160100地址的内存页设置为不可读，也就是我们上面讲的调用mprotect函数，

线程执行到test指令时就会产生一个自陷异常信号，在预先注册的异常处理器中暂停线程实现等待。
```

可以简单参考下图：

![](/images/posts/jvm/safepoint/12.png)

### 4.Blocked

safepoint只能处理正在运行的线程，它们可以主动运行到safepoint。而一些Sleep或者被blocked的线程不能主动运行到safepoint。这些线程也需要在GC的时候被标记检查，JVM引入了safe region的概念。safe region是指一块区域，这块区域中的引用都不会被修改，比如线程被阻塞了，那么它的线程堆栈中的引用是不会被修改的，JVM可以安全地进行标记。线程进入到safe region的时候先标识自己进入了safe region，等它被唤醒准备离开safe region的时候，先检查能否离开，如果GC已经完成，那么可以离开，否则就在safe region呆着。

### 5.VM or Transitioning between states

当线程处在状态转化的时候，线程会去安全点然后检查状态，如果要阻塞，就自己阻塞了。

## 挂起机制的触发

![](/images/posts/jvm/safepoint/13.png)

如果调用进程试图以违反保护的方式访问内存，也就是访问到不可读的内存页，那么内核会为该进程生成一个SIGSEGV信号。

调用如上图，最终会调用SafepointSynchronize::block(thread())阻塞。

## 执行GC操作

当线程在安全点阻塞后，调用evaluate_operation(_cur_vm_operation)执行当前vmOperation操作。

![](/images/posts/jvm/safepoint/14.png)

doit()方法会执行各种vm操作，如果当前是gc操作会调用vmGCOperations下的VM_GenCollectForAllocation::doit()方法实现相关操作。

## 线程如何恢复的？

![](/images/posts/jvm/safepoint/15.png)

make_polling_page_readable设置全局内存页为可读，操作一样只是此时传入的是PROT_READ可读。

重新将将DispatchTable从_safept_table更新为_nomal_table。

启动暂停的线程。

## 总结

安全点机制其实实现起来非常复杂，由于精力和水平有限，难免会出现错误，欢迎大家指正，并且还有很多细节没有继续往下抠。

不过到此为止，我们也能对GC时java线程的暂停和恢复机制有了大概了解。

还有一些比较有意义的问题有兴趣的可以继续往下研究:

例如：大多数人知道VMThread在进行GC时会等到所有的Java线程进入安全点阻塞后才可以进行（这里的“所有”指必须进入的java线程），否则VMThread会阻塞进行等待。

那VMThread在调用void SafepointSynchronize::begin()方法后是不是立马阻塞的？(ps:个人研究后，发现其实不是立马就阻塞的)

当所有java线程进入安全点阻塞后，VMThread又是如何被唤醒开始执行GC?

