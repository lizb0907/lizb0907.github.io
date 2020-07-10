---
layout: post
title: 每日tick
categories: Mmo-Game
description: 每天调用一次的DailyTick
keywords: dailyTick
---

每日清除数据

**目录**

* TOC
{:toc}

## 每天一次的一些数据和调用

例如：侠客宴会每天只能参加3次，一天结束后，次数需要重新置0。

那么会有这么一个问题，玩家如果下线了，那么当前场景tick不到玩家，玩家次数如何重新置0？

## 时钟设计

### 一些类的继承关系

#### 1.场景需要时钟，看下AbstractBPScene类的继承关系：

```sh
public abstract class AbstractBPScene extends AbstractSceneService implements IntervalTimberable

场景基类实现了，间隔时间定时Tick的接口IntervalTimberable
```

```sh
/**
 * 间隔时间定时Tick的接口
 */
public interface IntervalTimberable extends Timerable

IntervalTimberable 实现 Timerable接口，接口可以实现接口，并且接口可以实现多个接口。
```

```sh
/**
 * 可以定时tick的
 * Created by Wang Lizhi on 2017/8/29.
 */
public interface Timerable
{
}
```

#### 2.我们再看下定时Tick时钟，类的继承关系：

```java
/**
 * Tick时钟基类
 */
public abstract class BaseTimerTicker<T extends Timerable>
{
     /**
     * 定时执行体接口
     */
    private T timerable;

    public BaseTimerTicker(T timerable)
    {
        this.timerable = timerable;
    }
}
```

基类是一个泛型类，指定传入的泛型必须实现Timerable接口。

泛型类这么做有好处，成员属性T timerable，在构造方法传参时，T必须实现Timerable接口，否则编译不通过。我们可以利用这些性质，对传入的类做一些限制。

```java
/**
 * 间隔一定时间 Tick的时钟
 */
public class IntervalTimerTicker extends BaseTimerTicker<IntervalTimberable>
```
IntervalTimerTicker继承Tick时钟基类

```java
/**
 * 自然时间定时Tick的时钟
 */
public class NartualTimerTicker extends BaseTimerTicker<NaturalTimberable>
```
NartualTimerTicker继承Tick时钟基类

#### 3.泛型类测试

```java
/**
 * 鸟会飞 -- 实现飞接口
 * @author lizhibiao
 * @date 2020/7/10 17:48
 */
public class Bird implements FlyInterface
{

}
```

```java
/**
 * 鸟会飞 -- 实现飞接口
 * @author lizhibiao
 * @date 2020/7/10 17:48
 */
public class Bird implements FlyInterface
{

}
```

```java
/**
 * 鸭子
 * @author lizhibiao
 * @date 2020/7/10 17:49
 */
public class Duck
{

}
```


```java
/**
 * 构造类类
 * @author lizhibiao
 * @date 2020/7/10 17:44
 */
public class TBuild<T extends FlyInterface>
{
    private T obj;

    public TBuild(T obj)
    {
        this.obj = obj;
    }
}
```

```java
/**
 * 测试类
 * @author lizhibiao
 * @date 2020/7/10 17:50
 */
public class Test
{
    public static void main(String[] args)
    {
        //鸟会飞，实现了FlyInterface，编译通过
        Bird bird = new Bird();
        TBuild<Bird> birdTBuild = new TBuild<>(bird);

        //传入的对象duck不会飞，没有实现FlyInterface，所以编译不通过，直接报红色警告
        Duck duck = new Duck();
        TBuild<? extends FlyInterface> tBuild = new TBuild<>(duck);
    }
}
```
鸟会飞，实现了FlyInterface，编译通过。

传入的对象duck不会飞，没有实现FlyInterface，所以编译不通过，直接报红。

### 场景初始化和激活重置时钟

AbstractBPScene下：

init()和activeCallback，调用重置场景时钟

```java
/**
* 重置场景时钟
*/
private void resetTimeClock()
{

}
```

### 重置时钟resetTimeClock()方法

```java
/**
* 重置场景时钟
*/
private void resetTimeClock()
{
    long nowMillis = System.currentTimeMillis();

    //场景主时钟
    if (this.sysClock == null)
    {
        sysClock = new TimeClock();
    }
    sysClock.init(nowMillis);

    //场景定时时钟
    if (this.timerClock == null)
    {
        //this是AbstractBPScene,AbstractBPScene实现了IntervalTimberable接口
        timerClock = new TimerTickerHolder(this, null);
    }
    timerClock.init(nowMillis);
}
```

```sh
场景主时钟：存的都是一些当前的格林治时间，定时时钟需要拿当前时间和之前的时间作比较。

场景定时时钟：例如每秒进行一次tick,没分钟进行一次tick,需要从场景主时钟拿当前时间
和缓存的时间作比较。

this是AbstractBPScene,AbstractBPScene实现了IntervalTimberable接口
```

### 1.场景主时钟初始化主要操作

```java
private void updateTime(long nowMillis)
{
    //DEFAULT_RAW_OFFSET = TimeZone.getDefault().getRawOffset(); // 默认时 区时间偏移
    //当前时间戳会加上默认时区的偏移
    long rawOffset = nowMillis + TimeUtils.DEFAULT_RAW_OFFSET;

    //格林威治时间，距离现在的总天数，总小时，总分钟数
    curMils = nowMillis;
    curDay = rawOffset / TimeUtils.MILLIS_PER_DAY;
    curHour = rawOffset / TimeUtils.MILLIS_PER_HOUR;
    curMinute = rawOffset / TimeUtils.MILLIS_PER_MINUTE;
    curSecond = rawOffset / TimeUtils.MILLIS_PER_SECOND;
}
```
当前时间戳会加上默认时区的偏移，距离现在的总天数，总小时，总分钟数。

### 2.场景定时钟初始化主要操作

```java
/**
* 每毫秒tick上一次被Tick的时间戳
*/
protected long lastMills;

/**
* 每分钟tick上一次被Tick的时间戳
*/
protected long lastMinute;

/**
* 每小时tick上一次被Tick的时间戳
*/
protected long lastHour;
```
初始化缓存上一次tick时间戳，就是初始化传入的时间戳。

### 3.world上时钟初始化

WorldMessageHandler.java:
```java
public void handleCommonDataLoadResponse(BPWorldService service, BPLoadCommonDataRsp message)
{
    ...
    service.copyFrom(commonDataCache);
    ...
}
```
收到load服务器公共数据回复消息

BPWorldService.java:
```java
/**
* 从db读出来的数据赋值到world上的对应模块里
* @param dataCache
*/
public void copyFrom(CommonDataCache dataCache)
{
    serverInfoModule.copyFrom(dataCache.getServerBaseInfoModel());
}
```
调用serverInfoModule.copyFrom（）

```java
public void copyFrom(ServerBaseInfoModel model)
{
    TimeClock sysClock = new TimeClock();
    sysClock.init(getNowTickTime());

    TimerTickerHolder timerClock = new TimerTickerHolder(worldService, worldService);
    timerClock.init(timestamp);
}
```
其实每次都是重新new对象，重新初始化，并不是去数据库加载的，只是放在了加载数据库方法里。

### 4.每个生物对象挂的时钟

```java
/**
* 初始化时调用
*/
public void init()
{
    // init clock
    tickTimer = new TimerTickerHolder(this,this);

    long nowMillis =  MainClock.getInstance().currentTimeMillis();
    tickTimer.init(nowMillis);
}
```

```java
生物对象基类BPObject.java下，初始化时钟。

  public Actor createActor(BPLoginObject loginObject, String accountID, long actorID,
                             ActorDataCache actorDataCache, int birthX, int birthY, int birthZ,
                             int rotation, int defaultKnightID, int fashionDressID)
{
    ...
    actor.init();
    ...

```
创角的时候会初始化一次。

```java
@MessageHandler(MessageIDConst.LOAD_ACTOR_DATA_RESPONSE)
public void handleLoadActorDataResponse(AbstractBPLoginService loginService, BPLoadActorDataRsp message)
{
    ...
    //非常注意这里是新new了一个actor对象
    Actor actor = new Actor();
    loginObject.setActor(actor);
    actor.setCoreEntity(loginObject.getCoreEntity());
    actor.init();
    ...
}
```
登录的时候也会调用一次init()


### 5.场景下非自然日的更新机制

AbstractBPScene下：
```java
protected void tickCallback(int interval)
{
     // 场景主时钟
    sysClock.tick(nowTickTime);
    // 场景定时时钟
    timerClock.tick(sysClock);
}
```
定时时钟会去从主时钟拿当前时间，所以主时钟顺序在前。

场景主时钟更新：
```java
private void updateTime(long nowMillis)
{
    //DEFAULT_RAW_OFFSET = TimeZone.getDefault().getRawOffset(); // 默认时区时间偏移
    //当前时间戳会加上默认时区的偏移
    long rawOffset = nowMillis + TimeUtils.DEFAULT_RAW_OFFSET;

    //格林威治时间，距离现在的总天数，总小时，总分钟数
    curMils = nowMillis;
    curDay = rawOffset / TimeUtils.MILLIS_PER_DAY;
    curHour = rawOffset / TimeUtils.MILLIS_PER_HOUR;
    curMinute = rawOffset / TimeUtils.MILLIS_PER_MINUTE;
    curSecond = rawOffset / TimeUtils.MILLIS_PER_SECOND;
}
```
没啥看的，传入当前时间戳，每次重新更新时间。

场景定时时钟更新：
```java
@Override
public void tickImpl(TimeClock timeClock)
{
    long clockMillis = timeClock.getCurMils();

    // 秒
    if (clockMillis - lastMills < TimeUtils.MILLIS_PER_SECOND)
    {
        return;
    }

    IntervalTimberable timerable = getTimerable();

    lastMills = clockMillis;
    timerable.secondIntervalTick();

    // 分
    if (clockMillis - lastMinute < TimeUtils.MILLIS_PER_MINUTE)
    {
        return;
    }
    lastMinute = clockMillis;
    timerable.minuteIntervalTick();

    // 小时
    if (clockMillis - lastHour < TimeUtils.MILLIS_PER_HOUR)
    {
        return;
    }
    lastHour = clockMillis;
    timerable.hourIntervalTick();
}
```
需要拿当前时间和上一次时间做比较，传入主时钟timeClock。

接下来是满足秒、分、小时，那么就调用对应的秒tick、分tick、小时tick。

同理，其它生物对象、世界的时钟更新类似。

自然日时钟，更新机制类似。

## dailyTick玩家不在线，跨天处理机制

项目目前每天00:00调用一次dailyTick，玩家如果一直在线，那自然会调用没有什么问题。

但是，玩家如果下线了，那么当前场景tick不到玩家，玩家次数如何重新置0？

```java
/**
* 自然日Tick  :  每天的00:00调用一次；如果跨天0点时玩家不在线，那么玩家登录立即调用
*/
void dailyNaturalTick();
```
如果跨天0点时玩家不在线，那么玩家登录立即调用，如何做到？

### 1.玩家afterLoad传入上一次在线时间
@Override
public void afterLoad()
{
    ...
     // 初始化时钟，用玩家上次下线时间来初始化时钟，可以保证时钟不间断的tick
    long lastOffLineTime = getActorCommonModule().getLastOffLineTime();
    long nowMillis = getNowTickTime();
    if(lastOffLineTime <= 0 || lastOffLineTime > nowMillis)
    {
        lastOffLineTime = nowMillis;
        getActorCommonModule().setLastOffLineTime(lastOffLineTime);
        getActorCommonModule().setModify(true);
    }
    //传入上一次在线时间
    this.getTickTimer().init(lastOffLineTime);
    ...
}

玩家afterLoad传入上一次在线时间


### 2.对象上的时钟更新

AbstractBPScene下:
```java
tickObject()
{
     // 对象的时钟更新,并且调用时间间隔的tick
     // 传入场景主时钟
     object.tickUpdateClcok(sysClock);
}
```
场景基类下，tick对象时进行对象时钟更新，所以判定出跨天了，执行一次dailyTick。

到此，我们搞明白原理。


## 查看方法和类的调用栈，快捷键

crtl + alt + h
