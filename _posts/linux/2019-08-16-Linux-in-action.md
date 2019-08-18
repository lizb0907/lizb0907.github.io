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

## 一:基础指令操作

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

### 6.两个帮助指令

#### 1.--help
```sh
date --help

查看date指令的基本用法和选项参数
```

#### 2.man page
```sh
man date

查询用法和参数相关
```

### 7.搜寻关键字

#### 1.向下搜寻字符串 /date
```sh
/data

文本或man page页面里只要按下/，出现光标开始输入搜寻字符串"date"
```

#### 2.向上搜寻字符串 ？date
```sh
？date

文本或man page页面里只要按下？，出现光标开始输入搜寻字符串"date"
```

#### 3.向上或向上继续搜索
```sh
n, N

利用 / 或 ? 来搜寻字符串时，可以用 n 来继续下一个搜寻 (不论是 / 或 ?) ，可以利用 N 来进
行『反向』搜寻。举例来说，我以 /vbird 搜寻 vbird 字符串， 那么可以 n 继续往下查询，用 N 往
上查询。若以 ?vbird 向上查询 vbird 字符串， 那我可以用 n 继续『向上』查询，用 N 反向查
询。
```

### 8.找出系统中只要有man这个关键词就将该说明列出来
```sh
man -k man
```
### 9.查询系统中有哪些跟man这个指令有个说明文件
```sh
man -f man
```
### 10.info page
```sh
info 与 man 的用途其实差不多，都是用来查询指令的用法或者是文件的格式
```

### 11.服务说明文档路径
```sh
想要利用一整组软件来达成某项功能时，请赶快到/usr/share/doc 底
下查一查有没有该服务的说明
```

### 12.我想知道目前系统有多少指令是以 bz 为开头的，可以怎么作？
```sh
直接输入 bz[tab][tab]
```

### 13.观察系统的使用状态
```sh
who    目前谁在线

netstat -a  网络的联机状态
```

### 14.切换用户
```sh
直接键入su，然后输入密码切换为root
```

```sh
su bp，切换到bp用户
```

### 15.关机或重启操作

#### 1.关机之前将内存数据写入到硬盘
```sh
sync

关机之前将内存数据写入到硬盘,有些需要切换到root才能执行该命令
```

#### 2.查看shutdown用法
```sh
man shutdown
```

#### 3.例子
```sh
[root@study ~]# /sbin/shutdown [-krhc] [时间] [警告讯息]

选项与参数： 

-k ： 不要真的关机，只是发送警告讯息出去！

-r ： 在将系统的服务停掉之后就重新启动(常用)

-h ： 将系统的服务停掉后，立即关机。 (常用)
 
-c ： 取消已经在进行的 shutdown 指令内容。

时间 ： 指定系统关机的时间！时间的范例底下会说明。若没有这个项目，则默认 1 分钟后自动进行。

范例：

[root@study ~]# /sbin/shutdown -h 10 'I will shutdown after 10 mins'

Broadcast message from root@study.centos.vbird (Tue 2015-06-02 10:51:34 CST):

I will shutdown after 10 mins

The system is going down for power-off at Tue 2015-06-02 11:01:34 CST!
```

#### 4.取消关机指令
```sh
shutdown -c
```

#### 5.立刻关机
```sh
shutdown -h now

立刻关机，其中 now 相当于时间为 0 的状态
```

#### 6.立刻重新启动
```sh
shutdown -r now

立刻关机，其中 now 相当于时间为 0 的状态
```

#### 7.保存数据重新启动
```sh
sync; sync; sync; reboot

内存数据写入硬盘命令先执行3次，然后再重新启动硬盘

[root@study ~]# halt # 系统停止～屏幕可能会保留系统已经停止的讯息！

[root@study ~]# poweroff # 系统关机，所以没有提供额外的电力，屏幕空白！
```

#### 8.systemctl关机指令

```sh
上面谈到的 halt, poweroff, reboot, shutdown 等等，其实都是呼叫这个 systemctl 指令

[root@study ~]# systemctl [指令]

指令项目包括如下：
halt 进入系统停止的模式，屏幕可能会保留一些讯息，这与你的电源管理模式有关

poweroff 进入系统关机模式，直接关机没有提供电力喔！

reboot 直接重新启动

suspend 进入休眠模式

[root@study ~]# systemctl reboot 系统重新启动

[root@study ~]# systemctl poweroff 系统关机
```

#### 9.init

```sh
『 init 0 』来关机
```


### 16.在终端机里面登入后，看到的提示字符 $ 与 # 有何不同？平时操作应该使用哪一个？
```sh
"#" 代表以 root 的身份登入系统，而 "$"则代表一般身份使用者。依据提示字符的不同， 我们可以约略判断登入者身份。一般来说，建议日常操作使用一般身份使用者登入，亦即是 $ ！
```

## 二:Linux的文件权限与目录配置