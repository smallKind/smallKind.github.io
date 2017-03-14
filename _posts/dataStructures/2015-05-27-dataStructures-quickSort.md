---
layout: post
title: 快速排序
date: 2015-05-27 18:50:00 +8000
category: 数据结构
tags: 排序
---

* content
{:toc}

>又称划分交换排序（partition-exchange sort），一种排序算法

![](/img/dataStructuresAndAlgorithmAnalysis/quickSort.gif)

* 最差时间复杂度	O(n^2)
* 最优时间复杂度	O(nlog n)
* 平均时间复杂度	0(nlog n)
* 最差空间复杂度	根据实现的方式不同而不同

### 算法原理

快速排序使用分治法（Divide and conquer）策略来把一个序列（list）分为两个子序列（sub-lists）

步骤为：

1. 从数列中挑出一个元素，称为"基准"（pivot），也可以称为枢纽元
2. 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区结束之后，该基准就处于数列的中间位置。这个称为分区（partition）操作。
3. 递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序。

递归的最底部情形，是数列的大小是零或一，也就是永远都已经被排序好了。虽然一直递归下去，但是这个算法总会结束，因为在每次的迭代（iteration）中，它至少会把一个元素摆到它最后的位置去。

![](/img/dataStructuresAndAlgorithmAnalysis/quickSortTheory.png)

### 算法实现

    private static int[] arr = new int[]{1,0,10,20,3,5,6,4,9,8,12,17,34,11};
    private static void swap(int x, int y) {
        int temp = arr[x];
        arr[x] = arr[y];
        arr[y] = temp;
    }

    private static void quick_sort_recursive(int start, int end) {
        if (start >= end)
            return;
        int mid = arr[end];
        int left = start, right = end - 1;
        while (left < right) {
            while (arr[left] < mid && left < right)
                left++;
            while (arr[right] >= mid && left < right)
                right--;
            swap(left, right);
        }
        if (arr[left] >= arr[end])
            swap(left, end);
        else
            left++;
        quick_sort_recursive(start, left - 1);
        quick_sort_recursive(left + 1, end);
    }

    public static void sort(int[] arrin) {
        arr = arrin;
        quick_sort_recursive(0, arr.length - 1);
        printf(arr);
    }

    public static void printf(int[] a) {
        for (int i = 0; i < a.length; i++) {
            System.out.println(a[i]);
        }
    }

参考：

1. [维基百科](https://zh.wikipedia.org/wiki/快速排序)
2. 《数据结构与算法分析》