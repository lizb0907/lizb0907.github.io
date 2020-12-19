---
layout: post
title: 副本出生玩家位置不重叠简易算法
categories: Mmo-Game
description: 副本出生玩家位置不重叠简易算法
keywords: 副本，出生，位置，避免重叠
---

副本出生玩家位置简易算法，将玩家位置

**目录**

* TOC
{:toc}

## 需求

```sh
一般来说，游戏里每个场景策划会配置一个默认玩家出生坐标。玩家切场景后，都会以
配置的坐标为初始点。
由于，副本存在多人组队副本，如果只有一个坐标，玩家进入副本位置会重叠，表现效果比较差。
策划不想每个场景都配置一串坐标然后做随机，故实现一个摆阵型简易算法。
```


## 实现

```java
/**
*
* 获取摆整形坐标：
*
* P  Q  R  S  T
*
* K  L  M  N  O
*
* F  G  H  I  J
*
* A  B  C  D  E
*
* 依次输出方阵队形: A、C、E、G...
*
*    Q     S
*
* K     M     O
*
*    G     I
*
* A     C     E
*
* 出生点默认为A点，等间距为configLength
*
* @param index  第几个进来的人，从1开始，如果传入小于1默认等于1
* @param startX 初始坐标X
* @param startZ 初始坐标Z
* @param configLength 配置的等距间隔
* @return 返回X和Z坐标数组
*/
public static int[] getFormationPosition(int index, int startX, int startZ, int configLength)
{
    if (index < 1)
    {
        index = 1;
    }

    //余数
    int remainder = index % 5;
    //第几个5
    int divided = index / 5;

    //计算行数
    int line = divided * 2;
    if (remainder > 0)
    {
        line++;
    }
    if (remainder > 3)
    {
        line++;
    }

    int x = line - 1;
    int y = 0;
    switch (remainder)
    {
        case 0:
        {
            y = 3;
            break;
        }
        case 1:
        {
            y = 0;
            break;
        }
        case 2:
        {
            y = 2;
            break;
        }
        case 3:
        {
            y = 4;
            break;
        }
        case 4:
        {
            y = 1;
            break;
        }
        default:
        {
            break;
        }
    }

    int[] arr = new int[2];
    arr[0] = startX + x * configLength;
    arr[1] = startZ + y * configLength;
    return arr;
}
```