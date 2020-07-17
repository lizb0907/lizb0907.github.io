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

![](/images/posts/mmo_game/fight-area-01.jpg)

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

```sh
<Character extends AbstractCharacter>

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

### 3.onAddBpObject添加对象进场景

#### 1.比武场景抽象基类添加对象

AbstractBPScene:
```java
// 将对象加入场景
private void doAddBPObjectToScene(BPObject bpObject)
{
    ...
    // 加入场景事件
    onAddBpObject(bpObject);
    ...
}
```

AbstractBPCopySceneFight比武场景抽象基类添加对象
```java
@Override
protected void onAddBpObject(BPObject bpObject)
{
     if (bpObject.getObjectType() == ObjectTypeEnum.ACTOR)
     {
        if (isEnd())
        {
            exitCopyScene(actor);
            BPLog.BP_SCENE.error("【竞技场错误】副本已经进入结束状态了，不能进入玩家了 [sceneID] {} [line] {} [actorID] {} [objectID] {}", getSceneID(), getLineID(), actor.getActorID(), bpObject.getObjectID());
            return;
        }

        if (isStarted())
        {
            exitCopyScene(actor);
            BPLog.BP_SCENE.error("【竞技场错误】竞技场已经开始战斗了，不能进入玩家了 [sceneID] {} [line] {} [actorID] {} [objectID] {}", getSceneID(), getLineID(), actor.getActorID(), bpObject.getObjectID());
            return;
        }

        if (this.arenarObjectIdSet.size() >= MAX_ARENAR_COUNT)
        {
            exitCopyScene(actor);
            BPLog.BP_SCENE.error("【竞技场错误】竞技场已经进入两个玩家以上了，不能再进入玩家了 [sceneID] {} [line] {} [actorID] {} [objectID] {}", getSceneID(), getLineID(), actor.getActorID(), bpObject.getObjectID());
            return;
        }

        // 设置挑战者和被挑战者
        onActorEnter(actor);

        // 初始化侠客评价信息
        initKnightAppraise(actor);
     }
}
```
判断能不能进人，

设置挑战者和被挑战者，基类会给玩家派发一个buff（类似无敌，防止人没进来被打死），其余逻辑子类各自实现。

初始化侠客评价信息。


抽象的竞技场场景AbstractBPCopySceneFightArena添加对象处理：
```java
@Override
protected void onAddBpObject(BPObject bpObject)
{
    ...
    if (bpObject.getObjectType() == ObjectTypeEnum.ACTOR)
    {
        // 记录战斗前的侠客数据(两侠客上场前的血量和蓝量...)
        recordActorKnight(actor);

        // 进来就补满侠客血蓝、清怒气
        fillHpMp(actor);
    }
    ...
    //进入的竞技者的objectId(玩家，镜像；不包括宠物和傀儡等召唤物)
    if (bpObject.getObjectType() == ObjectTypeEnum.ACTOR || bpObject.getObjectType() == ObjectTypeEnum.MIRROR)
    {
        this.arenarObjectIdSet.add(bpObject.getObjectID());
    }
    
}
```
记录战斗前的侠客数据(两侠客上场前的血量和蓝量...)

进来就补满侠客血蓝、清怒气

进入的竞技者的objectId(玩家，镜像；不包括宠物和傀儡等召唤物),添加到arenarObjectIdSet集合，

会根据arenarObjectIdSet集合长度判断是否从等待阶段进入开始阶段。



```java
/**
* 记录侠客信息
*
* @param actor 玩家
*/
public void recordActorKnight(Actor actor)
{
    ...
    actorCopySceneKnightData.recordInfo(actor, knightIds);
    ...
}
```

```java
/**
* 记录信息
*
* @param actor
* @param knightIds
*/
public void recordInfo(Actor actor, int[] knightIds)
{
    ...
    ...
    //(当前血、蓝、怒气，索引值)
    int attrIndex = ATTRIBUTE_ARRAY[j];

    TIntIntHashMap attrMap = knightAttrMap.get(knightId);
    if (null == attrMap)
    {
        attrMap = new TIntIntHashMap(ATTRIBUTE_ARRAY.length, 1.0f);
        knightAttrMap.put(knightId, attrMap);
    }

    //根据索引，直接获取值
    int attrValue = knight.getAttributeLogic().getAttribute(attrIndex);
    //添加到map
    attrMap.put(attrIndex, attrValue);
    ...
    ...

}
```
记录当前血、蓝、怒气

#### 2.比武小游戏设置挑战者和被挑战者

这个名字没有起特别好，

```java
BPCopySceneFightArenaGame：
@Override
public void onActorEnter(Actor actor)
{
    ...
    ...
     // 设置挑战者
    setChallengerActor(actor);
    //设置竞技场战斗场景挑战者阵营
    actor.getHumanCommonModule().changeForce(DictGameConfig.getArenaScenechallengerForceID());

    //打镜像,一些逻辑会根据是否是镜像做处理
    setMirrorFight(true);
    ...
    ...
    //初始化镜像数据
    Mirror mirror = new Mirror(mirrorData);
    mirror.init();

    //镜像出生位置和朝向也是配置在MiniGameFightDifficulty表
    int[] knightBornPosStrArr = config.getKnightBornPosStrArr();
    mirror.setPos(knightBornPosStrArr[0], knightBornPosStrArr[1], knightBornPosStrArr[2]);
    mirror.setRotation(config.getKnightRotation());

    //添加镜像到场景
    addBPObject(mirror);

    // 设置镜像为被挑战者
    setBeChallengerHuman(mirror);
    //改变阵营
    mirror.getHumanCommonModule().changeForce(DictGameConfig.getArenaScenecBeInviterForceID());
}
```
设置玩家为挑战者更改阵营，

从小游戏模块拿到要挑战的机器人镜像数据，初始化镜像数据，添加机器人到场景...

#### 3.初始化镜像数据mirror.init()

Mirror.java:

```java
@Override
public void init()
{
    ...
    ...
    //初始化镜像AI模块
    aiModule.init();
    //初始化侠客模块
    knightModule.init();
    ...
    ...
    //镜像拥有侠客，侠客初始化
    for (int i = 0, length = mirrorData.getKnights().length; i < length; i++)
    {
        //设置侠客的主人为镜像
        Knight knight = new Knight(this);
        knight.setDictKnight(dictKnight);
        knight.init();
        knight.setFlowStatus(KnightFlowStatusEnum.UNLOCK);
        //战力
        knight.setFightForce(mirrorDataKnight.getFightForce());
        ...
         ////////////////// 初始化属性
        //是否初始化基础属性(侠客未初始化基础属性时改变属性会有问题)
        knight.getAttributeLogic().setInitBaseAttr(true);
        //获取战斗属性模块
        CharacterAttributeUseModule attributeLogic = knight.getAttributeLogic();
        attributeLogic.setFightVocation(knight.getFightVocation());
        if (mirrorData.isRobot())
        {
            //镜像如果是机器人需要设置星级, 添加进阵容基础属性变化
            int star = mirrorDataKnight.getStar();
            int state = mirrorDataKnight.getState();
            knight.getKnightCommonModule().setLevel(mirrorData.getLevel());
            knight.getKnightCommonModule().setStar(star);
            knight.getKnightCommonModule().setKnightState(state);
            DictKnightStarUp dictKnightStarUp = DictKnightStarUpData.getDictKnightStarUpData(mirrorDataKnight.getKnightId(), state, star);
            if (dictKnightStarUp == null)
            {
                BPLog.BP_LOGIC.error("[Mirror] init error dictKnightStarUp error, knightId:{} state:{} star:{}",
                        mirrorDataKnight.getKnightId(), mirrorDataKnight.getState(), mirrorDataKnight.getStar());
                return;
            }
            //设置星级配置
            knight.setDictKnightStarUp(dictKnightStarUp);
            //当被添加进阵容的时候, 基本属性的变化
            knight.baseAttrOnAddToOwner();
            //加一组属性
            attributeLogic.addAttributes(mirrorDataKnight.getAttributes());
        }
        else
        {
            //不是机器人直接set属性组
            attributeLogic.setAttributes(mirrorDataKnight.getAttributes());
        }
        ...
        // 补满血和蓝
        attributeLogic.fillHpMp();
        //计算并刷新推送属性组
        attributeLogic.refreshAttributes();
        ...

        ////////////////// 初始化技能

        // 把技能加到侠客身上
        List<BPSkill> skills = mirrorDataKnight.getDefenceSkills();
        for (int index = 0, size = skills.size(); index < size; index++)
        {
            BPSkill skill = skills.get(index);

            BPSkill bpSkill = skill.copy();

            knight.getSkillModule().initialAddSkill(bpSkill);
        }

        // 防守技能里没有普攻和怒气技
        TIntArrayList defenceSkillIds = new TIntArrayList();
        TIntIntHashMap slotSkills = DictSkillGroupData.getKnightSlotSkills(knightId);
        if (slotSkills != null)
        {
            for (int index = 1; index < GameConstant.SKILL_CONFIG_NUM; index++)
            {
                int skillId = slotSkills.get(index);
                if (skillId <= 0)
                {
                    break;
                }

                if (knight.getSkillModule().hasLearningSkill(skillId))
                {
                    defenceSkillIds.add(skillId);
                }
            }
        }

        //当前侠客的防守技能(没有普攻和大招)
        knightDefenceSkillIds.put(knightId, defenceSkillIds);

        ////////////////// 初始化buff
        ArrayList<AbstractSkillImpact> buffList = mirrorDataKnight.getBuffList();
        if(!CollectionUtils.isBlank(buffList))
        {
            ArrayList<AbstractSkillImpact> buffListCopy = new ArrayList<>(buffList.size());

            for (int j = 0,jSize = buffList.size();j < jSize; j++)
            {
                AbstractSkillImpact buff = buffList.get(j);
                buffListCopy.add(buff.makeExtendsImpact(this));
            }
            knight.getSkillModule().addInitialImpacts(buffListCopy);
        }

        knightModule.getOwnKnightMap().put(knightId, knight);
    }
    
}
```

初始化镜像AI模块

镜像拥有侠客，侠客初始化:初始化属性,补满血和蓝,计算并刷新推送属性组,初始化技能,初始化buff


### 4.比武场景tick逻辑处理

#### 1.tick场景逻辑

// tick场景逻辑
tickLogic(interval);

```java
@Override
protected void tickLogic(int interval)
{
    super.tickLogic(interval);

    if (!isEnd())
    {
        tickRunState(interval);
    }
}
```
tick场景逻辑下，调用tickRunState比武场景逻辑

#### 2.tickInit初始化

```java
/**
* 状态tick : 初始化
*
* @param interval 间隔
*/
private void tickInit(int interval)
{
    changeState(RUN_STATE_WAITING);
}
```
直接改变状态为，等待进入

同时进入下一个状态前，设置副本等待双方进入时间waitingRemainTime

#### 3.tick等待

```java
/**
* 状态tick : 等待
*
* @param interval 间隔
*/
private void tickWaiting(int interval)
{
    // 进入的竞技者超过2个，说明人齐了
    if (this.arenarObjectIdSet.size() >= 2)
    {
        //切换状态为-开始
        changeState(RUN_STATE_START);
        return;
    }

    // 超时没进入就算平局结束
    waitingRemainTime -= interval;
    if (waitingRemainTime < 0)
    {
        BPLog.BP_SCENE.warn("【竞技场】等待玩家进入超时 [serviceID] {} [lineId] {}", getServiceID(), getLineID());
        changeState(RUN_STATE_END);
    }
}
```
进入的竞技者超过2个，切换状态为-开始

超时没进入就算平局结束，状态切为结束

#### 4.tick开始

```java
/**
* 状态tick : 开始
*
* @param interval 间隔
*/
private void tickStart(int interval)
{
    //扣除剩余时间
    tickStart0(interval);

    //判断切断到ready的剩余时间是否为0
    //玩家如果提前loading完，会将剩余时间提前设置为0
    //如果在剩余时间30秒内，还没loading完，服务自动开始，玩家会被提前攻击
    if (canChangeReadyState())
    {
        changeState(RUN_STATE_READY);
    }
}
```
判断切断到ready的剩余时间是否为0

玩家如果提前loading完，会将剩余时间提前设置为0，

BPCopySceneFightArenaGame：
```java
@Override
public void onClientEnterSceneLoadingEnd(Actor actor)
{
    super.onClientEnterSceneLoadingEnd(actor);

    if (actor.getActorID() != getChallengerActorId())
    {
        BPLog.BP_SCENE.warn("[比武小游戏] onClientEnterSceneLoadingEnd actorID:{} != challengerActorId:{}", actor.getActorID(), getChallengerActorId());
        return;
    }

    // 可以进入ready状态了
    this.remainChangeReadyTime = 0;
}
设置允许进入ready状态的标识
```

如果在剩余时间30秒内，还没loading完，服务自动开始，玩家会被提前攻击

#### 5.tick准备阶段

```java
/**
* 状态tick : 准备
*
* @param interval 间隔
*/
private void tickReady(int interval)
{
    // 胜负分出后去结算，准备时候也可能对方退出了
    if (winnerActorId > 0)
    {
        //切结算
        changeState(RUN_STATE_DO_ACCOUNT);
        return;
    }

    // 准备倒计时结束
    remainTimeReady -= interval;
    if (remainTimeReady < 0)
    {
        // 状态迁移
        changeState(RUN_STATE_FIGHT);
    }
}
```
准备时候也对方退出，直接切结算

准备倒计时结束，切战斗，同时进入战斗需要设置战斗时间并推送客户端

#### 5.tick战斗

```java
/**
* 状态tick : 战斗
*
* @param interval
*/
private void tickFight(int interval)
{
    //中途提前退出，或者某方提前死亡，那么获胜者id大于0
    if (winnerActorId > 0)
    {
        //切为结算
        changeState(RUN_STATE_DO_ACCOUNT);
        return;
    }

    //累加战斗时间
    fightTime += interval;
    //判断战斗是否超时，超时切为结算状态
    tickFightTimeout(interval);
}
```
中途提前退出，或者某方提前死亡，那么获胜者id大于0

判断战斗是否超时，超时切为结算状态


```java
/**
* 状态进入 : 战斗
*/
protected void onEnterStateFight()
{
    //设置战斗时间
    remainFightTime = (int) (DictGameConfig.getArenaSceneAllTime() * TimeUtils.MILLIS_PER_SECOND);
    ...

    //去掉之前上的无敌buff
    actor.getSkillModule().finishBuffByImpactID(DictGameConfig.getArenaFightBeforeBuff());

}
```
设置战斗时间,去掉之前上的无敌buff

#### 6.判断某方胜利

AbstractBPCopySceneFight：
```java
/**
* 玩家离开设置结算
*/
protected void toDoAccountOnActorLeave(Actor actor)
```
离开场景会调用toDoAccountOnActorLeave()

```java
@Override
public void onMirrorDeath(int objectID)
{
    super.onMirrorDeath(objectID);
}
```
侠客死亡

```java
/**
* 战斗时间结束设置结算
*/
@Override
protected void toDoAccountOnFightTimeOut()
{
    if (isMirrorFight)
    {
        // 如果是打镜像，挑战者就输了
        this.winnerActorId = this.getBeChallengerActorId();
    }
    else
    {
        // 两个玩家战斗，伤害量高的获胜
        int challengerDamage = damageMap.get(this.getChallengerObjectId());
        int beChallengerDamage = damageMap.get(this.getBeChallengerObjectId());
        this.winnerActorId = challengerDamage > beChallengerDamage ? this.getChallengerActorId() : this.getBeChallengerActorId();
    }

    super.toDoAccountOnFightTimeOut();
}
```
战斗时间超时，根据伤害量判断获胜

#### 7.tick结算

```java
/**
* 结算处理
*/
protected void doAccount()
{
    ...
    //当前血量
    int hp = actor.getAttribute(AttributeTypeEnum.HP);
    //最大血量
    int hpMax = actor.getAttribute(AttributeTypeEnum.HP_MAX);
    ...
    //该侠客在战斗中损失血量低于30%
    knightAppraiseInfo.check(ArenaAppraiseTypeEnum.TYPE_2, (hpMax - hp) * GameConstant.HUNDRED / hpMax, 0, 0)
}
```
判断该侠客在战斗中损失血量低于30

```java
/**
* 状态进入 : 结算
*/
@Override
protected void onEnterDoAccount()
{
    // 停止镜像的AI
    if (isMirrorFight)
    {
        Mirror mirror = getMirror();
        if (mirror != null)
        {
            // 停止移动
            mirror.getMoveModule().stopMove();

            // 停止技能
            mirror.getSkillModule().breakSkill();

            // 停止AI
            mirror.getMirrorAiModule().inActiveAI();
        }
    }
}
```
如果是打镜像，进入结算前需要停止镜像Ai

#### 8.tick结束

```java
/**
* 状态进入 : 结束
*/
protected void onEnterStateEnd()
{
    // 设置父状态机(等待从场景中移除玩家)
    setParentState(CopySceneStateType.WAITING_REMOVE);

    // 调用父状态机的结束逻辑
    doEnd();

    // 发送给客户端清人
    TIntObjectHashMap<Actor> actors = getActors();
    for (TIntObjectIterator<Actor> iterator1 = actors.iterator(); iterator1.hasNext(); )
    {
        iterator1.advance();

        Actor actor = iterator1.value();
        //主动调用的脱战
        actor.getFightModule().secedeFight();
        //发送剩余时间消息
        sendTimeLeft(iterator1.value(), TIME_NOTICE_TYPE_END, getOverTimeOut());
    }
}
```
设置副本状态等待从场景中移除玩家

调用副本结束

发送给客户端清人


### 5.比武结束恢复蓝血...

AbstractBPCopySceneFightArena：
```java
@Override
protected void onLeaveScene(BPObject bpObject)
{
     // 死亡就复活
    if (!actor.getActorCommonModule().isAlive())
    {
        actor.revive();
    }
    //恢复侠客信息
    restoreActorKnight(actor);
    //回归默认势力
    actor.getActorCommonModule().changeToDefaultForce();
    
}
```
死亡就复活，恢复侠客信息，回归默认势力

```java
/**
* 恢复信息
*
* @param actor
*/
public void restoreInfo(Actor actor)
{
    ...
    // 恢复进来的时候的属性
    TIntObjectIterator<TIntIntHashMap> iterator = knightAttrMap.iterator();
    while (iterator.hasNext())
    {
        ...
        ...
        TIntIntHashMap attrMap = iterator.value();
        int hp = attrMap.get(AttributeTypeEnum.HP.getIndex());
        //如果死亡了先复活
        if (knight.isDead() && hp > 0)
        {
            knight.revive();
        }

        //直接set回之前的属性值
        TIntIntIterator attrIterator = attrMap.iterator();
        while (attrIterator.hasNext())
        {
            attrIterator.advance();

            knight.getAttributeLogic().setOneAttribute(attrIterator.key(), attrIterator.value());
        }

        //如果进来之前侠客是死亡的，那么继续变成死亡
        if (hp <= 0)
        {
            // 恢复死亡的侠客
            knight.getAttributeLogic().decreaseHP(actor.getObjectID(), Integer.MAX_VALUE, -1);
        }
        ...
        ...
    }

    if (knightModule.getCurrentKnight().isDead())
    {
        knightModule.swapWorkingAndBackupExternal(KnightSwapTypeEnum.FORCE);
    }

}
```
如果死亡了先复活

直接set回之前的属性值

如果进来之前侠客是死亡的，那么继续变成死亡