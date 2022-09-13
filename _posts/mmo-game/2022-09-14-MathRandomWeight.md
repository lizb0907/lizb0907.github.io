---
layout: post
title: 根据权重随机算法
categories: Mmo-Game
description: 根据权重随机算法
keywords: 根据权重随机算法，mmo
---

根据权重随机算法

**目录**

* TOC
{:toc}

## 根据权重，随机多个值,可重复

```sh
 /**
  * 根据权重，随机多个值,可重复
  * <p>
  * ( 随机数组和权重数组的长度不能小于随机次数 )
  *
  * @param values    值数组 与权重数组一一对应
  * @param weightArr 权重数组
  * @param num       要随机出个数
  * @return 如果错误返回null
  */
public static int[] randomManyRepeat(int[] values, int[] weightArr, int num)
{
    if (values == null || weightArr == null || values.length <= 0 || values.length <= num || weightArr.length != values.length)
    {
        BPLog.BP_SYSTEM.error("【随机错误】用于 '根据权重，随机多个值,可重复' 的随机条件错误 [values] {} [weightArr] {} [NUM] {}", values, weightArr, num);
        return null;
    }

    // 计算权重数组SUM
    int length = weightArr.length;
    int allWeight = 0;
    for (int i = length - 1; i >= 0; i--)
    {
        allWeight += weightArr[i];
    }

    // 随机过程
    int[] temp = new int[num];
    for (int i = 0; i < num; i++)
    {
        // 单次随机
        int randomInt = randomInt(allWeight);
        //迭代权重数组
        for (int j = 0; j < length; j++)
        {
            int weight = weightArr[j];
            if (randomInt < weightArr[j])
            {
                temp[i] = values[j];
                break;
            }
            else
            {
                //如果当前随机值大于的权重，就说明当前随机的值不在第一个权重区间，并且一定超出第一个区间。
                //所以需要减去第一个区间的权重，判断多余的值是落在剩余后面的哪个区间。
                //依次这么重复，一定能判断随机的值最终落在哪个区间。
                //举例:
                //values[1,2,3] 对应的权重weightArr[50,10,20]
                //如果randomInt（80）随机值为70：
                //   a.  70 不小于 50， 则70-50 = 20
                //   b.  20 不小于 10， 则20-10 = 10
                //   c.  10 小于 20， 则命中区间，所以本次随机的值就是权重20对应的3
                //
                randomInt -= weight;
            }
        }
    }
    return temp;
}
```

```sh
/**
* 随机一个整数(0<=res<n)
*
* @param n
* @return <=0 则返回0
*/
public static int randomInt(int n)
{
    if (n <= 0)
    {
        return 0;
    }
    ThreadLocalRandom random = ThreadLocalRandom.current();
    return random.nextInt(n);
}
```









