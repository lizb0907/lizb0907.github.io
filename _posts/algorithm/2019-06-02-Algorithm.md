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

## 堆排序，先建立大根堆，再进行排序

### HeapSort

```java
/**
 * 堆排序，先建立大根堆，再进行排序
 * @author lizhibiao
 * @date 2019/6/12 11:42
 */
public class HeapSort
{
    
    private static void heapSort(int[] arr)
    {
        if (null == arr || arr.length < 2)
        {
            return;
        }
        
        int size = arr.length;
        for (int i = 0; i < size; i++)
        {
            heapInsert(arr, i);
        }

        //完全二叉树头节点和尾节点交换位置，对应到数组就是0位置和数组末尾位置
        //数组长度减1，末尾值相当于有序了
        swap(arr, 0, --size);

        while (size > 0)
        {
            heapify(arr, 0, size);
            swap(arr, 0, --size);
        }

    }

    /**
     * 建立大根堆，当前索引和父节点比较，大继续往上和父节点比较
     * @param arr 数组
     * @param i 当前索引
     */
    private static void heapInsert(int[] arr, int i)
    {
        while (arr[i] > arr[(i - 1) / 2])
        {
            swap(arr, i, (i - 1) / 2);
            i = (i - 1) / 2;
        }
    }

    private static void swap(int[] arr, int i, int j)
    {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }

    /**
     * 如果树的头节点小于孩子节点，和大的孩子节点交互位置，往下沉，继续比较
     * @param arr 数组
     * @param index 当前头节点索引
     * @param size 逻辑上的堆边界值
     */
    private static void heapify(int[] arr, int index, int size)
    {
        //左孩子索引
        int leftChild = index * 2 + 1;
        while (leftChild < size)
        {
            //左孩子和右孩子比较大小，取大的孩子节点索引
            //注意这个三目运算比较，不满足的情况一定是要等于左孩子节点，因为右孩子节点可能越界
            int largeIndex = leftChild + 1 < size && arr[leftChild + 1] > arr[leftChild] ? leftChild + 1 : leftChild;
            largeIndex = arr[largeIndex] > arr[index] ? largeIndex : index;

            if (largeIndex == index)
            {
                break;
            }

            swap(arr, index, largeIndex);
            index = largeIndex;
            //更新新的头节点的左孩子节点
            leftChild = index * 2 + 1;
        }
    }


    public static void main(String[] args)
    {
        int[] arr = {8, 6, 7, 6, 3, 1, 2};
        heapSort(arr);
        for (int value : arr)
        {
            System.out.print(value);
        }

    }

}

```

## 比较器的使用

### ComparatorExample

```java
/**
 * 比较器的使用
 * @author lizhibiao
 * @date 2019/6/16 15:20
 */
public class ComparatorExample
{
    private static class Student
    {
        private String name;
        private int age;

        public Student(String name, int age)
        {
            this.name = name;
            this.age = age;
        }
    }

    /**
     * 按学生年龄升序排序
     */
    private static class MyAscendingComparator implements Comparator<Student>
    {

        @Override
        public int compare(Student o1, Student o2)
        {
            return o1.age - o2.age;
        }

    }

    /**
     * 按学生年龄降序排序
     */
    private static class MyDsendingComparator implements Comparator<Student>
    {

        @Override
        public int compare(Student o1, Student o2)
        {
            return o2.age - o1.age;
        }
    }

    public static void main(String[] args)
    {
        Student s1 = new Student("小菜", 20);
        Student s2 = new Student("大鸟", 30);

        Student[] arr = {s1, s2};

        //升序
        Arrays.sort(arr, new MyAscendingComparator());
        sysMessage(arr);

        System.out.println("===========分割线============");

        //降序
        Arrays.sort(arr, new MyDsendingComparator());
        sysMessage(arr);
    }

    /**
     * 输出信息
     * @param arr 对象组
     */
    private static void sysMessage(Student[] arr)
    {
        for (Student student : arr)
        {
            System.out.println("name:" + student.name + "" + "age:" + student.age);
        }
    }

}
```