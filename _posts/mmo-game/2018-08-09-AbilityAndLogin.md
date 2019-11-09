---
layout: post
title: 生活技能、登录开发
categories: Mmo-Game
description: 记录日常开发积累
keywords: ability,login
---
生活技能、登录开发开发积累



**目录**

* TOC
{:toc}

## 生活技能


### 1.Collections.sort(list)方法时，如果list是不可修改的，将报不可操作错误

```java
public static void main( String[] args )
{
    List<Integer> list = new ArrayList<>();
    list.add(1);
    list.add(2);
    list.add(3);
    //将list变为不可修改
    List<Integer> unModifiableList = Collections.unmodifiableList(list);
    test(unModifiableList);
}

private static void test(List<Integer> list)
{
    //对不可修改的list进行排序，将报错
    Collections.sort(list);

    for (int v : list)
    {
        System.out.println(v);
    }
}
```
结果：
```java
Exception in thread "main" java.lang.UnsupportedOperationException
 at java.util.Collections$UnmodifiableList.sort(Collections.java:1331)
 at java.util.Collections.sort(Collections.java:141)
 at com.lzb.App.test(App.java:38)
 at com.lzb.App.main(App.java:32)
```

游戏开发:

```java
// 生活技能合成操作 (烹饪、制药、熔矿)
message CSAbilityComposeOperate {
    required int32 abilityId = 1; // 生活技能id
    optional int32 composeId = 2; // 配方id(制药、熔矿、烹饪)
    required int32 composeNum = 3; // 合成数量
    required int32 useItemType = 4; // 使用材料类型 1 先绑定后非绑 2 只绑定 3 只非绑
    required int32 panEditorId = 5; // 烹饪锅的id
    optional bool useOptionalItem = 6; // 是否使用高级材料
    repeated int32 cookItems = 7; // 烹饪消耗食材列表
}
```
用protobuf时，例如repeated int32 cookItems = 7，服务器接受时默认是一个不可修改的list, 
所以我们可以先将list转成可修改的list，然后再调用Collections.sort()给list排序，就不会报错了。

```java
//转为可修改的list，再调用sort方法
List<Integer> modifiableList = new ArrayList<>(unmodifiableList);
Collections.sort(modifiableList);
```

### 2.jdk原生HashMap在get的时候，如果键值对没匹配到，那么返回的是null值

```java
HashMap<String, Integer> map1 = new HashMap<>();
map1.put("lzb", 100);

HashMap<Integer, Integer> map2 = new HashMap<>();
map2.put(1, 1);

System.out.println(map1.get("swf"));
System.out.println(map2.get(2));
```
结果:
```java
null
null
```

项目使用的trov4j里的 TIntIntHashMap如果key值没匹配上，那么返回是0

```java
TIntIntHashMap map3 = new TIntIntHashMap();
map3.put(1, 1);
System.out.println(map3.get(2));
```
结果:
```java
0
```

### 3.多个id组合成匹配唯一id

实际场景：烹饪配方id由策划可配的食材道具id组成（数量为1~5），不同的道具id组合成唯一的一个配方id。
前端发来乱序的食材道具id列表，我们去匹配是否有符合的烹饪配方id，这个场景如何做到性能较好的实现？

![](/images/posts/mmo_game/1.png)

实现方案：

初始化处理数据表数据时，将每个烹饪配方需要的食材id列表按从小到大先排序，然后将食材id列表拼接成一个字符串，字符串作为HashMap的key，value则是对应的烹饪配方id。

前端发来乱序的道具食材id列表，同样先将食材id进行从小到大排序，然后拼接成字符串，以这个字符串，去HashMap里匹配是否有烹饪配方id。


后来发现，字符串进行哈希时，每次都需要变量字符串，如果字符串过长，那么效率哈希效率低下。

改为，采用简版AC匹配算法，也就是多级索引:

代码如下：

```java
/**
 * 生活技能配方简版AC匹配算法
 * @author lizhibiao
 * @date 2019/11/5 20:48
 */
public class AbilityFormulaMatchFilter
{
    /**
     * DFA入口
     */
    private final AbilityFormulaMatchFilter.DFANode dfaEntrance;


    public AbilityFormulaMatchFilter(TIntObjectHashMap<TIntArrayList> formulaDataMap)
    {
        dfaEntrance = new DFANode();
        clear();

        if (null != formulaDataMap)
        {
            TIntObjectIterator<TIntArrayList> iterator = formulaDataMap.iterator();
            while (iterator.hasNext())
            {
                iterator.advance();

                //配方id
                int formulaId = iterator.key();
                TIntArrayList itemIdList = iterator.value();
                DFANode currentDFANode = dfaEntrance;
                int size = itemIdList.size();
                for (int i = 0; i < size; i++)
                {
                    //道具id
                    int itemId = itemIdList.get(i);
                    DFANode nextNode = currentDFANode.dfaTransition.get(itemId);
                    if (null == nextNode)
                    {
                        nextNode = new DFANode();
                        currentDFANode.dfaTransition.put(itemId, nextNode);
                        //注意这里是下一节点层数加1
                        nextNode.level = currentDFANode.level + 1;
                    }
                    currentDFANode = nextNode;
                }

                //如果当前节点不等于DFA入口，那么说明是终节点,赋值配方id
                if (currentDFANode != dfaEntrance)
                {
                    currentDFANode.formulaId = formulaId;
                }
            }
        }
    }

    /**
     * 根据道具id列表匹配烹饪配方
     * @param orderItemIdList 有顺序的道具id列表
     * @return 小于等于0未匹配， 大于0才是匹配成功返回配方id
     */
    public int matchFormulaId(List<Integer> orderItemIdList)
    {
        if (CollectionUtils.isBlank(orderItemIdList))
        {
            return -1;
        }

        if (null == dfaEntrance)
        {
            return -1;
        }

        //入口节点层数为0
        DFANode currentDFANode = dfaEntrance;
        int size = orderItemIdList.size();
        for (int i = 0; i < size; i++)
        {
            int itemId = orderItemIdList.get(i);
            DFANode nextDFANode = currentDFANode.dfaTransition.get(itemId);
            if (null != nextDFANode)
            {
                //当前节点等于下一节点，继续往下匹配
                currentDFANode = nextDFANode;

                //如果一直匹配，那么下一节点的层数和列表长度相等表明迭代完毕返回烹饪配方id
                if (size == currentDFANode.level)
                {
                    return currentDFANode.formulaId;
                }
            }
            else
            {
                //未匹配
                return -1;
            }
        }

        return -1;
    }


    /**
     * DFA节点.
     */
    private static class DFANode
    {
        /**
         * 配方id
         */
        private int formulaId;

        /**
         * key道具id, value为DFA节点
         */
        private final TIntObjectHashMap<AbilityFormulaMatchFilter.DFANode> dfaTransition;

        /**
         * 节点层数
         */
        private int level;


        public DFANode()
        {
            this.dfaTransition = new TIntObjectHashMap<>();
            formulaId = 0;
            level = 0;
        }
    }

    /**
     * 初始化时先调用此函数清理
     */
    private void clear()
    {
        // 清理入口
        dfaEntrance.dfaTransition.clear();
    }

}
```

### 4.采集读条广播动作增加玩家朝向

实际场景:当玩家开始读条进行采集时，玩家朝向可能是背对采集物的，所以需要服务调整好朝向并广播给周围玩家。
那么我们需要根据当前玩家坐标点（x, z）和采集物坐标点（x, z），准确计算朝向。那么该如何计算朝向？
```java
/** 面向某点 */
public void faceTo(int x,int z)
{
	if(this.x == x && this.z == z)
	{
		return;
	}
	setRotation(MapUtils.rotationBetweenPos(this.x,this.z, x, z));
}
```
分析：调用MapUtils.rotationBetweenPos(this.x,this.z, x, z)计算出朝向，直接set到当前场景对象

```java
/** 返回点1对于点0的角度 */
public static int rotationBetweenPos(float x0, float y0, float x1, float y1) 
{
	double d = Math.atan2(y1 - y0, x1 - x0);

	d = d / Math.PI * 180;

	return rotationCut((int) d);
}
```
分析：计算角度

```java
// 通用读条开始通知
message SCCommonStartStripe{
	required int32 stripeID = 1; //读条id,对应DictCommonStripe表中的id
	repeated int32 params = 2; // 服务器透传参数
	optional int32 objectId = 3; // 对象id
	optional int32 rotation = 4; // 朝向 0-360
}
```
分析:最后在广播协议中加上朝向参数即可同步到周围玩家

## 登录

### 1.断线重连机制

当客户端直接断开时会走断线重连机制，如果是选择退出游戏那么不会走断线重连。

#### 一：服务端等待断线，如果玩家没有在规定的时间内重新登录，那么时间到了才会将对象从内存中移除。

大体流程如下：

1.客户端异常关闭，netty的chanel会断开，我们在exception里将当前状态置为“断线重连状态”

```java
 @Override
public void channelInactive(ChannelHandlerContext ctx) throws Exception
{
    CoreLog.CORE_NET.info("channel inactive, remote:{}", ctx.channel().remoteAddress());

    exception(ctx, null);
}

private void exception(ChannelHandlerContext ctx, String reason)
{
    if (entity == null)
    {
        return;
    }

    CoreLog.CORE_NET.info("channel exception, remote:{}, exception:{}",
            ctx.channel().remoteAddress(), reason == null ? "" : reason);

    InterfaceObject interfaceObject = entity.getInterfaceObject();
    if (interfaceObject != null)
    {
        try
        {
            interfaceObject.setException(ObjectExceptionEnum.DISCONNECT);
        }
        catch (NullPointerException e)
        {
            CoreLog.CORE_NET.info("ClinetHanlder: interface object is null");
        }
    }

    ctx.close();

    entity = null;
}
```

2.场景tick对象时，发现对象状态为不正常状态，判断是断线重连后，设置断线重连时间

```java
/**
  * 场景对象的tick,包括玩家
  * @param interval
 */
public void tickObject(int interval) 方法下

actor.closeChannel();
actor.clearSendPacket();
// 如果断线并且不是主动退出游戏,才走断线重连回调
// 主动断开，那么还没设置断线重连时间, 所以actor.isWaitingReconnect()返回false
if (!actor.isWaitingReconnect())
{
    //断开连接的回调处理
    actor.disConnected();
}
```

3.tick处理断线重连,如果超时那么就将发送消息到world执行玩家退出场景
```java
/**
  * tick处理断线重连
  * @param interval
*/
public void tickReconnect(int interval)
{
    if (!isWaitingReconnect())
    {
        return;
    }

    if (exitStatus == ActorExitGameStatusEnum.LEAVING_SCENE)
    {
        return;
    }

    if (exitStatus == ActorExitGameStatusEnum.EXIT)
    {
        exitStatus = ActorExitGameStatusEnum.LEAVING_SCENE;
        // 发送退出场景
        sendLeaveSceneMessage(true);
        return;
    }

    waitReconnectRemainTime -= interval;

    // 如果是逻辑错误了,不需要断线重连等待,直接踢掉
    if (exceptionEnum == ObjectExceptionEnum.GAME_LOGIC_ERROR || exceptionEnum == ObjectExceptionEnum.LOGIC_KICKOFF)
    {
        waitReconnectRemainTime = 0;
    }

    if (waitReconnectRemainTime <= 0)
    {
        waitReconnectRemainTime = 0;
        exceptionEnum = ObjectExceptionEnum.WAIT_RECONNECT_TIMEOUT;
        BPLog.BP_SCENE.info("角色:{} 等待断线重连超时", actorID);

        exitStatus = ActorExitGameStatusEnum.LEAVING_SCENE;
        
        // 发送退出场景
        sendLeaveSceneMessage(true);
        return;
    }
}
```

4.world上场景线里玩家的数据回收以及其它的一些world信息处理,将消息回传到场景将玩家状态置为保存数据状态
```java
actor.setExitStatus(ActorExitGameStatusEnum.SAVE_DATA);
```

5.继续场景tick对象，判断玩家为保存数据状态，将玩家状态置为删除
```java
if (actor.getExitStatus() == ActorExitGameStatusEnum.SAVE_DATA)
{
    BPLog.BP_SCENE.info("scene:{}:{} actor:{} object:{} exception:{}",
            sceneID, lineID, actor.getActorID(), actor.getObjectID(), exceptionEnum);
    actor.deleteSafe();
    continue;
}
```

```java
if (object.isCanDestroy())
{
    removeBpObject(object);
    continue;
}
```
分析：进行移除对象

6.最后移除对象各种操作

```java
doRemoveBpObjectFromScene()
```
分析：主要是把actor对象从场景里缓存的一些map里移除，如果场景人数为空，那么通知world回收该场景线。


#### 二：在规定的时间内,玩家进行断线重连登录

大体流程：
1.客户端发起登录
```java
/ 登录游戏请求
message CSPlayerLogin {
    required string platformUid    = 1;   //玩家平台SDK的Uid
    optional string appVersion     = 2;   //程序版本号
    optional string resVersoin     = 3;   //资源版本号
    optional string channelId      = 4;   //渠道id，每个包唯一
    optional string deviceId       = 5;   //
    optional DeviceInfo deviceInfo = 6;   //
    optional string authCode       = 7;   //登录验证码，由checkPlatformToken返回
    optional int32 loginType       = 8;   //登录类型 0是玩家登录，1是本地开发测试登录(不验证token)
    optional int32 serverId        = 9;   //服务器id
    optional string sessionID 	   = 10; // 如果是断线重连才传此值,不是的话不需要传,只要消息里有sessionID服务器就会按照断线重连处理
    optional int32 globalId 	   = 11; // 服务器唯一id
}
```
分析：断线重连，客户端会赋值sessionID,服务端根据该值判断是否断线重连

2.服务端接受到消息后，去世界上场景线缓存验证该玩家是否还在内存中
```java
/**
* 玩家断线重连请求处理
* @param loginObject
* @param login
* @return
*/
public int playerReconnectRequest(BPLoginObject loginObject, BPLogic.CSPlayerLogin login)
```
分析：该方法里将登录状态置为重连状态，然后发送世界验证是否在场景线缓存

```java
@MessageHandler(MessageIDConst.WS_ACTOR_RECONNECT_REQUEST)
public void handleActorReconnectRequest(AbstractBPScene service, BPActorReconnectReq message)
{
    ...
    ...

     if (actor.isWaitingReconnect())
     {
        // 重置一下时间
        actor.setWaitReconnectRemainTime();
        service.sendServiceMessageToWorld(rsp);
     }
}
```
分析:多次收到世界下发到场景里的断线重连请求回复方法，然后上面这个方法要特别注意下，这里会重置下断线重连的时间，
也就是如果玩家一直连接、断线、连接、断线，那么因为断线重连时间一直被重置，玩家数据就会一直在内存里。

3.收到world返回的断线重连请求
```java
/**
* 收到world返回的断线重连请求
* @param result
* @param accountID
* @param actorID
* @param sceneID
* @param lineID
*/
public void actorReconnectResponse(int result, String accountID, long actorID, int sceneID, int lineID,
                                    long createTime, int posX, int posZ)
```
分析：这里会将登录状态置为等待客户端进入场景，并发送客户端SCReconnectInfoNotice下发重连信息

4.客户端发起断线重连进入场景请求
```java
// 断线重连进入场景请求
message CSReconnectEnterScene{
	required int32 sceneID = 1; // 场景id
	required int32 lineID = 2;	// 线id
}
```
分析：服务端接受到消息后，将登录状态置为重连进入场景成功，然后发送世界处理

5.经过多次下发
```java
@MessageHandler(MessageIDConst.WS_ACTOR_BIND_CHANNEL_REQUEST)
public void handleActorBindChannelRequest(AbstractBPScene service, BPActorBindChannelReq message)
{
     ...
     ...
     
     Actor actor = service.getActorByActorID(actorID);
     int result = actor.reconnectCallback(coreEntity);
}
```
分析:注意这里我们可以看到，玩家对象是根据玩家id值直接从内存获取到的。然后调用断线重连成功回调方法。

也就是说，断下重连的时间内，玩家的数据还是保存在服务器内存中没有删除。

断线重连，不会new一个新的Actor对象。

6.断线重连成功回调方法
```java
/**
* 断线重连成功回调
* @param coreEntity
* @return
*/
public int reconnectCallback(AbstractCoreEntity coreEntity)
{
    ...
    ...
     //发送断线重连的初始化数据
    sendReconnectInitData();
}
```
分析：这里断线重连成功回调方法里调用初始化数据方法，就是我们平时各个模块对应的初始化数据。


### 2.玩家登录

1.登录请求

```java
public void handleCSPlayerLogin(BPLogic.CSPlayerLogin packet, BPLoginObject loginObject)
{
    ...
    ...
    int result = loginObject.playerLoginRequest(packet);
}
```
分析：发起登录请求，调用玩家登录请求方法

```java
/**
* 玩家登录请求
*
* @param loginObject
* @param packet
* @return
*/
public int playerLoginRequest(BPLoginObject loginObject, BPLogic.CSPlayerLogin packet)
{
    ...
    ...
    else
    {
        BPLog.BP_LOGIN.info("登录 账号:{} 发送主服验证, type:{} authCode:{}", accountID, loginType, authCode);
        // 去主服验证token
        checkFromMainServer(loginObject, loginType, authCode);
    }
}
```
分析：去主服验证token

```java
/**
* 向主服验证玩家登录token
* @param object
* @param loginType
* @param authCode
*/
public void checkFromMainServer(BPLoginObject object, int loginType, String authCode)
{
    ReqCheckLoginToken req = new ReqCheckLoginToken();
    req.setLoginType(loginType);
    req.setAuthCode(authCode);
    req.setPlafformUid(object.getAccountID());
    req.setServerId(NetConfig.getInstance().getServerID());
    req.setDeviceId(object.getDeviceID());
    req.setClientIp(object.getRemoteAddr());
    req.setAppVersion(object.getClientVersion());
    req.setChannelId(object.getChannelID());

    String data = JSON.toJSONString(req);

    StringBuilder uriSb = new StringBuilder();
    uriSb.append(NetConfig.getInstance().getMainServerUrl()).append(HttpApi.CHECK_LOGIN_TOKEN);
    URI uri = URI.create(uriSb.toString());

    //这里new一个去主服检查玩家登录token回调即AbstractHttpResponse(子类ToMainCheckLoginTokenCallback)
    //发送到主服检验后，主服会会回应一个http响应
    ToMainCheckLoginTokenCallback callback = new ToMainCheckLoginTokenCallback();
    callback.setPlatformUID(object.getAccountID());

    StringBuilder dataSb = new StringBuilder();
    dataSb.append(HttpConstant.HTTP_CONTENT).append(HttpConstant.SYMBOL_EQUAL).append(StringUtils.urlEncode(data));

    httpClientModule.asyncRequest(uri, HttpMethod.POST, dataSb.toString(), HttpConstant.HTTP_CONTENT_TYPE_FORM, callback);
}
```
分析：主要就是拼接参数，转json串，发起http请求到主服验证。


```java
AbstractBPLoginService.java下:

/**
* 处理http
* login只会发送http请求,所以这里只处理的是http response
* @param interval
*/
private void tickHttp(int interval)
{
    // 每个tick处理50条http返回
    for (int i = 0; i < 50; i++)
    {
        BPHttpCallback callback = httpClientModule.pop();
        if (callback == null)
        {
            break;
        }

        callback.callback();
    }
}

```
进行tick回调处理


```java
**
 * 去主服检查玩家登录token回调
 *
 * @author guozhen
 * @date 2018/3/14
 */
public class ToMainCheckLoginTokenCallback extends BPHttpCallback
{
    @Override
    public void callback()
    {
        ...
        ...
        loginService.loadLoginData(loginObject);
    }

}
```
分析：主服验证成功，回调方法里，调用向db发送load登录数据请求

```java
/**
* 向db发送load登录数据请求
* 如果没有登录数据,会直接创建一个出来(现在注释了)
* @param loginObject
*/
public void loadLoginData(BPLoginObject loginObject)
{
    ...
    ...
    BPLoadLoginDataReq req = new BPLoadLoginDataReq(MessageIDConst.LOAD_LOGIN_DATA_REQUEST);

    //从角色池里拿出玩家数据对象，这里的角色中间过渡只是调用init将所有model模块集合加入到列表而已，没有数据
    ActorDataCache dataCache = BPGlobals.getActorDataCache();
    if (dataCache == null)
    {
        BPLog.BP_LOGIN.error("登录 actor cache池已满, account:{}", accountID);
        loginObject.setException(ObjectExceptionEnum.INTERNAL_ERROR);
        return;
    }
    ....
    loginObject.setFlowStatus(BPLoginFlowStatusEnum.LOAD_ACCOUNT_LOGIN_DATA);
    
}
```
分析：向db发送load登录数据请求,设置状态为正在load账号登录数据

```java
@MessageHandler(MessageIDConst.LOAD_LOGIN_DATA_REQUEST)
public void handleLoadLoginDataRequest(BPDBService dbService, BPLoadLoginDataReq message)
{
    ActorDBTask actorDBTask = new ActorDBTask();
    actorDBTask.clear();

    AbstractService sourceService = message.getLoginService();
    if (sourceService == null)
    {
        return;
    }

    actorDBTask.setSourceService(sourceService);
    actorDBTask.setOperateType(DBOperateTypeEnum.LOGIN);
    actorDBTask.setActorDataCache(message.getActorDataCache());

    dbService.addDBTask(actorDBTask);
}
```
分析：这里会new一个DB任务，然后调用addDBTask()添加到待处理的db任务列表...最后会交给线程池去执行DB任务,执行完DB任务会加到完成或失败列表，
tick列表时load数据会返回给登录线程。

```java
/**
* tick待处理的db task
* @param interval 上一个tick到本次tick的时间间隔
*/
private void tickHandleDBTask(int interval)
{
    ...
    ...
    int removePosition = 0;
    for (int i = 0; i < size; i++)
    {
        // 取出一个任务
        AbstractDBTask dbTask = handleTaskList.get(removePosition);
        if (dbTask == null)
        {
            break;
        }

        if (dbTask instanceof ActorDBTask)
        {
            long actorID = ((ActorDBTask) dbTask).getActorDataCache().getActorID();
            //玩家是否在执行db任务
            short isHandling = handlingTaskMap.get(actorID);

            if (actorID < 0)
            {
                isHandling = WITHOUT_TASK;
            }

            // 没有在执行的同ID任务
            if (isHandling == WITHOUT_TASK)
            {
                if (actorID > 0)
                {
                    //有任务
                    handlingTaskMap.put(actorID, HAS_TASK);
                }
                handleTaskList.remove(removePosition);
            }
            else
            {
                //集合里已经有，说明重复执行
                if (repeatHandleSet.contains(actorID))
                {
                    // 遍历做一个合并
                    for (int j = 0; j < removePosition; j++)
                    {
                        AbstractDBTask abstractDBTask = handleTaskList.get(j);
                        if (abstractDBTask instanceof ActorDBTask)
                        {
                            long id = ((ActorDBTask) abstractDBTask).getActorDataCache().getActorID();
                            if (id == actorID)
                            {
                                // 一次只做一次合并  一次就break了
                                if (mergeDBTask(abstractDBTask))
                                {
                                    handleTaskList.remove(j);
                                }
                                else
                                {
                                    handleTaskList.remove(removePosition);
                                }
                                break;
                            }
                        }
                    }
                }
                else
                {
                    removePosition++;
                    repeatHandleSet.add(actorID);
                }
                continue;
            }
        }
        ...
        ...
        // 派发执行任务
        dbHandleThreadPool.addThreadTask(dbTask);
    }

    repeatHandleSet.clear();
}
```
分析：这里主要就是从执行db任务列表取出db任务，然后派发执行。需要注意的是，这里会判断当前的actor是否正在执行db任务，如果正在执行，那么会做一次
合并db任务操作。

```java
/**
     * 合并db任务到待处理的DB任务列表
     * @param dbTask
     * @return boolean 是否合并了一个task,并且将缓存置空
     */
    private boolean mergeDBTask(AbstractDBTask dbTask)
    {
        //得处理的db任务列表如果长度为0，那么不用合并
        int size = handleTaskList.size();
        if (size == 0)
        {
            //直接将当前db任务加入到待处理的db任务列表
            addDBTask(dbTask);
            return false;
        }

        boolean merge = false;
        for (int i = 0; i < size; i++)
        {
            AbstractDBTask task = handleTaskList.get(i);
            //判断两个任务是否为同一类任务
            if (dbTask.isEqual(task))
            {
                //如果当前任务和遍历到待处理的db任务列表里不是同一个
                if (task != dbTask)
                {
                    task.mergeFrom(dbTask);
                    merge = true;
                    break;
                }
            }
        }

        if (!merge)
        {
            // 这里的修改原因是出现一种情况:handleTaskList.size!=0 && dbTask没有被合并，就直接走下面的释放任务了
            addDBTask(dbTask);
            return false;
        }

        // 释放公共数据内存池
        if (dbTask instanceof ActorDBTask)
        {
            BPGlobals.releaseActorDataCache(((ActorDBTask)dbTask).getActorDataCache());
            ((ActorDBTask)dbTask).setActorDataCache(null);
        }
        else if (dbTask instanceof CommonDBTask)
        {
            BPGlobals.releaseCommonDataCache(((CommonDBTask)dbTask).getCommonDataCache());
            ((CommonDBTask)dbTask).setCommonDataCache(null);
        }

        return true;
    }
```
分析：合并db任务

```java
private void tickThreadPool(int interval)
{
    ...
    ...
    else if (abstractDBTask.getOperateType() == DBOperateTypeEnum.LOAD)
    {
        // load数据需要返回给登录线程
        if (abstractDBTask instanceof ActorDBTask)
        {
            ActorDBTask actorDBTask = ((ActorDBTask)abstractDBTask);
            BPLoadActorDataRsp rsp = new BPLoadActorDataRsp(MessageIDConst.LOAD_ACTOR_DATA_RESPONSE);
            rsp.setResult(result);
            rsp.setActorDataCache(actorDBTask.getActorDataCache());
            sendMessage(actorDBTask.getSourceService(), rsp);
            continue;
        }
        else if(abstractDBTask instanceof CommonDBTask)
        {
            CommonDBTask commonDBTask = (CommonDBTask) abstractDBTask;
            BPLoadCommonDataRsp rsp = new BPLoadCommonDataRsp(MessageIDConst.COMMON_DATA_LOAD_RESPONSE);
            rsp.setCommonDataCache(commonDBTask.getCommonDataCache());
            rsp.setResult(result);

            sendMessage(commonDBTask.getSourceService(), rsp);
            continue;
        }
    }
}
```
分析：tick结果，load数据返回给登录线程

```java
public void handleLoadActorDataResponse(AbstractBPLoginService loginService, BPLoadActorDataRsp message)
{
    ...
    ...
    //非常注意这里是新new了一个actor对象
    Actor actor = new Actor();
    loginObject.setActor(actor);
    actor.setCoreEntity(loginObject.getCoreEntity());
    ...
    ...

    //挂在actor上的所有模块初始化
    actor.init();
    ....
    ....

    //挂在actor上的所有模块调用copyFrom方法从db赋值
    boolean result = actor.copyFrom(actorDataCache);
    if (!result)
    {
        loginObject.setException(ObjectExceptionEnum.INTERNAL_ERROR);
        BPLog.BP_LOGIN.error("登录 账号:{} 角色:{} copy from数据失败", accountID, actorID);
        // 释放data cache
        BPGlobals.releaseActorDataCache(actorDataCache);
    }

    actor.initCommonData(loginObject);

    //挂在actor上的所有模块afterLoad()方法处理
    actor.afterLoad();
    actor.setAllModuleModify(false);

    //设置状态load角色数据成功
    loginObject.setFlowStatus(BPLoginFlowStatusEnum.LOAD_ACTOR_DATA_SUCCESS);
    loginObject.getLoginTimeOut().cancelTimeout();
    loginObject.getLoginTimeOut().setTimeOutEnum(BPLoginTimeOutEnum.CLIENT_LOAD_SCENE);

    // 释放data cache
    BPGlobals.releaseActorDataCache(actorDataCache);

    // 加载完角色数据后把玩家数据同步给主服
    loginService.onActorLogin(loginObject);
    ...
    ...

}
```

分析：load角色数据回复流程:

1.特别注意，玩家下线登录是新new了一个Actor对象，放进内存。

2.挂在actor上的所有模块初始化

3.挂在actor上的所有模块调用copyFrom方法从db赋值

4.挂在actor上的所有模块afterLoad()方法处理

5.设置状态为登录成功

6.同步数据到主服


```java
**
 * 角色相关的db任务
 * Created by wangqiang on 2017/8/29.
 */
public class ActorDBTask extends AbstractDBTask
{
    @Override
    public int execute()
    {
        ...
        ...
        if (operateType == DBOperateTypeEnum.LOAD)
        {
            for (int i = 0; i < size; i++)
            {
                AbstractActorDataModel actorDataModel = list.get(i);
                result = actorDataModel.load();
                if (result < 0)
                {
                    BPLog.BP_DB.warn("db 角色:{} 模块:{} load 数据出错", actorDataModel.getActorID(), actorDataModel.getClass().getName();
                    break;
                }
            }
        }

    }

}
```
分析：DB任务最后会交给线程池去派发执行任务，然后调用ActorDBTask的execute()执行角色load任务，调用actorDataModel.load()将所有的各个模块数据加载到内存。


### 3.关闭服务器流程

#### 一：基本流程

```java
/**
 * 关闭信号处理器
 * 执行关闭流程:
 *  1. 通知登录线程,不再处理新登录
 *  2. 通知场景踢人,存玩家数据
 *  3. 存储world公共数据
 *  4. 调用exit,退出进程
 * Created by wangqiang on 2018/1/16.
 */
public class SystemSignalHandler implements SignalHandler
{
    @Override
    public void handle(Signal signal)
    {
        BPLog.BP_SYSTEM.info("收到关闭服务器信号:{}, 开始走关闭服务器流程", signal);
        shutdown();
    }

    public static void shutdown()
    {
        BPJVMShutdownMessage message = new BPJVMShutdownMessage(MessageIDConst.JVM_SHUTDOWN_NOTICE);
        AbstractWorldService worldService = ServiceManager.getInstance().getWorldService();
        worldService.sendMessage(worldService, message);
    }
}
```
分析：java中提供了signal的机制(需要实现注册机制)。在sun.misc包下，属于非标准包。服务器关闭，handle接受到消息，调用shutdown()方法，
发送世界，走关闭服务器流程。

```java
/**
*  1. 通知登录线程,不再处理新登录
*  2. 通知场景踢人,存玩家数据
*  3. 存储world公共数据
*  4. 调用exit,退出进程
* 关服
*/
public void shutdown()
{
    // 通知登录线程,停止登录
    {
        BPLog.BP_SYSTEM.info("停服 通知登录线程");
        BPJVMShutdownMessage message = new BPJVMShutdownMessage(MessageIDConst.WL_SHUTDOWN_SERVER_NOTICE);
        sendMessage(ServiceManager.getInstance().getLoginService(), message);
    }

    BPLog.BP_SYSTEM.info("停服 关闭进入场景功能");
    // 关闭场景进入功能
    setShutdown(true);

    // 通知场景线程玩家
    {
        BPLog.BP_SYSTEM.info("停服 广播给所有运行中的场景");
        BPJVMShutdownMessage message = new BPJVMShutdownMessage(MessageIDConst.WS_SHUTDOWN_SERVICE_NOTICE);
        broadcastMessageToRunningScene(message);
    }
}
```
分析：世界线程接受到线程消息后，调用shutdown()处理

    1. 通知登录线程,不再处理新登录

    2. 通知场景踢人,存玩家数据（这里其实是保存到文件）

    3. 存储world公共数据

    4. 调用exit,退出进程

#### 二：继续看通知登录线程jvm关闭

```java
/**
* 停服,踢掉所有正在登录的客户端,同时不接收新登录的客户端
*/
public void shutdown()
{
    isShutdown = true;

    Iterator<BPLoginObject> iterator = connectedList.iterator();
    while (iterator.hasNext())
    {
        BPLoginObject loginObject = iterator.next();
        sendLoginError(loginObject, BPErrorCodeEnum.LOGIN_SERVER_SHUTDOWN);
        iterator.remove();
    }

    Iterator<Map.Entry<String, BPLoginObject>> iterator1 = loginMap.entrySet().iterator();
    while (iterator1.hasNext())
    {
        Map.Entry<String, BPLoginObject> entry = iterator1.next();
        BPLoginObject loginObject = entry.getValue();
        sendLoginError(loginObject, BPErrorCodeEnum.LOGIN_SERVER_SHUTDOWN);
        iterator1.remove();
    }

    TLongObjectIterator<BPLoginObject> iterator2 = waitEnterSceneMap.iterator();
    while (iterator2.hasNext())
    {
        iterator2.advance();
        BPLoginObject loginObject = iterator2.value();
        waitEnterSceneSet.remove(loginObject.getAccountID());
        sendLoginError(loginObject, BPErrorCodeEnum.LOGIN_SERVER_SHUTDOWN);
        iterator2.remove();
    }

    Iterator<Map.Entry<String, BPLoginObject>> iterator3 = reconnectMap.entrySet().iterator();
    while (iterator3.hasNext())
    {
        BPLoginObject loginObject = iterator3.next().getValue();
        sendLoginError(loginObject, BPErrorCodeEnum.LOGIN_SERVER_SHUTDOWN);
        iterator3.remove();
    }

    BPLog.BP_LOGIN.info("停服 login收到停服请求 关闭登录");
    BPLog.BP_LOGIN.info("[登录统计] 当前在线人数:{}, 排队人数:{}, 正在登录的人数:{}, 重连中的人数:{}",
            loggedMap.size(), queueList.size(), loginMap.size() + connectedList.size(), reconnectMap.size());
}
```
分析：isShutdown状态置为true，然后是已连接的客户端队列、登录中的客户端map、等待进入场景的对象...

```java
private void tickShutdown(int interval)
{
    if (!isShutdown || !isStarted)
    {
        return;
    }

    shutdownRemain -= interval;

    if (shutdownRemain <= 0)
    {
        // 停服超时,则继续倒计时
        BPLog.BP_LOGIN.warn("停服超时");
        shutdownRemain = 300000;
    }

    if (offlineSaveMap.isEmpty() && loggedMap.isEmpty())
    {
        isStarted = false;
        BPLog.BP_LOGIN.info("登录线程已完成停服操作");
        // 发送给world线程,通知world线程执行停服存储操作
        BPJVMShutdownMessage message = new BPJVMShutdownMessage(MessageIDConst.LW_SHUTDOWN_SERVICE_DONE);
        sendMessage(ServiceManager.getInstance().getWorldService(), message);
        return;
    }
}
```
分析：登录线程tick,发送world线程，通知world线程执行停服存储操作

```java
/**
* 登录线程完成停服操作,也保存完了所有玩家的数据
*/
public void loginShutdownDone()
{
    BPLog.BP_SYSTEM.info("world线程开始保存数据");

    // 保存world上的数据
    for (AbstractWorldModule module : saveModuleArray)
    {
        module.setModify(true);
    }
    tradingMarketModule.setShutdown(true);
    serverInfoModule.setShutdownTimestamp((int) (nowTickTime / 1000));

    CommonDataCache dataCache = BPGlobals.getCommonDataCache();
    if (dataCache == null)
    {
        // 池中没有足够的对象了,要new一个
        BPLog.BP_SYSTEM.error("common data cache pool is empty!");
        dataCache = new CommonDataCache();
    }

    dataCache.copyFrom(this);

    BPSaveCommonDataReq req = new BPSaveCommonDataReq(MessageIDConst.WD_SHUTDOWN_SAVE_REQ);
    if (BPGlobals.getInstance().getServerMode() == ServerModeEnum.EMERGENCY_EXIT)
    {
        req.setSaveFile(true);
    }
    req.setDataCache(dataCache);
    req.setSrcService(this);
    sendMessage(ServiceManager.getInstance().getDbService(), req);
}
```
分析：登录线程完成停服操作,保存所有玩家的数据