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

##### 2.less (一页一页翻动）

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
