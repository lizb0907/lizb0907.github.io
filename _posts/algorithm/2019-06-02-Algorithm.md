---
layout: post
title: 常用算法
categories: Algorithm
description: 数据结构算法学习记录，一些常用算法实现
keywords: Algorithm
---

牛客网左神算法课程学习记录，常用的一些算法自己再实现一遍，为刷leetcode准备。



**目录**

* TOC
{:toc}

## 归并排序


### MergeSort
```java
/**
 * 归并排序
 * @author lizhibiao
 * @date 2019/6/2 17:27
 */
public class MergeSort
{
    private static void mergeSort(int[] arr)
    {
        if (null == arr || arr.length <= 1)
        {
            return;
        }

        //注意这里右边界传入的数组长度减1，数组下标是从0开始
        mergeSort(arr, 0, arr.length - 1);
    }

    private static void mergeSort(int[] arr, int l, int r)
    {
        if (l == r)
        {
            return;
        }

        int mid = l + ((r - l) >> 1);
        mergeSort(arr, l, mid);
        mergeSort(arr, mid + 1, r);
        merge(arr, l, mid, r);
    }

    /**
     * 合并过程
     * @param arr 原数组
     * @param l 起始
     * @param mid 中间值
     * @param r 末尾
     */
    private static void merge(int[] arr, int l, int mid, int r)
    {
        //辅助数组
        int[] help = new int[r - l + 1];
        //help数组索引，从0开始
        int i = 0;
        //左半部分起始位置是传进来的l，需要和右半部分比较谁小，往help里放
        int left = l;
        //右边部分起始位置，需要和左半部分比较谁小，往help里放
        int right = mid + 1;

        while (left <= mid && right <= r)
        {
            //谁小谁往help里放，同时小的索引右移，help数组索引也右移
            help[i++] = arr[left] < arr[right] ? arr[left++] : arr[right++];
        }

        while (left <= mid)
        {
            help[i++] = arr[left++];
        }

        while (right <= r)
        {
            help[i++] = arr[right++];
        }

        for (int j = 0; j < help.length; j++)
        {
            arr[l + j] = help[j];
        }

    }

    public static void main(String[] args)
    {
        int[] arr = {2, 4, 3, 1};
        mergeSort(arr);

        for (int value : arr)
        {
            System.out.print(value);
        }
    }

}
```

## 随机快排实现（降低最小划分值概率，节省划分变量）

### QuickSort

```java
/**
 * 随机快排
 * @author lizhibiao
 * @date 2019/6/4 19:36
 */
public class QuickSort
{
    private static void quickSort(int[] arr)
    {
        if (null == arr || arr.length <= 1)
        {
            return;
        }

        quickSort(arr, 0, arr.length - 1);
    }

    private static void quickSort(int[] arr, int l, int r)
    {
       if (l < r)
       {
           //从l到r范围内随机一个数放到数组末尾位置
           swap(arr, l + (int) (Math.random() * (r - l + 1)), r);

           int[] partition = partition(arr, l, r);

           quickSort(arr, l, partition[0] - 1);
           quickSort(arr, partition[1] + 1, r);
       }
    }

    private static void swap(int[] arr, int i, int j)
    {
        int tempValue = arr[i];
        arr[i] = arr[j];
        arr[j] = tempValue;
    }

    /**
     * 小于区、等于区、大于区 过程
     * @param arr 数组
     * @param l start
     * @param r end
     * @return 返回等于区的左边界和右边界下标
     */
    private static int[] partition(int[] arr, int l, int r)
    {
        int less = l - 1;
        int more = r;
        while (l < more)
        {
            if (arr[l] < arr[r])
            {
                swap(arr, ++less, l++);
            }
            else if (arr[l] > arr[r])
            {
                swap(arr, l, --more);
            }
            else
            {
                l++;
            }
        }

        //大于区第一个数和数组最后一个数交换（也就是划分值）
        swap(arr, more, r);

        //等于区的左边界和右边界下标
        return new int[]{less + 1, more};
    }

    public static void main(String[] args)
    {
        int[] arr = {5, 6, 3, 8, 9, 6, 1, 8};
        quickSort(arr);
        for (int value : arr)
        {
            System.out.print(value);
        }

    }

}

```