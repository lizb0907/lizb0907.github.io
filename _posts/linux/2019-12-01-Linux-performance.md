---
layout: post
title: linux性能优化
categories: Linux性能优化
description: 极课时间linux性能优化学习总结
keywords: linux, 性能，优化
---

极课时间linux性能优化学习总结, 常用命令

**目录**

* TOC
{:toc}

## uptime 查看当前用户数，平均负载

```sh
[root@hwsrv-348121 /]# uptime
 17:41:02 up 10 days, 16:38,  1 user,  load average: 0.03, 0.03, 0.05

load average: 0.03, 0.03, 0.05
意思是:过去1分钟，5分钟，15分钟内，平均活跃进程数目。

一般平均负载大于cpu数目的70%，就可能导致进程响应变慢，进而影响服务的正常功能。
```

```sh
平均负载: 平均活跃进程数目。

平均负载与 CPU 使用率:

1.平均负载不等于cpu使用率。

2.CPU 密集型进程，使用大量 CPU 会导致平均负载升高，此时这两者是一致的。

3.I/O密集型，大量等待I/O，此时平均负载很高，但是cpu使用率不一定很高。


```

## 两种查看cpu数目的方法

```sh
1.直接使用top命令

2.grep "model name" /proc/cpuinfo | wc -l
```

## stress是一个linux下的压力测试工具

stress
```sh
我们可以用模拟进程平均负载升高的场景。
```

安装
```sh
yum install -y epel-release

yum install -y stress
```

模拟一个 CPU 使用率 100% 的场景
```sh
$ stress --cpu 1 --timeout 600
```

这次模拟 I/O 压力，即不停地执行 sync
```sh
$ stress -i 1 --timeout 600
```

模拟的是 8 个进程使用
```sh
$ stress -c 8 --timeout 600
```

## sysstat软件包安装

```sh
这里我们直接安装sysstat: yum install sysstat

sysstat是一个软件包，包含监测系统性能及效率的一组工具:

sar、sadf、mpstat、iostat、pidstat等，这些工具可以监控系统
```

## mpstat 查看 CPU 使用率的变化情况

用来实时查看每个 CPU 的性能指标，以及所有 CPU 的平均指标。

显示所有 CPU 的指标，并在间隔 5 秒输出一组数据
```sh
$ mpstat -P ALL 5 1
```

## pidstat 查看哪个进程导致的

pidstat:

用来实时查看进程的 CPU、内存、I/O 以及上下文切换等性能指标。

差看哪个进程导致cpu或io使用率过高
```sh
# 间隔 5 秒后输出一组数据，-u 表示 CPU 指标
$ pidstat -u 5 1
```

