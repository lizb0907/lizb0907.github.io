---
layout: post
title: 游戏里trove4j TIntIntHashMap 迭代优化
categories: Mmo-Game
description: trove4j TIntIntHashMap 迭代优化
keywords: move，mmo，game
---

在我们的游戏里，使用trove4j以减少内存,同时保持性能。

**目录**

* TOC
{:toc}

## 一：传统采用迭代器遍历map

```java
TIntIntHashMap map = new TIntIntHashMap();
TIntIntIterator iterator = map.iterator();
while (iterator.hasNext())
{
    iterator.advance();

    int key = iterator.key();
    int value = iterator.value();
}
```

点进iterator源码，我们可以看到每次都会new一个TIntIntIterator对象:

```java
/** {@inheritDoc} */
public TIntIntIterator iterator() {
    return new TIntIntHashIterator( this );
}
```

采用迭代器方式遍历map取值，是比较常用的一种方式。但是，在游戏开发中有时候这种方式不满足我们的需求。

当我们有些场景不得不频繁的去迭代map时，如果采用的是迭代器方式，那么每次都会产生一个迭代器对象，频繁的创建对象对内存和GC都是一种较大的消耗。


## 二：开始优化之前，先简单认识下TIntIntHashMap的put存储过程

### 1：put传入key和value值

```java
/** {@inheritDoc} */
public int put( int key, int value ) {
    int index = insertKey( key );
    return doPut( key, value, index );
}
```

### 2：调用insertKey（）方法，传入key

```java
/**
* Locates the index at which <tt>val</tt> can be inserted.  if
* there is already a value equal()ing <tt>val</tt> in the set,
* returns that value as a negative integer.
*
* @param key an <code>int</code> value
* @return an <code>int</code> value
*/
protected int insertKey( int val ) {
    int hash, index;

    hash = HashFunctions.hash(val) & 0x7fffffff;
    index = hash % _states.length;
    byte state = _states[index];

    consumeFreeSlot = false;

    if (state == FREE) {
        consumeFreeSlot = true;
        insertKeyAt(index, val);

        return index;       // empty, all done
    }

    if (state == FULL && _set[index] == val) {
        return -index - 1;   // already stored
    }

    // already FULL or REMOVED, must probe
    return insertKeyRehash(val, index, hash, state);
}
```

我们抓重点看，认识主要部分:

#### 1：根据key值计算哈希索引，并根据索引从状态字节数组里取出状态
```java
hash = HashFunctions.hash(val) & 0x7fffffff;
index = hash % _states.length;
byte state = _states[index];
```
分析：

1.调用hash函数计算对应的哈希映射值index

2._states数组
```java
/**
* flags indicating whether each position in the hash is
* FREE, FULL, or REMOVED
*/
public transient byte[] _states;
```
这个字节数组记录的是哈希值对应的状态，该状态可能为空闲的、已满的或是已删除状态。

从byte[]数组中取出状态。

#### 2：如果取出的状态值是0即FREE状态，记录相应的值和状态，返回哈希映射索引index

```java
if (state == FREE) {
    consumeFreeSlot = true;
    insertKeyAt(index, val);

    return index;       // empty, all done
}
```

```java
void insertKeyAt(int index, int val) {
    _set[index] = val;  // insert value
    _states[index] = FULL;
}
```

根据哈希映射的索引，将key值记录到_set数组里,同时字节数组记录哈希值对应的状态为FULL，也就是有值状态。

```java
/** the set of ints */
public transient int[] _set;
```

```java
/**
* flags indicating whether each position in the hash is
* FREE, FULL, or REMOVED
*/
public transient byte[] _states;
```

两个哈希数组，_set记录的key值，_states记录的是状态。

#### 3：如果当前key值已经有记录了，那么返回负的哈希映射索引index并减1

```java
if (state == FULL && _set[index] == val) {
    return -index - 1;   // already stored
}
```

#### 4：insetKyeReHash（）发现最终也会调用insertKeyAt()

```java
insertKeyRehash(val, index, hash, state);
```

总结：到此我们认识了insertKey（）方法，最主要需要了解两个哈希数组，_set记录的key值，_states记录的是状态。

### 3：doPut( key, value, index )方法

```java
private int doPut( int key, int value, int index ) {
    int previous = no_entry_value;
    boolean isNewMapping = true;
    if ( index < 0 ) {
        index = -index -1;
        previous = _values[index];
        isNewMapping = false;
    }
    _values[index] = value;

    if (isNewMapping) {
        postInsertHook( consumeFreeSlot );
    }

    return previous;
}
```
这里我们只要明白，map的value值是存放在_values数组
```java
/** the values of the map */
protected transient int[] _values;
```


## 三：迭代优化

```java
TIntIntHashMap stateMap = new TIntIntHashMap();
stateMap.put(1, 1);
stateMap.put(2, 2);
stateMap.put(3, 3);

//这里们要知道哈希索引组里_state如果为满,那么一定可以根据索引值从_set组里取到key值（也就是map的键值）
//这里存的是根据key值计算的哈希索引index状态记录，空、满、删除三种状态
byte[] _state = stateMap._states;
//这里存的是key值
int[] _set = stateMap._set;

//当前map长度,也就是我们记录的值个数
int size = stateMap.size();

//从0开始到哈希索引数组长度遍历
for (int i = 0, iSize = _state.length; i < iSize; i++)
{

    if (size <= 0)
    {
        break;
    }

    //有值才往下走
    if (_state[i] != TIntObjectHashMap.FULL)
    {
        continue;
    }
    
    size--;
    //根据哈希索引index从_set里取出key值（也就是map的键值）
    int mapKey = _set[i];

    //当然我们也可以根据mapKey快速取得value值
    int value stateMap.get(mapKey);

}
```

总结：这里们要知道哈希索引组里_state如果为满,那么一定可以根据索引值从_set组里取到key值（也就是map的键值）。

所以，如果实际场景遍历非常频繁，我们可以不用迭代器迭代，掏出来map的内部数组进行遍历。

同理，TIntObjectMap...等等其他map结构也可以采用这种方式节省内存和减少GC。


