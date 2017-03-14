---
layout: post
title: "希尔排序"
date: 2015-05-24 13:31:01 +8000
category: 数据结构
tags: 排序
---

* content
{:toc}

>也叫缩小增量排序，它是通过比较相距一定间隔的元素来工作，各趟比较所用的距离随着算法的进行二减小，直到只比较相邻元素的最后一趟排序为止。

### 希爾排序法的概念

1. 將資料排列成二維陣列
2. 對二維陣列的每一行做排序
3. 利用前一次排序的結果來加快排序

### 希爾排序作法

1. 由大到小選定數個間距(Gap)，最後一個Gap一定要是1
2. 將資料依指定的間距(Gap)分組，進行插入排序
3. Gap的選擇對執行效率有很大的影響

### 常見的Gap

1. Shell原本的Gap：N/2、N/4、...1(反覆除以2)
2. Hibbard的Gap：1、3、7、...、2k-1
3. Knuth的Gap: 1、4、13、...、(3k - 1) / 2
S4. edgewick的Gap: 1、5、19、41、109、...

### 算法分析

假設有15筆資料(0~14)，Gap分別選用5,2,1

* 第一回合 Gap=5 :

    * 對位置0, 5, 10作插入排序
    * 對位置1, 6, 11作插入排序
    * 對位置2, 7, 12作插入排序
    * 對位置3, 8, 13作插入排序
    * 對位置4, 9, 14作插入排序

* 第二回合 Gap=2 :

    * 對位置0, 2, 4, 6, 8, 10, 12, 14作插入排序
    * 對位置1, 3, 5, 7, 9, 11, 13作插入排序

* 第三回合 Gap=1 :
    * 分別對位置0, 1, 2, 3, ..., 14作插入排序

* 時間複雜度(Time Complexity)
  * Best Case：Ο(n)
  * Worst Case：依選用的GAP而定，Ο(n2) ~ Ο(n1.5)
  * Average Case：Ο(n5/4)

 ![](/img/dataStructuresAndAlgorithmAnalysis/shell.gif)

* 空間複雜度(Space Complexity)：θ(1)
* 穩定性(Stable/Unstable)：不穩定(Unstable)

### 算法实现

    public static void shell(int[] a) {
        int i, j, increment, N = a.length;
        //增量序列
        for (increment = N / 2; increment > 0; increment /= 2) {
            //插入排序
            for (i = increment; i < N; i++) {
                int temp = a[i];
                for (j = i; j >= increment; j -= increment)
                    if (temp < a[j - increment])
                        a[j] = a[j - increment];
                    else
                        break;
                a[j] = temp;
            }
        }
        printf(a);
    }

    輸入：1個陣列名稱為a，1個長度名稱為n，陣列的編號從0到n - 1
    整數inc從n / 2到1每次迴圈inc變為inc / 2
        i從inc到n - 1每次迴圈i變為i + 1
                將a[i]的值丟到temp
                j從i - inc到0每次迴圈j變為j - inc
                        如果a[j]大於temp則將a[j - inc]的值丟到a[j]
                        否則跳出j迴圈
                j迴圈結束
                將temp的值丟到a[j - inc]
        i迴圈結束
    inc迴圈結束

    public static void printf(int[] a) {
        for (int i = 0; i < a.length; i++) {
            System.out.println(a[i]);
        }
    }

参考：

1.[维基百科](https://zh.wikipedia.org/zh-cn/希尔排序)