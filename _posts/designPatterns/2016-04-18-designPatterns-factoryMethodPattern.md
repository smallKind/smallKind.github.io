---
layout: post
title: 工厂方法模式
date: 2016-04-18 22:00:01 +8000
category: 设计模式
tags: 创建型
---

* content
{:toc}

>定义一个创建对象的接口，但让实现这个接口的类来决定实例化哪个类。工厂方法让类的实例化推迟到子类中进行

创建一个对象常常需要复杂的过程，所以不适合包含在一个复合对象中。创建对象可能会导致大量的重复代码，可能会需要复合对象访问不到的信息，也可能提供不了足够级别的抽象，还可能并不是复合对象概念的一部分。工厂方法模式通过定义一个单独的创建对象的方法来解决这些问题。由子类实现这个方法来创建具体类型的对象。

该模式中包含的角色及其职责:

#### 抽象产品（Product）角色

定义工厂方法所创建的对象的接口

    public class Operation
    {
       private double _numberA = 0;
       private double _numberB = 0;
       public double NumberA
       {
          get { return _numberA; }
          set { _numberA = value; }
       }
       public double NumberB
       {
          get { return _numberB; }
          set { _numberB = value; }
       }
       public virtual double GetResult()
       {
          double result = 0;
          return result;
       }
    }

#### 具体产品（Concrete Product）角色

具体的产品，实现了Product接口

    class OperationAdd : Operation
    {
       public override double GetResult()
       {
          double result = 0;
          result = NumberA + NumberB;
          return result;
       }
    }


    class OperationSub : Operation
    {
       public override double GetResult()
       {
          double result = 0;
          result = NumberA - NumberB;
          return result;
       }
    }

    class OperationMul: Operation
    {
       public override double GetResult()
       {
          double result = 0;
          result = NumberA * NumberB;
          return result;
       }
    }

    class OperationDiv : Operation
    {
       public override double GetResult()
       {
          double result = 0;
          if(NumberB == 0)
             throw new Exception("除数不能为0");
          result = NumberA / NumberB;
          return result;
       }
    }

#### 抽象工厂（Creator）角色

声明工厂方法，该方法返回一个Product类型的对象

    interface IFactory
    {
       Operation CreateOperation();
    }

#### 具体工厂（Concrete Creator）角色

重定义工厂方法以返回一个Concrete Product实例

    class AddFactory : IFactory
    {
       public Operation CreateOperation()
       {
          return new OperationAdd();
       }
    }

    class SubFactory : IFactory
    {
       public Operation CreateOperation()
       {
          return new OperationSub();
       }
    }

    class MulFactory : IFactory
    {
       public Operation CreateOperation()
       {
          return new OperationMul();
       }
    }

    class DivFactory : IFactory
    {
       public Operation CreateOperation()
       {
          return new OperationDiv();
       }
    }

### 适用性：

* 创建对象需要大量重复的代码。

* 创建对象需要访问某些信息，而这些信息不应该包含在复合类中。

* 创建对象的生命周期必须集中管理，以保证在整个程序中具有一致的行为。


简单工厂模式的最大优点在于工厂类中包含了必要的逻辑判断，根据客户端的选择条件动态实例化相关的类，对于客户端来说，去除了与具体产品的依赖。

工厂方法模式实现时，客户端需要决定实例化哪一个工厂来实现运算类，选择判断的问题还是存在的，也就是说，工厂方法把简单工厂的内部逻辑判断转到了客户端代码来进行。想要加功能，本来是修改工厂类的，而现在是修改客户端。

参考资料：

1. [维基百科-工厂方法](https://zh.wikipedia.org/wiki/工厂方法)

2. 《大话设计模式》第8章 工厂方法模式


