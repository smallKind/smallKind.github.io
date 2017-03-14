---
layout: post
title: 单例模式
date: 2016-04-13 22:00:01 +8000
category: 设计模式
tags: 创建型
---

* content
{:toc}

> 保证一个类仅有一个实例，并提供一个访问它的全局访问点

通常我们可以让一个全局变量使得一个对象被访问，但它不能防止你实例化多个对象。一个最好的办法就是，让类自身负责保存它的唯一实例。这个类可以保证没有其他实例可以被创建，并且它可以提供一个访问该实例的方法。

单例模式的优点：

* 在内存中只有一个对象，节省内存空间。

* 避免频繁的创建销毁对象，可以提高性能。

* 避免对共享资源的多重占用。

* 可以全局访问。

单例两种创建方法：

1). 饿汉式：在单例类被加载时候，就实例化一个对象交给自己的引用

    public class Singleton {
        private static Singleton singleton = new Singleton();
        private Singleton(){}
        public static Singleton getInstance(){
            return singleton;
        }
    }

但是存在如果不需要引用对象，就永远占着内存。改进如下：

#### 静态内部类：

    public class Singleton {
        private static class SingletonHolder {
           private static final Singleton INSTANCE = new Singleton();
        }
        private Singleton (){}
        public static final Singleton getInstance() {
          return SingletonHolder.INSTANCE;
       }
    }

这种实现了线程安全，又实现了懒加载功能。


2). 懒汉式：在调用取得实例方法的时候才会实例化对象

    public class Singleton {
        private static Singleton singleton;
        private Singleton(){}

        public static Singleton getInstance(){
            if(singleton==null){
                singleton = new Singleton();
            }
            return singleton;
        }
    }

但在多线程的程序中，多个线程同时访问Singleton类，调用getInstance()方法，会有可能创建多个实例。改进如下：

    public class Singleton {
        private static Singleton singleton;
        private Singleton(){}

        public static synchronized Singleton getInstance(){
            if(singleton==null){
                singleton = new Singleton();
            }
            return singleton;
        }
    }

但为了线程安全，这个实现牺牲了一定的效率。singleton如果初始化了，那么其他线程还需要同步进入getInstance()方法，会造成效率的损失。进一步改进:

#### 双重锁定：

     public class Singleton {
        private static Singleton singleton;
        private static readonly object syncRoot = new object();
        private Singleton(){}

        public static Singleton getInstance(){
            if(singleton==null){
                lock(syncRoot){
                   if(singleton == null){
                      singleton = new Singleton();
                   }
                }
            }
            return singleton;
        }
    }

对于singleton存在的情况，就直接返回。当singleton为null并且同时有两个线程调用getInstance()方法时，它们都将可以通过第一重singleton==null的判断。然后由于lock机制，两个线程只能进去一个，另外一个在外排队等候，必须要其中的一个进去并出来后，另一个才能进去。而如果没有了第二层的singleton==null的判断，则第一个线程创建了实例，而第二个线程还是可以继续创建一个新的实例，这就没有达到单例的目的。


参考资料：

1. [单例模式-23种设计模式-极客学院Wiki](http://wiki.jikexueyuan.com/project/java-design-pattern/singleton-pattern.html)

2.《大话设计模式》第21章 有些类也需计划生育——单例模式