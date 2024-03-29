---
layout: post
title: 排行榜设计实现
categories: Mmo-Game
description: 排行榜设计实现
keywords: rank,mmo,game
---

排行榜设计实现

**目录**

* TOC
{:toc}

## 排行榜功能需求

```sh
1.根据击杀胜利次数进行排名

2.显示排名Id，用户头像，用户名称，胜利次数

3.最多显示10个用户

4.增加胜率排名、等级排名

5.排名奖励功能（结算）

6.半实时（有变化不实时广播，获取排行榜的时候，才是最新的）
  定时排序（例如：1分钟排一次）
```

## 潜在需要考虑的问题

```sh
潜在需要考虑的问题:
  1.未来扩展功能，修改代码，需要重启服务器。

  2.在运行期间，如何修复bug。
    例如：到点的时候，没有给玩家发奖励，出bug了。刚开服没多久，不允许停服。
         为了改一个bug，停服影响太大。排行榜业务不那么重要。

  3.不影响主业务前提下，修复问题和扩展新功能。

解决方法：
  
                 生产                     消费
   游戏服务器--------------》RocketMQ 《-------------- 排行榜进程     
        -                                              -
         -                                            -
           -                                         -
             -    <读取>                     <写入>  -
               -                                  -
                 -                              -
                   >                          <
                           
                              Redis

1.排序可以用redis自身的排序功能（执行命令就可以）。
2.排行榜进程处理好数据后，写入redis进行存储。
3.这么做的好处之一，如果将来增加了榜的类型，游戏服务器如果同步到MQ的数据已经
  满足，那么可以不做任务修改，只修改排行帮进程相关就可以。
4.排行榜业务和主游戏分离。
5.正常游戏还会将数据存到mysql数据库里一份，方便redis挂了或者数据错误，回溯和查询问题。
```


