---
layout: post
title: LinkedList源码分析(JDK1.8)
categories: Java-Scource-Code-Analysis
description: LinkedList源码分析
keywords: Java,Base
---

LinkedList源码分析(JDK1.8)

**目录**

* TOC
{:toc}

## LinkedList类继承信息

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

继承抽象的AbstractSequentiaList类，提供了一个基本的List接口实现，为实现序列访问的数据储存结构的提供了所需要的最小化的接口实现。

同时LinkedList也实现了Cloneable、java.io.Serializable、Deque（双端队列）等接口


## 接着我们看主要的类成员属性

```java
/**
 * 当前链表长度
 */
transient int size = 0;

/**
 * 头结点
 */
transient Node<E> first;

/**
 * 尾结点
 */
transient Node<E> last;
```
transient不可序列化关键字。

## 内部类node

```java
/**
 * 静态内部类，它的创建是不需要依赖于外围类,可以被实例化
 * @param <E>
 */
private static class Node<E> {
    //当前结点值
    E item;
    //下一结点
    Node<E> next;
    //上一节点
    Node<E> prev;
    //构造函数初始化
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```
分析：我们知道java的LinkedList是个双向链表，java没有指针，那么是怎么找到前一个元素结点和后一个元素结点呢？

从上面的代码很容易就可以看出，其实就是在Node类的成员属性下直接记录了下一个结点Node<E> next 和 上一个结点 Node<E> prev，从而达到模拟指针的效果。

## 构造方法

### 1.不带参的构造方法，啥也没做

```java
/**
 * Constructs an empty list.
 */
public LinkedList() {
}
```

### 2.带参构造方法

```java
/**
 * 带参构造方法，传入集合
 */
public LinkedList(Collection<? extends E> c) {
    //构造方法
    this();
    //添加方法
    addAll(c);
}
```
分析：这里将集合直接传入addAll（）方法


```java
/**
 * 从索引为index结点的尾部，开始插入所有集合的元素
 */
public boolean addAll(int index, Collection<? extends E> c) {
    //检验长度是否在链表长度区间
    checkPositionIndex(index);
    //将集合转为数组
    Object[] a = c.toArray();
    //数组长度
    int numNew = a.length;
    //如果等于0直接返回false
    if (numNew == 0)
        return false;

    //两个记录结点
    Node<E> pred, succ;
    //如果当前传入的长度等于链表当前最大长度
    if (index == size) {
        //succ结点等于null
        succ = null;
        //pred结点等于尾结点
        pred = last;
    } else {
        //返回index索引的结点
        succ = node(index);
        //pred记录等于index索引结点的上一个结点
        pred = succ.prev;
    }

    //遍历数组
    for (Object o : a) {
        //将值强转为E类型
        @SuppressWarnings("unchecked") E e = (E) o;
        //每次循环new一个newNode结点，newNode结点的值等于e,newNode上一结点prev等于pred记录结点,newNode下一节点等于null
        //其实这个newNode就是pred的下一个结点，因为newNode的上一节点等于pred
        Node<E> newNode = new Node<>(pred, e, null);
        //如果当前记录结点pred等于null
        if (pred == null)
            //则头结点等于newNode
            first = newNode;
        else
            //当前记录pred的下一节点等于newNode,相当于把succ记录结点给覆盖了
            pred.next = newNode;
        //当前记录结点更新为newNode,也就是相当于链表往后移动一位
        pred = newNode;
    }

    if (succ == null) {
        //尾结点等于当前记录结点pred
        last = pred;
    } else {
        //重新把pred结点下一个结点赋值为succ记录结点
        pred.next = succ;
        //并且让succ记录结点的上一个结点等于最新的pred记录结点
        succ.prev = pred;
    }

    //链表的长度加上集合的长度
    size += numNew;
    //修改次数加1
    modCount++;
    return true;
}
```

我们要知道这个addAll(）方法是从索引为index结点开始向后插入元素的，链表的索引是从0开始的。那么到底是怎么插入元素的呢?

#### 1.将集合转为数组

```java
//检验长度是否在链表长度区间
checkPositionIndex(index);
//将集合转为数组
Object[] a = c.toArray();
//数组长度
int numNew = a.length;
//如果等于0直接返回false
if (numNew == 0)
    return false;
```

校验下传入的index长度是否在链表区间（0到size），然后将集合转为数组，如果数组长度等于0的话，直接返回了。


#### 2.pred, succ含义

```java
//两个记录结点
Node<E> pred, succ;
//如果当前传入的索引等于链表当前最大长度
if (index == size) {
    //succ结点等于null
    succ = null;
    //pred结点等于尾结点
    pred = last;
} else {
    //返回index索引的结点
    succ = node(index);
    //pred记录等于index索引结点的上一个结点
    pred = succ.prev;
}
```

分析：两个记录结点Node<E> pred，succ非常重要，它是为了方便我们我们操作链表的。

if(index == size)，当前传进来的索引index等于链表长度size，那么succ结点等于null，为什么呢？因为index=size下标已经超出链表了，所以值肯定为null了。

也因此，我们可以理解succ其实就是索引为index的结点，也就是我们要开始插入元素的结点，然后pred结点等于尾结点。

从这可以看出，我们前面构造方法传入当前链表长度和集合是从链表的尾部开始添加的（也就是尾结点的下一个结点），符合逻辑。


else当前传入的索引不大于链表长度size，succ等于index索引的结点，pred还是等于index索引的上一个结点。

我们看下这个node(index)方法，看完之后我们明白它确实是返回index索引的结点。

```java
/**
 * 传入index长度，链表返回索引为index的node结点（注意这里的链表索引也是从0开始）
 * 如果传入的长度小于链表长度一遍，那么从链表头结点开始遍历
 * 如果传入的长度大于或等于链表长度的一半，那么从链表的尾结点开始遍历
 */
Node<E> node(int index) {
    //如果传入的长度小于链表长度的一半（size >> 1链表长度除以2，这种写法运算速度稍快，可以根据实际需求应用）
    if (index < (size >> 1)) {
        //当前记录结点等于头结点
        Node<E> x = first;
        //从下表为0开始遍历
        for (int i = 0; i < index; i++)
            //取下一节点
            x = x.next;
        //所以返回的是索引为indx的点！！
        return x;
    } else {
        //如果传入的长度大于或者等于当前链表长度的一半
        //当前记录结点等于尾结点
        Node<E> x = last;
        //从链表末尾开始往前遍历
        for (int i = size - 1; i > index; i--)
            //当前记录结点等于上一节点
            x = x.prev;
        //返回索引为index的点！
        return x;
    }
}
```
遍历的时候有个小技巧，如果索引小于链表长度的一半从头结点开始往后遍历，否则从尾结点往前开始遍历，节省时间。遍历取到index索引的节点。

Node<E> pred, succ;

这两个结点的含义，succ代表当前传入索引index的结点，pred代表索引index结点的上一个结点，这两个结点是为了我们方便操作链表。


#### 3.遍历数组值添加node

```java
//遍历数组
for (Object o : a) {
    //将值强转为E类型
    @SuppressWarnings("unchecked") E e = (E) o;
    //每次循环new一个newNode结点，newNode结点的值等于e,newNode上一结点prev等于pred记录结点,newNode下一节点等于null
    //其实这个newNode就是pred的下一个结点，因为newNode的上一节点等于pred
    Node<E> newNode = new Node<>(pred, e, null);
    //如果当前记录结点pred等于null
    if (pred == null)
        //则头结点等于newNode
        first = newNode;
    else
        //当前记录pred的下一节点等于newNode,相当于把succ记录结点给覆盖了
        pred.next = newNode;
    //当前记录结点更新为newNode,也就是相当于链表往后移动一位
    pred = newNode;
}
```

分析：前面将集合转为了数组，这里遍历数组，每次循环将遍历的值转为类型E（泛型）。

new一个名为newNode的结点，构造函数初始化，newNode结点的上一结点等于pred结点（pred代表索引index结点的上一个结点），newNode结点的值为e，newNode结点的下一节点null。

看懂了没，其实这个newNode相当于“覆盖”了succ结点（当前索引为index的结点），这里画个图给大家理解下吧

![](/images/posts/java-scource-code-analysis/1.png)

newNode结点的上一结点等于pred结点，下一节点此时为null。

![](/images/posts/java-scource-code-analysis/2.png)

当pred == null时，链表的头结点等于newNode。因为如果当前传入索引index的结点succ为头结点，那么上一个结点pred就肯定会null。

![](/images/posts/java-scource-code-analysis/3.png)

pred不等于null时，pred结点的下一节点更改为newNode结点。


![](/images/posts/java-scource-code-analysis/4.png)

pred.next = newNode也就是当前传入索引index的结点succ的上一个结点变为了newNode结点，以便下次再添加时，pred变为newNode，

for循环一次结束，第二次循环的话，添加效果如上图。 


如此，直至for循环结束。


#### 4.最后增加修改次数

```java
if (succ == null) {
    //尾结点等于当前记录结点pred
    last = pred;
} else {
    //重新把pred结点下一个结点赋值为succ记录结点
    pred.next = succ;
    //并且让succ记录结点的上一个结点等于最新的pred记录结点
    succ.prev = pred;
}

//链表的长度加上集合的长度
size += numNew;
//修改次数加1
modCount++;
```

分析：当传入的index等于链表长度size时，链表索引是从0开始的，所以超出链表索引，那么当前索引succ == null，则直接让pred等于尾结点（很好理解吧？succ的上一个结点不就是索

引为index-1的尾结点）。如果不等于null，把pred（此时的pred等于最后一个newNode结点）下一个结点指向succ结点，succ的上一个结点指向pred，效果如下图：

![](/images/posts/java-scource-code-analysis/5.png)

到此，addAll()方法分析结束了，总结下，我们明白了在添加链表元素时并不是真正所谓上的“添加”，这里的添加其实是改变指针的指向从而达到往链表添加的目的，删除链表元素也类似。


## 常用的几个方法

### 1.往链表末尾添加单个元素方法

```java
/**
 * 往链表末尾添加元素
 */
public boolean add(E e) {
    linkLast(e);
    return true;
}
分析：传入元素的值，调用linkLast()方法
```

```java
/**
 * Links e as last element.
 */
void linkLast(E e) {
    //尾结点
    final Node<E> l = last;
    //new一个新结点，上一个结点等于尾结点，结点值等于e，下一个结点等于null
    final Node<E> newNode = new Node<>(l, e, null);
    //尾结点等于新结点
    last = newNode;
    //链表没有元素，则尾结点为null
    if (l == null)
        //让头结点等于新结点
        first = newNode;
    else
        //否则，尾结点的下一个结点等于新结点
        l.next = newNode;
    //链表长度加1
    size++;
    //修改次数加1
    modCount++;
}
```
分析：这里添加和上面思想其实是一致的，所以就不详解了和画图了，直接看上面我写的注释。

### 2.根据索引删除元素

```java
/**
 * 根据索引删除结点
 */
public E remove(int index) {
    //检查索引是否越界
    checkElementIndex(index);
    //node(index)索引为index的node结点,传入unLink()方法
    return unlink(node(index));
}

继续往下看unlink()方法，传入的是索引为index的node结点对象
/**
 * 删除指定节点元素
 */
E unlink(Node<E> x) {
    // assert x != null;
    //当前节点元素值
    final E element = x.item;
    //当前节点的下一节点
    final Node<E> next = x.next;
    //当前节点的上一节点
    final Node<E> prev = x.prev;

    //如果当前节点的上一节点prev为null
    if (prev == null) {
        //头节点等于下一节点
        first = next;
    } else {
        //上一节点成员变量下一节点改为next
        prev.next = next;
        //当前节点的上一节点置为null方便回收
        x.prev = null;
    }

    //如果当前节点的下一节点next为null(也就是链表最后一个元素)
    if (next == null) {
        //链表的尾节点等于当前节点的上一节点prev
        last = prev;
    } else {
        //当前节点的下一节点next的成员变量prev等于当前节点的上一节点prev
        next.prev = prev;
        //当前节点的下一节点置为null
        x.next = null;
    }

    //当前节点的值置为null
    x.item = null;
    //当前链表长度减1
    size--;
    //修改次数加1
    modCount++;
    //返回当前节点元素值
    return element;
}
```
分析：这里的删除指定节点，其实道理也是类似，通过改变上一节点和下一节点各自的成员变量引用从而达到删除当前元素节点的效果。

![](/images/posts/java-scource-code-analysis/6.png)


到此为止，LinkedList其它常用方法源码不再进行分析了，基本掌握了核心的原理，其它的都是类似。

这样我们就简单的看了下LinkedList源码。