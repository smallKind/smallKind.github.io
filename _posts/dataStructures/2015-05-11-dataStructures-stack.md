---
layout: post
title: 栈
date: 2015-05-11 17:31:01 +8000
category: 数据结构
tags: 栈
---

* content
{:toc}

>在计算机科学中，是一种特殊的串列形式的数据结构，它的特殊之处在于只能允许在链接串列或阵列的一端（称为堆叠顶端指标，英语：top）进行加入资料（英语：push）和输出资料（英语：pop）的运算。另外堆叠也可以用一维阵列或连结串列的形式来完成

### 栈的基本特点：

1. 先入后出，后入先出。(LIFO)

2. 除头尾节点之外，每个元素有一个前驱，一个后继。

堆栈可以用链表和数组两种方式实现，一般为一个堆栈预先分配一个大小固定且较合适的空间并非难事，所以较流行的做法是Stack结构下含一个数组。如果空间实在紧张，也可用链表实现，且去掉表头。

### 源码解析

Stack和Collection的关系如下图：

![](/img/dataStructuresAndAlgorithmAnalysis/stack.jpg)

Stack的继承关系：

    java.lang.Object
    ↳   java.util.AbstractCollection<E>
       ↳     java.util.AbstractList<E>
           ↳     java.util.Vector<E>
               ↳     java.util.Stack<E>

Stack源码：

    public class Stack<E> extends Vector<E> {

        public Stack() {
        }
        //加入item到栈顶,需要判断栈的大小,避免栈溢出
        public E push(E item) {
           addElement(item);
           return item;
       }
       //移除栈顶item并返回,需要判断栈的大小,避免栈为null
       public synchronized E pop() {
           E       obj;
           int     len = size();
           obj = peek();
           removeElementAt(len - 1);
           return obj;
       }
       //返回栈顶item
       public synchronized E peek() {
           int     len = size();
           if (len == 0)
               throw new EmptyStackException();
           return elementAt(len - 1);
       }
       //判断栈是否为空
       public boolean empty() {
           return size() == 0;
       }
       //查找o在栈中的位置
       public synchronized int search(Object o) {
           int i = lastIndexOf(o);
           if (i >= 0) {
               return size() - i;
           }
           return -1;
       }
    }

从源码看出Stack是线程安全，继承了Vector,而Vector是通过数组实现的。这也意味着Stack也是通过数组实现的。

### 栈的应用

* 平衡符号,可用于编译器检测程序的语法错误
* 后缀表达式
* 中缀到后缀的转换
