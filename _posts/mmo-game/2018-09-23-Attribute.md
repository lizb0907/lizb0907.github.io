---
layout: post
title: 属性系统
categories: Mmo-Game
description: mmo游戏里属性系统设计
keywords: Attribute，mmo，game
---

属性系统设计

**目录**

* TOC
{:toc}

## 属性系统设计考虑

### 1.初始化表数据到内存

#### 1.属性归类(Attribute表)

AttributeTypeDefine.init(reloadDictClazzs);

##### 1.初始化 当前值 - 最大值的对应

```java
public static void initCurrentMaxAttr()
{
    ...
    ...
    ...
}
```

```sh
public static int[] hasMaxAttrCurrentArray;                         //  当前值集合，有最大值的当前值集合

private static boolean[] currentAttrFlagArray;                      //  当前值集合

//这里的最大值还是指的attribute表的主键id,例如气血主键1，对应最大气血主键id等于2
//那么currentAttrToMaxAttrArray[1] = 2
public static int[] currentAttrToMaxAttrArray;                      //  当前-最大值 的映射，主要用于当前值不能大于最大值的计算

public static int[] maxAttrToCurrentAttrArray;                      //  最大值-当前 的映射，主要用于当前值不能大于最大值的计算

private static boolean[] maxAttrFlagArray;                          //  最大值集合
```

初始化当前值 - 最大值的对应，主要用于当前值不能大于最大值的计算。

这里的当前值指的是当前attribute表里配置的主键id,最大值指的表里配置的maxAttr列（attribute表里配置的主键id,只是和当前值不同id）


##### 2.初始化一级属性

//  当前值集合，有最大值的当前值集合

//  索引对应Attribute表主键id
private static boolean firstFlagArray[];                         

```java
private static void initFirstrAttr()
{
    firstFlagArray = new boolean[count];

    for (TIntObjectIterator<DictAttributes> iterator = DictAttributes.map.iterator(); iterator.hasNext(); )
    {
        iterator.advance();

        int attr = iterator.key();

        DictAttributes dict = iterator.value();

        if (dict.getIsFirstAttr() == 1)
        {
            //置为true
            firstFlagArray[attr] = true;
        }
    }
}
```
初始化表，如果配置的是一级属性组，那么对应firstFlagArray数组索引置为true。

索引对应Attribute表主键id。

目前暂时认为1级属性是有最大值的属性。


##### 3.初始化组属性，主属性和各个因子ABCD

```java
public static void initGroupMianFactorAttr()
{
    ...
    ...
    //就是groupAttrA()...有配置的attribute表主键id集合
    groupMainAttrList = groupMapping.keys();                                // 组属性集合
    groupMainAttrFlagArray = new boolean[AttributeTypeEnum.count];          // 组属性主属性标识集合
    //映射
    groupMainToFactorList = new TIntArrayList[AttributeTypeEnum.count];     // 组属性 主属性 -> 各个因子ABCD
    groupFactorToMainArray = new int[AttributeTypeEnum.count];              // 组属性 各个因子 -> 主属性
    Arrays.fill(groupFactorToMainArray, -1);
    for (TIntObjectIterator<int[]> iterator = groupMapping.iterator(); iterator.hasNext(); )
    {
        iterator.advance();
        //attribute表主键id
        int mianAttr = iterator.key();
        //ABCDEF值集合
        int[] factorArray = iterator.value();

        //当前是组属性的根据主键id做索引置为true
        groupMainAttrFlagArray[mianAttr] = true;

        groupMainToFactorList[mianAttr] = new TIntArrayList(factorArray.length);

        for (int i = 0, size = factorArray.length; i < size; i++)
        {
            //主属性(attribute表主键id)做数组索引,ABCDEF(该值也是表主键id)做数组值
            //主属性 -> 各个因子ABCD
            groupMainToFactorList[mianAttr].add(factorArray[i]);

            //各个因子 -> 主属性
            groupFactorToMainArray[factorArray[i]] = mianAttr;
        }
    }
    ...
    ...

}
```

什么是组属性？组属性ABCDEF有配置的就是组属性。


初始化:

///////////////////////////////
////////  组 属 性   /////////
///////////////////////////////

private static int[] groupMainAttrList;                         // 组属性主属性集合

private static boolean[] groupMainAttrFlagArray;                // 组属性主属性标识集合

private static TIntArrayList[] groupMainToFactorList;           // 组属性 主属性 -> 各个因子ABCD

private static int[] groupFactorToMainArray;                    // 组属性 各个因子 -> 主属性


##### 4.初始化元素攻击属性

```java
public static void initElementAttr()
{
    ...
    ...
     //冰火雷毒玄对应数组索引23456，对Attribute表主键id值21 24 27 30 33
    formulaAttackTypeToAttackAttrArray[FormulaAttackTypeEnum.ICE_ATTACK.getIndex()] = AttributeTypeEnum.ICE_ATTACK.getIndex();
    formulaAttackTypeToAttackAttrArray[FormulaAttackTypeEnum.FIRE_ATTACK.getIndex()] = AttributeTypeEnum.FIRE_ATTACK.getIndex();
    formulaAttackTypeToAttackAttrArray[FormulaAttackTypeEnum.THUNDER_ATTACK.getIndex()] = AttributeTypeEnum.THUNDER_ATTACK.getIndex();
    formulaAttackTypeToAttackAttrArray[FormulaAttackTypeEnum.POISON_ATTACK.getIndex()] = AttributeTypeEnum.POISON_ATTACK.getIndex();
    formulaAttackTypeToAttackAttrArray[FormulaAttackTypeEnum.DARK_ATTACK.getIndex()] = AttributeTypeEnum.DARK_ATTACK.getIndex();

    for (int attackType = FormulaAttackTypeEnum.ICE_ATTACK.getIndex(); attackType <= FormulaAttackTypeEnum.DARK_ATTACK.getIndex(); attackType++)
    {
        //对Attribute表主键id值21 24 27 30 33
        int attackAttr = formulaAttackTypeToAttackAttrArray[attackType];

        DictAttributes attactAttrDict = DictAttributes.getRecordById(attackAttr);

        //元素攻击对应元素防御  例如：21冰攻--元素攻击对应元素防御22--忽略冰防23--148冰伤害减免（受击万分比）--159冰伤害回血率(受击万分比)
        formulaAttackTypeToToDefenceArray[attackType] = attactAttrDict.getElementDefence();
        formulaAttackTypeToToIgnoreDefenceArray[attackType] = attactAttrDict.getElementIgnoreDefence();
        formulaAttackTypeToToAmplifyArray[attackType] = attactAttrDict.getElementAmplify();
        formulaAttackTypeToToDerateArray[attackType] = attactAttrDict.getElementDerate();
        formulaAttackTypeToToBeatHealArray[attackType] = attactAttrDict.getElementBeHeal();
    }
    ...
    ...
}

例如：21冰攻--元素攻击对应元素防御22--忽略冰防23--148冰伤害减免（受击万分比）--159冰伤害回血率(受击万分比)

```

##### 5.初始取值正负

```java
public static void initValuePositive()
{
//// 取值必须为正的属性集合
attrValuePositiveFlagArray = new boolean[AttributeTypeEnum.count];

for(int i = 0; i < AttributeTypeEnum.count; i++)
{
    attrValuePositiveFlagArray[i] = true;
}

for (TIntObjectIterator<DictAttributes> iterator = DictAttributes.map.iterator(); iterator.hasNext(); )
{
    iterator.advance();

    int attr = iterator.key();

    DictAttributes dict = iterator.value();

    // 先初始化为true, 因为大部分属性都必须为正.
    // 表中的注释是(是否可以为负数(1可以为负 0必须为正))
    attrValuePositiveFlagArray[attr] = true;

    // 特殊的可以为负值的, 设置为false
    if (dict.getIsCanNegative() == 1)
    {
        attrValuePositiveFlagArray[attr] = false;
    }
}
}
```
记录取值必须为正的属性集合和可以为负的属性集合（走表配置）


##### 6.初始化广播属性

根据Attribute表里配置的广播类型初始化到对应推送组里。

#### 2.初始化属性控制类

AttributeControl.init(reloadDictClazzs);

##### 1.初始化属性被影响组(AttrFactorConfig表)

```java
private static void initAttributeBeInfluence()
{
    attributeFirstInfluenceSecondDic = new int[DictAttrFactorConfig.map.size()][AttributeTypeEnum.count][];
    attributeSecondBeInfluenceFirstDic = new int[DictAttrFactorConfig.map.size()][AttributeTypeEnum.count][];

    //把表里的属性获取方法拿到(secondKey->firstKey->Method)
    Map<Integer, Map<Integer, Method>> map = new HashMap<>();
    //(firstKey->secondKey的HashSet集合)
    Map<Integer, Set<Integer>> map2 = new HashMap<>();

    Method[] methods = DictAttrFactorConfig.class.getMethods();

    for (Method m : methods)
    {
        if (m.getName().startsWith("getD_"))
        {
            String[] ar = m.getName().split("_");

            //一级属性
            int firstAttr = Integer.parseInt(ar[1]);//1
            //二级属性
            int secondAttr = Integer.parseInt(ar[2]);//2

            Map<Integer, Method> dic = map.get(secondAttr);

            if (dic == null)
            {
                dic = new HashMap<>();
                map.put(secondAttr, dic);
            }

            dic.put(firstAttr, m);

            Set<Integer> dic2 = map2.get(firstAttr);

            if (dic2 == null)
            {
                dic2 = new HashSet<>();
                map2.put(firstAttr, dic2);
            }

            dic2.add(secondAttr);
        }
    }

    TIntObjectIterator<DictAttrFactorConfig> it = DictAttrFactorConfig.map.iterator();

    while (it.hasNext())
    {
        it.advance();

        DictAttrFactorConfig dictAttrFactorConfig = it.value();

        //根据职业序号AttrFactorConfig主键获取二维数组
        int[][] tempInfluenceDic = attributeSecondBeInfluenceFirstDic[dictAttrFactorConfig.getId()];

        for (Entry<Integer, Map<Integer, Method>> entry : map.entrySet())
        {
            //初始化Map<Integer, Method>的长度 * 2
            int[] tempInfluenceArr = new int[entry.getValue().size() * 2];

            int i = 0;

            for (Entry<Integer, Method> kv2 : entry.getValue().entrySet())
            {
                //firstKey一级属性
                tempInfluenceArr[i] = kv2.getKey();

                try
                {
                    //通过反射获取影响的值
                    tempInfluenceArr[i + 1] = (int) (kv2.getValue().invoke(dictAttrFactorConfig));
                } catch (Exception e)
                {
                    e.printStackTrace();
                }

                i += 2;
            }

            // int[secondKey][] = int[firstKey, 影响值]
            tempInfluenceDic[entry.getKey()] = tempInfluenceArr;
        }

        //根据职业序号（AttrFactorConfig主键）获取二维数组
        int[][] firstToSecondArray = attributeFirstInfluenceSecondDic[dictAttrFactorConfig.getId()];

        for (Entry<Integer, Set<Integer>> kv : map2.entrySet())
        {
            int[] tempArr = new int[kv.getValue().size()];

            int i = 0;

            //v为二级属性
            for (int v : kv.getValue())
            {
                //通过组属性 各因子ABCD获取主属性(也就是二级属性的组属性)
                tempArr[i] = AttributeTypeDefine.getMainAttrByFactor(v);

                i++;
            }

            firstToSecondArray[kv.getKey()] = tempArr;
        }
    }
}
```

```java
/**
* 二级属性被影响组(每个二级属性受那些一级属性影响)(vocation->secondKey->[firstKey0,firstValue0,firstKey1,firstValue1])
*/
private static int[][][] attributeSecondBeInfluenceFirstDic;

/**
* 二级属性影响组(每个一级属性影响那些二级属性)(vocation->firstKey->[secondMainKey0,secondMainKey1])
*
* secondMainKey0的主属性
*/
private static int[][][] attributeFirstInfluenceSecondDic;
```

这里特别要注意:secondMainKey0的主属性，是通过组映射或取主属性的。

![](/images/posts/mmo_game/20.png)

```sh
表主键属性类型id其实也就是职业序号。

d_10_43 = 120000:
10是一级属性，43是二级属性，一级属性影响二级属性的值120000。

体质对生命的影响系数，万分比。
```

##### 2.通过配置表计算出每个职业，每个等级的基础属性(人物部分)

initAttributeLevel();

```java
private static void initAttributeLevel()
{
    ...
    ...
    TIntObjectIterator<DictAttrFactorConfig> it = DictAttrFactorConfig.map.iterator();
    while (it.hasNext())
    {
        it.advance();

        DictAttrFactorConfig dictAttrFactorConfig = it.value();

         //侠客职业系数统计等级字典(vocation->属性key->value) 万分比
        double[] knightBaseAttrArray = knightAttributeBaseDic[dictAttrFactorConfig.getId()];
        //主角属性职业统计等级字典(vocation->level->key->value)
        int[][] actorAttributeCoubtLevelArray = actorAttributeCountLevelDic[dictAttrFactorConfig.getId()];
        //等级-》属性
        double[][] tempArr = dLevelDic[dictAttrFactorConfig.getId()];

         // s组,初始属性
        for (Entry<Integer, Method> baseSEntry : baseSMap.entrySet())
        {
            int value = 0;

            try
            {
                Method baseSMethod = baseSEntry.getValue();
                value = (int) (baseSMethod.invoke(dictAttrFactorConfig));
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }

            //基础属性key值(对应Attribute表主键id)
            int attrIndex = baseSEntry.getKey();

            //迭代最大等级
            for (int i = 0; i < levelLen; ++i)
            {
                //累加上一个等级的基础值
                //tempArr[等级][key] = value累加上一个等级
                tempArr[i][attrIndex] += value;

                //先赋值一次,以防等级组里没有该属性
                actorAttributeCoubtLevelArray[i][attrIndex] = (int) (Math.ceil(tempArr[i][attrIndex]));
            }
        }

        // 侠客属性率
        for (Entry<Integer, Method> knightAttrRatioKEntry : knightAttrRatioKMap.entrySet())
        {
            int value = 0;

            try
            {
                Method method = knightAttrRatioKEntry.getValue();
                value = (int) (method.invoke(dictAttrFactorConfig));
            }
            catch (Exception e)
            {
                e.printStackTrace();
            }

            int attrIndex = knightAttrRatioKEntry.getKey();

            // 统一万分比
            double valuedouble = value / GameConstant.TEN_THOUSAND_DOUBLE;

            knightBaseAttrArray[attrIndex] = valuedouble;

        }

        ...
        ...

        //计算战力
        actorFightForceCountLevelDic = new int[DictAttrFactorConfig.map.size()][levelLen];

        for (int i = 0; i < DictAttrFactorConfig.map.size(); ++i)
        {
            //tempArr[等级][key]  level->key->value
            int[][] tempArr = actorAttributeCountLevelDic[i];

            for (int j = 0; j < levelLen; ++j)
            {
                //tempArr[等级][key]对应的战力
                actorFightForceCountLevelDic[i][j] = getAttributeFightForceForOnlySecond(tempArr[j], i, defaultAptitudes);
            }
        }
}
}
```
根据AttrFactorConfig表:

```java
//主角属性职业统计等级字典(vocation->level->key->value)
//当前等级基础等于累加上一个等级的基础值
//初始属性组,这里的key是表里配置的s_121的数字部分121
int[][] actorAttributeCoubtLevelArray = actorAttributeCountLevelDic[dictAttrFactorConfig.getId()];

//侠客职业系数统计字典(vocation->属性key->value) 万分比
double[] knightBaseAttrArray = knightAttributeBaseDic[dictAttrFactorConfig.getId()];

//初始化战力tempArr[等级][key]对应的战力
getAttributeFightForceForOnlySecond(tempArr[j], i, defaultAptitudes);
```

```java
/**
* 将组中所有一级属性转化为二级属性,然后计算战力(不考虑百分比,只初始化(角色/宠物)基础用)
* <p>
*     等级对应的基础属性组int[] attributes int[key] = value (例如：等级10对应的所有基础属性105、197、120...值，
*      * 那么int[105,197,120] = xxx)
* </p>
* int vocation 职业
* int[] aptitudes通用权重分母值 （千分之）组
*/
public static int getAttributeFightForceForOnlySecond(int[] attributes, int vocation, int[] aptitudes)
{
    if(aptitudes == null)
    {
        BPLog.BP_LOGIC.error("【评分计算错误】资质不存在，这里应该只有宠物评分调用",new RuntimeException("该异常只是为了打印堆栈"));
        return -1;
    }
    //二级属性被影响组(每个二级属性受那些一级属性影响)secondKey->[firstKey0,firstValue0,firstKey1,firstValue1])
    int[][] beInfluences = attributeSecondBeInfluenceFirstDic[vocation];

    //int[主属性] = 累加后端基础属性值
    int[] firstTs = new int[AttributeTypeEnum.count];

    //遍历当前等级的基础属性组值
    for (int i = attributes.length - 1; i >= 0; --i)
    {
        if (i < AttributeTypeEnum.count && attributes[i] != 0)
        {
            //获取主属性
            int groupKey = AttributeTypeDefine.getMainAttrByFactor(i);

            //如果有主属性并且是一级属性并且自身的key和主属性不相等
            if (groupKey > 0 && AttributeTypeDefine.isFirstAttr(groupKey) && i != groupKey)
            {
                // XXX !!!  属性的B,D也按A,C一样处理了，因为策划说不会配B,D
                //主属性为索引，累加所有的基础属性值
                firstTs[groupKey] += attributes[i];
            }
        }
    }

    //计算战力
    double re = 0.0;

    double v;

    for (int i = attributes.length - 1; i >= 0; --i)
    {
        v = attributes[i];

        if (i < AttributeTypeEnum.count)
        {
            //跳过1级属性
            int groupM = AttributeTypeDefine.getMainAttrByFactor(i);
            if (groupM > 0 && AttributeTypeDefine.isFirstAttr(groupM))
            {
                continue;
            }

            //二级属性被影响组(每个二级属性受那些一级属性影响)secondKey->[firstKey0,firstValue0,firstKey1,firstValue1])
            //所以ks代表会影响当前基础属性的所有一级属性
            int[] ks = beInfluences[i];

            if (ks != null)
            {
                int len = ks.length;

                for (int j = 0; j < len; j += 2)
                {
                    //key
                    int fType = ks[j];

                    //ks[j + 1]即对应的值

                    //根据公式累加战力
                    v += firstTs[fType] * (ks[j + 1] / GameConstant.TEN_THOUSAND_DOUBLE) * (aptitudes[fType] / GameConstant.THOUSAND_DOUBLE);
                }
            }
        }

        if (v != 0.0)
        {
            re += DictAttributes.getRecordById(i).getValue() / GameConstant.TEN_THOUSAND_DOUBLE * v;
        }
    }

    return (int) re;
}
```
初始化战力，tempArr[等级][key]对应的战力。

现在最新的侠客机制，已经不使用这个战力组了。

最新的机制，根据出战侠客计算玩家战力，也就是玩家战力等于出战侠客（队伍战力）。



##### 2.计算每个侠客星级带来的属性中B部分的加成（KnightLevelAttribute侠客属性表）

initAttributePerStar();

侠客星级和B属性统计等级字典(vocation->星级->属性key->value) 万分比,也就是初始化，星级改变引起的属性加成值

```java
/**
* 计算每个侠客星级带来的属性中B部分的加成 
*/
private static void initAttributePerStar()
{
    //B组,除了173外, 其他的属性变成+1的B属性
    //    力量加1从53-A属性变成54-B属性
    //    STRENGTH_A(53),
    //    STRENGTH_B(54),
    // 对应key索引数组值置为true
    boolean[] BList = new boolean[AttributeTypeEnum.count];
    Method[] methods = DictKnightLevelAttribute.class.getMethods();
    for (Method m : methods)
    {
        if (m.getName().startsWith("getS_"))
        {
            String[] ar = m.getName().split("_");

            int key = Integer.parseInt(ar[1]);// 属性id

            if(key != AttributeTypeEnum.RECOVERY_MP.getIndex())
            {
                // 除了173(回蓝)外, 其他的都会加1
                BList[key + 1] = true;
            }
        }
    }

    //侠客最大星级
    int maxStar = DictKnightStarUpData.getMaxStar() + 1;
    //侠客星级和B属性统计等级字典(vocation->星级->属性key->value) 万分比
    knightStarAttributeBDic = new double[DictAttrFactorConfig.map.size()][maxStar][AttributeTypeEnum.count];
    
    TIntObjectIterator<DictAttrFactorConfig> it = DictAttrFactorConfig.map.iterator();

    while (it.hasNext())
    {
        it.advance();

        DictAttrFactorConfig dd = it.value();

        //星级->属性key->value
        double[][] starB = knightStarAttributeBDic[dd.getId()];
        
        for(int star = 1; star< maxStar; star++)
        {
            //根据侠客id和星级获取数据
            DictKnightStarUp dataByKnightIdAndStar = DictKnightStarUpData.getDataByKnightIdAndStar(dd.getId(), star);
            if(dataByKnightIdAndStar == null)
            {
                continue;
            }
            
            double[] ds = starB[star];
            //长度等于枚举长度AttributeTypeEnum.count
            int len = ds.length;
            for(int i = 0; i< len; i++)
            {
                if(BList[i])
                {
                    //成长系数(万分比)
                    //    力量加1从53-A属性变成54-B属性
                    //    STRENGTH_A(53),
                    //    STRENGTH_B(54),
                    // 假设遍历刚好到了54,那么星级改变，这个54作为key对应的成长系数会改变
                    ds[i] = dataByKnightIdAndStar.getGrowRatio();
                }
            }
        }
    }
}
```
说白了，就是KnightLevelAttribute表配置的key值，除了173回蓝外, 其他的属性变成+1的B属性，

侠客随着星级改变成长系数会改变-成长系数值(万分比)对应KnightStarUp表，

然后职业序号-》星级-》B属性key-》成长系数，做映射。

更通俗的讲：

double[] starBValue = AttributeControl.getKnightStarAttributeBDic()[fightVocation][star];

根据侠客职业和星级就可以获取所有星级改变然后属性会变的值。

#### 3.总结

这里的侠客职业，现在修改了，其实就是侠客的属性类型ID（侠客表的attrTypeID）。

当前值就是Attribute表里配置了maxAttr,例如:当前主键id等于1动态当前生命，对应的最大值maxAttr = 2 生命上限(总)。

其它的就是初始化属性组字典:

1.初始化当前值和最大值---AttributeTypeDefine.init(reloadDictClazzs);

2.初始化基础属性（初始化属性控制类），为后面升级，升星，添加，减少，改变侠客基础属性做准备--AttributeControl.init(reloadDictClazzs);

### 2.属性推送基本流程

#### 1.创建新号属性推送过程

public Actor createActor(BPLoginObject loginObject, String accountID, long actorID,
                            ActorDataCache actorDataCache, int birthX, int birthY, int birthZ,
                            int rotation, int defaultKnightID)
{
    ...
    actor.afterLoad();

    ...
    int result = actor.getKnightModule().addAndUnlockKnight(defaultKnightID);
    
}

##### 1.actor.afterLoad()下attribute.afterLoad()

ActorAttributeModule.java

```java
@Override
public void afterLoad()
{
    ...
    ...

    //设置设置战斗职业，属性影响组，资质，都是从缓存在内存里的字典里拿取
    getActor().getAttributeLogic().setFightVocation(commonModule.getFightVocationIndex());
    
    //根据职业和等级从内存中添加基础属性(怒气啥的,AttrFactorConfig表的S组)
    getActor().getAttributeLogic().addAttributes(AttributeControl.getActorAttributeCountLevelDic()[commonModule.getFightVocationIndex()][commonModule.getLevel()]);

    //刷新属性
	getActor().getAttributeLogic().refreshAttributes();

    //刷新战力
	refreshFightForce();	
    ...
}
```
设置设置战斗职业，属性影响组。

刷新战力。

添加基础属性和刷新属性其实不用在这里操作，因为后面添加侠客了，会重新计算。


##### 2.addAndUnlockKnight(defaultKnightID)添加侠客属性计算

actor.getKnightModule().addAndUnlockKnight(defaultKnightID);

```java
/**
* 添加并解锁侠客
* @param knightID 要添加的侠客id
* @return
*/
public int addAndUnlockKnight(int knightID)
{
    //这里新new了一个Knight实体，并初始化挂在AbstractCharacter类下的模块(创建属性模块并初始化属性组)
    Knight knight = addKnight(knightID);

    //添加天赋技能,计算战力,更新基础属性。
    //回满蓝血霸体（当前属性,需要存库）
    int result = unlockKnight(knight);
}
```
这里新new了一个Knight实体，并初始化挂在AbstractCharacter类下的模块(所以需要重新创建属性模块并初始化属性组)。

添加天赋技能,计算战力,更新基础属性。回满蓝血霸体（当前属性,需要存库）。

```java
@Override
public void init()
{
    ...
    attributeModule = createAttributeLogic();

    this.getAttributeLogic().init(hasAptitude());
    ...
}
```

创建属性模块并初始化属性组长度。

```java
/**
* 解锁侠客
*
* @param knight     侠客实体
*/
private int unlockKnight(Knight knight)
{
    ...
    ...
    // 计算战力
    calcKnightFightForce(knight, true);

     //更新基础属性
    updateBaseAttributeByAddKnight(knightId);

     //回满蓝血霸体
    knight.getAttributeLogic().fillHpMp();
    ...
    ...
}
```
所以每次侠客增加会进行如下操作：

更新基础属性（相当于侠客增加）,也可以算是总的属性会随等级变化而变化。

回满蓝血霸体，当前属性，下线需要存库。

#### 2.侠客基础属性的计算和推送

##### 1.等级改变的时候, 基础属性变化


```java
/**
* 等级改变的时候, 基础属性变化
*
* @param beforeLevel 之前的等级
* @param afterLevel 现在的等级
*/
public void baseAttrOnLevelChange(int beforeLevel, int afterLevel)
{
    ...
    //升级重置侠客等级的基本属性
    resetKnightLevelAttr(beforeLevel, afterLevel);

    if (beforeLevel > 0)
    {
        changeKnightBaseAttr_LevelChange(beforeLevel, star, KnightConstant.KNIGHT_MINUS_ATTR_TYPE);
    }
    //等级变化综合属性
    changeKnightBaseAttr_LevelChange(afterLevel, star, KnightConstant.KNIGHT_ADD_ATTR_TYPE);

    //计算并刷新推送属性组
    getAttributeLogic().refreshAttributes();
}
```
升级重置侠客等级的基本属性是根据主角基础属性职业S组,也就是计算DictAttrFactorConfig中的属性。

等级变化综合属性KnightLevelAttribute表，DictAttrFactorConfig表中K组。。。等等

计算并刷新推送属性组。

```java
/**
* 升级重置侠客等级的基本属性
*
* @param beforeLevel 之前的等级
* @param afterLevel 之后的等级
*/
private void resetKnightLevelAttr(int beforeLevel, int afterLevel)
{
    ...
    ...
    if(beforeLevel > 0)
    {
        int[] attr = AttributeControl.getActorAttributeCountLevelDic()[fightVocation][beforeLevel];
        getAttributeLogic().minusAttributes(attr);
    }

    if(afterLevel > 0)
    {
        int[] attr = AttributeControl.getActorAttributeCountLevelDic()[fightVocation][afterLevel];
        getAttributeLogic().addAttributes(attr);
    }
    ...
    ...
}
```
升级重置侠客等级的基本属性是根据主角基础属性职业S组,也就是计算DictAttrFactorConfig中的属性。

先减去之前等级的，加上当前等级的。

```java
/**
* 等级变化
* @param level
* @param star 
* @param type 1减少，2增加
*/
private void changeKnightBaseAttr_LevelChange(int level, int star, int type)
{
    // 侠客基础属性 = attrFactor + 基础属性（等级X职业系数）*(1+（星级系数+收集加成)/10000.0), 只需要各自算出A的总和, B的总和. 然后带入AttributeTypeEnum的公式计算 
    // (ABCD含义,Total=(A*(1+B/10000.0)+C)*(1+D/10000.0))   
    // 其中 attrFactor 部分是策划默认加的, 公式中并未给出
    int fightVocation = getFightVocation();

    //根据侠客和等级获取KnightLevelAttribute初始属性
    int[] attr = DictKnightLevelAttributeData.getInitAttrByLevel(level);

    double[] levelRatio =  AttributeControl.getKnightAttributeBaseDic()[fightVocation];
    double[] starBValue = AttributeControl.getKnightStarAttributeBDic()[fightVocation][star];
    for(int attrType = 0; attrType < AttributeTypeEnum.count; attrType++)
    {
        //根据公式计算值
        int value = calBaseAttr(attr[attrType], levelRatio[attrType], starBValue[attrType]);
        
        int collectType = DictKnightCollectAttrData.getCollectTypeByAttribute(attrType);
        if(collectType > 0)
        {
            int collectRatioValue = getMasterActor().getKnightModule().getCollectAttrRatio(collectType);
            value += KnightModule.calMaxAndCollectTypePartValue(collectRatioValue);
        }
        
        if(value > 0)
        {
            if (type == KnightConstant.KNIGHT_MINUS_ATTR_TYPE)
            {
                getAttributeLogic().minusOneAttribute(attrType, value);
            }
            else
            {
                getAttributeLogic().addOneAttribute(attrType, value);
            }
        }
    }
}
```
等级变化综合属性KnightLevelAttribute表，DictAttrFactorConfig表中K组。。。等等


##### 2.设置属性值

设置当个属性值

```java
/**
* 设置单个属性值
*/
public void setOneAttribute(int type, int value)
{
    if(value < 0 && AttributeTypeDefine.isMustAttrValuePositive(type))
    {
        value = 0;
    }

    this.attributes[type] = value;

    //是否需要置脏推送标志
    makeOneDirty(type);
}
```
增加、减少属性都和设置当个属性值类似，会根据属性类型type判断是否需要置脏推送。

```java
private void makeOneDirty(int type)
{
    ...
     // 是否需要置脏推送标志，表类配置了通知自己、通知他人、npc广播字段为1，当前属性需要广播
    boolean[] allMaybeSendSet = AttributeTypeDefine.needDispatchAttrAllSet;
    if (allMaybeSendSet[type])
    {
        this.dispatchDirty = true;
    }

    // 是否当前属性,如果是当前属性变化了,置相关状态并且return返回，不进行组属性操作。
    // 那么在getAttributeLogic().refreshAttributes()时会调用countAttributes()计算
    if (AttributeTypeDefine.isCurrentAttr(type))
    {
        this.attributeModifications[type] = true;
        this.attributeModified = true;
        return;
    }

      // 如果当前设置的type属性改变了，type属于组属性，因子变了主属性也需要修改，那么组属性需要重新计算
    // 如果是组属性和当前属性互斥(组属性就是ABCDEF字段有值)
    // 通过因子ABCDEF取主属性
    int groupMainAttr = AttributeTypeDefine.getMainAttrByFactor(type);
    if (groupMainAttr > 0)
    {
        //主属性刚好等于当前type属性,主属性不能set设置，一定是通过其他因子计算出来
        if (groupMainAttr == type)
        {
            //【属性错误】不能设置组属性主属性也就是总值
            if(BPGlobals.getInstance().isDebugVersion())
            {
                BPLog.BP_LOGIC.warn("【属性错误】不能设置组属性主属性也就是总值 [mianAttr] {}",groupMainAttr,new RuntimeException("该异常用于打印堆栈"));
                return;
            }
        }

        // 需要推送,当前属性需要广播
        if (allMaybeSendSet[groupMainAttr])
        {
            this.dispatchDirty = true;
        }

        // 置脏,那么在getAttributeLogic().refreshAttributes()时会调用countAttributes()计算
        this.attributeModifications[groupMainAttr] = true;
        this.attributeModified = true;

        // 如果最大值属性(五项值结果中有一部分是最大值)变了，那么同样修改最大值影响的当前值
        // type变化导致上面主属性置脏了说明主属性也改变了，那么主属性影响的当前属性也要修改所以置脏
        int curAttr = AttributeTypeDefine.getCurrentAttrByMaxAttr(groupMainAttr);
        if (curAttr > 0)
        {
            this.attributeModifications[curAttr] = true;
        }

        // 置脏一级属性影响的二级属性
        // 如果主属性是一级属性影响的二级属性不为空
        int[] firstInfluenceSecondArray = this.attributeInfluenceDic[groupMainAttr];

        if (firstInfluenceSecondArray != null)
        {
            for (int i = firstInfluenceSecondArray.length - 1; i >= 0; --i)
            {
                int secondAttr = firstInfluenceSecondArray[i];

                //二级属性被一级属性影响，所以置脏重新计算
                this.attributeModifications[secondAttr] = true;

                // 如果二级属性最大值属性(五项值结果中有一部分是最大值)变了，那么同样修改最大值影响的当前值
                // 上面置脏说明二级属性修改了，那么二级属性影响也会影响当前属性也要计算
                int secondCurrentAttr = AttributeTypeDefine.getCurrentAttrByMaxAttr(secondAttr);
                if (secondCurrentAttr != 0)
                {
                    //当前属性为curAttr
                    this.attributeModifications[curAttr] = true;
                }
            }
        }
    }
    ...

}
```
是否需要置脏推送标志，表类配置了通知自己、通知他人、npc广播字段为1，当前属性需要广播

是否当前属性,如果是当前属性变化了,置相关状态并且return返回，不进行组属性操作。
那么在getAttributeLogic().refreshAttributes()时会调用countAttributes()计算。

当前修改的type是组属性，那么主属性需要重新计算，并且主属性影响的当前属性重新计算，

如果主属性又是一级属性，那么受一级属性影响的二级属性需要重新计算，二级属性影响的当前属性也要重新计算，

全部置脏，等待调用getAttributeLogic().refreshAttributes()时重新计算。


##### 3.计算并刷新推送属性组

getAttributeLogic().refreshAttributes()

```java
/**
* 计算并刷新推送属性组
*/
public void refreshAttributes()
{
    //属性修改标记要先计算
    if (this.attributeModified)
    {
        countAttributes();
    }

    //需要推送
    if (!this.dispatchDirty)
    {
        return;
    }

    this.dispatchDirty = false;

    //属性改变回调,同时推送
    dispatchAttributeChange();
}
```

```java
/**
* 计算属性(不推送)
*/
public void countAttributes()
{
    if (!this.attributeModified)
    {
        return;
    }

    this.attributeModified = false;

    //修改标记(只用于当前属性和组属性)
    boolean[] attributeModifications = this.attributeModifications;

    // 获取组属性主属性集合
    int[] groupList = AttributeTypeDefine.getGroupMainAttrList();
    for (int i = 0, iSize = groupList.length; i < iSize; ++i)
    {
        int type = groupList[i];

        //修改了
        if (attributeModifications[type])
        {
            attributeModifications[type] = false;

            // 根据公式计算单个的组属性
            countOneGroupAttribute(type);
        }
    }

    //刷当前属性
    int[] attributes = this.attributes;
    //当前值集合，有最大值的当前值集合
    int[] hasMaxAttrCurrentArray = AttributeTypeDefine.hasMaxAttrCurrentArray;
    for(int i = 0 , size = hasMaxAttrCurrentArray.length; i < size; i++)
    {
        //当前属性索引
        int curAttr = hasMaxAttrCurrentArray[i];

        int maxAttr = AttributeTypeDefine.getMaxAttrByCurrent(curAttr);

        //修改了
        if (attributeModifications[curAttr])
        {
            //最新的当前值
            int nowValue = attributes[curAttr];
            //允许最大值
            int maxValue = attributes[maxAttr];

            //范围
            if (nowValue > maxValue)
            {
                nowValue = maxValue;
                attributes[curAttr] = nowValue;
            }

            if (nowValue < 0  && AttributeTypeDefine.isMustAttrValuePositive(curAttr))
            {
                attributes[curAttr] = 0;
            }
        }
    }
}
```
迭代获取组属性主属性集合，迭代当前值集合（有最大值的属性），然后根据置脏数据判断是否修改了，如果修改了，

重新计算值，这个方法是不进行推送给前端的。

```java
//属性改变回调,同时推送
private void dispatchAttributeChange()
{
    int type;
    int value;

    //改变属性数量
    int num = 0;

    //最新的属性组（最新值已经计算过了）
    int[] attributes = this.attributes;
    //上次的属性组(推送用)
    int[] lastAttributes = this.lastDispatches;
    //上次的属性组(用于记录两次属性变化事件)
    int[] lastAttributesRecord = this.lastAttributes;
    //改变组(也做临时组)
    int[] changeList = this.changeList;
    // 锁定属性组(不能锁计算因子)
    int[] lockAttributes = this.lockAttributes;
    //锁定属性状态组(不能锁计算因子)（这里是指例如帮派货运这种的，调用锁定方法，锁住了速度等等）
    boolean[] lockAttributesStatus = this.lockAttributesStatus;

    //所有有关推送的属性
    int[] needDispatchList = AttributeTypeDefine.needDispatchAttrAllList;
    //迭代需要推送的所有属性
    for (int i = needDispatchList.length - 1; i >= 0; --i)
    {
        //需要推送的属性索引
        type = needDispatchList[i];
        //当前值
        value = attributes[type];
        if (lockAttributesStatus[type])
        {
            //直接让当前值等于之前缓存的锁定属性值（覆盖当前值）
            value = lockAttributes[type];
        }

        //如果上次推送给客户端的值和当前value不相等,说明改变了
        if (lastAttributes[type] != value)
        {
            //更新上一次推送组
            lastAttributes[type] = value;
            //并记录到改变组changeList
            changeList[num++] = type;
        }
    }

    if (num > 0)
    {
        //改变组changeList,数量，上次的属性组(用于记录两次属性变化事件)
        //属性改变回调
        onAttributesChange(changeList, num,lastAttributesRecord);
    }

    //上次的属性组(推送用),拷贝给上次的属性组(用于记录两次属性变化事件)
    System.arraycopy(lastAttributes,0,lastAttributesRecord,0,lastAttributes.length);

}
```



##### 4.自定义属性推送组？






