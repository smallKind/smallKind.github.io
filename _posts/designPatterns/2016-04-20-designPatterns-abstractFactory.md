---
layout: post
title: 抽象工厂模式
date: 2016-04-20 22:00:01 +8000
category: 设计模式
tags: 创建型
---

* content
{:toc}

>提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类

### 适用性：

在以下情况可以考虑使用抽象工厂模式

* 一个系统要独立于它的产品的创建、组合和表示时
* 一个系统要由多个产品系列中的一个来配置时
* 需要强调一系列相关的产品对象的设计以便进行联合使用时
* 提供一个产品类库，而只想显示它们的接口而不是实现时

### 优点:
* 具体产品从客户代码中被分离出来
* 容易改变产品的系列
* 将一个系列的产品族统一到一起创建

### 缺点:
* 在产品族中扩展新的产品是很困难的，它需要修改抽象工厂的接口

### 类结构图:

![](/img/designPatterns/abstractFactory.gif)

大话设计模式中的例子，用抽象工厂模式的数据库访问程序

该模式中包含的角色及其职责:

#### 抽象产品（Product）角色

    interface IUser
    {
       void Insert(User user);

       User GetUser(int id);
    }

    interface IDepartment
    {
       void Insert(Department department);

       Department GetDepartment(int id);
    }

#### 具体产品（Concrete Product）角色

    class SqlserverUser : IUser
    {
       public void Insert(User user)
       {
         println("在SQL Server中给User表增加一条记录");
       }

       public User GetUser(int id)
       {
         println("在SQL Server中根据ID得到User表一条记录");
       }
    }

    class AccessUser : IUser
    {
       public void Insert(User user)
       {
         println("在Access中给User表增加一条记录");
       }

       public User GetUser(int id)
       {
         println("在Access中根据ID得到User表一条记录");
       }
    }

    class SqlserverDepartment : IDepartment
    {
       public void Insert(Department department)
       {
         println("在SQL Server中给Department表增加一条记录");
       }

       public Department GetDepartment(int id)
       {
         println("在SQL Server中根据ID得到Department表一条记录");
       }
    }

    class AccessDepartment : IDepartment
    {
       public void Insert(Department department)
       {
         println("在Access中给Department表增加一条记录");
       }

       public Department GetDepartment(int id)
       {
         println("在Access中根据ID得到Department表一条记录");
       }
    }

#### 抽象工厂（Creator）角色

    interface IFactory
    {
       IUser CreateUser();

       IDepartment CreateDepartment();
    }

#### 具体工厂（Concrete Creator）角色

    class SqlServerFactory : IFactory
    {
       public IUser CreateUser()
       {
          return new SqlserverUser();
       }

       public IDepartment CreateDepartment()
       {
          return new SqlserverDepartment();
       }
    }

    class AccessFactory : IFactory
    {
       public IUser CreateUser()
       {
          return new AccessUser();
       }

       public IDepartment CreateDepartment()
       {
          return new AccessDepartment();
       }
    }

抽象工厂模式与工厂方法模式有很多地方还是相似的。但抽象工厂模式与工厂方法模式最大的区别在于，工厂方法模式针对的是一个产品等级结构，而抽象工厂模式则需要面对多个产品等级结构，一个工厂等级结构可以负责多个不同产品等级结构中的产品对象的创建 。当一个工厂等级结构可以创建出分属于不同产品等级结构的一个产品族中的所有对象时，抽象工厂模式比工厂方法模式更为简单、有效率。

参考资料：

1. [维基百科-抽象工厂](https://zh.wikipedia.org/wiki/抽象工厂)

2. 《大话设计模式》第15章 抽象工厂模式