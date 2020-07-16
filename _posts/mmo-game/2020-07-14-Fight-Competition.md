---
layout: post
title: 比武小游戏
categories: Mmo-Game
description: 比武小游戏学习记录
keywords: fight,competition
---

比武小游戏学习记录

**目录**

* TOC
{:toc}

## 切磋入口

玩家与NPC对话，若NPC配置的武艺等级>0级，则显示“武艺切蹉”按钮，点击按钮开始切磋，

类似竞技场。

## 世界上初始化比武小游戏机器人

### 1.世界init初始化

ApplicationBoot下：
```java
 // world服务 (初始化地图会用到dict数据，放在读dict后)
AbstractWorldService worldService = (AbstractWorldService) SpringContainer.getInstance().getBeanById("WorldService");
flag &= worldService.init();
CoreGlobals.getInstance().getProcessorPool().addService(worldService, 100);
ServiceManager.getInstance().setWorldService(worldService);
if(!flag)
{
    BPLog.BP_SYSTEM.error("配置加载错误");
    return -1;
}
```
调用worldService.init();

BPWorldService：

```java
@Override
public boolean init()
{
    worldMiniGameModule.init();
}
```
调用worldMiniGameModule.init();

```java
private void initMirrorData()
{
    ...
    //获取机器人
    RobotData robotData = getWorldService().getWorldRobotModule().createMiniGameFightRobot(dictMiniGameFightKnight.getRobotId());
    ...
    //机器人数据生成镜像数据
     MirrorData mirrorData = getWorldService().getWorldRobotModule().buildMirrorData(robotData);
    ...
     //生成比武小游戏机器人玩家id
     mirrorData.setActorID(getWorldService().getWorldRobotModule().buildRobotActorIdByMiniGame(dictMiniGameFightKnight.getId()));
    for (int level = dictMiniGameFightKnight.getActorMinLevel(), maxLevel = dictMiniGameFightKnight.getActorMaxLevel(); level <= maxLevel; level++)
    {
        actorLevel2MirrorDataMap.put(level, mirrorData);
    }
}
```
获取机器人。

机器人数据生成镜像数据:属性，战力，技能，等级，名字...组装成类似玩家。

生成比武小游戏机器人玩家id。


### 2.生成机器人

```java
/**
* 生成机器人
*
* @param robotId 机器人id
* @param robotType 机器人类型
*/
private RobotData createRobot(int robotId, RobotTypeEnum robotType)
{
    ...
    ...
    //主侠客对应的机器人id
    boolean initKnight = initRobotKnight(robotData, dictRobot.getRobotKnightId1(), 0);
    if (!initKnight)
    {
        return null;
    }
    //副侠客对应的机器人id
    initKnight = initRobotKnight(robotData, dictRobot.getRobotKnightId2(), 1);
    if (!initKnight)
    {
        return null;
    }
    ...
     robotData.setFightForce(sumFightForce);
    ...

}
```
主、副侠客，设置战力。

### 3.初始化机器人侠客


```java
/**
* 初始化机器人侠客
*
* @param robotData 机器人
*/
private boolean initRobotKnight(RobotData robotData, int robotKnightId, int index)
{
    ...
    //根据机器人侠客id获取机器人侠客数据
    DictRobotKnight dictRobotKnight = DictRobotKnight.getRecordById(robotKnightId);

     // 初始化属性
    boolean initAttr = initRobotKnightAttr(robotKnightData);

      // 初始化技能
    boolean initSkill = initRobotKnightSkill(robotKnightData);
    ...

    // 计算修为
     boolean initFightForce = initRobotKnightFightForce(robotKnightData, robotData.getDictRobot().getRobotLevel());
}
```
根据机器人侠客id获取机器人侠客数据

初始化属性,初始化技能,计算修为

```java
/**
* 初始化机器人侠客属性
*
* @param robotKnightData 机器人侠客
* @return
*/
private boolean initRobotKnightAttr(RobotKnightData robotKnightData)
{
    //RobotKnightAttr将处理后的属性表数据设置到RobotKnightData
}
```
RobotKnightAttr将处理后的属性表数据设置到RobotKnightData

RobotKnightAttData属性初始化的时候，根据反射将对应索引的值装进数组里，例如s_77 = 100,  int[77] = 100

```java
/**
* 初始化机器人侠客技能
*
* @param robotKnightData 机器人侠客
* @return
*/
private boolean initRobotKnightSkill(RobotKnightData robotKnightData)
{
    ...
    ...
    //根据侠客id 和 类型  获取技能配置列表
    ArrayList<DictRobotKnightSkillData> dataList = DictRobotKnightSkillData.getDataListByKnightIdAndType(knightId, type);
    ...
    ...
    TIntObjectHashMap<BPSkill> skills = robotKnightData.getSkills();
    for (int i = 0, size = dataList.size(); i < size; i++)
    {
        DictRobotKnightSkillData dictRobotKnightSkillData = dataList.get(i);

        BPSkill bpSkill = skills.get(dictRobotKnightSkillData.getSkillId());
        if (null == bpSkill)
        {
            bpSkill = new BPSkill(dictRobotKnightSkillData.getSkillId());
            skills.put(dictRobotKnightSkillData.getSkillId(), bpSkill);
        }
        //状态和状态等级 解锁技能
        bpSkill.initAddStateLevel(dictRobotKnightSkillData.getState(), dictRobotKnightSkillData.getLevel());
    }
}
```
根据侠客id和类型从RobotKnightSkill表里获取技能配置列表

根据状态和状态等级，解锁机器人侠客技能

```java
/**
* 初始化战力
*
* @param robotKnightData 机器人侠客
* @param level 等级
* @return
*/
private boolean initRobotKnightFightForce(RobotKnightData robotKnightData, int level)
{
    ...
    // 这个只是针对机器人的特殊侠客基础战斗力，和玩家侠客基础战力不同
    // 侠客基础战斗力=侠客战力基础值*对应星级的侠客星级战力系数*对应等级的侠客等级战力系数

    // 机器人战力 = 侠客基础战力 + 机器人侠客附加属性战力 + 机器人侠客附加战力
    ...

}
```

### 4.机器人数据生成镜像数据

```java
/**
/**
* 机器人数据生成镜像数据
*
* @param robotData 机器人数据
* @return
*/
public MirrorData buildMirrorData(RobotData robotData)
{
    ...
    MirrorData mirrorData = new MirrorData();
    mirrorData.init();
    mirrorData.setRobot(true);

    mirrorData.setLevel(robotData.getDictRobot().getRobotLevel());
    mirrorData.setName(robotData.getName());
    ...

}
```
名字，性别，等级，属性
    
## 世界返回机器人数据到场景

```java
@MessageHandler(MessageIDConst.WS_MINI_GAME_FIGHT_ROBOT_DATA_RESP)
public void handleBPWSMiniGameFightRobotDataResp(AbstractBPScene service, BPWSMiniGameFightRobotDataResp message)
{
    ...
    int result = actor.getGameModule().callbackStartMiniGameFight(message.getMirrorData());
    ...
}
```
world回调开始进副本


```java
/**
* world回调开始进副本
*
* @param actor 玩家
* @param mirrorData 机器人数据
* @return
*/
public int callbackStartMiniGameFight(Actor actor, MirrorData mirrorData)
{
    //添加额外属性,MiniGameFightDifficulty有额外的属性添加值
    //MirrorData镜像数据里包含镜像侠客数据
    //迭代机器人侠客数组长度(主，副)
    for (int i = 0, length = mirrorData.getKnights().length; i < length; i++)
    {
        MirrorKnightData mirrorDataKnight = mirrorData.getKnights()[i];
        if (mirrorDataKnight == null)
        {
            continue;
        }

        int[] attributes = mirrorDataKnight.getAttributes();

        //MiniGameFightDifficulty有额外的属性添加值
        DictMiniGameFightDifficultyData config = (DictMiniGameFightDifficultyData) this.config;
        if (config.getAttrs() == null)
        {
            continue;
        }

        for (int j = 0, size = config.getAttrs().length; j < size; j++)
        {
            //防止越界而已
            if (j >= attributes.length)
            {
                break;
            }

            //有额外的值
            int attrValue = config.getAttrs()[j];
            if (attrValue != 0)
            {
                attrValue += attributes[j];
                if (attrValue < 0)
                {
                    attrValue = 0;
                }
                //镜像侠客数据设置最终值
                mirrorDataKnight.getAttributes()[j] = attrValue;
            }
        }

        ...

        //发送响应客户端
        int[] knightIds = new int[KnightConstant.MAX_WORKING_COUNT];
        //把当前出站侠客拷贝到数组中，不能直接拿formationArray，防止外面误改
        actor.getKnightModule().copyFormationKnight(knightIds);
        actor.getArenaModule().sendEnterArenaSuccess(knightIds, this.mirrorData.getName(), targetKnightIds);
        //进副本
        boolean result = actor.getCopySceneModule().applyEnter(config.getCopySceneId());
    }
    
}
```
添加额外属性,MiniGameFightDifficulty有额外的属性添加值

镜像侠客数据需要加上额外的属性添加值。

为什么在这里添加额外属性呢？
```sh
猜测是，因为不同的难度表，机器人厉害程度不一样，所以需要通过额外添加属性值控制。

因此，在MiniGameFightDifficulty有额外的属性添加值。
```

发送响应客户端,进副本