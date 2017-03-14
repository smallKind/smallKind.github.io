---
layout: post
title: HashMap
date: 2016-05-24 20:00:00 +8000
category: JDK源码解析
tags: 集合
---

* content
{:toc}

### HashMap概述

HashMap是基于哈希表的 Map 接口的非同步实现。此实现提供所有可选的映射操作，并允许使用 null 值和 null 键。此类不保证映射的顺序，特别是它不保证该顺序恒久不变。

需要注意的是：HashMap 不是同步的，如果多个线程同时访问一个 HashMap，而其中至少一个线程从结构上（指添加或者删除一个或多个映射关系的任何操作）修改了，则必须保持外部同步，以防止对映射进行意外的非同步访问。

### HashMap的数据结构

在 Java 编程语言中，最基本的结构就是两种，一个是数组，另外一个是指针（引用），HashMap 就是通过这两个数据结构进行实现。HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体。

![](/img/api/hashMap.jpg)

从上图中可以看出，HashMap 底层就是一个数组结构，数组中的每一项又是一个链表。当新建一个 HashMap 的时候，就会初始化一个数组。

### HashMap源码解析

##### HashMap的继承关系

    java.lang.Object
        java.util.AbstractMap<K,V>
            java.util.HashMap<K,V>

##### HashMap的构造函数

HashMap提供四个构造函数：

* HashMap()

  Constructs an empty HashMap with the specified initial capacity and the default load factor (0.75).构造一个具有默认初始容量 (16) 和默认负载因子 (0.75) 的空 HashMap

* HashMap(int initialCapacity)

  Constructs an empty HashMap with the specified initial capacity and the default load factor (0.75).构造一个带指定初始容量和默认负载因子 (0.75) 的空 HashMap

* HashMap(int initialCapacity, float loadFactor)

  Constructs an empty HashMap with the specified initial capacity and load factor.构造一个带指定初始容量和负载因子的空 HashMap

* HashMap(Map<? extends K,? extends V> m)

  Constructs a new HashMap with the same mappings as the specified Map.构造一个有相同指定映射的新HashMap

初始容量，负载因子。这两个参数是影响HashMap性能的重要参数，其中容量表示哈希表中桶的数量，初始容量是创建哈希表时的容量，负载因子是哈希表在其容量自动增加之前可以达到多满的一种尺度，它衡量的是一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度越高，反之愈小。对于使用链表法的散列表来说，查找一个元素的平均时间是O(1+a)，因此如果负载因子越大，对空间的利用更充分，然而后果是查找效率的降低；如果负载因子太小，那么散列表的数据将过于稀疏，对空间造成严重浪费。系统默认负载因子为0.75，一般情况下我们是无需修改的。

    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        //设置负载因子
        this.loadFactor = loadFactor;
        //设置扩容的大小
        this.threshold = tableSizeFor(initialCapacity);
    }

从构造函数源码分析并没有看到数组的初始化？

##### HashMap存储实现

Java中最常用的两种结构是数组和模拟指针(引用)，几乎所有的数据结构都可以利用这两种来组合实现，HashMap也是如此。实际上HashMap是一个“链表散列”。

![](/img/api/23-1.png)

    //数组，数组的元素为Node<K,V>
    transient Node<K,V>[] table;
    //Node<K,V>实现了Map中内部类Entry<K,V>（链表）
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;//散列值
        final K key;//键
        V value;//值
        Node<K,V> next;//下一个节点的指针

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }

在HashMap中，并没有看到初始化table数组,其初始化数组的操作放到了put()函数中。

    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

实际调用的是putVal()函数，在看putVal()函数：

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab;
        Node<K,V> p;
        int n, i;
        //如果table为空，或者table数组的长度等于0，则调用resize()方法，这个方法就是初始化数组和扩容
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //如果存入的位置，table数组的元素为空，就新建一个Node<K,V>元素,存入该位置
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        //剩下就是存入的位置，存在元素，往该链表的尾添加新元素
        else {
            Node<K,V> e;
            K k;
            //新加入的元素与链表的头元素相等
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            //该位置的元素如果是TreeNode,调用putTreeVal()
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            //剩下的是，遍历该链表，在链表的尾部，加入新元素
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //存在K,把旧的V替换成新的V
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        //扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

在哈希映射表中，内部数组中的每个位置称作“存储桶”(bucket)，而可用的存储桶数（即内部数组的大小）称作容量 (capacity)。随着HashMap中元素的数量越来越多，发生碰撞的概率就越来越大，所产生的链表长度就会越来越长，这样势必会影响HashMap的速度，为了保证HashMap的效率，系统必须要在某个临界点进行扩容处理。该临界点在当HashMap中元素的数量等于table数组长度*加载因子。但是扩容是一个非常耗时的过程，因为它需要重新计算这些数据在新table数组中的位置并进行复制处理。所以如果我们已经预知HashMap中元素的个数，那么预设元素的个数能够有效的提高HashMap的性能。

下面是 HashMap 调整容器大小的过程，通过下面的代码我们可以看到其扩容过程的复杂性：

    //调整数组大小（初始化数组或扩容）
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        //旧的数组大小
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            //如果数组容量超过最大值，返回原数组
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //如果没有超过最大值，在原数组上扩容一倍
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }//数组容量小于等于0，则用默认的扩容大小来设置数组大小
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
            //都为0的情况下，全部初始化为默认的
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            //初始化新数组
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
            //将新数组赋值给table(如果table为空，在这里初始化)
        table = newTab;
        //如果旧数组不为空，将旧数组的值赋值给新数组
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }

但是,如何决定扩容了?

为了确认何时需要调整 Map 容器，Map 使用了一个额外的参数并且粗略计算存储容器的密度。在 Map 调整大小之前，使用”负载因子”来指示 Map 将会承担的“负载量”，也就是它的负载程度，当容器中元素的数量达到了这个“负载量”，则 Map 将会进行扩容操作。负载因子、容量、Map 大小之间的关系如下：负载因子 * 容量 > map 大小 —–>调整 Map 大小。

例如：如果负载因子大小为 0.75（HashMap 的默认值），默认容量为 11，则 11 * 0.75 = 8.25 = 8，所以当我们容器中插入第八个元素的时候，Map 就会调整大小。

##### HashMap读取实现

相对于 HashMap 的存而言，取就显得比较简单了。通过 key 的 hash 值找到在 table 数组中的索引处的 Entry，然后返回该 key 对应的 value 即可。

    public V get(Object key) {
        Node<K,V> e;
        return (e = getNode(hash(key), key)) == null ? null : e.value;
    }

 实际调用的是getNode()函数，把K的散列值，和K作为参数。

    //散列函数，计算K的hash值
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

    final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            //链表头元素相等，返回头元素
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            //遍历链接，知道找到该元素为止
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }

在这里能够根据 key 快速的取到 value 除了和 HashMap 的数据结构密不可分外，还和 Entry 有莫大的关系，在前面就提到过，HashMap 在存储过程中并没有将 key，value 分开来存储，而是当做一个整体 key-value 来处理的，这个整体就是 Entry 对象。同时 value 也只相当于 key 的附属而已。在存储的过程中，系统根据 key 的 hashcode 来决定 Entry 在 table 数组中的存储位置，在取的过程中同样根据 key 的 hashcode 取出相对应的 Entry 对象，然后遍历 Entry 对象。

参考:

1. [java提高篇（二三）—–HashMap](http://cmsblogs.com/?p=176)
2. [极客学院-HashMap](http://wiki.jikexueyuan.com/project/java-enhancement/java-twentythree.html)










