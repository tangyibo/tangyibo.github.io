---
layout: post
title: jvm的内存结构:| java
category: java
tag: [java]
---

这篇文章主要介绍了jvm的内存结构与垃圾回收。
本文分为以下几个部分：
1. jvm的内存结构
2. java堆的垃圾回收


# 1.jvm的内存结构

jvm的内存结构如图所示:

![structure](https://github.com/tangyibo/tangyibo.github.io/blob/master/_posts/imgs/jvm.jpg?raw=true)

## 1.1.程序计数器

程序计数器是一块较小的内存空间，它可以看做是当前线程执行的字节码写的行号指示器。他是线程私有的，按照我的理解就是，它相当于马路上的路标，当程序执行的时候，他会获取相应的指令，让代码运行下去，程序计数器是java虚拟机中唯一没有 OutOfMemoryError情况的区域

## 1.2.Java虚拟机栈

Java虚拟机栈和程序计数器一样是线程私有的，他的生命周期和线程相同，当运行每个方法的时候都会创建一个栈帧，这里面存储局部变量表，操作数栈，动态链接，方法出口等信息。
局部变量表就是储存的基本数据类型(\boolean,byte,char,short,int,float,long,double),对象引用等等，long和double会占用2个局部变量的空间，其余的数据类型都只占用1个，方法运行期间局部变量表的大小固定
Java 虚拟机栈规定了两种异常，线程请求栈的深度大于当前虚拟机所允许的深度，会抛出StackOverflowError异常（比如递归的时候）还有就是当虚拟机栈扩展时无法申请到足够的内存会有OutOfMemoryError异常。

## 1.3.本地方法栈

本地方法栈和Java虚拟机栈所发挥的作用非常相似，他们之间的区别就是Java虚拟机栈执行的Java方法，本地方法栈执行的是native方法，本地方法栈和Java虚拟机栈一样都会有StackOverflowError和OutOfMemoryError异常

## 1.4.Java堆

- Java堆是虚拟机内存中最大的一块，他是线程共享的，唯一的作用就是存放对象的实例，几乎所有的对象实例都在堆里面分配内存，
- Java 堆是垃圾收集器管理的主要区域，
- Java 堆可以细分为：新生代和老年代，再详细就分为：Eden（伊甸园）空间，From Survivor和To Survivor（幸存者）空间等，这样分配主要目的是为了更好地回收内存，内存图所示：

![structure](https://github.com/tangyibo/tangyibo.github.io/blob/master/_posts/imgs/heap_meory.png?raw=true)

- Java堆可以处于物理上不连续的空间，只要逻辑上连续即可，Java堆可以通过-Xmx和-Xms来控制大小当堆没有内存完成实例分配的时候，会抛出OutOfMemoryError异常

## 1.5.方法区

- 方法区（永久代）也是线程共享的内存区域，他用于储存一杯虚拟机加载的类的信息，常量，静态变量，即时编译器编译后的代码等数据。
- 方法区的大小可以通过-XX：MaxPermSize设置上限，当无法满足内存分配需求时会抛出OutOfMemoryError异常
- 方法区不需要连续的内存，可以选择固定大小，可扩展，还能选择不实现垃圾收集

# 2.java堆的垃圾回收



