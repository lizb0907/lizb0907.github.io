---
layout: post
title: 鸟哥linux实战
categories: linux
description: 系统学习linux
keywords: 系统，基础，命令
---

鸟哥linux实战学习记录

**目录**

* TOC
{:toc}

## 基础指令操作

### 1.显示日期指令

```sh
date
```

```sh
date +%Y%m%d 

显示日期格式为【年/月/日】
```

### 2.显示日历指令

```sh
cal 
```

```sh
cal 8 2019

显示2019年8月的月历
```

### 3.计算器

```sh
bc
```

```sh
quit

退出计算器
```

### 4.几个重要的热键[Tab], [ctrl]-c, [ctrl]-d

#### 1.[Tab]按键 『命令补全』与『文件补齐』

```sh
 ca[tab][tab]

 所有以 ca 为开头的指令都被显示
```

```sh
 ls -al ~/.bash[tab][tab]

 .bash 为开头的文件名都会被显示出来
```

#### 2.[Ctrl]-c 按键

```sh
[Ctrl]-c

将正在运作中的指令中断
```
#### 3.[Ctrl]-d 按键

```sh
[Ctrl]-d

键盘输入结束,也可以用来取代 exit 的输入呢
```

### 5.[shift]+{[PageUP]|[Page Down]}按键

```sh
[shift]+{[PageUP]|[Page Down]}

翻页
```