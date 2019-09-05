---
layout: post
title: 传送
categories: Mmo-Game
description: mmo游戏里传送设计一些记录
keywords: transfer，mmo，game
---

在游戏里传送模块和各个模块都结合的比较紧密，我们有必要认真学习和思考下。

**目录**

* TOC
{:toc}

## 服务端切换场景一些状态标识

```java
/**
 * 玩家对象切换场景,在当前场景的状态枚举,此状态只是针对当前场景而言
 * 进入的新场景状态不需要维护,是通过transfer module的一个boolean值来控制何时加入新场景tick
 * Created by wangqiang on 2017/10/23.
 */
public enum SwitchSceneStatusEnum
{
    NONE(), // 正常状态
    PRE_ENTER_SCENE(), // 预进入状态,客户端刚刚发起了切换场景请求,等待world返回预进入消息
    SERVER_SWITCHING(), // 服务器内部切换场景中
    LEAVING_SCENE(), // 正在离开当前场景
    WAIT_CLIENT_READY(), // 等待客户端发送ready
}
```
切场景每个步骤会对应某个状态标识，依据状态标识会做不同的处理。


## 简单看下具体流程,我们以前端发起切场景请求为例

### 1.前端发起切场景，传入参数

```java
// 进入场景请求
message CSEnterSceneRequest{
	required int32 sceneID = 1; // 场景id
	required int32 enterType = 2; // 进入场景的类型：0：手动切线或登录进入，1：传送点切线，2：NPC传送切线,3:gm传送（后端用）,4:点击小地图传送点传送
	optional int32 line = 3; //场景线, 如果需要自动切线的,则不填写此值
	optional int32 transferID = 4; // 如果是传送点切线,需要将此值设置成SceneAreaTransfer表对应的id
	optional int32 transferParam = 5; // 传送参数 默认不传值 1：双人坐骑不管乘客能否传送，司机一定过去
//	optional int32 guildId = 6; // 要进入的帮派id (本帮派id 或者 做任务的帮派id)
}
```

```java
/**
* 处理客户端的传送场景消息
*
* @param sceneID 场景id
* @param lineID 线id
* @param transferType 传送类型
* @param transferID 传送id
* @param transferParam 传送参数
* @param guildId 帮派id
*/
public void clientTransferScene(int sceneID, int lineID, int transferType, int transferID, int transferParam, int guildId)
{
    ...
    ...
    int sceneType = config.getSceneType();
    if(sceneType == SceneTypeConst.WAR_REST_ROOM || SceneTypeConst.isCopyScene(sceneType))
    {
        //客户端不能主动进入
        actor.sendError(BPErrorCodeEnum.SCENE_TRANSFER_IS_BATTLE);
        return;
    }

    SceneTransferTypeEnum typeEnum = SceneTransferTypeEnum.valueOf(transferType);
    int result = actor.getTransferModule().preSwitchSceneRequest(typeEnum, sceneID, lineID, transferID, transferParam, guildId);
    ...
    ...
}
```
clientTransferScene()方法会先校验一下预进入的场景是否可以客户端主动进入，战场休息室、副本客户端不能主动进入。

然后调用preSwitchSceneRequest（）方法，进入预切换场景状态。

### 2.预切换场景

```java
/**
* 预切换场景
* @param transferType
* @param sceneID 目标场景
* @param lineID
* @param transferID 如果是npc传送或者踩点传送,此值不为-1
* @param isTeam 是否是组队副本 （双人坐骑传送也设置为true，如果这个字段有了别的用途，需要注意坐骑）
* @param transferParam 传送参数：1：双人坐骑强制传送
* @param guildId 帮派id
* @param isForceTransfer 是否强制传送，目前只给帮派驻地清人使用
*/
private int preSwitchSceneRequest(SceneTransferTypeEnum transferType, int sceneID, int lineID, int transferID, 
                                    boolean isTeam, int transferParam, int guildId, boolean isForceTransfer, boolean checkOnline)
{
    ...
    ...
    各种检查，例如是否需要读条...
    ...
    ...
    //预进入状态,客户端刚刚发起了切换场景请求,等待world返回预进入消息
	setSwitchSceneStatus(SwitchSceneStatusEnum.PRE_ENTER_SCENE);
    ...
    ...
    //发送世界进行预进入场景请求处理
    service.sendMessage(ServiceManager.getInstance().getWorldService(), preEnterSceneRequest);
    ...
}                                   
```
各种检查，检查通过后设置当前状态标识为PRE_ENTER_SCENE(), 预进入状态,客户端刚刚发起了切换场景请求,等待world返回预进入消息


### 3.等待world返回预进入消息

```java
@MessageHandler(MessageIDConst.PRE_ENTER_SCENE_REQUEST)
public void handlePreEnterSceneRequest(BPWorldService service, BPPreEnterSceneRequest message)
{
    ...
    ...
    //预进入场景请求世界处理
    int lineID = sceneManager.preEnterSceneRequest(message.getDstSceneID(), message.getDstLineID(),message.getActorID(), message.isTeam()
    ...
    ...

    //将预进入的玩家加入到预进入玩家映射数据中
    sceneManager.addPreEnterMap(actorID, sceneID, lineID);
    ...

    //返回预进入消息到场景
    service.sendMessage(message.getSrcService(), response)
    ...
    ...
}
```
预进入场景请求世界处理preEnterSceneRequest（）方法，最主要做的是先把人加上,把位置占上   --------场景当前总人数增加。

将预进入的玩家加入到预进入玩家映射数据中，然后返回场景。

### 4.预进入场景回复

```java
/**
* 预切换场景回复
* 此时只是将用户在场景中占位,并没有真正进入
* @param message
*/
public void preSwitchSceneResponse(BPPreEnterSceneResponse message)
{
    ...
    ...
    //预切换场景完成的回调
    result = preSwitchSceneCallback(result, sceneID, lineID, message.isForceTransfer());
    if (result >= 0)
    {
        // 设置为等待客户端load
        setSwitchSceneStatus(SwitchSceneStatusEnum.SERVER_SWITCHING);

        onPreEnterSceneSuccess(sceneID,lineID);

        BPLog.BP_SCENE.info("actor:{} received BPPreEnterSceneResponse, dstScene:{}, line:{}",
                actor.getActorID(), sceneID, lineID);
    }
    else
    {
        ...
        //预进入失败，重置状态
        setSwitchSceneStatus(SwitchSceneStatusEnum.NONE);
        ...
        //发送客户端失败
        actor.sendPacket(PacketIDConst.SCEnterSceneResponse, response.build());
    }
    ...
    ...
}
```
预进入场景回复,这里会做一些事件回调。

如果回调成功，那么设置状态标识为SERVER_SWITCHING服务器内部切换场景中，然后服务器开始根据
状态标识进行tick处理。

如果回调失败，重置状态为NONE正常状态，并通知客户端进入场景失败。

### 5.tick处理状态标识为SERVER_SWITCHING服务器内部切换场景

```java
@Override
public void tick(int interval)
{
    if (switchSceneStatus == SwitchSceneStatusEnum.SERVER_SWITCHING)
    {
        clientStartLoading();
    }
}
```

```java
public void clientStartLoading(int sceneID, int lineID)
{
    ...
    ...
    if(actor.getCopySceneModule().nextIsSpaceScene())
    {
        //下个是位面场景
        actor.getCopySceneModule().enterSpace();
    }
    else if(actor.getCopySceneModule().isSceneBacking())
    {
        //退出位面
        actor.getCopySceneModule().exitSpace();
    }
    else
    {
        //正式切换场景,玩家会后续的离开当前场景,进入新场景
        actor.getTransferModule().switchSceneRequest(sceneID, lineID);
    }
    ...
}
```
进位面、退出位面、切其它场景

```java
/**
* 正式切换场景,玩家会后续的离开当前场景,进入新场景
* @param sceneID
* @param lineID
*/
public void switchSceneRequest(int sceneID, int lineID)
{
    ...
    ...
    else
    {
        // 正常切换场景时
        request.setSrcLineID(scene.getLineID());
        request.setSrcSceneID(scene.getSceneID());
        service = scene;
        // 从场景中移除aoi,但是将对象保留在场景中
        actor.deleteSafe();
        transferModule.setSwitchingScene(true);
        transferModule.setSrcScene(scene);
        transferModule.setSwitchSceneStatus(SwitchSceneStatusEnum.LEAVING_SCENE);

        BPLog.BP_SCENE.info("actor:{} load scene ready, dst:{}, line:{}, src:{}, line:{}",
                actor.getActorID(), sceneID, lineID, scene.getSceneID(), scene.getLineID());
    }

    service.sendMessage(ServiceManager.getInstance().getWorldService(), request);
    ...
    ...
}
```
调用deleteSafe（）删除玩家，设置状态标识LEAVING_SCENE()--正在离开当前场景，发送世界处理正式进入场景请求。

### 6.世界线程处理正式进入场景请求

```java
@MessageHandler(MessageIDConst.ENTER_SCENE_REQUEST)
public void handleEnterSceneRequest(BPWorldService service, BPEnterSceneRequest message)
{
    ...
    ...
    //拿到场景管理器
    SceneManager sceneManager = service.getSceneManager();
    //请求进入场景
    int result = sceneManager.enterSceneRequest(dstSceneID, dstLineID, srcSceneID, srcLineID, object);
    ...
    ...

    service.sendMessage(message.getSrcService(), response);
    ...
}
```

```java
public int enterSceneRequest(int dstSceneID, int dstLineID, int srcSceneID, int srcLineID, Actor object)
{
    ...
    //根据目标场景拿到场景容器
    SceneContainer dstContainer = allSceneMap.get(dstSceneID);
    if (dstContainer == null)
    {
        BPLog.BP_SCENE.warn("no sceneID create, dst id:{}", dstSceneID);
        return BPErrorCodeEnum.SCENE_NO_SCENE_ID_BE_CREATED;
    }
    //玩家进入场景---判断是否激活，并加入场景待加入对象队列
    int result = dstContainer.playerEnterScene(dstLineID, object);

    ...

    //记录到真正的玩家的场景分线数据
    playerOnlineMap.put(object.getActorID(), getPlayerSceneIndentify(dstSceneID, dstLineID));
    //将预进入的玩家从预进入玩家映射数据中删除
    removePreEnterMap(object.getActorID());
    ...
}
```
世界线程处理正式进入场景请求,正式请求进入场景，判断场景是否激活，将玩家加入待加入对象队列。

将玩家记录到真正的玩家的场景分线数据，将预进入的玩家从预进入玩家映射数据中删除。

世界线程正式进入场景，下发正式进入场景回复。

### 7.正式进入场景回复处理

```java
@MessageHandler(MessageIDConst.ENTER_SCENE_RESPONSE)
public void handleEnterSceneResponse(AbstractBPScene service, BPEnterSceneResponse message)
{
    Actor actor = message.getObject();
    if (actor == null)
    {
        return;
    }
    actor = service.getActorByActorID(actor.getActorID());
    if (actor == null)
    {
        return;
    }

    actor.getTransferModule().switchSceneResponse(message);

    if (message.getResult() < 0)
    {
        BPLog.BP_SCENE.warn("actor:{} 切换场景 dst:{} 最后一步出错:{}", actor.getActorID(),
                message.getDstLineID(), message.getResult());
    }
    else
    {
        service.actorSwitchDone(actor.getActorID());
    }
}
```

#### 1.switchSceneResponse（）

```java
/**
* 正式进入场景回复
* @param message
*/
public void switchSceneResponse(BPEnterSceneResponse message)
{
    // 从当前场景退出  是否可以从源场景的切换场景容器中删除
    canRemoveFromOld = true;
    BPLog.BP_SCENE.info("actor:{} received EnterSceneResponse, dstScene:{}, line:{}",
            actor.getActorID(), sceneID, lineID);

    //设置使用传送技能结束
    useSkillOver = false;

    //旧场景中玩家身上的场景对象
    AbstractBPScene scene = actor.getScene();
    if (scene != null)
    {
        actor.getTransferModule().setSrcScene(null);
        scene.removeSwitchingActor(actor.getActorID());
        //切换场景时,源场景彻底删除用户后,通知新场景将玩家正式加入场景
        BPSwitchSceneLeaveNotice notice = new BPSwitchSceneLeaveNotice(MessageIDConst.SS_SWITCH_SCENE_LEAVE_NOTICE);
        notice.setActor(actor);
        scene.sendMessage(message.getScene(), notice);

    }
    else
    {
        BPLog.BP_SCENE.warn("玩家切换场景, 旧场景中玩家身上的场景对象为空");
    }
}
```

```java
@MessageHandler(MessageIDConst.SS_SWITCH_SCENE_LEAVE_NOTICE)
public void handleSwitchSceneLeaveNotice(AbstractBPScene service, BPSwitchSceneLeaveNotice message)
{
    Actor actor = message.getActor();

    //如果当前玩家在待加入对象队列里
    if (service.actorIsWaitingAdd(actor))
    {
        //是否在切换场景的过程中,此过程为源场景删除,目标场景待加入
        //当源场景完全删除以后,目标场景检测此值为false后才加入场景tick
        actor.getTransferModule().setSwitchingScene(false);
    }
}
```
设置为false，在场景基类tick时，就会开始将对象加入场景tick。

#### 2.actorSwitchDone（）

```java
/**
* 玩家完成切换,此方法在源场景调用,调用此方法时,其实玩家已经从actor map中移除了
* @param actorID
*/
public void actorSwitchDone(long actorID)
{
    ...
    if (!DictSceneDefineData.isGuildScene(sceneID) && !isSpaceScene())
    {
        // 通知world,需要销毁场景
        BPSceneEmptyNotice notice = new BPSceneEmptyNotice(MessageIDConst.SW_SCENE_EMPTY_NOTICE);
        notice.setSceneID(sceneID);
        notice.setLineID(lineID);
        sendServiceMessageToWorld(notice);
    }
    ...
}
```
判断场景人数，是否为空，空销毁场景。

### 8.场景基类tick对象

```java
private void tickActorWaittingAdd()
{
    ...
    ...
    //出队列
    Actor actor = this.actorWaittingAdd.poll();
    if (actor == null)
    {
        return;
    }

    ActorTransferModule transferModule = actor.getTransferModule();
    //目标场景检测此值为false才会往下走，否则从新加入队列
    if (transferModule.isSwitchingScene())
    {
        this.actorWaittingAdd.add(actor);
        continue;
    }
    else if (!actor.isLoginEnd())
    {
        this.actorWaittingAdd.add(actor);
        continue;
    }
    else
    {
        transferModule.setSrcScene(this);
        //设置状态WAIT_CLIENT_READY(), // 等待客户端发送ready
        transferModule.setSwitchSceneStatus(SwitchSceneStatusEnum.WAIT_CLIENT_READY);
    }

    //加入场景
    addBPObject(actor);
    ...
    ...
}

```
isSwitchingScene如果不为false，那么对象重新加入队列等待。


isSwitchingScene等于false，那么状态标识设置为设置状态WAIT_CLIENT_READY()--等待客户端发送ready，对象加入场景。

### 9.tick状态WAIT_CLIENT_READY()---等待客户端发送ready

```java
@PacketHandler(PacketIDConst.CSEnterSceneReady)
public void handleEnterSceneReady(BPLogic.CSEnterSceneReady message, Actor actor)
{
    ActorTransferModule transferModule = actor.getTransferModule();

    BPLogic.SCEnterSceneReady.Builder builder = BPLogic.SCEnterSceneReady.newBuilder();
    builder.setSceneID(message.getSceneID());
    builder.setLineID(message.getLineID());

    builder.setResult(0);
    actor.sendPacket(PacketIDConst.SCEnterSceneReady, builder.build());

    transferModule.setSwitchSceneStatus(SwitchSceneStatusEnum.NONE);
    transferModule.setClientLoadingSceneTime(0);

    AbstractBPScene scene = actor.getScene();
    if (scene != null)
    {
        scene.sendOtherPostionAndMovePath(actor);
    }
}
```

服务端等待前端发送进入场景,准备完毕请求。也就是现在机制是服务端不管客户端是否loading完毕，服务端是先进入场景的。

ActorTransferModule类下，tick状态WAIT_CLIENT_READY()---等待客户端发送ready。

接收到到CSEnterSceneReady协议后，服务端将状态标识为NONE()-----正常状态,然后回复客户端服务端已经准备完毕SCEnterSceneReady。


## 设置传送出生点最终调用（结合需求：出生点为中心随机半径范围内出生）

```java
private void tickActorWaittingAdd()
{
    ...
    ...
    ...
    // 给前端发送消息
    BPLogic.SCEnterSceneResponse.Builder response = BPLogic.SCEnterSceneResponse.newBuilder();
    response.setResult(0);
    response.setSceneID(sceneID);
    response.setLine(lineID);

    //出生点为中心随机
    int switchPosX = transferModule.getSwitchPosX();
    int switchPosZ = transferModule.getSwitchPosZ();
    if (getDictSceneDefine() != null && getDictSceneDefine().getBirthRandomRadius() > 0)
    {
        int radius = getDictSceneDefine().getBirthRandomRadius();
        int minX = switchPosX - radius;
        int maxX = switchPosX + radius;
        switchPosX = MathUtils.randomRange(minX, maxX);
        if (switchPosX < 0)
        {
            switchPosX = 0;
        }
        else if (switchPosX > aoiModule.getWidth())
        {
            switchPosX = aoiModule.getWidth();
        }

        int minZ = switchPosZ - radius;
        int MaxZ = switchPosZ + radius;
        switchPosZ = MathUtils.randomRange(minZ, MaxZ);
        if (switchPosZ < 0)
        {
            switchPosZ = 0;
        }
        else if (switchPosZ > aoiModule.getHeight())
        {
            switchPosZ = aoiModule.getHeight();
        }

        transferModule.setSwitchPos(switchPosX, transferModule.getSwitchPosY(), switchPosZ);
    }

    response.setPosX(switchPosX);
    response.setPosZ(switchPosZ);
    actor.sendPacket(PacketIDConst.SCEnterSceneResponse, response.build());
    ...
    ...
}
```

现在服务端各个模块设置出生点，没有整合成一个通用的事件设置，每个模块走自己逻辑导致想在上层实现出生点为中心随机半径范围内出生需求很难。

所以，在最终调用方法里实现。
