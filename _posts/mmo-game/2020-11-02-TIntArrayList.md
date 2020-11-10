---
layout: post
title: TIntArrayList的resetQuick()方法使用
categories: Java-Scource-Code-Analysis
description: TIntArrayList的resetQuick()
keywords: trove4j,TIntArrayList,resetQuick()
---

TIntArrayList的resetQuick()方法使用注意事项！

**目录**

* TOC
{:toc}

## TIntArrayList的 clear() 方法对比和 resetQuick() 方法对比

```java
public void clear() {
        this.clear(10);
    }

public void clear(int capacity) {
    this._data = new int[capacity];
    this._pos = 0;
}
```
clear方法每次回new一个长度为10的新数组，同时将下标重置。


```java
 public void resetQuick() {
        this._pos = 0;
    } 
```

resetQuick()是一个快速清除方法，没有new新的数组。

有些场景可以使用resetQuick()方法而不是clear()方法，减少new对象。

可以看到resetQuick()只是把记录下标索引的值pos置为了0。

数组内的值并没有清除，还保留在内存当中。


## 使用resetQuick()注意事项

### TIntArrayList的 add（）方法

```java
public boolean add(int val) {
        this.ensureCapacity(this._pos + 1);
        //对应索引赋值
        this._data[this._pos++] = val;
        return true;
    }
```
调用ensureCapacity确认数组容量

```java
public void ensureCapacity(int capacity) {
        //如果当前索引大于存值数组的长度，扩容
        if (capacity > this._data.length) {
            //数组扩容一倍和当前索引比较，取大值
            int newCap = Math.max(this._data.length << 1, capacity);
            int[] tmp = new int[newCap];
            //将旧数组的值拷贝到临时数组tmp
            System.arraycopy(this._data, 0, tmp, 0, this._data.length);
            //改变引用
            this._data = tmp;
        }
    }
```
如果当前索引大于存值数组的长度进行数组扩容

```sh
this._data[this._pos++] = val;

添加是根据下标索引覆盖原先值， resetQuick()把pos下标清0，所以不会有问题。
```

### TIntArrayList的 get（）方法

```java
public int get(int offset) {
        if (offset >= this._pos) {
            throw new ArrayIndexOutOfBoundsException(offset);
        } else {
            return this._data[offset];
        }
    }
```
get值时，会先判断是否大于下标索引值，大于等于抛异常，不会有问题。


### TIntArrayList的迭代

```java
int size = list.size();
for (int i = 0; i < size; i++)
{
    ...
}
```

```java
 public int size() {
        return this._pos;
    }
```

size方法判断的是下标长度，所以不会有问题。

### 总结

```sh
其它几个操作就不一一看了，基本上都没有问题。

使用TIntArrayList的 resetQuick() 方法，最主要注意是数组内的值并没有清除，还保留在内存当中。
```

