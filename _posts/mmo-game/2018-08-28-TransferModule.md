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
    //返回预进入消息到场景
    service.sendMessage(message.getSrcService(), response)
    ...
    ...
}
```
预进入场景请求世界处理preEnterSceneRequest（）方法，最主要做的是先把人加上,把位置占上   --------场景当前总人数增加。

然后返回场景。