---
layout: post
title: mmo游戏跨服玩法
categories: Mmo-Game
description: mmo游戏跨服玩法
keywords: mmo, 跨服，活动，晋梁
---

mmo游戏跨服玩法

**目录**

* TOC
{:toc}

## 跨服玩法简单介绍

```sh
1.游戏里，一般各个服相互之间是屏蔽的。例如：001服务器玩家不能登录002服务器，
不同服之间玩家不能组队、聊天...
2.为了游戏运营以及增加玩法之间的乐趣，跨服活动就登场了。不同服的玩家可以在活动开启时，参加同一个
临时服务器，不同服务器的玩家在这里可以进行 pk, 组队，聊天...
3.当然跨服活动时会屏蔽很多玩法，毕竟最后还是要回到自己原先的服务器，所以类似加好友这种功能，交易...在跨服是会屏蔽
的，仅会开放有限的一些功能。
```


## 开始流程图

![](/images/posts/mmo_game/war_ground/1.png)



## 晋梁战场活动玩法流程

### 1.活动最终确认报名

1.world验证通过之后的回调,调用跨服模块
```sh
/**
* world验证通过之后的回调
*
* @param time
*/
public int callbackConfirmSignUp(long time)
{
    int result = getActor().getCrossModule()
            .clientCrossMatchRequest(ActivityType.WAR_GROUND, DictWarGroundConfig.getSceneDefineId());
}
```

2.发起跨服匹配请求
```sh
public int clientCrossMatchRequest(int matchTypeID, int activitySceneID)
{
    ...
     // 发送跨服匹配请求给server 线程
    BPSSRequestCrossMatch message = new BPSSRequestCrossMatch(MessageIDConst.SS_SERVER_MATCH_REQUEST);
    message.setMatchData(data);

    sendMessageToService(message, ServiceManager.getInstance().getServerService());
    crossStatus = ActorCrossStatusEnum.GAME_MATCHING_REQUEST;
    ...
}

设置 
```
