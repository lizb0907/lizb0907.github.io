---
layout: post
title: 初探字节码
categories: Jvm
description: 初探字节码
keywords: JVM,ByteCode
---

字节码介绍，开启字节码大门

**目录**

* TOC
{:toc}

## 字节码简介

```sh
1.不同平台的Java虚拟机，统一支持的程序存储格式——字节码（Byte Code），是构成平台无关性的基石。
2.Java虚拟机不与包括Java语言在内的任何程序语言绑定，它只与“Class文件”这种特定的二进制文件格式所关联，Class文
  件中包含了Java虚拟机指令集、符号表以及若干其他辅助信息。
  用Java编译器可以把Java代码编译为存储字节码的Class文件，使用JRuby等其他语言的编译器一样可以把它们的源程序代
  码编译成Class文件。虚拟机丝毫不关心Class的来源是什么语言。

3.官方文档：https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html#jvms-4.4.4
```

## 一些概念

```sh
1.一个字节(Byte) 等于 8 位(bit).
2.十六进制转十进制：
    16进制就是逢16进1，但我们只有0~9这十个数字，所以我们用A，B，C，D，E，F这六个字母来分别表示10，11，12，13，14，15。字母不区分大小写。
    十六进制数的第0位的权值为16的0次方，第1位的权值为16的1次方，第2位的权值为16的2次方……
    所以，在第N（N从0开始）位上，如果是是数 X （X 大于等于0，并且X小于等于 15，即：F）表示的大小为 X * 16的N次方。
    假设有一个十六进数 2AF5, 那么如何换算成10进制？
    用竖式计算：
    2AF5换算成10进制:
    第0位：5 * (16 ^ 0) = 5
    第1位：F * (16 ^ 1) = 15 * (16 ^ 1) = 240
    第2位：A * (16 ^ 2) = 10 * (16 ^ 2) = 2560
    第3位：2 * (16 ^ 3) = 2 * (16 ^ 3) = 8192
    直接计算就是：5 + 240 + 2560 + 8192
    可以看出，所有进制换算成10进制，关键在于各自的权值不同
```

## 可视化class文件

```sh
1.编译java文件为class文件：
  进入对应java文件目录，执行命令javac即可产生class格式文件。

2.用文本编辑器打开class文件：
  一般会乱码，class文件是十六进制格式，需要装插件。
  
3.这里我以notepade++举例：
  点击插件-->Plugin Manager-->Show Plugin Manager：
          在Available窗口找到HEX-Editor，安装。
           
4.安装完毕HEX-Editor重启后，工具栏末尾有一个“H”工具，点击即可显示格式友好的class文件。
```

## class文件结构

```sh
1.Class文件是一组以8个字节为基础单位的二进制流，没有任何分隔符。
2.无符号数：u1、u2、u4、u8来分别代表1个字节、2个字节、4个字节和8个
           字节的无符号数。
3.表：表是由多个无符号数或者其他表作为数据项构成的复合数据类型，
      以“_info”结尾。

4.无论是无符号数还是表，当需要描述同一类型但数量不定的多个数据时，经常会使用一个前置的
  容量计数器加若干个连续的数据项的形式，这时候称这一系列连续的某一类型的数据为某一类型的“集
  合”。
```

```sh
ClassFile {
    u4             magic; //魔数
    u2             minor_version; //次版本号
    u2             major_version; //主版本号
    u2             constant_pool_count; //常量池数量+1
    cp_info        constant_pool[constant_pool_count-1]; //常量池
    u2             access_flags; // 访问标识
    u2             this_class; // 常量池的有效下标
    u2             super_class; // 常量池的有效下标
    u2             interfaces_count; // 接口数
    u2             interfaces[interfaces_count];// 下标从0开始，元素为常量池的有效下标
    u2             fields_count;
    field_info     fields[fields_count];
    u2             methods_count;
    method_info    methods[methods_count];
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

![](/images/posts/jvm/bytecode/2.png)

### 1.魔数与文件版本号

![](/images/posts/jvm/bytecode/1.png)

```sh
1.每个Class文件的头4个字节被称为魔数（Magic Number），它的唯一作用是确定这个文件是否为
  一个能被虚拟机接受的Class文件（ca fe ba be）。
  使用魔数而不是扩展名来进行识别主要是基于安全考虑，因为文件扩展名可以随意改动
2.紧接着魔数的4个字节存储的是Class文件的版本号：第5和第6个字节是次版本号（Minor
  Version），第7和第8个字节是主版本号（Major Version）。
```

### 2.常量池

紧接着主、次版本号之后的是常量池入口。

![](/images/posts/jvm/bytecode/3.png)

```sh
1.由于常量池中常量的数量是不固定的，所以在常量池的入口需要放置一项u2类型的数据（两个字节也就是上图中的第8和第9，0X0016），代表常
  量池容量计数值（constant_pool_count）。
  常量池容量为十六进制数0x0016，即十进制的22，这就代表常量池中有21项常量，索引值范围为1～21

2.常量池容量计数是从1而不是0开始，如果后面某些指向常量池的索引值的数据在特定情况下
  需要表达“不引用任何一个常量池项目”的含义，可以把索引值设置为0来表示。
3.Class文件结构中只有常量池的容量计数是从1开始，其它都是从0开始。
```

```sh
常量池中主要存放两大类常量：字面量（Literal）和符号引用（Symbolic References）
   
字面量：
     近于Java语言层面的常量概念，文本字符串、被声明为final的常量值。

符合引用：
    ·被模块导出或者开放的包（Package）
    ·类和接口的全限定名（Fully Qualified Name）
    ·字段的名称和描述符（Descriptor）
    ·方法的名称和描述符
    ·方法句柄和方法类型（Method Handle、Method Type、Invoke Dynamic）
    ·动态调用点和动态常量（Dynamically-Computed Call Site、Dynamically-Computed Constant）
Class文件中不会保存各个方法、字段最终在内存中的布局信息，这些字段、方法的符号引用不经过虚拟机在运行期转换的话是无法得到真正的
内存入口地址。
```

### 1.分析常量池

![](/images/posts/jvm/bytecode/4.png)
===============================图1===========================

![](/images/posts/jvm/bytecode/5.png)
===============================图2===========================

![](/images/posts/jvm/bytecode/6.png)
===========图3 CONSTANT_Class_info型常量的结构================

![](/images/posts/jvm/bytecode/7.png)
===========图4 CONSTANT_Utf8_info型常量的结构=================

```sh
1.我们已经知道，常量池的入口第8和第9的0X0016代表常量池容量计数值，常量池中有21项常量，索引值范围为1～21。
那么我们继续往下分析。

2.截至JDK13，常量表中分别有17种不同类型的常量。这17类表都有一个共同的特点，表结构起始的第一位是个u1类型的标志位，
也就是占用一个字节：
    所以图1，A列对应的0x07代表常量池类型标志，也就是7，对应图2中的CONSTANT_Class_info类型。

3.CONSTANT_Class_info型常量的结构（图3）：
    1.u1一个字节的tag,标志位，它用于区分常量类型。这里其实就是A列对应的0x07代表常量池类型标志。
    2.u2两个字节的name_index，常量池的索引值，它指向常量池中一个CONSTANT_Utf8_info类型常量，
      此常量代表了这个类（或者接口）的全限定名。

      我们看到B和C两个字节列，指向了常量池中的第二项常量。从图1中查找第二项常量，也就是D列，它的标志位是0x01，
      对应图2，确实是一个CONSTANT_Utf8_info类型的常量。

4.CONSTANT_Utf8_info型常量的结构（图4）:
   1.length值说明了这个UTF-8编码的字符串长度是多少字节，它后面紧跟着的长度为length字节的连
     续数据是一个使用UTF-8缩略编码表示的字符串。
     length值为E和F列对应的0x001D，也就是长29个字节，往后29个字节正好都在1～127的ASCII码范围以内，
     内容为“org/fenixsoft/clazz/TestClass”，如上图1。

总结：到此为止，我们仅仅分析了TestClass.class常量池中21个常量中的两个。
```

```sh
由于Class文件中方法、字段等都需要引用CONSTANT_Utf8_info型常量来描述名
称，所以CONSTANT_Utf8_info型常量的最大长度也就是Java中方法、字段名的最大长度。而这里的
最大长度就是length的最大值，既u2类型能表达的最大值65535。所以Java程序中如果定义了超过64KB
英文字符的变量或方法名，即使规则和全部字符都是合法的，也会无法编译。
```

![](/images/posts/jvm/bytecode/9.jpg)

使用javap -verbose TestClass命令输出常量表

```sh
其中有些常量似乎从来没有在代码中出现过，如“I”“V”“<init>”“LineNumberTable”“LocalVariableTable”等。

它们都是编译器自动生成的，会被后面即将讲到的字段表
（field_info）、方法表（method_info）、属性表（attribute_info）所引用，它们将会被用来描述一些不
方便使用“固定字节”进行表达的内容，譬如描述方法的返回值是什么，有几个参数，每个参数的类型
是什么。
```

### 2.访问标志

```sh
在常量池结束之后，紧接着的2个字节代表访问标志（access_flags），这个标志用于识别一些类或
者接口层次的访问信息，包括：这个Class是类还是接口；是否定义为public类型；是否定义为abstract
类型；如果是类的话，是否被声明为final；

注意纯看字节码和和javap -verbose命令输出的顺序有一些不一样，
字节码常量池后面紧跟着的是访问标志，而javap -verbose命令先看到访问标志再常量池。
我们这里指的是字节码，所以在常量池结束之后，紧接着的2个字节代表访问标志（access_flags）。
```

![](/images/posts/jvm/bytecode/10.jpg)

```sh
类型同时存在时进行 | 操作，如public final的值就是0x0011.
```

### 3.类索引、父类索引与接口索引集合

```sh
1.类索引（this_class）和父类索引（super_class）都是一个u2类型的数据，而接口索引集合
（interfaces）是一组u2类型的数据的集合，Class文件中由这三项数据来确定该类型的继承关系。
2.类索引用于确定这个类的全限定名，父类索引用于确定这个类的父类的全限定名。
3.类索引、父类索引和接口索引集合都按顺序排列在访问标志之后，类索引和父类索引用两个u2类
  型的索引值表示，它们各自指向一个类型为CONSTANT_Class_info的类描述符常量，通过
  CONSTANT_Class_info类型的常量中的索引值可以找到定义在CONSTANT_Utf8_info类型的常量中的
  全限定名字符串。
4.对于接口索引集合，入口的第一项u2类型的数据为接口计数器（interfaces_count），表示索引表
  的容量。如果该类没有实现任何接口，则该计数器值为0，后面接口的索引表不再占用任何字节。
```

