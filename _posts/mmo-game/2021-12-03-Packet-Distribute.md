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
  a.将消息ID对应的Method放入一个数组中，利用消息ID为数组下标，找到方法调用invoke执行消息对应的方法。
  b.利用javassist，在起服时的时候动态生成代码，将消息ID作为switch的key,case执行具体方法，过程中没
    有用到反射。
```

## 消息派发机制实现


### 1.调用invoke执行消息对应的方法

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


### 2.javassist动态生成代码组件

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
                registerPacketFromReflectLoader(id, method.getParameters()[0].getType().getCanonicalName(),
                                object.getClass().getName(), method.getName(),
                                method.getParameters()[1].getParameterizedType().getTypeName(), method.getParameterCount());
            }
        }
    }
}
```
根据注解扫描，在加载器加载的时候注册packet的一些信息


```java
/**
* 生成网络消息包分发处理方法
* @throws NotFoundException
* @throws CannotCompileException
* @throws IllegalAccessException
* @throws InstantiationException
*/
public void generatePacketDispatcherMethod() throws Exception
{
    BPLog.BP_SYSTEM.info("开始生成消息处理switch case代码!");
    ClassPool classPool = ClassPool.getDefault();

    CtClass bpDispatcher = classPool.get("com.game2sky.application.common.BPDispatcher");
    CtMethod method = bpDispatcher.getDeclaredMethod("handlePacket");

    // 动态写method里的代码(BPDispatcher的handlePacket方法里的代码)
    StringBuilder stringBuilder = new StringBuilder();
    stringBuilder.append("switch(packetID){\n");

    //handler class的类名对应map
    //key:packetID, value:handler class name
    TIntObjectHashMap<String> idHandlerClassNameMap = BPReflectLoader.getIdHandlerClassNameMap();
    //handler class 的方法里参数的interface object的类型名
    //key:packetID, value:handler class param objectName
    TIntObjectHashMap<String> idHandlerParamObjectNameMap = BPReflectLoader.getIdHandlerParamObjectNameMap();
    //key:packetID, value:protobuf message name
    TIntObjectHashMap<String> idProtobufMessageNameMap = BPReflectLoader.getIdProtobufMessageNameMap();
    //key:packetID, value: handler class Method name
    TIntObjectHashMap<String> idHandlerMethodNameMap = BPReflectLoader.getIdHandlerMethodNameMap();
    //key:packetID, value: params count
    TIntIntHashMap idHandlerMethodParamsCountMap = BPReflectLoader.getIdHandlerMethodParamsCountMap();

    TIntObjectIterator<String> iterator = idHandlerMethodNameMap.iterator();
    while (iterator.hasNext())
    {
        iterator.advance();
        int packetID = iterator.key();

        // 过滤掉需要2个参数以上的handler方法,也就是目前的处理服务器内部包的方法
        if (idHandlerMethodParamsCountMap.get(packetID) != 2)
        {
            continue;
        }

        //value:protobuf message name
        String protobufMessage = idProtobufMessageNameMap.get(packetID);
        if (StringUtils.isBlank(protobufMessage))
        {
            throw new NotFoundException("packetID:" + packetID +  " protobuf message 名字为空");
        }

        //value:handler class param objectName
        String objectType = idHandlerParamObjectNameMap.get(packetID);
        if (StringUtils.isBlank(objectType))
        {
            throw new NotFoundException("packetID:" + packetID + " object type 名字为空");
        }

        //value:handler class name
        String handler = idHandlerClassNameMap.get(packetID);
        if (StringUtils.isBlank(handler))
        {
            throw new NotFoundException("packetID:" + packetID + " handler 名字为空");
        }

        //handler class Method name
        String methodName = idHandlerMethodNameMap.get(packetID);

        stringBuilder.append("case ");
        stringBuilder.append(packetID);
        stringBuilder.append(":\n{\n");

        stringBuilder.append(protobufMessage);
        stringBuilder.append(" message = ");
        stringBuilder.append(protobufMessage);
        stringBuilder.append(".parseFrom(body);\n");
        stringBuilder.append("if (message == null)\n" +
                "                {\n" +
                "                    com.game2sky.core.log.CoreLog.CORE_COMMON.error(\"handle packet, protobuf parseFrom return null, id:{}\" + \"packetID\");\n" +
                "                    return;\n" +
                "                }\n");
        stringBuilder.append("((");
        stringBuilder.append(handler);
        stringBuilder.append(")(");
        stringBuilder.append("packetHandlerObjectArray[packetID])).");
        stringBuilder.append(methodName);
        stringBuilder.append("(message, (");
        stringBuilder.append(objectType);
        stringBuilder.append(")object);\n");
        stringBuilder.append("break;}\n");
    }

    stringBuilder.append("default:{break;}}\n");

    BPLog.BP_SYSTEM.info("生成消息处理switch case代码 string 已生成, 开始redefine");
    method.insertBefore(stringBuilder.toString());

    Class clazz = Class.forName("com.game2sky.application.common.BPDispatcher");
    ClassDefinition classDefinition = new ClassDefinition(clazz, bpDispatcher.toBytecode());
    int result = ClassReloader.redefineClass(classDefinition);
    if (result < 0)
    {
        throw new Exception("重定义生成的dispatcher代码失败");
    }
    BPLog.BP_SYSTEM.info("生成消息处理switch case代码已完成");
    BPDispatcher dispatcher = (BPDispatcher) clazz.newInstance();
    CoreGlobals.getInstance().setDispatcher(dispatcher);
}
```
生成网络消息包分发处理方法，也就是动态拼接handlePacket里的方法代码

```java
public static void handlePacket(int packetID, byte[] body, InterfaceObject object) throws InvalidProtocolBufferException
{
    switch (packetID)
    {
        case 1:
        {
            BPRide.CSRideInstall message = BPRide.CSRideInstall.parseFrom(body);
            if (null == message)
            {
                //打印错误
                return;
            }
            ((RidePacketHandler)(packetHandlerObjectArray[1])).rideInstall(message, (Actor) object);
        }
        case 2:
        {

        }
        default:
        {
            break;
        }

    }
}
```
以坐骑RidePacketHandler下的rideInstall()方法来说明，handlePacket里的方法代码是动态生成的，大概就是上面的代码。

```java
/**
 */
public class BPDispatcher extends Dispatcher
{
    /**
     * 客户端网络包handler对象的数组 数组下标为消息id
     */
    private static Object[] packetHandlerObjectArray = new Object[MAX_ID];

    static
    {
        int length = packetHandlerObjectArray.length;
        for (int i = 0; i < length; i++)
        {
            packetHandlerObjectArray[i] = null;
        }
    }

    public static void dispatchClientPacket(InterfaceObject object, ClientNetPacket netPacket)
    {
        if (object == null)
        {
            CoreLog.CORE_COMMON.error("on dispatch object is null");
            return;
        }

        if (netPacket == null)
        {
            CoreLog.CORE_COMMON.error("on dispatch netPacket is null");
            return;
        }

        int packetID = netPacket.getPacketID();
        byte[] body = netPacket.getBody();

        if (body == null)
        {
            CoreLog.CORE_COMMON.error("客户端发送了一个空包:{}", packetID);
            object.setException(ObjectExceptionEnum.HANDLE_PACKET);
            return;
        }

        try
        {
            handlePacket(packetID, body, object);
        }
        catch (Exception e)
        {
            CoreLog.CORE_COMMON.error("dispatch client packet:{} error:{}",packetID, ExceptionUtils.exceptionToString(e));
            e.printStackTrace();
            object.setException(ObjectExceptionEnum.HANDLE_PACKET);
        }
    }

    public static void handlePacket(int packetID, byte[] body, InterfaceObject object) throws InvalidProtocolBufferException
    {

    }

    /**
     * 在加载器加载的时候注册packet的一些信息
     * @param id
     * @param protobufName
     * @param packetHandlerName
     * @param methodName
     * @throws Exception
     */
    public void registerPacketFromReflectLoader(int id, String protobufName,
                                                String packetHandlerName,
                                                String methodName,
                                                String objectName) throws Exception
    {
        if (id >= MAX_ID)
        {
            throw new Exception("id invalid:" + id);
        }

        Class clazz = Class.forName(packetHandlerName);
        packetHandlerObjectArray[id] = clazz.newInstance();
    }
}

```
如上dispatchClientPacket就是消息分发方法。


### 性能对比

```sh
1.利用javassist，在起服时的时候动态生成代码，将消息ID作为switch的key,case执行具体方法，过程中没
  有用到反射，肯定远大于用反射的性能。

```
