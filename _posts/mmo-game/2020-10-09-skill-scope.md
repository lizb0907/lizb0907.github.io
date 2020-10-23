---
layout: post
title: AOE技能范围索敌
categories: Mmo-Game
description: mmo游戏里技能范围索敌学习
keywords: skill，scope, mmo，game
---

AOE技能范围索敌学习

**目录**

* TOC
{:toc}

## 1.弧度与角度

1.区别在于角所对的弧长大小不同。度的是等于圆周长的360分之一，而弧度的是等于半径。

简单的说，弧度的定义是，当角所对的弧长等于半径时，角的大小为1弧度。

2.圆的周长 = 2πr, 所以一个周角是360度对应的弧长就是2πr， 那么360度对应2π弧度。

3.1度 = 2π弧度 / 360度， 即 1度 = π / 180 (弧度)

## 2.sin30° 和 sin30

sin30° 的含义是：在直角三角形中，30°角所对的直角边与斜边的比值，因为在直角三角形中，30°角所对的直角边是斜边的1/2，所以sin30°=1/2。

sin30 中的30表示30弧度(1弧度=57.3°)

在计算机中math函数用的都是弧度，所以我们一般都需要先将角度转为弧度再调用math函数计算（例如:Math.cos(弧度)）。

## 3.我们项目约定服务器和客户端使用坐标系

![](/images/posts/mmo_game/skill_scope/3.jpg)

1.Unity的世界空间是左手坐标系，而观察空间是右手坐标系。我们现在讨论的是世界空间也就是左手坐标系，而在观察坐标系中正前方是－z轴方向。

2.服务器使用右手坐标系，Unity使用左手坐标系，服务器0度朝向x正方向, 90度朝向z正方向. unity 0度朝向z正方向。

3.服务器顺时针旋转，从x+轴 转向 z+轴。  客户端也顺时针旋转，从z+轴 转向 x+轴。

4.服务器和客户端虽然使用的是不同坐标系，单我们维护的是同一套朝向机制。在各自的坐标系中，服务端的面向哪里，客户端也需要面向哪里。

也就是如果服务端此时面向自己坐标系的x+轴，那么客户端面向的也应是自己坐标系的x+轴。

5.公式：450° - 服务器朝向角度(0°~360°) = 客户端朝向角度

![](/images/posts/mmo_game/skill_scope/4.png)

如上图，服务端顺时针旋转90度到达z+轴，那么客户端顺时针需要旋转360°到达z+轴相当于原地不动。

![](/images/posts/mmo_game/skill_scope/5.png)

同理，如果服务端顺时针旋转270°到达z-轴， 那么客户端瞬时针旋转180°到达z-轴。

## 4.极坐标和笛卡尔坐标

极坐标就是角度和一条直线长度确定某点。

笛卡尔坐标就是直角坐标系，根据x和y确定某点。

### 1.极坐标转迪卡尔坐标

![](/images/posts/mmo_game/skill_scope/2.jpg)

```sh
使用余弦函数 x:	cos( 22.6°) = x/13
               x = 13×cos(22.6°)

　	　
使用正弦函数为Y: sin(22.6°) = y/13
             	y = 13 × sin( 22.6°)
```

## 5.网上一篇将矩形和扇形的博客还不错

https://lifan.tech/2020/01/21/game/aoe-skill-selector/

核心思想差不多，方法不唯一，我们项目在矩形上采用的是另外一种方法。

## 6.我们项目技能索敌

### 1.矩形

![](/images/posts/mmo_game/skill_scope/6.png)

```sh
1.前向矩形范围，人在矩形的宽边中央位置, 也就是上图的A点。

2.我们的技能AOE扫敌，不只是只能扫正前方的怪物，还可以有一定的向后延伸距离，
也就是B点所在的虚线框。所以初始点，就变成了B点。

3.C为被攻击者。

4.以初始点B为参照，最远的长度其实就是BE,也就是从宽的中央B点到矩形对角点E。

5.A的朝向是A点为轴，建立坐标系，如图我们将A的朝向角度往下平移，就是图中的θ角。

6.计算出BD的长度h, CD的长度w, 判断h是否小于等于最大长度BE，w是否小于等于
宽的一半即BG长度，即可以判断受击点C是否在矩形内。
```

核心代码如下:
```java
/**
* 初始化
* @param unit 施法者自身
* @param width 宽
* @param height 高
* @param list 待加入对象的列表
*/
public RectVisionConsumer(BPObject unit, int width, int height, List<BPObject> list)
{
    this.list = list;

    //自身坐标和朝向
    this.x = unit.getX();
    this.z = unit.getZ();
    this.rotation = unit.getRotation();

    MapUtils.polar(DictGameConfig.getVisionBackLength(), this.rotation, re);

    //向后的x和z坐标
    rx = this.x - re[0];
    rz = this.z - re[1];

    //宽度一半
    this.halfWidth = width / 2;

    this.height = height + DictGameConfig.getVisionBackLength();

    dsq = (long) width * (long) width / 4 + (long) this.height * (long) this.height;

    if (unit instanceof AbstractCharacter)
    {
        AbstractCharacter character = (AbstractCharacter) unit;
        if (character.getSkillModule().isUsingSkill() && character.getFightModule().isFightStatus())
        {
            isFightScan = true;
        }
    }
}

@Override
public void accept(BPObject unit)
{
    int ux = unit.getX();
    int uy = unit.getZ();

    float distanceSQ = MapUtils.distanceSqBetweenPos(rx, rz, ux, uy);
    float beHitDistanceSQ = distanceSQ;
    float distance = -1;

    //被攻击者距离（rx, rz）
    float beHitDistance = -1;

    if (isFightScan)
    {
        if (unit.isCloseBeHit())
        {
            return;
        }
        distance = (float) Math.sqrt(distanceSQ);
        beHitDistance = unit.getBeHitDistance(distance);
        beHitDistanceSQ = beHitDistance * beHitDistance;
    }

    if (beHitDistanceSQ <= dsq)
    {
        if(beHitDistance < 0)
        {
            beHitDistance = (float) Math.sqrt(beHitDistanceSQ);
        }

        int rotationA = this.rotation - MapUtils.rotationBetweenPos(rx, rz, ux, uy);

        //角度转为弧度
        double radiansA = Math.toRadians(rotationA);

        double h = Math.cos(radiansA) * beHitDistance;
        double w = Math.sin(radiansA) * beHitDistance;

        if (Math.abs(w) <= halfWidth)
        {
            if (h >= 0 && h <= height)
            {
                list.add(unit);
            }
        }
    }
}

```

### 2.扇形
