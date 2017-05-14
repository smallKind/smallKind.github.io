---
layout: post
title: Class类文件结构
date: 2016-05-31 22:00:01 +8000
category: JVM
tags: jvm
---

* content
{:toc}

Java虚拟机规范的规定，Class文件格式采用一种类似C语言结构体的伪结构来存储，这种伪结构中只有两种数据类型：无符号数和表。

### 无符号数

无符号数属于基本的数据类型，以u1,u2,u4,u8来分别代表1个字节，2个字节，4个字节，8个字节的无符号数，无符号数可以用来描述数字，索引引用，数量值或者按照UTF-8编码构成字符串值。

### 表

表是由多个无符号数或其他表作为数据项构成的复合数据类型，所有表都习惯以“_info”结尾。表用于描述有层次关系的复合结构的数据，整个Class文件本质上就是一张表。

![](/img/virtual/class.png)

### ClassFile结构

#### 魔数（magic）与Class的版本（minor_version,major_version）

每个Class的文件的头4个字节成为魔数（Magic Number）,它唯一的作用就是用于确认这个文件是否为一个能被虚拟机接受的Class文件。

紧接这魔数的4个字节存储的是Class的版本号：第5个字节和第6个字节是次版本号（Minor Version）,第7个字节和第8个字节是住版本号（Major Version）。Java的版本号是从45开始的。

#### 常量池（constant_pool）

紧接着主次版本的号之后的是常量池入口，常量池是Class文件结构中与其他项目关联最多的数据类型，也是占用Class文件空间最大的数据项目之一，同时它还是Class文件中第一个出现的表类型数据项目。

由于常量池中的常量的数量是不固定的，所以在常量池的入口需要放置一项u2类型的数据，代表常量池容量计数值（constant_pool_count）。

常量池之中主要存放两大类常量：字面量和符号引用。字面量比较接近Java语言层次的常量概念，如文本字符串，被声明为final的常量值等。而符号引用则属于编译原理方面的概念，包括了下面三类常量：类和接口的全限定名，字段的名称和描述符，方法的名称和描述符。

常量池中每一项常量都是一个表，共有11种结构各不相同的表结构数据，这11种表结构都有一个相同的特点，就是表开始的第一位都是一个u1类型的标志位。11中常量项的结构定义为如下：

![](/img/virtual/constant1.png)

![](/img/virtual/constant2.png)

#### 访问标志

在常量结束后，接着的2个字节代表访问标志（access_flags），这个标志用于识别一些类或接口层次的访问信息。具体的标志位和标志的含义如下：

![](/img/virtual/accessflags.png)

access_flags中一共有32个标志位可以使用，当前值定义了8个，没有使用到的标志位要求一律为0。

#### 类索引，父类索引和接口索引集合

类索引（this_class）和父类索引（super_class）都是一个u2类型的数据，而接口索引集合（interfaces）是一组u2类型的数据的集合，Class文件中由这三项数据来确定这个类的继承关系。类索引确定这个类的全限定名，父类索引确定这个类的父类的全限定名，接口索引集合就是用来描述这个类实现了哪些接口。

#### 字段表集合

字段表（field_info）用于描述接口或类中声明的变量。字段（field）包括了类级变量或实例级变量，但不包括方法内部声明的变量。

字段可以包含什么信息？

1. 字段的作用域（public，protected，private修饰符）
2. 是类级变量还是实例级变量（static修饰符）
3. 可变性（final）
4. 并发可见性（volatile修饰符，是否强制从主内存读写）
5. 可否序列化（transient修饰符）
6. 字段数据类型（基本类型，对象，数组）
7. 字段名称

字段表结构如下：

![](/img/virtual/field.png)

access_flags可以设置的标志位和含义如下：

![](/img/virtual/fieldflags.png)

接下来是name_index和descriptor_index，它们是对常量池的引用，分别代表这字段的简单名称及字段和方法的描述符。描述符的作用就是用来描述字段的数据类型，方法的参数列表（包括数量，类型以及顺序）和返回值。描述符标识字符含义如下：

![](/img/virtual/descriptor.png)

#### 方法表集合

Class文件存储格式中对方法的描述与对字段的描述几乎采用了完全一致的方式，方法表的结构如同字段表一样。

![](/img/virtual/method.png)

#### 属性表集合

属性表（attribute_info）用于描述某些场景专有的信息。在Class文件，字段表，方法表中都可以携带自己的属性表集合。虚拟机规范预定义的属性：

![](/img/virtual/attribute.png)

对于每一个属性，它的名称都需要从常量池中引用一个CONSTANT_Utf8_info类型的常量来表示，属性的结构完全自定义的。一个符合规则的属性表应该满足如下：

![](/img/virtual/attributestruct.png)

##### Code属性

Java程序方法体里面的代码经过Javac编译器出来后，最终变为字节码指令存储在Code属性内。如果一个方法表内有Code属性存在，它的结构如下：

![](/img/virtual/code.png)

attribute_name_index代表该属性的属性名称。attribute_length指示了属性值的长度。max_stack代表了操作数栈深度的最大值。max_locals代表了局部变量表所需的存储空间。code_length和code用来存储Java源程序编译后生成的字节码指令。code_length代表字节码长度，code用于存储字节码指令的一系列字节流。后面的就是显式异常表集合，异常表对于Code属性来说不是必须存在的。

##### Exceptions属性

Exceptions属性的作用就是列举出方法中可能抛出的受查异常，也就是方法描述时throws关键字后面的异常。结构表如下：

![](/img/virtual/exceptions.png)

##### LineNumberTable属性

LineNumberTable属性用于描述Java源码行号和字节码行号（字节码偏移量）之间的关系。结构表如下：

![](/img/virtual/linenumbertable.png)

##### LocalVariableTable属性

LocalVariableTable属性用于描述栈帧中局部变量表中变量与Java源码中定义的变量之间的关系。结构表如下：

![](/img/virtual/localvariabletable.png)

##### SourceFile属性

SourceFile属性用于记录生成这个Class文件的源码文件名称。结构表如下：

![](/img/virtual/sourcefile.png)

##### ConstantValue属性

ConstantValue属性的作用是通知虚拟机自动为静态变量赋值。结构表如下：

![](/img/virtual/constantvalue.png)

##### InnerClasses属性

InnerClasses属性用于记录内部类与宿主类之间的关联。结构表如下：

![](/img/virtual/innerclasses.png)

##### Depreated及Synthetic属性

Depreated及Synthetic属性两个属性都属于标志类型的布尔属性，只存在有和没有的区别，没有属性值的概念。

### Java虚拟机限制

* 每个类或接口的常量池项最多为65535个
* 类或接口可以声明的字段数最多为65535个
* 类或接口中可以声明的方法数最多为65535个
* 类或接口的直接父类接口最多为65535个
* 方法调用时创建的栈帧，其局部变量表中的最大局部变量数为65535个
* 方法帧中操作数栈的最大深度为635535个
* 方法的参数最多有255个
* 字段和方法名称，字段和方法描述符以及其他常量字符串值的最大长度为65535个字符
* 数组的维度最大为255维

参考：

1. 《深入理解Java虚拟机》