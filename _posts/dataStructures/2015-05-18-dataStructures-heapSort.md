---
layout: post
title: 堆排序
date: 2015-05-18 09:31:01 +8000
category: 数据结构
tags: 排序
---

* content
{:toc}

>堆排序（Heapsort）是指利用堆这种数据结构所设计的一种排序算法。堆积是一个近似完全二叉树的结构，并同时满足堆积的性质：即子结点的键值或索引总是小于（或者大于）它的父节点。

## 堆

堆是一棵被完全填满的二叉树，有可能的例外在底层，底层上的元素从左到右填入。这样的树被称为完全二叉树。

##### 逻辑定义

n个元素序列{k1,k2...ki...kn},当且仅当满足下列关系时称之为堆：
(ki <= k2i,ki <= k2i+1)或者(ki >= k2i,ki >= k2i+1), (i = 1,2,3,4...n/2)

##### 性质

堆的实现通过构造二叉堆（binary heap），实为二叉树的一种；由于其应用的普遍性，当不加限定时，均指该数据结构的这种实现。这种数据结构具有以下性质。

* 任意节点小于（或大于）它的所有后裔，最小元（或最大元）在堆的根上（堆序性）。
* 堆总是一棵完全树。即除了最底层，其他层的节点都被元素填满，且最底层尽可能地从左到右填入。

将根节点最大的堆叫做最大堆或大根堆，根节点最小的堆叫做最小堆或小根堆。

## 堆排序

![](/img/dataStructuresAndAlgorithmAnalysis/heapSort.gif)

##### 堆节点的访问

通常堆是通过一维数组来实现的。在数组起始位置为0的情形中：

* 父节点i的左子节点在位置(2*i+1);
* 父节点i的右子节点在位置(2*i+2);
* 子节点i的父节点在位置floor((i-1)/2);

##### 堆排序的操作（实现原理）

在堆的数据结构中，堆中的最大值总是位于根节点。堆中定义以下几种操作：

1. 创建最大堆（Build_Max_Heap）：将堆所有数据重新排序
2. 堆排序（HeapSort）：移除位在第一个数据的根节点，并做最大堆调整的递归运算
3. 最大堆调整（Max_Heapify）：将堆的末端子节点作调整，使得子节点永远小于父节点

##### 实现示例

    public class HeapSort {
        private static int[] sort = new int[]{1,0,10,20,3,5,6,4,9,8,12,17,34,11};
        public static void main(String[] args) {
              buildMaxHeapify(sort);
              heapSort(sort);
              print(sort);
    }

    private static void buildMaxHeapify(int[] data){
        //没有子节点的才需要创建最大堆，从最后一个的父节点开始
        int startIndex = getParentIndex(data.length - 1);
        //从尾端开始创建最大堆，每次都是正确的堆
        for (int i = startIndex; i >= 0; i--) {
            maxHeapify(data, data.length, i);
        }
    }

    /**
     * 创建最大堆
     * @param data
     * @param heapSize需要创建最大堆的大小，一般在sort的时候用到，因为最多值放在末尾，末尾就不再归入最大堆了
     * @param index当前需要创建最大堆的位置
     */
    private static void maxHeapify(int[] data, int heapSize, int index){
        // 当前点与左右子节点比较
        int left = getChildLeftIndex(index);
        int right = getChildRightIndex(index);

        int largest = index;
        if (left < heapSize && data[index] < data[left]) {
            largest = left;
        }
        if (right < heapSize && data[largest] < data[right]) {
            largest = right;
        }
        //得到最大值后可能需要交换，如果交换了，其子节点可能就不是最大堆了，需要重新调整
        if (largest != index) {
            int temp = data[index];
            data[index] = data[largest];
            data[largest] = temp;
            maxHeapify(data, heapSize, largest);
        }
    }

    /**
     * 排序，最大值放在末尾，data虽然是最大堆，在排序后就成了递增的
     * @param data
     */
    private static void heapSort(int[] data) {
        //末尾与头交换，交换后调整最大堆
        for (int i = data.length - 1; i > 0; i--) {
            int temp = data[0];
            data[0] = data[i];
            data[i] = temp;
            maxHeapify(data, i, 0);
        }
    }

    /**
     * 父节点位置
     * @param current
     * @return
     */
    private static int getParentIndex(int current){
        return (current - 1) >> 1;
    }

    /**
     * 左子节点position注意括号，加法优先级更高
     * @param current
     * @return
     */
    private static int getChildLeftIndex(int current){
        return (current << 1) + 1;
    }

    /**
     * 右子节点position
     * @param current
     * @return
     */
    private static int getChildRightIndex(int current){
        return (current << 1) + 2;
    }

    public static void print(int[] data){
        int pre = -2;
        for (int i = 0; i < data.length; i++) {
            if (pre < (int)getLog(i+1)) {
                pre = (int)getLog(i+1);
                System.out.println();
            }
            System.out.print(data[i] + " |");
        }
    }

    /**
     * 以2为底的对数
     * @param param
     * @return
     */
    private static double getLog(double param){
        return Math.log(param)/Math.log(2);
    }
}

优先队列基于堆排序的原理上实现。堆排序是一个非常稳定的算法：它平均使用的比较只比最坏情况界指出的略少。


参考：

1. [维基百科](https://zh.wikipedia.org/wiki/堆排序)
2. 《数据结构与算法分析》