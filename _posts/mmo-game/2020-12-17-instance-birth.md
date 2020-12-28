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
* 出生点默认为C点，等间距为configLength
*
* 先以C点等间距计算出其它各个点坐标，
* 如果不是第一个进入，绕着初始点旋转朝向角度 + 额外90度
*（ps:绕着初始点旋转朝向角度会和队长同一条直线，所以额外再转90度）
*
* 依次输出方阵队形: C E A I G
*
*    Q     S
*
* K     M     O
*
*    G     I
*
* A     C     E
*
* @param index  第几个进来的人，从1开始，如果传入小于1默认等于1
* @param startX 初始坐标X
* @param startZ 初始坐标Z
* @param configLength 配置的等距间隔
* @param rotation 朝向角度
* @return 返回X和Z坐标数组
*/
public static int[] getFormationPosition(int index, int startX, int startZ, int configLength, int rotation)
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

    //保证朝向在0~360范围内
    int rRotation = MapUtils.rotationCut(rotation);
    return getPositionArr(startX, startZ, line, remainder, configLength,
            rRotation, index);
}


/**
* 先计算往哪个坐标轴偏移，
*
* 如果不是第一个进入，绕着初始点旋转某个角度
*
* @param startX
* @param startZ
* @param line 行数
* @param remainder 余数
* @param configLength 间距
* @param rRotation 朝向角度(0~360)
* @param index 第几个进来
* @return
*/
private static int[] getPositionArr(int startX, int startZ, int line, int remainder,
                                    int configLength, int rRotation, int index)
{
    //行间距
    int lineInterval = line - 1;
    //列间距
    int columnInterval = 0;
    switch (remainder)
    {
        case 0:
        {
            columnInterval = 1;
            break;
        }
        case 1:
        {
            columnInterval = 0;
            break;
        }
        case 2:
        {
            columnInterval = -2;
            break;
        }
        case 3:
        {
            columnInterval = 2;
            break;
        }
        case 4:
        {
            columnInterval = -1;
            break;
        }
        default:
        {
            break;
        }
    }

    //先固定往x+和z-轴偏
    int[] arr = new int[2];
    arr[0] = startX + columnInterval * configLength;
    arr[1] = startZ - lineInterval * configLength;

    //如果不是第一个进入，绕着初始点旋转某个角度
    if (index > 1)
    {
        return getPosARoundPosB(arr[0], arr[1], startX, startZ, rRotation + 90);
    }

    return arr;
}

/**
* A点绕着B点旋转某个角度，计算旋转后的坐标
* @param x1 A点X坐标
* @param z1 A点Z坐标
* @param x2 B点X坐标
* @param z2 B点Z坐标
* @param rotation 旋转角度
* @return 旋转后的坐标 x，z
*/
public static int[] getPosARoundPosB(int x1, int z1, int x2, int z2, int rotation)
{
    //保证朝向在0~360范围内
    int rRotation = MapUtils.rotationCut(rotation);
    //转为弧度
    double radian = rRotation * Math.PI / 180;
    double x = (x1 - x2) * Math.cos(radian) - (z1 - z2) * Math.sin(radian) + x2;
    double y = (z1 - z2) * Math.cos(radian) + (x1 - x2) * Math.sin(radian) + z2;
    return new int[]{(int)x, (int)y};
}
```