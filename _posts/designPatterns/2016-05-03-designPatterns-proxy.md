---
layout: post
title: 代理模式
date: 2016-05-03 22:00:01 +8000
category: 设计模式
tags: 结构型
---

* content
{:toc}

>给某个对象提供一个代理对象，并由代理对象控制对于原对象的访问，即客户不直接操控原对象，而是通过代理对象间接地操控原对象。

![](/img/designPatterns/proxy.gif)

为了保持行为的一致性，代理类和委托类通常会实现相同的接口，所以在访问者看来两者没有丝毫的区别。通过代理类这中间一层，能有效控制对委托类对象的直接访问，也可以很好地隐藏和保护委托类对象，同时也为实施不同控制策略预留了空间，从而在设计上获得了更大的灵活性。

代理模式分为：

*  静态代理：代理类是在编译时就实现好的。也就是说 Java 编译完成后代理类是一个实际的 class 文件
*  动态代理：代理类是在运行时生成的。也就是说 Java 编译完之后并没有实际的 class 文件，而是在运行时动态生成的类字节码，并加载到JVM中。

## 静态代理

    interface Subject{
       void request();
    }

    class RealSubject implements Subject{
       public void request(){
          System.out.println("request");
       }
    }

    class Proxy implements Subject{
       private Subject subject;
       public Proxy(){
          //代理关系在编译时确定
          subject = new Subject;
       }
       /**
       public Proxy(Subject subject){
          //装饰器模式关系在运行时确定，由客户端具体传入
          this.subject = subject;
       }
       **/
       public void request(){
          System.out.println("PreProcess");
          subject.request();
          System.out.println("PostProcess");
       }
    }

代理模式和装饰模式都实现了同一个接口，不论使用哪一个模式，都可以很容易地在真实对象的方法前面或者后面加上自定义的方法，有什么区别了？装饰器模式关注于在一个对象上动态的添加方法，然而代理模式关注于控制对对象的访问。用代理模式，代理类（proxy class）可以对它的客户隐藏一个对象的具体信息。因此，当使用代理模式的时候，我们常常在一个代理类中创建一个对象的实例。并且，当我们使用装饰器模式的时候，我们通常的做法是将原始对象作为一个参数传给装饰者的构造器。

## 动态代理

为了让开发人员能在运行时间动态创建代理对象，进一步提高软件的可维护性与可复用性，Java 在反射库中使用 Proxy, InvocationHandler 类支持了动态代理模式。

    public class LogInvocationHandler implements InvocationHandler {

       /**
        * 被代理对象
        */
       private Object obj;

       public LogInvocationHandler(Object obj) {
           this.obj = obj;
       }

       @Override
       public Object invoke(Object proxy, Method method, Object[] args)
            throws Throwable {
           System.out.println("Start.");
           method.invoke(obj, args);
           System.out.println("End.");
           return null;
       }
    }

    public class Client {

       public static void main(String[] args) {

           Subject subject = new RealSubject();
           // 使用 Proxy.newProxyInstance 方法生成代理对象
           Subject proxy = (Subject) Proxy.newProxyInstance(subject.getClass()
                .getClassLoader(), subject.getClass().getInterfaces(),
                new LogInvocationHandler(subject));
           proxy.request();

       }
    }


了解一下如何使用 Java 动态代理。具体有如下四步骤：

1. 通过实现 InvocationHandler 接口创建自己的调用处理器；
2. 通过为 Proxy 类指定 ClassLoader 对象和一组 interface 来创建动态代理类；
3. 通过反射机制获得动态代理类的构造函数，其唯一参数类型是调用处理器接口类型；
4. 通过构造函数创建动态代理类实例，构造时调用处理器对象作为参数被传入。

### 动态代理对象创建过程

    // InvocationHandlerImpl 实现了 InvocationHandler 接口，并能实现方法调用从代理类到委托类的分派转发
    // 其内部通常包含指向委托类实例的引用，用于真正执行分派转发过来的方法调用
    InvocationHandler handler = new InvocationHandlerImpl(..);

    // 通过 Proxy 为包括 Interface 接口在内的一组接口动态创建代理类的类对象
    Class clazz = Proxy.getProxyClass(classLoader, new Class[]{ Interface.class, ... });

    // 通过反射从生成的类对象获得构造函数对象
    Constructor constructor = clazz.getConstructor(new Class[] { InvocationHandler.class });

    // 通过构造函数对象创建动态代理类实例
    Interface Proxy = (Interface)constructor.newInstance(new Object[] { handler });

实际使用过程更加简单，因为 Proxy 的静态方法 newProxyInstance 已经为我们封装了步骤 2 到步骤 4 的过程，所以简化后的过程如下

### 简化的动态代理对象创建过程

    // InvocationHandlerImpl 实现了 InvocationHandler 接口，并能实现方法调用从代理类到委托类的分派转发
    InvocationHandler handler = new InvocationHandlerImpl(..);

    // 通过 Proxy 直接创建动态代理类实例
    Interface proxy = (Interface)Proxy.newProxyInstance( classLoader,
	 new Class[] { Interface.class },
	 handler );

### 代码是最好的老师

通过源代码来了解一下 Proxy 到底是如何实现的。首先记住 Proxy 的几个重要的静态变量：

#### Proxy 的重要静态变量

    //代理类构造函数的参数类型（parameter types of a proxy class constructor ）
    private static final Class<?>[] constructorParams =
        { InvocationHandler.class };

    // 代理级别的缓存（a cache of proxy classes）
    private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
        proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());

    // 关联的调用处理器引用
    protected InvocationHandler h;

#### Proxy 构造方法

    // 由于 Proxy 内部从不直接调用构造函数，所以 private 类型意味着禁止任何调用
    private Proxy() {
    }

    // 由于 Proxy 内部从不直接调用构造函数，所以 protected 意味着只有子类可以调用
    protected Proxy(InvocationHandler h) {
        this.h = h;
    }

####  Proxy 静态方法 newProxyInstance

    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
        // 检查 h 不为空，否则抛异常
        Objects.requireNonNull(h);

        final Class<?>[] intfs = interfaces.clone();
        final SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
        }

        /*
         * Look up or generate the designated proxy class.
         * 查找或者生产指定的代理类
         */
        Class<?> cl = getProxyClass0(loader, intfs);

        /*
         * Invoke its constructor with the designated invocation handler.
         * 通过指定的invocation handler反射获取它的构造函数并生产代理类实例
         */
        try {
            if (sm != null) {
                checkNewProxyPermission(Reflection.getCallerClass(), cl);
            }
            final Constructor<?> cons = cl.getConstructor(constructorParams);
            final InvocationHandler ih = h;
            if (!Modifier.isPublic(cl.getModifiers())) {
                AccessController.doPrivileged(new PrivilegedAction<Void>() {
                    public Void run() {
                        cons.setAccessible(true);
                        return null;
                    }
                });
            }
            return cons.newInstance(new Object[]{h});
        } catch (IllegalAccessException|InstantiationException e) {
            throw new InternalError(e.toString(), e);
        } catch (InvocationTargetException e) {
            Throwable t = e.getCause();
            if (t instanceof RuntimeException) {
                throw (RuntimeException) t;
            } else {
                throw new InternalError(t.toString(), t);
            }
        } catch (NoSuchMethodException e) {
            throw new InternalError(e.toString(), e);
        }
    }

参考资料：

1. [Java 动态代理机制分析及扩展](https://www.ibm.com/developerworks/cn/java/j-lo-proxy1/)

2. 《大话设计模式》第7章 代理模式

