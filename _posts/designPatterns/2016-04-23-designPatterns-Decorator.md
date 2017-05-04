---
layout: post
title: 装饰模式
date: 2016-04-23 22:00:01 +8000
category: 设计模式
tags: 结构型
---

* content
{:toc}

>动态的给一个对象添加一些额外的职责，就增加功能来说，装饰模式比生产子类更为灵活。

通过使用修饰模式，可以在运行时扩充一个类的功能。原理是：增加一个修饰类包裹原来的类，包裹的方式一般是通过在将原来的对象作为修饰类的构造函数的参数。装饰类实现新的功能，但是，在不需要用到新功能的地方，它可以直接调用原来的类中的方法。修饰类必须和原来的类有相同的接口。

修饰模式是类继承的另外一种选择。类继承在编译时候增加行为，而装饰模式是在运行时增加行为(理解:因为类继承,在编译期就已经确定了对象类型,对象调用哪个方法.而装饰模式,在编译期是无法确定包裹对象的真实类型的,只能在运行时才可以确定)。

#### 装饰模式与类继承的区别

* 装饰模式是一种动态行为，对已经存在类进行随意组合，而类的继承是一种静态的行为，一个类定义成什么样的，该类的对象便具有什么样的功能，无法动态的改变。
* 装饰模式扩展的是对象的功能，不需要增加类的数量，而类继承扩展是类的功能，在继承的关系中，如果我们想增加一个对象的功能，我们只能通过继承关系，在子类中增加方法。
* 装饰模式是在不改变原类文件和使用继承的情况下，动态的扩展一个对象的功能，它是通过创建一个包装对象，也就是装饰来包裹真实的对象。

#### 装饰模式的特点
* 装饰对象和真实对象具有相同的接口，这样客户端对象就可以以真实对象的相同的方式和装饰对象交互。
* 装饰对象包含一个真实对象的引用(reference).
* 装饰对象接受所有来自客户端的请求，它把这些请求转发给真实的对象。
* 装饰对象可以在转发这些请求以前或者以后增加一些附加的功能。这样就能确保在运行时，不用修改给定对象结构就可以在外部增加附加的功能。在面向对象的程序设计中，通常是使用继承的关系来扩展给定类的功能。

### 类结构图

![](/img/designPatterns/decorator.png)

#### 抽象构件（Component）角色

Component是定义一个对象接口，可以给这些对象动态地添加职责。

    abstract class Component
    {
       public abstract void Operation();
    }

#### 具体构件（ConcreteComponent）角色

ConcreteComponent是定义了一个具体的对象，也可以给这个对象添加一些职责。

    class ConcreteComponent : Component
    {
       public override void Operation()
       {
          println("具体对象的操作");
       }
    }

#### 装饰（Decorator）角色

Decorator,装饰抽象类，继承了Component,从外类来扩展Component类的功能，但对于Component来说，是无需知道Decorator的存在。

    abstract class Decorator : Component
    {
       protected Component component;

       public void SetComponent(Component component)
       {
          this.component = component;
       }

       public override void Operation()
       {
          component.Operation();
       }
    }

#### 具体装饰（ConcreteDecorator）角色

ConcreteDecorator就是具体的装饰对象，起到给Component添加职责的功能。

    class ConcreteDecoratorA : Decorator
    {
       private String addedState;

       public override void Operation()
       {
          component.Operation();
          addedState = "New State";
          println("具体装饰对象A的操作");
       }
    }

学习知识就是要去思考，去举一反三。这样学习才会有效率。所以我们可以把装饰模式变通一下，如果只有一个ConcreteComponent类而没有抽象的Component类，那么Decorator类可以使ConcreteComponent的一个子类。同样道理，如果只有一个ConcreteDecorator类，那么就没有必要建立一个单独的Decoration类，而可以把Decorator和ConcreteDecorator的责任合并成一个类。

JAVA中IO流就是这种方式,以BufferedReader类作为分析一下。

具体构件（ConcreteComponent）角色:

    public abstract class Reader
    {

       public int read(java.nio.CharBuffer target) throws IOException {
        int len = target.remaining();
        char[] cbuf = new char[len];
        int n = read(cbuf, 0, len);
        if (n > 0)
            target.put(cbuf, 0, n);
        return n;
       }

       public int read() throws IOException {
        char cb[] = new char[1];
        if (read(cb, 0, 1) == -1)
            return -1;
        else
            return cb[0];
       }

       public int read(char cbuf[]) throws IOException {
        return read(cbuf, 0, cbuf.length);
       }

       abstract public int read(char cbuf[], int off, int len) throws IOException;

    }

具体装饰（ConcreteDecorator）角色:

    public class BufferedReader extends Reader
    {
       private Reader in;

       private char cb[];

       private int nChars, nextChar;

       private static int defaultCharBufferSize = 8192;

       public BufferedReader(Reader in, int sz) {
        super(in);
        if (sz <= 0)
            throw new IllegalArgumentException("Buffer size <= 0");
        this.in = in;
        cb = new char[sz];
        nextChar = nChars = 0;
       }

       public BufferedReader(Reader in) {
        this(in, defaultCharBufferSize);
       }

       public String readLine() throws IOException {
        return readLine(false);
       }
    }

从上面源码，不难看出，BufferedReader是Reader的子类，并持有Reader对象的一个引用。具体装饰角色本质还是在操作具体构建角色，动态的给具体构建角色增加了一些额外的职责，这个readLine()方法就是对read()方法进行了增强。

参考资料:

1. [维基百科-装饰模式](https://zh.wikipedia.org/wiki/修饰模式)

2. 《大话设计模式》第6章 装饰模式






