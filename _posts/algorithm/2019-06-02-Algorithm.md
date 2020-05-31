---
layout: post
title: 常用算法
categories: Algorithm
description: 数据结构算法学习记录，一些常用算法实现
keywords: Algorithm
---

左神算法课程学习记录，常用的一些算法自己再实现一遍，为刷leetcode准备。



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

## 桶排序（求排序后相邻数最大差值，时间复杂度O（N））

### MaxGap

```java
/** 
 *  桶排序
 * @author lizhibiao
 * @date 2019/7/8 21:30
 */
public class MaxGap
{
    public static int maxGap(int[] nums) 
    {
        if (nums == null || nums.length < 2) 
        {
            return 0;
        }
        //注意这里的代码
        int len = nums.length;
        int min = Integer.MAX_VALUE;
        int max = Integer.MIN_VALUE;

        //找这个数组中的最大值和最小值, 这里进来的时候数组长度肯定大于等于2
        for (int i = 0; i < len; i++)
        {
            //找最小值，所以min一开始要传入最大的数去比较。min为int最大值,
            // 下一次比较max传进来的就是取nums[i]
            min = Math.min(min, nums[i]);
            //找最大值，所以，max一开始要传最小值进去比较，max最int最小值
            max = Math.max(max, nums[i]);
        }

        //最大值和最小值相等，说明nums数组里值都相等，所以差值为0
        if (min == max) 
        {
            return 0;
        }

        //三个数组,表示i号桶是否为空和最大值、最小值
        //hasNum数组用来记录每一个桶是否为空
        boolean[] hasNum = new boolean[len + 1];
        //maxs数组统计每个桶的最大值
        int[] maxs = new int[len + 1];
        //mins数组统计每个桶的最小值
        int[] mins = new int[len + 1];

        int bid = 0;
        //建立所有桶信息
        for (int i = 0; i < len; i++) 
        {
            //每个数根据最大值和最小值算出你该进哪个桶
            bid = bucket(nums[i], len, min, max);
            //因为这个数，判断当前桶的最小值是否需要更新
            mins[bid] = hasNum[bid] ? Math.min(mins[bid], nums[i]) : nums[i];
            //因为这个数，判断当前桶的最大值是否需要更新
            maxs[bid] = hasNum[bid] ? Math.max(maxs[bid], nums[i]) : nums[i];
            //桶肯定不为空了
            hasNum[bid] = true;
        }

        int res = 0;
        //上一个桶的最大值
        int lastMax = maxs[0];
        int i = 1;
        //下标从1开始
        for (; i <= len; i++) 
        {
            if (hasNum[i])
            {
                //每一桶的最小值都去找上一个桶的最大值
                //取当前桶最小值减去上一通最大值，差值的最大值
                res = Math.max(res, mins[i] - lastMax);
                //更新上一个桶的最大值
                lastMax = maxs[i];
            }
        }
        return res;
    }

    /**
     * 均分桶，然后根据公式算出你该进哪个桶
     * (包括最大值和最小值也是根据这个公式选进第0号桶和最后一个桶)
     * @param num 当前值，判断要进哪个桶
     * @param len 数组的长度
     * @param min 当前桶的最小值
     * @param max 当前桶的最大值
     * @return 放入几号桶
     */
    public static int bucket(long num, long len, long min, long max)
    {
        return (int) ((num - min) * len / (max - min));
    }
}
```

## KMP（求Str1是否包含str2，时间复杂度O（N））

### Kmp

```java
/**
 * KMP算法
 * @author lizhibiao
 * @date 2019/7/3 15:11
 */
public class Kmp
{
    /**
     * 字符串str1是否包含字符串str2
     * @param str1
     * @param str2
     * @return 如果包含返回字符串str1匹配的起始位置，不包含返回-1
     */
    private static int getIndexOf(String str1, String str2)
    {
        if (null == str1 || null == str2)
        {
            return -1;
        }

        if (str2.length() < 1 || str1.length() < str2.length())
        {
            return -1;
        }

        char[] char1 = str1.toCharArray();
        char[] char2 = str2.toCharArray();

        int[] nextArr = getNextArr(char2);

        int first = 0;
        int second = 0;

        while (first < char1.length && second < char2.length)
        {
            if (char1[first] == char2[second])
            {
                first++;
                second++;
            }
            else if (nextArr[second] == -1)
            {
                first++;
            }
            else
            {
                second = nextArr[second];
            }
        }

        return second == char2.length ? first - second: -1;
    }

    /**
     * next[]数组求解
     * @param arr 传入的str2字符数组
     * @return next[]数组
     */
    private static int[] getNextArr(char[] arr)
    {
       if (arr.length <= 1)
       {
           return new int[]{ - 1};
       }

       int[] next = new int[arr.length];
       next[0] = -1;
       next[1] = 0;

       int index = 2;
       //每次跳的索引，刚好等于当前要求数的上一个数的最大前缀。
       int jump = 0;

       //从0开始，所以小于数组长度!
       while (index < next.length)
       {
           if (arr[index - 1] == arr[jump])
           {
               next[index++] = ++jump;
           }
           else if (jump > 0)
           {
               //往前跳,数组是从0开始,所以刚好等于当前要求数的上一个数的最大前缀
               jump = next[jump];
           }
           else
           {
               //没法再往前跳为了，说明当前缀长度为0,并且index加1继续下一个数的缀长度
               next[index++] = 0;
           }

       }

       return next;
    }

    public static void main(String[] args) {
        String str = "abcabcababaccc";
        String match = "ababa";
        System.out.println(getIndexOf(str, match));
    }

}
```

## Manacher(求最长回文子串，时间复杂度O(N))

### Manacher

```java
/**
 * @author lizhibiao
 * @date 2019/8/25 14:24
 */
public class Manacher
{
    public static char[] manacherString(String str)
    {
        char[] charArr = str.toCharArray();
        char[] res = new char[str.length() * 2 + 1];
        int index = 0;
        for (int i = 0; i != res.length; i++) {
            res[i] = (i & 1) == 0 ? '#' : charArr[index++];
        }
        return res;
    }

    public static int maxLcpsLength(String str)
    {
        if (str == null || str.length() == 0)
        {
            return 0;
        }
        //先处理成manacher字符数组也就是包含"#"的字符串
        char[] charArr = manacherString(str);
        //每一个数的回文半径长度记录数组
        int[] pArr = new int[charArr.length];
        //回文中心索引
        int pC = -1;
        //最右回文边界索引
        int pR = -1;

        int max = Integer.MIN_VALUE;
        for (int i = 0; i != charArr.length; i++)
        {
            //i位置起码的回文半径是多少，不管是什么情况
            //pR > i说明当前数在回文边界里面

            // Math.min(pArr[2 * pC - i], pR - i), pArr[2 * pC - i]对称点i'的回文半径和pR到i的回文半径谁小就是不用验证的距离
            // 这个其实就是：
            //    情况2：如果i‘的回文半径在L和C之间，那么i的回文半径肯定和i’一样长，i回文半径等于pArr[2 * pC - i]
            //    情况3：如果i‘的回文半径有部分在L和C之外，那么i的回文半径是i到R的距离，即pR - i
            //    情况2和情况3两者之前取最小
            //    然后就是情况4：i只需要判断从R的下一个数开始继续往两边扩，i的回文半径和i'的回文半径相等，所以随便取一个就可以
            //    总结：Math.min(pArr[2 * pC - i], pR - i), pArr[2 * pC - i] 就是pR > i当前数在回文边界里面，最小的不用验证的距离

            //pR <= i说明当前数不在回文边界里面那么需要往两边扩，最小的不用验证的回文半径长度就是自己本身所以就是1

            //pArr[i] = pR > i ? Math.min(pArr[2 * pC - i], pR - i) : 1;
            // 这个求得就是每个数起码的回文半径长度，现在只是为了code的短减少分支，整合在一起了，否则需要写4个分支
            pArr[i] = pR > i ? Math.min(pArr[2 * pC - i], pR - i) : 1;

            //继续判断能不能扩，能扩多远？
            //判断是否越界i + pArr[i] < charArr.length && i - pArr[i] > -1
            while (i + pArr[i] < charArr.length && i - pArr[i] > -1)
            {
                //如果没有越界，判断新的边界是否相等，如果相等回文边界加1
                //当前数i的索引 + 当前数回文半径长度pArr[i] = 等于回文半径右边界的下一索引，也就是新的右边界
                //当前数i的索引 - 当前数回文半径长度pArr[i] = 等于回文半径左边界的前一索引，也就是新的左边界
                if (charArr[i + pArr[i]] == charArr[i - pArr[i]])
                {
                    pArr[i]++;
                }
                else
                {
                    break;
                }
            }

            //如果当前数的最右回文半径边界大于当前记录的最右回文边界，需要更新最右回文边界值和回文中心索引
            if (i + pArr[i] > pR)
            {
                pR = i + pArr[i];
                pC = i;
            }

            //记录着整个全局的最大回文半径长度值，更新
            max = Math.max(max, pArr[i]);
        }

        //#a#b#c#1#2#3#4#3#2#1#a#b#
        //每一个数的回文半径长度记录包含了“#”符号，所以pArr[i] - 1刚好就是回文子串
        //例如pArr[13] == 8, 即上面那个数字4的回文半径长度是8，回文半径里包含了“#”符号，减1刚好等于回文子串长度
        return max - 1;
    }

    public static void main(String[] args)
    {
        String str1 = "abc1234321ab";
        System.out.println(maxLcpsLength(str1));

    }
}
```

## 用栈实现队列（借助一个辅助栈）

### StackImplementQueue

```java
/**
 * 两个栈实现队列
 * @author lizhibiao
 * @date 2020/1/1 22:27
 */
public class StackImplementQueue
{
    /**
     * 数据栈
     */
    private Stack<Integer> dataStack = new Stack<>();

    /**
     * 辅助栈
     */
    private Stack<Integer> helpStack = new Stack<>();

    /**
     * 添加
     * @param value 值
     */
    public void add(int value)
    {
        dataStack.push(value);
    }

    /**
     * 弹出
     */
    public int pop()
    {
        if (dataStack.isEmpty() && helpStack.isEmpty())
        {
            throw new RuntimeException("Queue pop error!");
        }

        //只有辅助栈为空，那么就需要一次性把数据栈数据全部倒入
        if (helpStack.isEmpty())
        {
            while (!dataStack.isEmpty())
            {
                Integer value = dataStack.pop();
                helpStack.push(value);
            }
        }

        return helpStack.pop();
    }

    public static void main(String[] args)
    {
        StackImplementQueue queue = new StackImplementQueue();
        queue.add(1);
        queue.add(2);
        queue.add(3);

        System.out.println("弹出的值" + queue.pop());

        queue.add(4);

        System.out.println("弹出的值" + queue.pop());
        System.out.println("弹出的值" + queue.pop());
        System.out.println("弹出的值" + queue.pop());
    }

}

```

```sh
输出:
弹出的值1
弹出的值2
弹出的值3
弹出的值4
```

## 用队列实现栈

### QueueImplementStack

```java

/**
 * 用队列实现栈
 * @author lizhibiao
 * @date 2020/1/16 20:45
 */
public class QueueImplementStack
{
    /**
     * 原数据队列
     */
    private Queue<Integer> dataQueue = new LinkedList<>();

    /**
     * 辅助队列
     */
    private Queue<Integer> helpQueue = new LinkedList<>();

    /**
     * 默认开始弹出数据的长度
     */
    private static final int DEFAULT_POP_LENGTH = 1;


    /**
     * 压栈
     * @param value 值
     */
    public void push(int value)
    {
        dataQueue.add(value);
    }

    /**
     * 出栈
     * @return 弹出的值，没有返回-1
     */
    public int pop()
    {
        if (dataQueue.isEmpty())
        {
            throw new RuntimeException("dataQueue is empty !");
        }

        while (dataQueue.size() != DEFAULT_POP_LENGTH)
        {
            helpQueue.add(dataQueue.poll());
        }

        int pollValue = dataQueue.poll();

        swap();

        return pollValue;

    }

    /**
     * 交换队列引用
     */
    private void swap()
    {
        Queue<Integer> tmp = helpQueue;
        helpQueue = dataQueue;
        dataQueue = tmp;
    }


    public static void main(String[] args)
    {
        QueueImplementStack stack = new QueueImplementStack();
        stack.push(1);
        stack.push(2);
        stack.push(3);
        stack.push(4);
        stack.push(5);

        System.out.println(stack.pop());
        System.out.println(stack.pop());
        System.out.println(stack.pop());
        System.out.println(stack.pop());
        System.out.println(stack.pop());

    }

}
```

## 转圈打印矩阵

### PrintCircleMatrix

```java
/**
 * 转圈打印矩阵
 *
 * @author lizhibiao
 * @date 2020/5/31 9:47
 */
public class PrintCircleMatrix
{
    public static void main(String[] args)
    {
        //构建一个4行5列二维数组
        int[][] arrData = buildArrData(4, 5);
        System.out.println(Arrays.deepToString(arrData));

        int startLineIndex = 0;
        int startColumnIndex = 0;
        int endLineIndex = arrData.length - 1;
        int endColumnIndex = arrData[0].length - 1;

//        printOneCircle(arrData, 0, 0, 0, 5);
//        printOneCircle(arrData, 0, 0, 4, 0);

        //一层一层打印
        while (startLineIndex <= endLineIndex
                || startColumnIndex <= endColumnIndex)
        {
            printOneCircle(arrData, startLineIndex++, startColumnIndex++, endLineIndex--, endColumnIndex--);
        }


    }

    /**
     * 构建一个nLine行nColumn列的二维数组, 值累加
     * @param nLine 行
     * @param nColumn 列
     * @return 返回数组
     */
    private static int[][] buildArrData(int nLine, int nColumn)
    {
        int value = 1;
        int[][] arr = new int[nLine][nColumn];
        //行
        int line = arr.length;
        for (int i = 0; i < line; i++)
        {
            //列
            int column = arr[i].length;
            for (int j = 0; j < column; j++)
            {
                arr[i][j] = value++;
            }
        }

        return arr;
    }

    /**
     * 打印一圈的值
     * @param arr 传入的数组
     * @param startLineIndex 起始行索引
     * @param startColumnIndex 起始列索引
     * @param endLineIndex 终止行索引
     * @param endColumnIndex 终止列索引
     */
    private static void printOneCircle(int[][] arr, int startLineIndex, int startColumnIndex,
                                       int endLineIndex, int endColumnIndex)
    {
        //只有一行
        if (startLineIndex == endLineIndex)
        {
            for (int i = startColumnIndex; i < endColumnIndex; i++)
            {
                System.out.print(arr[startLineIndex][i] + " ");
            }

            return;
        }

        //只有一列
        if (startColumnIndex == endColumnIndex)
        {
            for (int i = startLineIndex; i < endLineIndex; i++)
            {
                System.out.print(arr[i][startColumnIndex] + " ");
            }
            return;
        }

        //其它情况,打印圈
        for (int i = startColumnIndex; i < endColumnIndex; i++)
        {
            System.out.print(arr[startLineIndex][i] + " ");
        }

        for (int i = startLineIndex; i < endLineIndex; i++)
        {
            System.out.print(arr[i][endColumnIndex] + " ");
        }

        for (int i = endColumnIndex; i > startColumnIndex; i--)
        {
            System.out.print(arr[endLineIndex][i] + " ");
        }

        for (int i = endLineIndex; i > startLineIndex; i--)
        {
            System.out.print(arr[i][startColumnIndex] + " ");
        }

    }
}

```

