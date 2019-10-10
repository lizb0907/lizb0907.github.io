---
layout: post
title: 鸟哥linux实战
categories: Linux
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

### 1.查看文件名
```sh
ls  
```

### 2.查看文件名和属性
```sh
ls -al  包括显示隐藏文件

ls -l   不显示隐藏文件

ls -lh 可以查看当前路径总共占用多少k
```

### 3.查看当前目录文件完整的时间格式
```sh
ls -l --full-time
```

### 4.查看某个指定文件的详细属性
```sh
ls -al shadow

查看shadow文件的详细属性（前提是在文件目录下并且是文件！！）
```
### 5.查看“目录”权限命令
```sh
查看“目录”权限:

[root@VM_0_8_centos tmp]# ls -ld testing/

drwxr-xr-x 2 root root 4096 Aug 24 15:02 testing/
```
```sh
一次查看testing目录和testing目录下testing文件权限：

[root@VM_0_8_centos tmp]# ls -ald testing testing/testing 

drwxr-xr-x 2 root root 4096 Aug 24 15:16 testing

-rw------- 1 root root    0 Aug 24 15:16 testing/testing
```
### 6.属性含义

所有属性:

![](/images/posts/linux/1.png)

权限属性:

![](/images/posts/linux/2.png)


举例：-rw-r--r--

第一个字符代表这个文件是『目录、文件或链接文件等等』：

[ d ]则是目录，例如上表档名为『.config』的那一行；

[ - ]则是文件，例如上表档名为『initial-setup-ks.cfg』那一行；

[ l ]则表示为连结档(link file)； 

[ b ]则表示为装置文件里面的可供储存的接口设备(可随机存取装置)； 

[ c ]则表示为装置文件里面的串行端口设备，例如键盘、鼠标(一次性读取装置)。

### 6.如何改变文件属性与权限

#### 1.三个指令

```sh
chgrp ：改变文件所属群组

change group的缩写

chown ：改变文件拥有者

change owner的缩写

chmod ：改变文件的权限, SUID, SGID, SBIT 等等的特性
```

#### 2.改变所属群组, chgrp

要被改变的组名必须要在/etc/group文件内存在

```sh
chgrp users test

将test文件群组权限改成为users（前提users是/etc/group文件内存在）

改变群组权限后，我们可以通过ls -l查看
```
![](/images/posts/linux/3.png)

#### 3.改变文件拥有者, chown

用户必须是已经存在/etc/passwd文件中
```sh
chown -R bp3 test

将test目录和目录下所有文件递归赋予拥有者bp3
```

```sh
chown root:root test

test拥有者与群组改回为 root：
```

```sh
chown .root test

单纯的修改所属群组拥有者,这就是小数点用途
```

```sh
cp start.sh start_copy.sh

复制start.sh文件并重名命为start_copy.sh（注意只能复制文件并且权限不改变）
```

#### 4.改变文件权限, chmod

##### 1.数字类型改变文件权限
```sh
数字类型改变文件权限:

『-rwxrwxrwx』， 这九个权限是三个三个一组的

各个权限的分数值:r = 4, w = 2, x = 1。
```

```sh
[-rwxrwx---] 分数则是：

owner = rwx = 4+2+1 = 7

group = rwx = 4+2+1 = 7

others= --- = 0+0+0 = 0

所以该文件的权限数字就是 770
```

```sh
[root@VM_0_8_centos test]# ls -al start.sh

-rw-r--r-- 1 bp3 root 405 Nov 22  2018 start.sh

如果想要将start.sh的权限全部启用，可以输入：

[root@VM_0_8_centos test]# chmod 777 start.sh

[root@VM_0_8_centos test]# ls -al start.sh

-rwxrwxrwx 1 bp3 root 405 Nov 22  2018 start.sh
```

```sh
如果要将权限变成『 -rwxr-xr-- 』呢？那么权限的分数就成为 [4+2+1][4+0+1][4+0+0]=754
```
##### 2.符号类型改变文件权限

```sh
基本上就九个权限分别是(1)user (2)group (3)others 三种身份,

那么我们就可以藉由 u, g, o 来代表三种身份的权限！此外， a 则代表 all 亦即全部的身份！

那么读写的权限就可以写成 r, w, x

+(加入) 

-(除去)

=(设定)
```

```sh
我们要『设定』一个文件的权限成为『-rwxr-xr-x』:

[root@VM_0_8_centos test]# chmod u=rwx,go=rx start.sh 

# 注意！那个 u=rwx,go=rx 是连在一起的，中间并没有任何空格符！

[root@VM_0_8_centos test]# ls -l start.sh

-rwxr-xr-x 1 bp3 root 405 Nov 22  2018 start.sh
```

```sh
我不知道原先的文件属性，而我只想要增加start.sh 这个文件的每个人均可写入的权限:

chmod a + w start.sh
```

```sh
要拿掉全部人的start.sh可执行权限:

chmod a - x start.sh
```

### 7.目录与文件之权限意义

#### 1.例题
```sh
有个目录的权限如下所示：

drwxr--r-- 3 root root 4096 Jun 25 08:35 .ssh

d代表是目录，rwx拥有者root权限为读写和执行，r--群组权限只为读，r--其它权限只为读

系统有个账号名称为 vbird，这个账号并没有支持 root 群组，请问 vbird 对这个目录有何权限？是否可切换到此目
录中？

vbird 对此目录仅具有 r 的权限，因此 vbird 可以查询此目录下的文件名列表。因为 vbird 不具有 x 的权限，
所以没有这个抽屉的钥匙啦！ 因此 vbird 并不能切换到此目录内！(相当重要的概念！)。

如果你在某目录下不具有 x 的权限， 那么你就无法切换到该目录下（不能cd到此目录），
也就无法执行该目录下的任何指令，即使你具有该目录的 r 或 w 的权限。
```

#### 2.例题
```sh
假设有个账号名称为 dmtsai，他的家目录在/home/dmtsai/，dmtsai 对此目录具有[rwx]的权限。 若在此目录下有个
名为 the_root.data 的文件，该文件的权限如下：

-rwx------ 1 root root 4365 Sep 19 23:20 the_root.data

请问 dmtsai 对此文件的权限为何？可否删除此文件？

如上所示，由于 dmtsai 对此文件来说是『others』的身份，因此这个文件他无法读、无法编辑也无法执行，也就
是说，他无法变动这个文件的内容就是了。

但是由于这个文件在他的家目录下， 他在此目录下具有 rwx 的完整权限，因此对于 the_root.data 这个『档名』来
说，他是能够『删除』的！ 结论就是，dmtsai 这个用户能够删除 the_root.data 这个文件！
```

#### 3.例题
```sh
[root@study ~]# cd /tmp <==切换工作目录到/tmp

[root@study tmp]# mkdir testing <==建立新目录

[root@study tmp]# chmod 744 testing <==变更权限

[root@study tmp]# touch testing/testing <==建立空的文件

[root@study tmp]# chmod 600 testing/testing <==变更权限

[root@study tmp]# ls -ald testing testing/testing

drwxr--r--. 2 root root 20 Jun 3 01:00 testing

-rw-------. 1 root root 0 Jun 3 01:00 testing/testing
```

### 8.绝对路径与相对路径

```sh
绝对路径：由根目录(/)开始写起的文件名或目录名称， 例如 /home/dmtsai/.bashrc； 

相对路径：相对于目前路径的文件名写法。 例如 ./home/dmtsai 或 ../../home/dmtsai/ 等等。反正开头不是 / 
就属于相对路径的写法
```

#### 1.当前目录为/home目录下， 如果想要进入/var/log 这个目录，如何操作？
```sh
首先我们要清除home和var这两个目录都在根目录下，那么当前在/home目录下，执行../返回上一目录，就是var所在的根目录。

操作方法两种:

1. cd /var/log (absolute)

2. cd ../var/log (relative)

注意:

. ：代表当前的目录，也可以使用 ./ 来表示；

.. ：代表上一层目录，也可以 ../ 来代表。
```

#### 2.网络文件常常提到类似『./run.sh』之类的数据，这个指令的意义为何？
```sh
『./』代表『本目录』的意思，所以『./run.sh』代表『执行本目录下， 名为 run.sh 的文件』啰！
```

### 9.Centos信息输出
```sh
uname -r 

查看核心版本

man uname

查看命令
```

## 三:Linux 文件与目录管理
 
### 1.相对路径的用途
```sh
例如：

/cluster/raid/output/taiwan2006/smoke 这个目录，而另一个目录在 /cluster/raid/output/taiwan2006/cctm ，

那么我从第一个要到第二个目录去的话，怎么写比较方便？ 当然是『 cd ../cctm 』
```

### 2.绝对路径的用途

但是对于档名的正确性来说，绝对路径的正确度要比较好。

### 3.目录相关操作
```sh
. 代表此层目录

.. 代表上一层目录 - 代表前一个工作目录

~ 代表『目前用户身份』所在的家目录

~account 代表 account 这个用户的家目录(account 是个账号名称)
```

```sh
cd：变换目录

pwd：显示当前目录

mkdir：建立一个新的目录

rmdir：删除一个空的目录
```

#### 1.mkdir

[root@study ~]# mkdir [-mp] 目录名称

选项与参数：

 -m ：配置文件案的权限喔！直接设定，不需要看预设权限 (umask) 的脸色～

-p ：帮助你直接将所需要的目录(包含上层目录)递归建立起来！

范例：请到/tmp 底下尝试建立数个新目录看看：
```sh
[root@study tmp]# mkdir test1/test2/test3/test4

mkdir: cannot create directory ‘test1/test2/test3/test4’: No such file or directory

# 话说，系统告诉我们，没可能建立这个目录啊！就是没有目录才要建立的！见鬼嘛？

[root@study tmp]# mkdir -p test1/test2/test3/test4

# 原来是要建 test4 上层没先建 test3 之故！加了这个 -p 的选项，可以自行帮你建立多层目录！
```

范例：建立权限为 rwx--x--x 的目录
```sh
[root@study tmp]# mkdir -m 711 test2

[root@study tmp]# ls -ld test*

drwxr-xr-x. 2 root root 6 Jun 4 19:03 test

drwxr-xr-x. 3 root root 18 Jun 4 19:04 test1

drwx--x--x. 2 root root 6 Jun 4 19:05 test2

# 仔细看上面的权限部分，如果没有加上 -m 来强制设定属性，系统会使用默认属性。
```

#### 2.rmdir (删除『空』的目录！注意是“目录”)

[root@study ~]# rmdir [-p] 目录名称

选项与参数： -p ：连同『上层』『空的』目录也一起删除

```sh
rmdir tick

删除tick空目录
```

```sh
如果目录和子目录都是目录并且都为空：

rmdir -p tick/tick1

我们可以连同tick目录下的tick1目录一起全部删除
```

```sh
但是如果tick目录下既有目录还有文件，那么整个删除只能用

rm -rf 目录

用这个命令一定要特别小心
```

### 4.文件与目录管理

#### 1.文件与目录的检视： ls
```sh
man ls

查看ls用法
```
```sh
ls -al

所有文件列出来(含属性与隐藏文件)

ls -l

所有文件列出来(含属性不包含隐藏文件)
```
```sh
ls -al --full-time

所有文件详细时间列出来(含属性与隐藏文件)
```

#### 2.cp (复制文件或目录)

[root@study ~]# cp [-adfilprsu] 来源文件(source) 目标文件(destination)

[root@study ~]# cp [options] source1 source2 source3 .... directory

选项与参数： 
-a ：相当于 -dr --preserve=all 的意思，至于 dr 请参考下列说明；(常用) 

-d ：若来源文件为链接文件的属性(link file)，则复制链接文件属性而非文件本身；

-f ：为强制(force)的意思，若目标文件已经存在且无法开启，则移除后再尝试一次；

-i ：若目标文件(destination)已经存在时，在覆盖时会先询问动作的进行(常用) -l ：进行硬式连结(hard link)的连结档建立，而非复制文件本身；

-p ：连同文件的属性(权限、用户、时间)一起复制过去，而非使用默认属性(备份常用)； 

-r ：递归持续复制，用于目录的复制行为；(常用) 

-s ：复制成为符号链接文件 (symbolic link)，亦即『快捷方式』文件；

-u ：destination 比 source 旧才更新 destination，或 destination 不存在的情况下才复制。

--preserve=all ：除了 -p 的权限相关参数外，还加入 SELinux 的属性, links, xattr 等也复制了。

最后需要注意的，如果来源档有两个以上，则最后一个目的文件一定要是『目录』才行！

##### 1.将当前目录下的文件拷贝到另一目录下并重命名
```sh
[root@VM_0_8_centos test]# ls

start_copy.sh  start.sh

[root@VM_0_8_centos test]# cp start.sh /data/bp3/update.sh

将当前目录下start.sh文件拷贝到bp3下并重名为update.sh
```

```sh
[root@VM_0_8_centos test]# cp -i start.sh /data/bp3/update.sh

cp: overwrite ‘/data/bp3/update.sh’? y

[root@VM_0_8_centos test]# 

将当前目录下start.sh文件拷贝到bp3下并重名为update.sh,如果update.sh已经存在bp3目录，是否覆盖？
```

##### 2.复制整个目录到另一个目录，拥有者会更改为当前账号

cp -r bp3 /data/test/
```sh
[root@VM_0_8_centos data]# ls

bp3  test

[root@VM_0_8_centos data]# ls -ld bp3

drwxr-xr-x 2 bp3 root 4096 Aug 28 12:49 bp3

这里的拥有者权限为bp3！

[root@VM_0_8_centos data]# cp -r bp3 /data/test/

我们用root执行递归复制目录命令

[root@VM_0_8_centos data]# ls -ld /data/test/bp3

drwxr-xr-x 2 root root 4096 Aug 28 13:19 /data/test/bp3

所以这里目录的拥有者更改为当前用户root了
```

cp -a bp3 /data/test/
```sh
[root@VM_0_8_centos data]# cp -a bp3 /data/test/

[root@VM_0_8_centos data]# ls -ld /data/test/bp3

drwx--x--x 2 bp3 root 4096 Aug 28 12:49 /data/test/bp3

这里用root权限，执行cp -a 复制目录，但是拥有者权限没有改变

当我们在进行备份的时候，某些需要特别注意的特殊权限文件， 例如密码文件 (/etc/shadow) 以及一些配置文件，就不能直接以 cp 来复制，而必须要加上 -a 或者是 -p 等等可以完整复制文件权限的选项才行！
```

#### 3.rm (移除文件或目录)

[root@study ~]# rm [-fir] 文件或目录

选项与参数： -f ：就是 force 的意思，忽略不存在的文件，不会出现警告讯息；

-i ：互动模式，在删除前会询问使用者是否动作 

-r ：递归删除啊！最常用在目录的删除了！这是非常危险的选项！！！

#### 4.mv (移动文件与目录，或更名)

范例三：再建立两个文件，再全部移动到 /tmp/mvtest2 当中

[root@study tmp]# cp ~/.bashrc bashrc1

[root@study tmp]# cp ~/.bashrc bashrc2

[root@study tmp]# mv bashrc1 bashrc2 mvtest2

注意到这边，如果有多个来源文件或目录，则最后一个目标文件一定是『目录！』

意思是说，将所有的数据移动到该目录的意思！

#### 5.取得路径的文件名与目录名称

[root@VM_0_8_centos test]# basename /data/test/start.sh

start.sh  //取最后的文件名

[root@VM_0_8_centos test]# dirname /data/test/start.sh

/data/test  //取得目录名称

#### 6.文件内容查阅

最常使用的显示文件内容的指令可以说是 cat 与 more 及 less 了！

此外，如果我们要查看一个很大型的文件 (好几百 MB 时)，但是我们只需要后端的几行字而已，用 tail 呀，

此外， tac 这个指令也可以达到这个目的喔

```sh
cat 由第一行开始显示文件内容

tac 从最后一行开始显示，可以看出 tac 是 cat 的倒着写！

nl 显示的时候，顺道输出行号！

more 一页一页的显示文件内容

less 与 more 类似，但是比 more 更好的是，他可以往前翻页！

head 只看头几行

tail 只看尾巴几行

od 以二进制的方式读取文件内容！
```

##### 1.cat (concatenate)

[root@study ~]# cat [-AbEnTv]

选项与参数： -A ：相当于 -vET 的整合选项，可列出一些特殊字符而不是空白而已；

-b ：列出行号，仅针对非空白行做行号显示，空白行不标行号！ 

-E ：将结尾的断行字符 $ 显示出来； 

-n ：打印出行号，连同空白行也会有行号，与 -b 的选项不同；

-T ：将 [tab] 按键以 ^I 显示出来；

-v ：列出一些看不出来的特殊字符

cat 是 Concatenate (连续) 的简写， 主要的功能是将一个文件的内容连续的印出在屏幕上面！

范例一：cat -n 打印行号
```sh
[root@VM_0_8_centos test]# cat -n start.sh
     1	#!/bin/sh
     2	
     3	export JAVA_HOME=/usr/java/jdk1.8.0_191
     4	export PATH=$JAVA_HOME/bin:$PATH
     5	
     6	cd /data/bp3/test/
     7	
     8	nohup java -server -Xms2048m -Xmx2048m -Xss1024K  -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=512m -verbose:gc -Xloggc:gc.log -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+PrintGCDetails -Dsun.reflect.noInflation=true -cp  resources:BPTest-1.jar com.game2sky.bp.test.MainTest > nohup.out 2>&1 &
[root@VM_0_8_centos test]# 
```

##### 2.tac（反向列显示）

```sh
[root@study ~]# tac /etc/issue
Kernel \r on an \m \S
# 嘿嘿！与刚刚上面的范例一比较，是由最后一行先显示喔！
```

##### 3.nl (添加行号打印)

```sh
[root@VM_0_8_centos test]# nl start.sh
     1	#!/bin/sh
       
     2	export JAVA_HOME=/usr/java/jdk1.8.0_191
     3	export PATH=$JAVA_HOME/bin:$PATH
       
     4	cd /data/bp3/test/
       
     5	nohup java -server -Xms2048m -Xmx2048m -Xss1024K  -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=512m -verbose:gc -Xloggc:gc.log -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+PrintGCDetails -Dsun.reflect.noInflation=true -cp  resources:BPTest-1.jar com.game2sky.bp.test.MainTest > nohup.out 2>&1 &
```

#### 7.可翻页检视

##### 1.more （一页一页翻动）

[root@study ~]# more /etc/man_db.con

```sh
在 more 这个程序的运作过程中，你有几个按键可以按的：
 空格键 (space)：代表向下翻一页；
 Enter ：代表向下翻『一行』；
 /字符串 ：代表在这个显示的内容当中，向下搜寻『字符串』这个关键词；
 :f ：立刻显示出文件名以及目前显示的行数；
 q ：代表立刻离开 more ，不再显示该文件内容。
 b 或 [ctrl]-b ：代表往回翻页，不过这动作只对文件有用，对管线无用。
```

##### 2.less (一页一页翻动,强烈建议）

[root@study ~]# less /etc/man_db.conf

可以使用 [pageup] [pagedown] 等按键的功能来往前往后翻看文件。

```sh
 空格键 ：向下翻动一页；
 [pagedown]：向下翻动一页；
 [pageup] ：向上翻动一页；
 /字符串 ：向下搜寻『字符串』的功能；
 ?字符串 ：向上搜寻『字符串』的功能；
 n ：重复前一个搜寻 (与 / 或 ? 有关！) 
 N ：反向的重复前一个搜寻 (与 / 或 ? 有关！)
 g ：前进到这个资料的第一行去；
 G ：前进到这个数据的最后一行去 (注意大小写)； 
 q ：离开 less 这个程序；
```

注意：这里有技巧，我们平时查看日志内容时太大了，我们需要看最近的输出的日志，所以需要从底部往上看能翻页看。

因此，我们可以less nohup.out, 然后按下G调到底部，再用PageUp一页一页往上翻。

tail -100f nohup.out 实时监测输出的倒数100行数据。

类似

#### 8.资料撷取

head(取出前面几行) 和 tail (取出后面几行)

```sh
[root@study ~]# tail [-n number] 
文选项与参数： 
-n ：后面接数字，代表显示几行的意思
-f ：表示持续侦测后面所接的档名，要等到按下[ctrl]-c 才会结束 tail 的侦测

持续侦测/var/log/messages 的内容[root@study ~]# tail -f /var/log/messages
<==要等到输入[crtl]-c 之后才会离开 tail 这个指令的侦测！
```

##### 1.显示 gc.log 的第 11 到第 20 行呢？

```sh
在第 11 到第 20 行，那么我取前 20 行，再取后十行，所以结果就是：
『 head -n 20 gc.log  | tail -n 10 』，这样就可以得到第 11 到第 20 行之间的内容了！

[root@VM_0_8_centos data]# head -n 20 gc.log | tail -n 10
2019-09-09T17:03:22.105+0800: 3.437: Total time for which application threads were stopped: 0.0001994 seconds, Stopping threads took: 0.0000350 seconds
{Heap before GC invocations=0 (full 0):
 garbage-first heap   total 8388608K, used 417792K [0x00000005c0000000, 0x00000005c0404000, 0x00000007c0000000)
  region size 4096K, 102 young (417792K), 0 survivors (0K)
 Metaspace       used 15132K, capacity 15722K, committed 15872K, reserved 1062912K
  class space    used 1821K, capacity 1924K, committed 2048K, reserved 1048576K
2019-09-09T17:03:22.182+0800: 3.514: [GC pause (G1 Evacuation Pause) (young) 3.514: [G1Ergonomics (CSet Construction) start choosing CSet, _pending_cards: 0, predicted base time: 10.00 ms, remaining time: 190.00 ms, target pause time: 200.00 ms]
 3.514: [G1Ergonomics (CSet Construction) add young regions to CSet, eden: 102 regions, survivors: 0 regions, predicted young region time: 6180.99 ms]
 3.514: [G1Ergonomics (CSet Construction) finish choosing CSet, eden: 102 regions, survivors: 0 regions, old: 0 regions, predicted pause time: 6190.99 ms, target pause time: 200.00 ms]
, 0.0420786 secs]


这两个指令中间有个管线 (|) 的符号存在，这个管线的意思是：『前面的指令所输出的讯息，请透过管线交由后续
的指令继续使用』的意思。 所以， gc.log  会将文件内的 20 行取出来，但不输出到屏幕上，而是转交给后续的 tail 指令继续处理。 因此 tail 『不需要接档名』，因为 tail 所需要的数据是来自于 head 处理后的结果！
```

##### 2.显示 gc.log 的第 11 到第 20 行并带行号

```sh
通过cat -n 带出行号
[root@VM_0_8_centos data]# cat -n gc.log | head -n 20 | tail -n 10
    11	2019-09-09T17:03:22.105+0800: 3.437: Total time for which application threads were stopped: 0.0001994 seconds, Stopping threads took: 0.0000350 seconds
    12	{Heap before GC invocations=0 (full 0):
    13	 garbage-first heap   total 8388608K, used 417792K [0x00000005c0000000, 0x00000005c0404000, 0x00000007c0000000)
    14	  region size 4096K, 102 young (417792K), 0 survivors (0K)
    15	 Metaspace       used 15132K, capacity 15722K, committed 15872K, reserved 1062912K
    16	  class space    used 1821K, capacity 1924K, committed 2048K, reserved 1048576K
    17	2019-09-09T17:03:22.182+0800: 3.514: [GC pause (G1 Evacuation Pause) (young) 3.514: [G1Ergonomics (CSet Construction) start choosing CSet, _pending_cards: 0, predicted base time: 10.00 ms, remaining time: 190.00 ms, target pause time: 200.00 ms]
    18	 3.514: [G1Ergonomics (CSet Construction) add young regions to CSet, eden: 102 regions, survivors: 0 regions, predicted young region time: 6180.99 ms]
    19	 3.514: [G1Ergonomics (CSet Construction) finish choosing CSet, eden: 102 regions, survivors: 0 regions, old: 0 regions, predicted pause time: 6190.99 ms, target pause time: 200.00 ms]
    20	, 0.0420786 secs]
```

#### 9.修改文件时间或建置新档： touch

##### 1. ls查看文件更新时间

```sh
[root@VM_0_8_centos data]# ls -l gc.log
-rw-r--r-- 1 root root 25593736 Sep 11 12:50 gc.log

默认的情况下，ls 显示出来的是该文件的 mtime,
当该文件的『内容数据』变更时，就会更新这个时间！内容数据指的是文件的内容，而不是文件的属性或
权限喔！
```

```sh
[root@VM_0_8_centos data]# ls -l --time=atime gc.log
-rw-r--r-- 1 root root 25593736 Sep 11 12:52 gc.log

显示读取过内容时间(atime),
当『该文件的内容被取用』时，就会更新这个读取时间 (access)。举例来说，我们使用 cat 去读取
/etc/man_db.conf ， 就会更新该文件的 atime 了。
```

```sh
[root@VM_0_8_centos data]# chmod 777 gc.log
[root@VM_0_8_centos data]# ls -l gc.log
-rwxrwxrwx 1 root root 25593736 Sep 11 12:50 gc.log
[root@VM_0_8_centos data]# ls -l --time=ctime gc.log
-rwxrwxrwx 1 root root 25593736 Sep 17 23:17 gc.log

改变权限后，执行ls -l --time=ctime，时间显示为最新更新时间。
当该文件的『状态 (status)』改变时，就会更新这个时间，举例来说，像是权限与属性被更改了，都会更新
这个时间啊。
```

##### 2.touch可以轻易的修订文件的日期与时间
```sh
[root@study ~]# touch [-acdmt] 文件
选项与参数： -a ：仅修订 access time； 
-c ：仅修改文件的时间，若该文件不存在则不建立新文件； 
-d ：后面可以接欲修订的日期而不用目前的日期，也可以使用 --date="日期或时间" 
-m ：仅修改 mtime ； 
-t ：后面可以接欲修订的时间而不用目前的时间，格式为[YYYYMMDDhhmm]
```

## 四:文件与文件系统的压缩

### 1.常见的压缩指令

#### 1.gizp

```sh
[dmtsai@study ~]$ gzip [-cdtv#] 檔名
[dmtsai@study ~]$ zcat 檔名.gz
选项与参数：
-c ：将压缩的数据输出到屏幕上，可透过数据流重导向来处理；
-d ：解压缩的参数；
-t ：可以用来检验一个压缩文件的一致性～看看文件有无错误； 
-v ：可以显示出原文件/压缩文件案的压缩比等信息；
-# ：# 为数字的意思，代表压缩等级，-1 最快，但是压缩比最差、-9 最慢，但是压缩比最好！预设是 -6
```

```sh
[dmtsai@study tmp]$ gzip -v services

services: 79.7% -- replaced with services.gz

压缩文件同时显示出原文件/压缩文件案的压缩比等信息
```

```sh
gzip -d services.gz

解压缩
```

#### 2.xz

 xz这个命令压缩比gzip命令压缩比更高。

```sh
[dmtsai@study ~]$ xz [-dtlkc#] 檔名
[dmtsai@study ~]$ xcat 檔名.xz
选项与参数： -d ：就是解压缩啊！
-t ：测试压缩文件的完整性，看有没有错误
-l ：列出压缩文件的相关信息
-k ：保留原本的文件不删除～
-c ：同样的，就是将数据由屏幕上输出的意思！
-# ：同样的，也有较佳的压缩比的意思！
```

#### 3.tar

tar 可以将多个目录或文 件打包成一个大文件，同时还可以透过 gzip/bzip2/xz 的支持，将该文件同时进行压缩

```sh
[dmtsai@study ~]$ tar [-z|-j|-J] [cv] [-f 待建立的新檔名] filename... <==打包与压缩
[dmtsai@study ~]$ tar [-z|-j|-J] [tv] [-f 既有的 tar 檔名] <==察看檔名
[dmtsai@study ~]$ tar [-z|-j|-J] [xv] [-f 既有的 tar 檔名] [-C 目录] <==解压缩
选项与参数：
-c ：建立打包文件，可搭配 -v 来察看过程中被打包的档名(filename)
-t ：察看打包文件的内容含有哪些档名，重点在察看『档名』就是了；

-x ：解打包或解压缩的功能，可以搭配 -C (大写) 在特定目录解开 特别留意的是， -c, -t, -x 不可同时出现在一串指令列中。

-z ：透过 gzip 的支持进行压缩/解压缩：此时档名最好为 *.tar.gz
-j ：透过 bzip2 的支持进行压缩/解压缩：此时档名最好为 *.tar.bz2
-J ：透过 xz 的支持进行压缩/解压缩：此时档名最好为 *.tar.xz
特别留意， -z, -j, -J 不可以同时出现在一串指令列中
-v ：在压缩/解压缩的过程中，将正在处理的文件名显示出来！ -f filename：-f 后面要立刻接要被处理的档名！建议 -f 单独写一个选项啰！(比较不会忘记)
-f : 待建立的新檔名] filename
-C(大写) 目录 ：这个选项用在解压缩，若要在特定目录解压缩，可以使用这个选项。其他后续练习会使用到的选项介绍

-p(小写) ：保留备份数据的原本权限与属性，常用于备份(-c)重要的配置文件
-P(大写) ：保留绝对路径，亦即允许备份数据中含有根目录存在之意；
--exclude=FILE：在压缩的过程中，不要将 FILE 打包！
```

一般-vf参数必须加上，代表在压缩/解压缩的过程中，将正在处理的文件名显示出来同时待建立的新档名。

```sh
打包命令tar:
tar -cvf 打包文件名 源文件
选项：
-c ：打包
-v ：显示过程
-f ：指定打包后的文件名
```

```sh
解打包命令
tar -xvf 打包文件名
选项：
-x : 解打包
```

```sh
利用tar并添加gzip压缩文件:
tar -zcvf [自己取的压缩文件名] [test]
[root@VM_0_8_centos data]# ls
bp3  bp3_copy  gc.log  test  test.tar
[root@VM_0_8_centos data]# tar -zcvf test.tar.gz test

-z 添加gzip 的支持，所以档名最好取为 *.tar.gz。
将test文件压缩为test.tar.gz

解压test.tar.gz:
[root@VM_0_8_centos data]# tar -zxvf test.tar.gz
```

## 五:vim 程序编辑器

### 1.vi指令

所有的 Unix Like 系统都会内建 vi 文书编辑器，其他的文书编辑器则不一定会存在(vim不一定有)。

#### 1.使用『 vi filename 』进入一般指令模式

```sh
[dmtsai@study ~]$ /bin/vi welcome.txt

在 CentOS 7 当中，由于一般账号预设 vi 已经被 vim 取代了，因此得要输入绝对路径来执行才行！
```

#### 2.按下 i 进入编辑模式，开始编辑文字

```sh
按下 i 进入编辑模式，左下角状态栏中会出现 –INSERT- 的字样。
```

#### 3.按下 [ESC] 按钮回到一般指令模式

```sh
编辑完毕, 按下[Esc] 这个按钮
```
#### 4.进入指令列模式，文件储存并离开 vi 环境

```sh
输入分好“:”  + 

wq  保存并退出
wq! 强制保存并退出
q   不保存退出
q!  不保存强制退出
```

#### 5.一般指令模式常用操作，搜素...

一般模式下按数字不能按小键盘上的数字，按字母上面的数字。

```sh
n<Enter> n 为数字。光标向下移动 n 列(常用)

G 移动到这个文件的最后一列(常用)

gg 移动到这个文件的第一列，相当于 1G 啊！ (常用)

/word向光标之下寻找一个名称为 word 的字符串。例如要在文件内搜寻 vbird 这个字符串，
就输入 /vbird 即可！ (常用)

?word 向光标之上寻找一个字符串名称为 word 的字符串。

n 这个 n 是英文按键。代表『重复前一个搜寻的动作』。

N这个 N 是英文按键。与 n 刚好相反，为『反向』进行前一个搜寻动作。 例如 /vbird 
后，按下 N 则表示『向上』搜寻 vbird 。

yy 复制游标所在的那一列(常用)。

u 复原前一个动作。(常用)
```

### 2.vim指令

上面的指令同样适用于vim指令，可以简单理解为vim是vi的加强指令，现在我们一般用vim指令。

vim 具有颜色显示的功能，并且还支持许多的程序语法。

vim 支持区块选择(Visual Block)，多文件编辑，多窗口。

## 六:认识 BASH 这个 Shell

### 1.硬件、核心与 Shell

```sh
透过『 Shell 』将我们输入的指令与 Kernel(核心管理：操作系统的核心可以支持这个芯片组，当然还需要提供芯片的驱动程序啰) 沟通，好让 Kernel 可以控制硬件来正确无误的工作。

简单理解就是应用程序，最外部的壳程序。

壳程序的功能只是提供用户操作系统的一个接口，因此这个壳程序需要可以呼叫其他软件，man, chmod, chown, vi, fdisk, mkfs 等等指令，这些指令都是独立的应用程序， 但是我们可以透过壳程序 (就是指令列模式) 来操作这些应用程序，让这些应用程序呼叫核心来运作所需的工作。

只要能够操作应用程序的接口都能够称为壳程序。

狭义的壳程序指的是指令列方面的软件，包括 bash，也就是说bash其实壳shell（壳程序）的一种。
```

### 2.linux使用的Shell

Linux 使用的shell称为『 Bourne Again SHell (简称 bash) 』。

Linux 预设就是使用 bash ，所以最初你只要学会 bash 就非常了不起了。

### 3.查询指令是否为 Bash shell 的内建命令： type

```sh
[dmtsai@study ~]$ type [-tpa] name
选项与参数：

：不加任何选项与参数时，type 会显示出 name 是外部指令还是 bash 内建指令
-t ：当加入 -t 参数时，type 会将 name 以底下这些字眼显示出他的意义：
     file ：表示为外部指令；
     alias ：表示该指令为命令别名所设定的名称；
     builtin ：表示该指令为 bash 内建的指令功能；
-p ：如果后面接的 name 为外部指令时，才会显示完整文件名；
-a ：会由 PATH 变量定义的路径中，将所有含 name 的指令都列出来，包含 alias
```

```sh
查看CD命令是否Bash内建命令：

[root@VM_0_8_centos ~]# type cd
cd is a shell builtin
[root@VM_0_8_centos ~]# 

builtin表面cd是Bash内建命令
```

### 4.如果指令串太长的话，如何使用两行来输出？
```sh
[root@VM_0_8_centos data]# cp gc.log \
> gc.sss
[root@VM_0_8_centos data]# ls
bp3  bp3_copy  gc.log  gc.sss  test  test.tar  test.tar.gz

指令太长，让 [Enter] 按键不再具有『开始执行』的功能！好让指令可以继续在下一行输入。

需要特别留意， [Enter] 按键是紧接着反斜杠 (\) 的，两者中间没有其他字符
```

### 5.请在屏幕上面显示出您的环境变量 HOME 与 MAIL？
```sh
[root@VM_0_8_centos data]# echo ${HOME}
/root
[root@VM_0_8_centos data]# echo ${MAIL}
/var/spool/mail/root
```
### 6.变量

#### 1.如何『设定』或者是『修改』 某个变量的内容啊？
```sh
[dmtsai@study ~]$ echo ${myname}
<==这里并没有任何数据～因为这个变量尚未被设定！是空的！
[dmtsai@study ~]$ myname=VBird
[dmtsai@study ~]$ echo ${myname}
VBird <==出现了！因为这个变量已经被设定了！
```

#### 2.取消变量的方法为使用 unset

```sh
『unset 变量名称』例如取消 myname 的设定：
『unset myname』

[root@VM_0_8_centos data]# echo ${myset}

[root@VM_0_8_centos data]# myset=lizhibiao
[root@VM_0_8_centos data]# echo ${myset}
lizhibiao
[root@VM_0_8_centos data]# unset myset
[root@VM_0_8_centos data]# echo ${myset}

[root@VM_0_8_centos data]# 

输入unset myset，再输入echo ${myset}，变量为空
```

#### 3.变量的设定规则

```sh
1. 变量与变量内容以一个等号『=』来连结，如下所示：『myname=VBird』

2. 变量名称只能是英文字母与数字，但是开头字符不能是数字，如下为错误：
『2myname=VBird』

3. 变量内容若有空格符可使用双引号"" 或单引号''将变量内容结合起来，但
     o 双引号内的特殊字符如 $ 等，可以保有原本的特性，如下所示：
     『var="lang is $LANG"』则『echo $var』可得『lang is zh_TW.UTF-8』 
     o 单引号内的特殊字符则仅为一般字符 (纯文本)，如下所示：
     『var='lang is $LANG'』则『echo $var』可得『lang is $LANG』
     
4. 可用跳脱字符『 \ 』将特殊符号(如 [Enter], $, \, 空格符...)变成一般字符，如：
『myname=VBird\ Tsai』

5.若该变量为扩增变量内容时，则可用 "$变量名称" 或 ${变量} 累加内容:
『PATH="$PATH":/home/bin』或『PATH=${PATH}:/home/bin』

6.若该变量需要在其他子程序执行，则需要以 export 来使变量变成环境变量：
『export PATH』

7. 通常大写字符为系统默认变量，自行设定变量可以使用小写字符

8.在一串指令的执行中，还需要藉由其他额外的指令所提供的信息时，可以使用反单引号『`指令`』或 『$(指 令)』
```

#### 4.export： 自定义变量转成环境变量

子程序仅会继承父程序的环境变量， 子程序不会继承父程序的自定义变量。

```sh
[dmtsai@study ~]$ export 变量名称

将自定义变量变成环境变量的话，那就可以让该变量值继续存在于子程序。
```



