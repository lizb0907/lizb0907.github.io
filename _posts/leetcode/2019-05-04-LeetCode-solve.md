---
layout: post
title: 数据结构与算法
categories: LeetCode
description: 本系列主要是想系统的再过一遍数据结构与算法，重新打下基础。记录，方便以后翻阅。
keywords: leetcode, solve
---

练练leetcode上的题目，顺便学学英语。除了习题，还会记录一些基础的基本数据结构。

学习数据结构，个人感觉很有必要，锻炼逻辑能力，熟悉常用结构以便在实际场景更好运用。



**目录**

* TOC
{:toc}

## 数组


### 283. Move Zeroes
Given an array nums, write a function to move all 0's to the end of it while maintaining the relative order of the non-zero elements.

Example:

Input: [0,1,0,3,12]
Output: [1,3,12,0,0]
Note:

You must do this in-place without making a copy of the array.
Minimize the total number of operations.

给定一个数组，写一个函数将0元素移动到数组末尾，同时保持非0元素的位置不变。

思路:

给定一个下标变量k。然后判断当前数组位置下标的值是否为0，不为0和k位置数组值交换，然后k++。为0,不动，数组下标后移。

```java
public class MoveZeros_02 {
	public static void moveZeros_02(int[] arr){
		int k=0;
		for (int i=0;i<arr.length;i++){
			if (arr[i]!=0){
				swap(arr,i,k);
				k++;
			}
		}

	}

	public static void swap(int[] arr,int i,int j){
		int temp=arr[i];
		arr[i]=arr[j];
		arr[j]=temp;
	}

	public static void main(String[] args) {
		int[] arr={5,2,0,6,0,4,8,0};
		MoveZeros_02 mz=new MoveZeros_02();
		mz.moveZeros_02(arr);
		for (int items:arr){
			System.out.print(items);
		}
	}
}
```




## 字符串




## 动态规划


