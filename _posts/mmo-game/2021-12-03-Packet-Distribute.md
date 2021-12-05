---
layout: post
title: 游戏里的消息派发机制
categories: Mmo-Game
description: 游戏里的消息派发机制
keywords: packet,distribute,mmo,game
---

游戏里的消息派发机制

**目录**

* TOC
{:toc}

## 概要

```sh
1.游戏里前后端通信采用的是protobuf协议，提前约定好消息号，每个消息号是一个short类型的值。该值表示
  特定类的某个方法。那么，我们又是如何根据消息号，将消息转发到对应类的方法里呢？这里用到的就是本次
  要讲的消息派发机制。

2.消息派发机制实现一般用两种：
  a.反射
  b.javassist提升发射效率
```

## 消息派发机制实现


### 1.反射

#### 1.注解实现

```java
/** 
网络消息包处理模块
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Component
public @interface PacketModule
{
}
```

类的注解"@PacketModule"


```java
/** 
用户网络包处理方法注解
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface PacketHandler
{
    /**
     * 网络消息包的 packet id
     * @return
     */
    short value() default -1;
}
```
方法的注解"@PacketModule"

```java
/**
 * 坐骑客户端消息处理类
 */
@PacketModule
public class RidePacketHandler
{

    /**
     * 装备坐骑
     *
     * @param packet 客户端请求消息包
     * @param actor  玩家
     */
    @PacketHandler(PacketIDConst.CSRideInstall)
    public void rideInstall(BPRide.CSRideInstall packet, Actor actor)
    {
        int result = actor.getActorRideModule().rideInstall(packet.getRideId(), false);

        BPRide.SCRideInstall.Builder resp = BPRide.SCRideInstall.newBuilder();
        resp.setResult(result);
        resp.setInstallRideId(actor.getActorRideModule().getInstallRideId());
        actor.sendPacket(PacketIDConst.SCRideInstall, resp.build());
    }
}
```
一个完整的消息类，类和方法上有对应的注解

#### 2.根据注解构建消息缓存

```java
public void registerPacket() throws Exception
{
    Map<String, Object> modules = SpringContainer.getInstance().getCtx().getBeansWithAnnotation(PacketModule.class);
    for (Object object : modules.values())
    {
        Method[] methods = object.getClass().getMethods();
        for (Method method : methods)
        {
            if (method.isAnnotationPresent(PacketHandler.class))
            {
                PacketHandler handler = method.getAnnotation(PacketHandler.class);
                short id = handler.value();
                Dispatcher.registerPacket(id, object, method);
                registerPacketFromReflectLoader(id, method.getParameters()[0].getType().getCanonicalName(),
                                object.getClass().getName(), method.getName(),
                                method.getParameters()[1].getParameterizedType().getTypeName(), method.getParameterCount());
            }
        }
    }
}
```
运营spring获取所有带有"@PacketModule"注解的类和带有"@PacketHandler"的方法

```java
public static void registerPacket(Short id, Object moduleClass, Method method) throws Exception
{
    if (id >= MAX_ID)
    {
        throw new Exception("id invalid:" + id);
    }
    if (packetMethods[id] != null)
    {
        throw new Exception("duplication id:" + id);
    }

    try
    {
        packetMethods[id] = method.getParameterTypes()[0].getDeclaredMethod("parseFrom", byte[].class);
        packetModules[id] = new PacketModule(moduleClass, method);
    }
    catch (Exception e)
    {
        throw e;
    }
}
```
packetMethods[]消息id对应的方法值，packetModules[]消息id对应的类和方法实例

```java
 public static void dispatchClientPacket(InterfaceObject object, ClientNetPacket netPacket)
{
    int packetID = netPacket.getPacketID();

    if (packetID < 0 || packetID > MAX_ID)
    {
        CoreLog.CORE_COMMON.error("dispatch packet id:{} < 0 or > {}", packetID, MAX_ID);
        return;
    }

    PacketModule module = packetModules[packetID]; //存的实例
    Method method = packetMethods[packetID];  //方法的内容
    try
    {
        if (method == null)
        {
            CoreLog.CORE_COMMON.warn("handle method not found packetID:{}", packetID);
            object.setException(ObjectExceptionEnum.HANDLE_PACKET);
            return;
        }
        if (module == null)
        {
            CoreLog.CORE_COMMON.warn("moudle not found, packetID:{}", packetID);
            object.setException(ObjectExceptionEnum.HANDLE_PACKET);
            return;
        }
        //将内容反射传入到对应类的方法
        module.getMethod().invoke(module.getObject(), packet, object);
    }
    catch (Exception e)
    {
        CoreLog.CORE_COMMON.error("dispatch client packet:{} error:{}",packetID, ExceptionUtils.exceptionToString(e));
        e.printStackTrace();
        object.setException(ObjectExceptionEnum.HANDLE_PACKET);
    }
}
```
根据反射将消息进行分发，到此，我们就将运用反射进行的消息派发机制总结完毕了。


### 2.javassist提升发射效率

```sh
待补充
```


### 对比

```sh
待补充
```
