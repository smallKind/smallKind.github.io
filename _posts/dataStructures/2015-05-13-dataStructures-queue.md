---
layout: post
title: 队列
date: 2015-05-13 19:20:01 +8000
category: 数据结构
tags: 队列
---

* content
{:toc}

>又称为伫列（queue），是先进先出（FIFO, First-In-First-Out）的线性表。在具体应用中通常用链表或者数组来实现。队列只允许在后端（称为rear）进行插入操作，在前端（称为front）进行删除操作。

### 源码解析

Queue的体系关系：

     All Superinterfaces:

        Collection<E>, Iterable<E>

     All Known Subinterfaces:

        BlockingDeque<E>, BlockingQueue<E>, Deque<E>, TransferQueue<E>

     All Known Implementing Classes:

        AbstractQueue, ArrayBlockingQueue, ArrayDeque, ConcurrentLinkedDeque,
        ConcurrentLinkedQueue, DelayQueue, LinkedBlockingDeque, LinkedBlockingQueue,
        LinkedList, LinkedTransferQueue, PriorityBlockingQueue, PriorityQueue,
        SynchronousQueue


Queue源码：

    public interface Queue<E> extends Collection<E> {
         //在队列后端加入一个新元素
         boolean add(E e);

         //在队列后端加入一个新元素，会判断队列是否满了
         boolean offer(E e);

         //返回并移除队列的头元素，如果元素为空，throws一个异常
         E remove();

         //返回并移除队列的头元素
         E poll();

         //返回队列的头元素，但不移除,如果元素为空，throws一个异常
         E element();

         //返回队列的头元素，但不移除
         E peek();
    }

    AbstractQueue继承体系关系：

    java.lang.Object
        java.util.AbstractCollection<E>
            java.util.AbstractQueue<E>

AbstractQueue源码：

    public abstract class AbstractQueue<E>
         extends AbstractCollection<E>
         implements Queue<E> {

         protected AbstractQueue() {}

         public boolean add(E e) {
              if (offer(e))
                  return true;
              else
                  throw new IllegalStateException("Queue full");
         }

         public E remove() {
              E x = poll();
              if (x != null)
                 return x;
              else
                 throw new NoSuchElementException();
         }

         public E element() {
              E x = peek();
              if (x != null)
                 return x;
              else
                 throw new NoSuchElementException();
         }

         public void clear() {
              while (poll() != null)
              ;
         }

         public boolean addAll(Collection<? extends E> c) {
              if (c == null)
                 throw new NullPointerException();
              if (c == this)
                 throw new IllegalArgumentException();
              boolean modified = false;
              for (E e : c)
                 if (add(e))
                    modified = true;
              return modified;
         }

从AbstractQueue源码看，add()函数实际还是调用的offer()函数在队列的后端加入一个新元素，offer（）函数判断了队列是否超出队列大小的限制。

## 优先队列

>在优先队列中，元素被赋予优先级。当访问元素时，具有最高优先级的元素最先删除。优先队列具有最高级先出 （largest-in，first-out）的行为特征。

优先队列中的每个元素都有各自的优先级，优先级最高的元素最先得到服务；优先级相同的元素按照其在优先队列中的顺序得到服务。优先队列往往用堆来实现。

### 源码解析

PriorityQueue继承体系关系：

    java.lang.Object
        java.util.AbstractCollection<E>
            java.util.AbstractQueue<E>
                java.util.PriorityQueue<E>


PriorityQueue源码：

    public class PriorityQueue<E> extends AbstractQueue<E>
           implements java.io.Serializable {
       //默认初始化大小
       private static final int DEFAULT_INITIAL_CAPACITY = 11;
       //堆以数组实现
       private transient Object[] queue;
       //当前大小
       private int size = 0;
       //外部比较器，默认为自然排序即内部比较器，比较器的规则决定是最大堆还是最小堆
       private final Comparator<? super E> comparator;
       //修改次数（增、删、改、查）
       transient int modCount = 0;
       //初始化容量和比较器
       public PriorityQueue(int initialCapacity,
                         Comparator<? super E> comparator) {
           // Note: This restriction of at least one is not actually needed,
           // but continues for 1.5 compatibility
           if (initialCapacity < 1)
               throw new IllegalArgumentException();
           this.queue = new Object[initialCapacity];
           this.comparator = comparator;
       }
       //如果当前队列容量小于64，容量扩充一倍，反之容量扩充一半
       private void grow(int minCapacity) {
           int oldCapacity = queue.length;
           // Double size if small; else grow by 50%
           int newCapacity = oldCapacity + ((oldCapacity < 64) ?
                                         (oldCapacity + 2) :
                                         (oldCapacity >> 1));
           // overflow-conscious code
           if (newCapacity - MAX_ARRAY_SIZE > 0)
               newCapacity = hugeCapacity(minCapacity);
           queue = Arrays.copyOf(queue, newCapacity);
      }
      //add()实际调用的还是offer()
      public boolean add(E e) {
           return offer(e);
      }
      //如果添加元素为空，报空指针异常，如果队列已满，则扩容，反之则调用siftUp()方法
      public boolean offer(E e) {
           if (e == null)
               throw new NullPointerException();
           modCount++;
           int i = size;
           if (i >= queue.length)
               grow(i + 1);
           size = i + 1;
           if (i == 0)
               queue[0] = e;
           else
               siftUp(i, e);
           return true;
      }
      //根据比较器来选择不同的上滤规则
      private void siftUp(int k, E x) {
           if (comparator != null)
               siftUpUsingComparator(k, x);
           else
               siftUpComparable(k, x);
      }
      //外部比较器为空的时候，对象自带排序规则
      @SuppressWarnings("unchecked")
      private void siftUpComparable(int k, E x) {
         Comparable<? super E> key = (Comparable<? super E>) x;
           while (k > 0) {
               //父节点下标，无符号右移以为，等于/2
               int parent = (k - 1) >>> 1;
               Object e = queue[parent];
               //如果比父节点大，无需上滤，结束
               if (key.compareTo((E) e) >= 0)
                    break;
               //如果小，则父节点元素下移
               queue[k] = e;
               //改变k的位置
               k = parent;
           }
           //找到一个合适的位置k，赋值
           queue[k] = key;
      }
      //外部比较器不为空的时候，与上面唯一区别就是这里采用的是外部排序，上面采用的是内部排序
      @SuppressWarnings("unchecked")
      private void siftUpUsingComparator(int k, E x) {
           while (k > 0) {
               int parent = (k - 1) >>> 1;
               Object e = queue[parent];
               if (comparator.compare(x, (E) e) >= 0)
                   break;
               queue[k] = e;
               k = parent;
           }
           queue[k] = x;
      }
      //出栈（不删除）
      public E peek() {
           return (size == 0) ? null : (E) queue[0];
      }
      //出栈（删除）
      public E poll() {
           // 优先队列为空，返回null
           if (size == 0)
                return null;
           int s = --size;
           modCount++;
           //返回队首
           E result = (E) queue[0];
           //队尾赋值为空
           E x = (E) queue[s];
           queue[s] = null;
           //判断是否需要下滤
           if (s != 0)
               siftDown(0, x);
           return result;
      }
      //根据比较器选择不同的下滤规则
      private void siftDown(int k, E x) {
           if (comparator != null)
               siftDownUsingComparator(k, x);
           else
               siftDownComparable(k, x);
      }
      //外部比较器为空的时候，对象自带排序规则
      @SuppressWarnings("unchecked")
      private void siftDownComparable(int k, E x) {
           Comparable<? super E> key = (Comparable<? super E>)x;
           // 计算非叶子节点元素的最大位置
           int half = size >>> 1;        // loop while a non-leaf
           //如果k不是叶子节点
           while (k < half) {
               //左孩子
               int child = (k << 1) + 1; // assume left child is least
               Object c = queue[child];
               //右孩子
               int right = child + 1;
               //如果右孩子小于左孩子，c赋值为右孩子的值
               if (right < size &&
                   ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
                   c = queue[child = right];
               //如果c大于等于key,结束
               if (key.compareTo((E) c) <= 0)
                   break;
               //改变k的位置，向下移动
               queue[k] = c;
               k = child;
           }
           queue[k] = key;
      }
      //外部比较器不为空的时候，与上面唯一区别就是这里采用的是外部排序，上面采用的是内部排序
      @SuppressWarnings("unchecked")
      private void siftDownUsingComparator(int k, E x) {
           int half = size >>> 1;
           while (k < half) {
               int child = (k << 1) + 1;
               Object c = queue[child];
               int right = child + 1;
               if (right < size &&
                   comparator.compare((E) c, (E) queue[right]) > 0)
                   c = queue[child = right];
               if (comparator.compare(x, (E) c) <= 0)
                   break;
               queue[k] = c;
               k = child;
           }
           queue[k] = x;
      }
      //删除对象o，调用的是removeAt
      public boolean remove(Object o) {
           int i = indexOf(o);
           if (i == -1)
               return false;
           else {
               removeAt(i);
               return true;
           }
      }
      //删除位置为i的元素
      private E removeAt(int i) {
           // assert i >= 0 && i < size;
           modCount++;
           int s = --size;
           //如果是队尾，队尾赋值为空
           if (s == i) // removed last element
               queue[i] = null;
           else {
               E moved = (E) queue[s];
               queue[s] = null;
               //下滤
               siftDown(i, moved);
               //如果下滤后，元素没有发生改变，则执行上滤
               if (queue[i] == moved) {
                   siftUp(i, moved);
                   if (queue[i] != moved)
                       return moved;
               }
           }
           return null;
       }
    }

PriorityQueue类在Java1.5中引入并作为 Java Collections Framework 的一部分。PriorityQueue是基于优先堆的一个无界队列，这个优先队列中的元素可以默认自然排序或者通过提供的Comparator（比较器）在队列实例化的时排序。
优先队列不允许空值，而且不支持non-comparable（不可比较）的对象，比如用户自定义的类。优先队列要求使用Java Comparable和Comparator接口给对象排序，并且在排序时会按照优先级处理其中的元素。

优先队列的头是基于自然排序或者Comparator排序的最小元素。如果有多个对象拥有同样的排序，那么就可能随机地取其中任意一个。当我们获取队列时，返回队列的头对象。

优先队列的大小是不受限制的，但在创建时可以指定初始大小。当我们向优先队列增加元素的时候，队列大小会自动增加。

PriorityQueue是非线程安全的，所以Java提供了PriorityBlockingQueue（实现BlockingQueue接口）用于Java多线程环境。




