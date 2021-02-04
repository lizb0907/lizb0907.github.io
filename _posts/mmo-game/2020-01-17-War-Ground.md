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

1.设置状态为游戏服请求跨服匹配状态。
2.发送跨服匹配请求给server线程（内网线程）。
3.添加到内网线程的waitMatchMap待匹配列表。
```

3.内网线程向Match匹配服发送匹配请求
```sh
ServerCrossModule:
根据活动id分组，添加到map列表
/**
* 跨服匹配请求
*
* 游戏服
*/
public void matchRequest(ActorCrossMatchData matchData, AbstractService srcService)
{
    int activityID = matchData.getActivityID();
    ArrayList<ActorCrossMatchData> list = waitMatchMap.get(activityID);
    if (list == null)
    {
        list = new ArrayList<>();
        waitMatchMap.put(activityID, list);
    }
    list.add(matchData);
    BPLog.BP_CROSS.info("[跨服] 游戏服server收到客户端的跨服匹配请求 {}", matchData);
}

/**
* tick处理待匹配的数据
* @param interval
*/
private void tickWaitMatch(int interval)
{
    1.构建数据
    2.加入到待移除列表，等待匹配服响应后从列表删除。
    3.将构建好的数据发送的匹配服。
}
```

### 2.匹配服match逻辑

#### 1.协议结构
```sh
repeated int64 actorID = 1; // 玩家id
repeated int64 crossSequenceID = 2; // 跨服请求的序列id,针对单个玩家递增
repeated string accountID = 3; // 账号id
repeated int32 gameServerID = 4; // 游戏服务器的id
required int32 activityID = 5; // 活动id
repeated int32 level = 6; // 玩家等级
repeated int32 fightForce = 7; // 玩家战斗力
```

#### 2.匹配请求逻辑
```sh
/**
* 匹配请求
*
* @param srcServerID
* @param matchGroup
*/
public int matchRequest(int srcServerID, MatchGroup matchGroup)
{
    //根据server id 获取所在的逻辑分区id
    //这里是内网注册时broker从主服拿到数据下发到match服（主服是查数据库配置）
    int zoneLogicID = getZoneLogicIDByServerID(srcServerID);
    if (zoneLogicID < 0)
    {
        return MatchErrorCode.LOGIC_ZONE_ID_NOT_FOUND;
    }

    //加入待匹配组
    int result = addWaitingMatchGroup(zoneLogicID, matchGroup);
    if (result < 0)
    {
        return result;
    }

    return 0;
}
获取逻辑分区id,加入待匹配组
```

### 3.tick处理待匹配组

```sh
 /**
* tick处理匹配
*
* @param interval
*/
public void tickMatch(int interval)
{
    tickMatchRemainTime -= interval;
    if (tickMatchRemainTime <= 0)
    {
        // 匹配
        handleMatch();

        tickMatchRemainTime = MATCH_TICK_INTERVAL;
    }
}

每200毫秒统一从待匹配列表取出进行匹配。
```

```sh
最终匹配：

 /**
* 最终进行匹配处理
* 目前匹配代码比较简单,就是来一个就分配一个,实现不了PVP匹配,这个匹配
* 需要后期重构下面的代码来实现
*
* @param logicZoneID 逻辑大区ID,理论上应该只有同大区的才可以进行匹配
* @param activityID  活动ID
* @param groups
*/
private void matchByLogicZoneID(int logicZoneID, int activityID, HashSet<MatchGroup> groups)
{
    ...
     //匹配逻辑管理类(目前每次都会进)
    IMatchLogicManager manager = ReloadManagerContext.getInstance().getMatchLogicManager();
    if (manager != null)
    {
        //如果是延时匹配那么 策略为0。 如果是及时匹配，策略为1。
        //Enum类提供了一个ordinal()方法，用来返回枚举对象的序数。
        //如果是延时匹配
        if (manager.getMatchMode(activityID) == MatchLogicMatchMode.DELAY.ordinal())
        {
            matchLogic = activityMatchLogicMap.get(activityID);
            if (matchLogic == null)
            {
                //是否每次调用新建算法实例 0是，1不是
                if (manager.getMatchOnce(activityID) == MatchLogicLifeCycle.ONCE.ordinal())
                {
                    matchLogic = manager.getMatchLogicInstance(activityID);
                }
                else
                {
                    //构建算法伪单例
                    matchLogic = manager.getMatchLogicSingle(activityID);
                }
                if(matchLogic == null)
                {
                    groups.clear();
                    MatchLog.MATCH.error("没有找到活动对应的匹配处理类, activityID:{}", activityID);
                    return;
                }
                //设置创建时间戳
                matchLogic.setCreateTimestamp(MainClock.getInstance().currentTimeMillis());
                //添加到缓存匹配逻辑处理对象的map
                activityMatchLogicMap.put(activityID, matchLogic);
            }

            //当前时间 - 创建时间 > 超时时间
            long current = MainClock.getInstance().currentTimeMillis();
            //如果还不到延时时间，则等下一个tick去判断
            if (!matchLogic.isStartMatchTimeout(current))
            {
                return;
            }

        }
        else
        {
            if (manager.getMatchOnce(activityID) == MatchLogicLifeCycle.ONCE.ordinal())
            {
                matchLogic = manager.getMatchLogicInstance(activityID);
            }
            else
            {
                matchLogic = manager.getMatchLogicSingle(activityID);
            }
        }
    }

    上面都是创建策略对象。

    各个活动更加匹配规则，进行分组匹配。
    //进行匹配逻辑!目前只有战场匹配和debug匹配逻辑。
    MatchResult matchResult = matchLogic.match(groups);
    if (matchResult != null)
    {
        // 判断是否有超时的匹配
        ArrayList<MatchGroup> matchTimeoutGroups = matchResult.getMatchTimeoutGroups();
        if (matchTimeoutGroups != null)
        {
            int size = matchTimeoutGroups.size();
            for (int i = 0; i < size; i++)
            {
                MatchGroup matchGroup = matchTimeoutGroups.get(i);
                ArrayList<MatchActor> group = matchGroup.getGroup();
                int groupSize = group.size();
                for (int j = 0; j < groupSize; j++)
                {
                    MatchActor matchActor = group.get(j);
                    if (matchActor == null)
                    {
                        continue;
                    }
                    sendMatchTimeoutPacket(matchActor.getActorID(), matchActor.getCrossSequenceID(), matchActor
                            .getActivityID(), matchActor.getGameServerID());
                }
                // 从待匹配里删除
                groups.remove(matchGroup);
            }
        }
    }
    ...


    发送匹配成功消息给游戏

}
```

### 3.游戏服响应匹配成功

#### 1.成功协议结构

```sh
required int64 actorID = 1;
required int32 mirrorServerID = 2; // mirror的server id
required int32 activityID = 3; // 活动id
required int64 crossSequenceID = 4; // 跨服请求的序列id,针对单个玩家递增
required string accountID = 5; // 账号id
required int32 result = 6; // 匹配结果 0:成功 负数:错误码
optional string mirrorServerIP = 7; // 镜像服服务器ip
optional int32 mirrorServerPort = 8; // 镜像服服务器port
optional int64 matchSequenceID = 9; // 匹配成功的匹配序列号
optional SpecificationMatchData matchData = 10; // 匹配的特化数据
repeated int64 actorIDList = 11; // 匹配成一组的所有actor id数据
```

### 2.游戏服内网收到匹配成功消息

内网模块：ServerCrossModule
```sh
/**
* 处理match服务的匹配结果
* 游戏服
* @param packet
*/
public void matchResponse(Global.MGMatchResponse packet)
{
    ActorCrossMatchData data = matchingMap.remove(actorID);
    if (data == null)
    {
        // 匹配已经被取消 打日志
        BPLog.BP_CROSS.info("跨服 [游戏服] 收到match匹配结果 match已经被取消, actor:{}, account:{}, result:{}",
                packet.getActorID(), packet.getAccountID(), packet.getResult());
        return;
    }

    ...
    // 放入匹配完成的map
    matchDoneMap.put(actorID, data);

    // 通知场景 匹配结果, 通过world
    BPSSCrossMatchResponse response = new BPSSCrossMatchResponse(MessageIDConst.SW_SERVER_MATCH_RESPONSE);
}

1.从正在匹配列表中移除。
2.放入匹配完成的map。
3.发送world，通过world通知场景匹配结果。
```

世界：
```sh
@MessageHandler(MessageIDConst.SW_SERVER_MATCH_RESPONSE)
public void handleServerMatchResponse(BPWorldService service, BPSSCrossMatchResponse message)
{
    message.modifyMessageID(MessageIDConst.SS_SERVER_MATCH_RESPONSE);

    service.sendMessageToActor(message.getActorID(), message);
}
只是做中转往场景转发协议。
```

场景：
```sh
/**
* 游戏服 match的匹配回复处理
* @param message
*/
public void matchResponse(BPSSCrossMatchResponse message)
{
    //失败


    //成功
    
}
```
