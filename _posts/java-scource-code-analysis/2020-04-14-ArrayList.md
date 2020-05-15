---
layout: post
title: ArrayList源码分析
categories: Java-Scource-Code-Analysis
description: ArrayList源码分析
keywords: Java,Base
---

ArrayList源码分析

**目录**

* TOC
{:toc}

## ArrayList类图

```java

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

继承AbstracList抽象父类

然后分别实现：List接口（规定一些方法）、RandomAccess(可随机访问)、Cloneable（克隆接口）、Serializable（可序列化接口）


## 接着我们看主要的类成员属性

### 1.版本号

private static final long serialVersionUID = 8683452581122892189L;

### 2.容量为10的常量

```java
/**
 * Default initial capacity.
 */
private static final int DEFAULT_CAPACITY = 10;
```

### 3.空数组实例（List list = new ArrayList(0) 如果带参初始化，但是参数为0，那么数组初始为该空数组）

```java
/**
 * Shared empty array instance used for empty instances.
 *
 */
private static final Object[] EMPTY_ELEMENTDATA = {};
```

### 4.用于默认大小的空实例的共享空数组实例。

```java
/**
 * Shared empty array instance used for default sized empty instances. We
 * distinguish this from EMPTY_ELEMENTDATA to know how much to inflate when
 * first element is added.
 */
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
```

我们将其与EMPTY_ELEMENTDATA区分开来，以了解在添加第一个元素时要膨胀多少。

（ List list = new ArrayList() 不传参这种初始化方式，数组会初始为该默认的空数组，与上面区分开来）

### 5.元素数组（从这里我们其实就能知道ArrayList底层是以数组为数据结构进行操作的）

```java
/**
 * The array buffer into which the elements of the ArrayList are stored.
 * The capacity of the ArrayList is the length of this array buffer. Any
 * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
 * will be expanded to DEFAULT_CAPACITY when the first element is added.
 */
transient Object[] elementData; // non-private to simplify nested class access
```

### 6.记录数组下标用的，默认为0，每次往数组里add一个元素，size++

```java
/**
 * The size of the ArrayList (the number of elements it contains).
 *
 * @serial
 */
private int size;
```

## 看下ArrayList初始化，ArrayList是通过构造方法进行初始化的。有三种不同的初始化方法，对应三种不同的构造方法。

### 1.事先传进来一个ArrayList初始化长度。

```java
/**
 * Constructs an empty list with the specified initial capacity.
 *
 * @param  initialCapacity  the initial capacity of the list
 * @throws IllegalArgumentException if the specified initial capacity
 *         is negative
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        //如果传进来的长度大于0，则直接初始化Object[]传进来的长度数组
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        //如果长度为0则让数组等于Object[] EMPTY_ELEMENTDATA = {};空数组
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        //抛错
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
```

也就是说我们可以通过 List list = new ArrayList(20)  来构造一个具有指定初始容量的空列表


### 2. List list = new ArrayList() 不传参（ps:jdk1.8版本现在初始化是为空数组了！！）

```java
/**
 * 不传参，初始化一个空数组，与上面空数组不为同一个
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```


### 3.传入的参数为集合时，先将集合转为数组再赋值给elementData数组

```java
/**
 * 当传递的参数为集合类型时，会把集合类型转化为数组类型，并赋值给elementData
 */
public ArrayList(Collection<? extends E> c) {
    //先转为数组
    elementData = c.toArray();
    //参数长度不为0
    if ((size = elementData.length) != 0) {
        //如果没有成功转为Object型数组,Object[].class得到的是class [Ljava.lang.Object;
        if (elementData.getClass() != Object[].class)
            //那么就进行拷贝
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

## 添加方法 add()

### 1.这里我们首先假设是以List list = new ArrayList() 不传参的方式来初始化的ArrayList

然后我们第一次执行添加方法 list.add() 

注意这里的假设是下面所有分析的前提！

```java
/**
 * 将指定的元素添加到此列表的尾部。
 */
public boolean add(E e) {
    //确保数组有合适的大小
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //放入对应的数组下表
    elementData[size++] = e;
    return true;
}
```

分析：在执行添加的时候，会先调用ensureCapacityInternal()方法来确保当前的数组有合适的大小来添加元素

ensureCapacityInternal(size + 1)因为是第一次调用，并且初始化方法为List list = new ArrayList()，所以传入的参数为1


```java
private void ensureCapacityInternal(int minCapacity) {
    //如果elementData数组等于默认空元素数组则进入下面的逻辑（ps:我们假设是不传参初始化的ArrayList，所以会进入下面括号内的逻辑）
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        //DEFAULT_CAPACITY=10，比较两者数的大小，较大值
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    //将参数继续传入该函数
    ensureExplicitCapacity(minCapacity);
}
```
分析：这里minCapacity传入的时候为1，经过比较取大值后，minCapacity为10。

这里我们可能会联想到，如果以List list = new ArrayList()方式初始化的空数组，在我们第一次进行add添加时，空数组需要进行扩容，那么扩容是不是一下扩容了10个长度大小？

我们继续往下分析


```java
private void ensureExplicitCapacity(int minCapacity) {
    //记录修改次数加1
    modCount++;
    //当前传入minCapacity长度大于当前数组长度
    if (minCapacity - elementData.length > 0)
        //将参数传入该函数
        grow(minCapacity);
}
```

分析：modCount++，然后执行grow（）方法

```java
/**
 * 真正执行数组扩容的方法，先判断扩容的长度，最后执行Arrays.copyOf扩容
 */
private void grow(int minCapacity) {
    //当前旧的数组长度
    int oldCapacity = elementData.length;
    //新数组长度等于 = 旧数组长度 + 旧数组长度除以2
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //小于传进来的参数则等于传进来的参数
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //大于int的最大长度(2^31 - 1) 再减去8长度，传入minCapacity参数,执行hugeCapacity（）方法,指定新容量
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    //执行Arrays.copyOf将旧数组扩容成新数组长度为newCapacity
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```
分析：在我们假设的前提下，minCapacity当前等于10，当前oldCapacity为0。

所以，经过逻辑判断后，newCapcity等于10。最后执行Arrays.copyOf(elementData, newCapacity)

将空数组，扩容成长度为10的新数组！ 

这我们也就搞明白了，如果我们初始化ArrayList不传参，那么第一次添加元素时，数组会先扩容10个长度。

注意我们这里只是明确搞懂了初始化方式为不传参，第一次添加元素时，数组扩容机制。


接下来，根据前面的判断，我们很容易分析到如果当前list的元素小于10个，那么数组是不进行扩容的。

（你想想写这玩意的那帮家伙，肯定是这么干的，数组只有满了，才会去扩容，不然就是傻子。。。）

我们可以分析下第二次添加元素的情景:

```java
/**
 * 将指定的元素添加到此列表的尾部。
 */
public boolean add(E e) {
    //确保数组有合适的大小
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //
    elementData[size++] = e;
    return true;
}
```
分析：这里现在ensureCapacityInternal()传入的参数为2


```java
private void ensureCapacityInternal(int minCapacity) {
    //如果elementData数组等于默认空元素数组则进入下面的逻辑（ps:我们前面假设是不传参初始化的ArrayList，所以会进入下面括号内的逻辑）
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        //DEFAULT_CAPACITY=10，比较两者数的大小，较大值
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    //将参数等于10继续传入该函数
    ensureExplicitCapacity(minCapacity);
}
```
分析:第二次添加，那么minCapacity就等于2。也就是minCapacity可以理解为当前元素个数。

现在数组已经不是默认的空数组，if条件不满足，所以直接走下面ensureExplicitCapacity（）方法

```java
private void ensureExplicitCapacity(int minCapacity) {
    //记录修改次数加1
    modCount++;
    //当前传入minCapacity长度大于当前数组长度
    if (minCapacity - elementData.length > 0)
        //将参数传入该函数
        grow(minCapacity);
}
```
分析：到这里就是记录修改次数加1了，2小于当前数组长度10，不满足，不往下执行。

所以，我们知道，当数组元素没有满时，或者更准确的说没有达到第二次条件时，它是不扩容的。

然后，我们继续考虑，那么什么时候进行第二次扩容，第二次扩容的大小又是多少？

```java
private void ensureExplicitCapacity(int minCapacity) {
    //记录修改次数加1
    modCount++;
    //当前传入minCapacity长度大于当前数组长度
    if (minCapacity - elementData.length > 0)
        //将参数传入该函数
        grow(minCapacity);
}
```

分析：我们从这里很容易就看出，第二次准备扩容时，elementData.length当前的数组长度是10。当前的元素个数minCapacity为11个时，放不下，

也就是数组个数已经满了，开始扩容。传入参数11，进入grow（）方法。

```java
/**
 * 真正执行数组扩容的方法，先判断扩容的长度，最后执行Arrays.copyOf扩容
 */
private void grow(int minCapacity) {
    //当前旧的数组长度
    int oldCapacity = elementData.length;
    //新数组长度等于 = 旧数组长度 + 旧数组长度除以2 (也就是1.5倍)
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //小于传进来的参数则等于传进来的参数
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    //大于int的最大长度(2^31 - 1)再减去8长度，传入minCapacity参数,执行hugeCapacity（）方法
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    //执行Arrays.copyOf将旧数组扩容成新数组长度为newCapacity
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```
分析：此时，newCapactiy值等于旧数组的1.5倍，然后进行复制扩容。也就是说，第二次包括以后数组满了再扩容，

在满足没有超过MAX_ARRAY_SIZE的前提下每次数组扩容都是扩容为原来数组长度的1.5倍长。


至于如果当前扩容的长度大于了MAX_ARRAY_SIZE，执行hugeCapacity（）方法，不按照1.5倍扩容机制，而是重新计算扩容长度。很简单，我们进去看下就懂了。

```java
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```
MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8

分析：这里的minCapacity代表的是需要扩容的长度，如果需要扩容长度大于了int最大值减去8

那么，扩容的长度直接等于int最大值，也就是2^31-1。不大于，则直接等于2^31-1-8。

### 2.假设是以List list = new ArrayList(5) 传参的方式来初始化的ArrayList，初始化长度为5

还是按照上面代码去理思路，很容易就可以分析，在添加的元素不大于5时，不会去扩容。

```java
//当前传入minCapacity长度大于当前数组长度
if (minCapacity - elementData.length > 0)
    //将参数传入该函数
    grow(minCapacity);
```

同理，接下来每次添加元素，如果当前数组元素已经填满，那么就需要去扩容。

扩容扔然是按1.5倍去扩容，如果当前数组乘以1.5倍后长度已经大于MAX_ARRY_SIZE，那么扩容仍然会走hugeCapacity（）策略，同分析1。

### 3.简单总结

```sh
我们已经知道了ArrayList的扩容机制，简单来说就是在满足扩容新数组长度不大于MAX_ARRY_SIZE长度下：

不传参初始化ArrayList（）第一次扩容长度为10，接下来每次都是按1.5倍扩容。

传参初始化，每次扩容都是按1.5倍来扩的。
```

## remove（）方法

```java
/**
 * 通过移动数组后面所有的元素覆盖当前索引元素，从而达到删除当前元素的效果
 * 数组最后一个索引值置为null，说明ArrayList可以查找null值
 */
public E remove(int index) {
    //检查删除的索引是否超出数组的索引
    rangeCheck(index);

    //修改记录
    modCount++;
    //根据索引获取数组值
    E oldValue = elementData(index);
    //需要移动的元素个数
    int numMoved = size - index - 1;
    if (numMoved > 0)
        //删除元素索后面的所有元素都要往前移动一个索引
        //也表明了数组的删除是通过后面的元素往前移动进行覆盖而达到删除的目的
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    //数组最后一个索引值置为null，并且索引减1
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
```

## set(int index, E element)

```java
/**
 * 因为底层是数组，所以set时，可以直接根据索引进行覆盖旧的值
 */
public E set(int index, E element) {
    //检查索引是否越界
    rangeCheck(index);
    
    E oldValue = elementData(index);
    //直接根据索引覆盖值
    elementData[index] = element;
    return oldValue;
}
```

## index0f()

```java
/**
 * 从首部开始查找，返回第一个值相同的索引，遍历完没找到返回-1
 */
public int indexOf(Object o) {
    //传进来的值为null，从首部开始遍历数组，找到第一个为null的值，返回当前索引
    if (o == null) {
        for (int i = 0; i < size; i++)
            if (elementData[i]==null)
                return i;
    } else {
        //不为空，从首部遍历，返回第一个值相同的下标索引
        for (int i = 0; i < size; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    //没找到返回-1
    return -1;
}
```

## ArrayList的迭代器

### 1.类的继承关系

```java
/**
* Returns an iterator over the elements in this list in proper sequence.
*
* <p>The returned iterator is <a href="#fail-fast"><i>fail-fast</i></a>.
*
* @return an iterator over the elements in this list in proper sequence
*/
public Iterator<E> iterator() {
     return new Itr();
}
```
ArrayList调用iterator，会new一个Itr对象，这是我们要特别注意的。

···sh
在游戏服务端开发中，对性能要求特别高，所以我们迭代的时候一般迭代器跌停，
能用for循环尽量用for循环，可以减少new对象，减少gc频率。
···

```java
/**
* An optimized version of AbstractList.Itr
*/
rivate class Itr implements Iterator<E> {
}
```
itr实现Iterator接口,并且该类是一个嵌套类，直接嵌套在ArrayList类下，

嵌套类好处：

1.嵌套类可以访问外部类的所有数据成员和方法，即使它是私有的。

2.提高可读性和可维护性：因为如果一个类只对另外一个类可用，那么将它们放在一起，这更便于理解和维护。

3.提高封装性：给定两个类A和B，如果需要访问A类中的私有成员，则可以将B类封装在A类中，这样不仅可以使得B类可以访问A类中的私有成员，并且可以在外部隐藏B类本身。

4.减少代码的编写量。


### 2.类的成员属性

```java
/**
* An optimized version of AbstractList.Itr
*/
private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;
}
```
cursor 返回下一个元素的索引

lastRet

expected

expectedModCount 


### 3.重要的方法

#### 1. hasNext()

```java
/**
* Returns {@code true} if the iteration has more elements.
* (In other words, returns {@code true} if {@link #next} would
* return an element rather than throwing an exception.)
*
* @return {@code true} if the iteration has more elements
*/
boolean hasNext();
```

如果还有下一个值，那么hasNext返回true。


```java
 public boolean hasNext() {
    return cursor != size;
}
```




## 本文适当参考下面两篇文章分析:

https://www.cnblogs.com/leesf456/p/5308358.html

http://cmsblogs.com/?p=108