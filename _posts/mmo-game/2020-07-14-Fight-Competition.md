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

发送响应客户端,进副本。

## 比武流程

### 1.比武场景类的继承关系

![](/images/posts/mmo_game/fight_area/01.jpg)

1.最上层的抽象的比武场景
```java
/**
 * 抽象的比武场景，处理玩家与玩家/镜像/NPC的战斗流程
 * <pre>
 *     场景名称：抽象的比武场景
 *
 *     场景说明：0.初始化: 不做什么实质性工作，只是状态机的第一环，代表状态
 *               1.等待：  等待双方入场，倒计时，记录玩家入场前的血量
 *               2.开始:   补满血量，去除对应的BUFF
 *               3.准备:   倒计时
 *               4.战斗:   双方进入战斗区域战斗，侠客死亡后若备用侠客活着则派出，否则判断胜负
 *               5.结算：  双方胜负结算
 *               6.结束：  结束副本，踢出玩家
 *               7.摧毁：  不做什么实质性的工作，只是状态机最后一环，主要用于标识状态
 * </pre>
 */
public abstract class AbstractBPCopySceneFight<Character extends AbstractCharacter> extends BPCopyScene
```
主要是所有子类公用的，记录流程（等待进入，倒计时，开始），一些统计。

<Character extends AbstractCharacter>

```sh
public void setBeChallengerCharacter(Character beChallengerCharacter)
{
    this.beChallengerCharacter = beChallengerCharacter;
}

setBeChallengerCharacter(Character beChallengerCharacter)设置被调战者，

传入对象必须是继承于生物基类AbstractCharacter。
```

2.处理玩家与玩家/镜像的战斗

```java
/**
 * 抽象的竞技场场景，处理玩家与玩家/镜像的战斗流程
 */
public abstract class AbstractBPCopySceneFightArena extends AbstractBPCopySceneFight<AbstractHuman>
```
处理玩家与玩家/镜像的战斗流程的抽象类，再抽出了一层，玩家与镜像都是AbstractHuman的子类。

3.NPC比武场景
```java
/**
 * NPC比武副本
 */
public class BPCopySceneFightNpc extends AbstractBPCopySceneFight<Npc>
```
只是与npc比武，所以extends AbstractBPCopySceneFight<Npc>，传入npc。

4.BPCopySceneFightArena竞技场副本
```java
/**
 * 比武竞技场副本
 *
 * <pre>
 *     场景名称：竞技场
 *
 *     场景描述：竞技场场景是参加竞技场挑战的挑战者和被挑战者共同参与的一个非组队一对一的PVP战斗场景。场景中没有NPC。
 *               挑战者可以携带宠物和傀儡进入。被挑战者如果是玩家可以携带宠物和傀儡进入；被挑战者是玩家镜像，那么镜像只可以在
 *               该场景内召唤傀儡，并且镜像没有宠物。双方战斗时侠客死亡后若还有备用侠客则暂停战斗，派出备用侠客继续战斗，
 *               直至一方侠客全部死亡/退出，则另一方胜利。战斗至场景计时结束，则比较双方伤害，输出伤害高的一方胜利。
 *               双方战斗结束并传送回去，则场景结束并回收。
 * </pre>
 */
public class BPCopySceneFightArena extends AbstractBPCopySceneFightArena
```
竞技场可以是玩家与玩家，玩家与镜像，继承自AbstractBPCopySceneFightArena

5.比武小游戏副本
```java
/**
 * 比武小游戏副本
 */
public class BPCopySceneFightArenaGame extends AbstractBPCopySceneFightArena
```
比武小游戏副本，玩家和镜像战斗，继承自AbstractBPCopySceneFightArena

### 2.初始化操作

AbstractBPCopySceneFight：

```java
@Override
public boolean init()
{
    boolean flag = super.init();
    if (!flag)
    {
        return false;
    }

    this.clear();

    return true;
}
```
初始化，没做什么复杂操作，只是清一下数据。子类，覆写类似。