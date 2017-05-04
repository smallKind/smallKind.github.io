---
layout: post
title: 简单工厂模式
date: 2016-04-18 22:00:01 +8000
category: 设计模式
tags: 创建型
---

* content
{:toc}

>又叫做静态工厂方法模式，是由一个工厂对象决定创建出哪一种产品类的实例

简单工厂模式的实质是由一个工厂类根据传入的参数，动态决定应该创建哪一个产品类（这些产品类继承自一个父类或接口）的实例。

该模式中包含的角色及其职责:

#### 抽象产品（Product）角色

简单工厂模式所创建的所有对象的父类，它负责描述所有实例所共有的公共接口。

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

是简单工厂模式的创建目标，所有创建的对象都是充当这个角色的某个具体类的实例。

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

#### 工厂（Creator）角色

简单工厂模式的核心，它负责实现创建所有实例的内部逻辑。工厂类的创建产品类的方法可以被外界直接调用，创建所需的产品对象。

    public class OperationFactory
    {
       public static Operation createOperate(String operate)
       {
          Operation oper = null;
          switch(operation)
          {
             case "+":
                oper = new OperationAdd();
                break;
             case "-":
                oper = new OperationSub();
                break;
             case "*":
                oper = new OperationMul();
                break;
             case "/":
                oper = new OperationDiv();
                break;
          }
          return oper;
       }
    }

### 优点

工厂类是整个模式的关键.包含了必要的逻辑判断,根据外界给定的信息,决定究竟应该创建哪个具体类的对象.通过使用工厂类,外界可以从直接创建具体产品对象的尴尬局面摆脱出来,仅仅需要负责“消费”对象就可以了。而不必管这些对象究竟如何创建及如何组织的．明确了各自的职责和权利，有利于整个软件体系结构的优化。

### 缺点

由于工厂类集中了所有实例的创建逻辑，违反了高内聚责任分配原则，将全部创建逻辑集中到了一个工厂类中；它所能创建的类只能是事先考虑到的，如果需要添加新的类，则就需要改变工厂类了。
当系统中的具体产品类不断增多时候，可能会出现要求工厂类根据不同条件创建不同实例的需求．这种对条件的判断和对具体产品类型的判断交错在一起，很难避免模块功能的蔓延，对系统的维护和扩展非常不利；

参考资料：

1. [百度百科-简单工厂模式](http://baike.baidu.com/view/1227908.htm)

2. 《大话设计模式》第一章 简单工厂模式


