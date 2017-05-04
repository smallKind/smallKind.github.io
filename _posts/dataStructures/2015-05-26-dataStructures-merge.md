---
layout: post
title: 归并排序
date: 2015-05-26 22:10:01 +8000
category: 数据结构
tags: 排序
---

* content
{:toc}

>指的是将两个已经排序的序列合并成一个序列的操作

![](/img/dataStructuresAndAlgorithmAnalysis/merge.gif)

* 最差时间复杂度	O(nlog n)
* 最优时间复杂度	O(n)
* 平均时间复杂度	O(nlog n)
* 最差空间复杂度	O(n)

原理如下（假设序列共有n个元素）：

1. 将序列每相邻两个数字进行归并操作，形成floor(n/2)个序列，排序后每个序列包含两个元素
2. 将上述序列再次归并，形成floor(n/4)个序列，每个序列包含四个元素
3. 重复步骤2，直到所有元素排序完毕

### 算法实现

    public static void Msort(int[] a,int[] temp,int left,int right){
        int center;
        if(left < right){
            center = (left + right) >> 1;
            Msort(a,temp,left,center);
            Msort(a,temp,center+1,right);
            Merge(a,temp,left,center+1,right);
        }
    }

    public static void Merge(int[] a,int[] temp,int Lpos,int Rpos,int RightEnd){
        int i,LeftEnd,NumElements,TmpPos;
        LeftEnd = Rpos - 1;
        TmpPos = Lpos;
        NumElements = RightEnd - Lpos + 1;
        while(Lpos <= LeftEnd && Rpos <= RightEnd)
            if(a[Lpos] <= a[Rpos])
                temp[TmpPos++] = a[Lpos++];
            else
                temp[TmpPos++] = a[Rpos++];
        while(Lpos <= LeftEnd)
            temp[TmpPos++] = a[Lpos++];
        while(Rpos <= RightEnd)
            temp[TmpPos++] = a[Rpos++];
        for(i = 0;i < NumElements;i++,RightEnd--)
            a[RightEnd] = temp[RightEnd];
    }

    public static void Mergesort(int[] a){
        int N = a.length;
        int[] temp = new int[N];
        if(N!=0){
            Msort(a,temp,0,N-1);
            printf(a);
        }
    }

    public static void printf(int[] a) {
        for (int i = 0; i < a.length; i++) {
            System.out.println(a[i]);
        }
    }

 参考：

 1.《数据结构与算法分析》