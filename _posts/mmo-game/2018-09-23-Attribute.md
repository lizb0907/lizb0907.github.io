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
初始化战力，tempArr[等级][key]对应的战力


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