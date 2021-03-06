---
layout: post
title: 代码优化小技巧
categories: Review
description: 本系列主要记录关于代码的优化小技巧（持续更新）
keywords: review, code
---

总结工作、Effective java、秦小波java的151个建议

**目录**

* TOC
{:toc}

## Effective Java

### 1.for-each循环优先于传统的for循环

注意，利用for-each循环不会有性能损失，甚至于数组也一样。
在某些情况下，它还稍有性能优势，因为它对数组的索引的边界值只会计算一次。

例子: 扑克牌花色  两个集合进行嵌套迭代
```java
enum Suit {CLUB , DIAMODN, HEART, SPADE}
enum Rank {ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT, NINE, TEN, JACK, QUEEN, KING}
...
Collection<Suit> suits = Arrays.asList(Suit.values());
Collection<Rank> ranks = Arrays.asList(Rank.values());
List<Card> deck = new ArrayList<Card>();
for(Iterator<Suit> i = suits.iterator(); i.hasNext();)
       for（Iterator<Rank> j = ranks.iterator(); j.hasNext();）
                 deck.add(new Card(i.next(), j.next()));
```
上面这个例子有缺陷，在迭代器上对外部的集合（suits）调用了太多次next方法了。


正确的做法：  外部循环域中添加一个变量来保存元素
```java
for(Iterator<Suit> i = suits.iterator(); i.hasNext();)
       Suit suit = i.next();
       for（Iterator<Rank> j = ranks.iterator(); j.hasNext();）
                 deck.add(new Card(suit, j.next()));
```


用for-each改进：  这个问题会完全消失，并且使代码更加简洁
```java
for(Suit suit : suits）
  for(Rank rank : ranks)
       deck.add(new Card(suit, rank));
```
for-each循环不仅让你遍历集合和数组，还让你遍历任何实现Iterable接口的对象。


在以下情况中不要使用for-each循环
1.过滤
需要遍历集合，并删除选的的元素，就需要使用显式的迭代器，方便可调用remove方法
2.转换
需要遍历列表或者数组，并取代它部分或者全部元素，就需要列表迭代器或者数组索引，以便设定元素的值。
3.平行迭代
需要并行的地遍历多个集合，就需要显式地控制迭代器或者索引变量，以便可以同步前移



### 2.使用类库Random.nextInt(int)

假设你希望产生位于0和某个上界之间的随机整数。
很多人会这样做
```java
private static final Random rnd = new Random();
static int random(int n)
{
    return Math.abs(rnd.nextInt()) % n;
}
```
有缺点：
           1.n是比较小的2的乘方，。经过一段相当短的周期后，它产生的随机数列将会重复
           2.n不是2的乘方，平均起来，有些数会比其他的数出现得更为频繁
           3.n比较大，有random方法产生的数字会有2/3落在随机数取值范围的前半部分
		   

所以直接用类库
Random.nextInt(int)		   


### 3.需要精确的答案，不要用float和double

float 和 double 不能精确的表示 0.1，所以不适合用于货币计算
```java
public static void main(String[] args)
{
    System.out.println(1.00 - 9 * .10);
}
```
输出的结果却是：
0.09999999999999998

你可能会认为，只要在打印之前将结果做一下舍入就可以，但这种方法在有些场景并适合。

解决这个问题的正确办法是使用BigDecimal、int或者long进行货币计算。
 
使用BigDecimal与使用基本运算类型相比，这样做很不方便，而且很慢。



### 4.基本类型优于装箱类型

基本类型优先于装箱基本类型
装箱基本类型中对应int、double和boolean的是Integer、Double、Boolean
还有long对应Long

基本类型与装箱基本类型的区别：
1.基本类型只有值，而两个装箱基本类型可以具有相同的值和不同的同一性。
2.基本类型只有功能完备的值，而每个装箱基本类型除了它对应基本类型的所有功能值之外，还有个非功能值：null
3.基本类型通常比装箱基本类型更节省时间和空间
（个人还有理解：装箱基本类型可以调用方法，例如Integer.parsInt() ）


一：对于装箱基本类型运用==操作符几乎总是错误的
例：
```java
 Comparator<Integer> naturalOrder = new Comparator<Integer>()
    {
           public int compare<Interger first, Inter second>
           {
               return first  < second ? -1 : (first == second ? 0 : 1);
           }
    };
```
这个比较器只要打印naturalOrder .Compare（new Integer(42), new Integer(42)）就会出现错误  输出值为1
为什么？
对表达式first  < second执行计算会导致被first 和 second引用的Integer实例会被自动拆箱。也就是说，它提取了他们的基本类型值
下一步就是执行first == second，它在两个对象引用上执行同一性比较。如果first和second引用表示同一个int值的不用Integer实例，
这个比较操作会返回false，比较器错误的返回1,表示一个Integer值大于第二个。

改进：
```java
 Comparator<Integer> naturalOrder = new Comparator<Integer>()
    {
           public int compare<Interger first, Inter second>
           {
               int f = first;          //Integer赋值给int，导致自动拆箱 
               int s = second;  //Integer赋值给int，导致自动拆箱 
               return first  < second ? -1 : (first == second ? 0 : 1);
           }
    };
```

 二：如果null对象引用被自动拆箱，就会得到一个NullPointerException异常
 例：
```java
public class Unbelievable
{ 
   static Integer i;
   public static void main(String[] args)
   {
		if(i == 42)
		  System.out.println("Unbelievable");
   }
}
```
虽然不会输出Unbelievable---但是它的行为也是很奇怪的。
它在计算 i == 42 时会抛出NullPointerException异常。
当在一项操作中混合使用基本类型和装箱类型时，装箱基本类型就会自动拆箱。
要修正这个例子，声明i是int就行，而不是Integer。


三：注意声明变量时不要不小心声明为Long，会导致性能严重下降
```java
public static void main(String[] args)
{ 
   Long sum = 0L;
   for(long i = 0; i < Interger.MAX_VALUE; i++)
   {
	   sum += i;
   }
   System.out.println(sum);
}
```
这个程序运行起来比预计要慢，因为它不小心将一个局部变量sum声明为是装箱基本类型，而不是
基本类型long。变量反复地装箱和拆箱，导致性能明显的下降。


什么时候用装箱类型呢？
1.作为集合中的元素、键和值。你不能将基本类型放在集合中，因此必须使用装箱基本类型。
2.参数化类型中，必须使用
3.进行反射方法调用


### 5.当心字符串连接性能

字符串连接符，为连接n个字符串而重复地使用字符串连接操作符，需要n的平方级的时间。这是
因为字符串不可变，当两个字符串被连接在一起时，他们的内容都要被拷贝。
例：
```java
public String statement()
{
   String result = " ";
   for(int i = 0; i < numItems(); i++)
   {
		 //lineForItem(i)迭代的值为字符串
		 result += lineForItem(i);
   }
  return result;
}
```
这种做法开销随项目数量而呈平方级增加

优化：   
```java
public String statement()
   {
       StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
       for(int i = 0; i < numItems(); i++)
       {
            b.append(lineForItem(i));
       }
      return b.toString;
   }
```
这种方法预先分配了一个StringBuilder,大到足以容纳结果字符串。一定数据量测试下，比上一种方法，几十倍。
即使因为预先不知道字符串长度，使用默认大小的StringBuilder，它仍然比第一种做法快50倍。


## 秦小波java151个建议

### 1.不要陷入自增++的陷阱

例：
```java
int count = 0;
for (int i = 0; i < 10; i++)
{
    count = count++;
}
System.out.println(count);
```
该程序输出的值为0
count值是0，保存到临时变量，count++返回的是临时变量的值，所以count返回的还是0.
你每次循环时，初始值都为0.所以返回的临时变量都是0.
最后输出的就是0.

```java
int count = 0;
for (int i = 0; i < 10; i++)
{
     count++;
}
System.out.println(count);
```
该程序输出值才是为10
count++虽然返回0，但是它没有覆盖count的初始值，所以下一次，count的值就是从加1开始了。
也可以理解为，右++要下一次再拿的时候才加1

如何理解?
count++是一个表达式，是有返回值的，它的返回值就是count自加前的值。
java对自加是这样处理的:
首先把count的值（注意是值，不是引用）拷贝到一个临时变量区，然后对count变量加1，最后返回临时变量的值。
程序第一次循环时的详细处理步骤如下:
1.JVM把count值（其值是0）拷贝到临时变量区。
2.count值加1，这时候count的值是1。
3.返回临时变量区的值，注意这个值是0，没修改过。
4.返回值赋值给count,此时count值被重置为0。

"count=count++"这条语句可以用如下代码来理解:
```java
public static int mockAdd(int count)
{
     //先保存初始值
     int temp = count;
     //做自增操作
     count = count +  1;
     //返回原始值
     return temp；
}
```

### 2.三元操作符类型务必一致

三元操作符是if-else的简化写法，常用
注意：三元操作符的类型务必一致:
```java
public class Client
{
     public static void main(String[] args)
     {
             int i = 80;
             String s = String.valueOf(i < 100 ? 90 : 100);
             String t = String.valueOf(i < 100 ? 90 : 100.0);
             System.out.println("两者是否相等:" +s.equals(t));
      }
}
```
结果是false
因为 t 的值是 90.0 所以和 s不相等
三元操作符，一个是int型，一个是浮点数，操作类型不一致，编译器会进行类型转换。



## 工作中代码优化积累

### 1.一定不要在for循环迭代中修改数据！

### 2.起服时缓存好的数据结构，获取数据时，一定要注意小心直接操作引用！

### 3.侠客技能拷贝给当前玩家时，一定要注意重新复制一份，不能直接操作引用！
