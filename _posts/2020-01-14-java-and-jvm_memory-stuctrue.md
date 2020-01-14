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

## 1.3.本地方法区

本地方法栈和Java虚拟机栈所发挥的作用非常相似，他们之间的区别就是Java虚拟机栈执行的Java方法，本地方法栈执行的是native方法，本地方法栈和Java虚拟机栈一样都会有StackOverflowError和OutOfMemoryError异常

## 1.4.Java堆

- Java堆是虚拟机内存中最大的一块，他是线程共享的，唯一的作用就是存放对象的实例，几乎所有的对象实例都在堆里面分配内存，
- Java 堆可以细分为：新生代和老年代，再详细就分为：Eden（伊甸园）空间，From Survivor和To Survivor（幸存者）空间等，这样分配主要目的是为了更好地回收内存，内存图所示：

![structure](https://github.com/tangyibo/tangyibo.github.io/blob/master/_posts/imgs/heap_meory.png?raw=true)

- Java堆可以处于物理上不连续的空间，只要逻辑上连续即可，Java堆可以通过-Xmx和-Xms来控制大小当堆没有内存完成实例分配的时候，会抛出OutOfMemoryError异常

- Java 堆是垃圾收集器管理的主要区域

## 1.5.方法区

- 方法区（永久代）也是线程共享的内存区域，他用于储存一杯虚拟机加载的类的信息，常量，静态变量，即时编译器编译后的代码等数据。
- 方法区的大小可以通过-XX：MaxPermSize设置上限，当无法满足内存分配需求时会抛出OutOfMemoryError异常
- 方法区不需要连续的内存，可以选择固定大小，可扩展，还能选择不实现垃圾收集

# 2.java堆的垃圾回收

java的堆内存主要被分为三块，新生代、老年代、持久代。三代的特点不同，造就了他们所用的GC算法不同，新生代适合那些生命周期较短，频繁创建及销毁的对象，旧生代适合生命周期相对较长的对象，持久代在Sun HotSpot中就是指方法区（有些JVM中根本就没有持久代这中说法）。首先介绍下新生代、旧生代、持久代的概念及特点：
- 新生代：New Generation或者Young Generation。上面大致分为Eden区和Survivor区，Survivor区又分为大小相同的两部分：From和To。新建的对象都是用新生代分配内存，Eden空间不足的时候，会把存活的对象转移到Survivor中，新生代的大小可以由-Xmn来控制，也可以用-XX:SurvivorRatio来控制Eden和Survivor的比例.
- 老年代：Old Generation。用于存放新生代中经过多次垃圾回收仍然存活的对象，例如缓存对象。老年代占用大小为-Xmx值减去-Xmn对应的值。
- 持久代：Permanent Generation。在Sun的JVM中就是方法区的意思，尽管有些JVM大多没有这一代。主要存放常量及类的一些信息默认最小值为16MB，最大值为64MB，可通过-XX:PermSize及-XX:MaxPermSize来设置最小值和最大值。

## 2.1 垃圾回收机制中的算法

垃圾回收算法需要做的基本事情：

- 发现无用对象

- 回收被无用对象占用的内存空间，使该空间可被程序再次使用

## 2.1.1 可达性检测算法

- (1) 引用计数法（Reference Counting Collector）

引用计数是垃圾收集器中的早期策略。此方法中，堆中的每个对象都会添加一个引用计数器。每当一个地方引用这个对象时，计数器值 +1；当引用失效时，计数器值 -1。任何时刻计数值为 0 的对象就是不可能再被使用的。

这种算法无法解决对象之间相互引用的情况。比如对象有一个对子对象的引用，子对象反过来引用父对象，它们的引用计数永远不可能为 0。

- (2) 根搜索算法

由于引用计数法存在缺陷，所有现在一般使用根搜索算法。

根搜索算法是从离散数学中的图论引入的，程序把所有的引用关系看作一张图，从一个节点 GC ROOT 开始，寻找对应的引用节点，找到这个节点以后，继续寻找这个节点的引用节点，当所有的引用节点寻找完毕之后，剩余的节点则被认为是没有被引用到的节点，即无用的节点。

Java 中可作为 GC Root 的对象：
- 虚拟机栈中引用的对象（本地变量表）
- 方法区中静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中引用的对象（Native对象）


## 2.1.2 垃圾收集算法

在确定了哪些垃圾可以被回收后，垃圾收集器要做的就是进行垃圾的回收，有下面的几中算法：

- (1) 标记-清除（Mark-Sweep）算法

标记-清除算法分为两个阶段：
- 标记阶段：标记出需要被回收的对象。
- 清除阶段：回收被标记的可回收对象的内部空间。

![structure](https://github.com/tangyibo/tangyibo.github.io/blob/master/_posts/imgs/vm_a_1.png?raw=true)

标记-清除算法实现较容易，不需要移动对象，但是存在较严重的问题：
- 算法过程需要暂停整个应用，效率不高。
- 标记清除后会产生大量不连续的内存碎片，碎片太多可能会导致后续过程中需要为大对象分配空间时无法找到足够的空间而提前触发新的一次垃圾收集动作。

- (2) 复制（Copying）算法

为了解决标志-清除算法的缺陷，由此有了复制算法。

复制算法将可用内存分为两块，每次只用其中一块，当这一块内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已经使用过的内存空间一次性清理掉。

![structure](https://github.com/tangyibo/tangyibo.github.io/blob/master/_posts/imgs/vm_a_2.png?raw=true)

但是优缺点总结如下：
- 优点：实现简单，不易产生内存碎片，每次只需要对半个区进行内存回收。
- 缺点：内存空间缩减为原来的一半；算法的效率和存活对象的数目有关，存活对象越多，效率越低。

- (3) 标记-整理（Mark-Compact）算法

为了更充分利用内存空间，提出了标记-整理算法。此算法结合了“标记-清除”和“复制”两个算法的优点。该算法标记阶段和“标志-清除”算法一样，但是在完成标记之后，它不是直接清理可回收对象，而是将存活对象都向一端移动，然后清理掉端边界以外的内存。

![structure](https://github.com/tangyibo/tangyibo.github.io/blob/master/_posts/imgs/vm_a_3.png?raw=true)

- (4) 分代收集（Generational Collection）算法

分代收集算法是目前大部分 JVM 的垃圾收集器采用的算法。

核心思想是根据对象存活的生命周期将内存划分为若干个不同的区域。一般情况下将堆区划分为老年代（Tenured Generation）和新生代（Young Generation），老年代的特点是每次垃圾收集时只有少量对象需要被回收，而新生代的特点是每次垃圾回收时都有大量的对象需要被回收，那么就可以根据不同代的特点采取最适合的收集算法。

区域划分：

```
年轻代（Young Generation）

1. 所有新生成的对象首先都是放在年轻代的。年轻代的目标就是尽可能快速的收集掉那些生命周期短的对象。

2. 新生代内存按照8:1:1的比例分为一个 eden 区和两个 survivor(survivor0,survivor1) 区。一个 Eden 区，两个  Survivor 区(一般而言)。大部分对象在 Eden 区中生成。回收时先将  eden 区存活对象复制到一个 survivor0 区，然后清空 eden 区，当这个 survivor0 区也存放满了时，则将 eden 区和 survivor0 区存活对象复制到另一个 survivor1 区，然后清空 eden 和这个 survivor0 区，此时 survivor0 区是空的，然后将 survivor0 区和 survivor1 区交换，即保持 survivor1 区为空， 如此往复。

3. 当 survivor1区不足以存放 eden 和 survivor0 的存活对象时，就将存活对象直接存放到老年代。若是老年代也满了就会触发一次 Full GC ，也就是新生代、老年代都进行回收。

4.新生代发生的 GC 也叫做 Minor GC ，Minor GC 发生频率比较高(不一定等 Eden 区满了才触发)。

```

```
年老代（Old Generation）

1. 在年轻代中经历了 N 次垃圾回收后仍然存活的对象，就会被放到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象。
2. 内存比新生代也大很多(大概比例是1:2)，当老年代内存满时触发 Major GC 即 Full GC，Full GC 发生频率比较低，老年代对象存活时间比较长，存活率标记高。
```

```
持久代（Permanent Generation） 

用于存放静态文件，如 Java 类、方法等。持久代对垃圾回收没有显著影响，但是有些应用可能动态生成或者调用一些 class ，例如 Hibernate 等，在这种时候需要设置一个比较大的持久代空间来存放这些运行过程中新增的类。
```

## 2.1.3 GC 类型划分

- (1) Minor GC(新生代 GC): 

新生代 GC，指发生在新生代的垃圾收集动作，因为 Java 对象大多都具备朝生熄灭的特点，所以 Minor GC 十分频繁，回收速度也较快。

- (2) Major GC(老年代 GC): 

老年代 GC，指发生在老年代的垃圾收集动作，当出现 Major GC 时，一般也会伴有至少一次的 Minor GC（并非绝对，例如 Parallel Scavenge 收集器会单独直接触发 Major GC 的机制）。 Major GC 的速度一般会比 Minor GC 慢十倍以上。

- (3) Full GC: 

清理整个堆空间—包括年轻代和老年代。Major GC == Full GC。

```
产生 Full GC 可能的原因：

1. 年老代被写满。
2. 持久代被写满。
3. System.gc() 被显示调用。
4. 上一次 GC 之后 Heap 的各域分配策略动态变化。
```

## 2.1.4 垃圾收集器（GC）

不同虚拟机所提供的垃圾收集器可能会有很大差别，下面的例子是 HotSpot虚拟机的垃圾回收器：

- 新生代收集器使用的收集器：Serial、PraNew、Parallel Scavenge。
- 老年代收集器使用的收集器：Serial Old、Parallel Old、CMS。

![structure](https://github.com/tangyibo/tangyibo.github.io/blob/master/_posts/imgs/vm_a_4.png?raw=true)

上面有7中收集器，分为两块，上面为新生代收集器，下面是老年代收集器。如果两个收集器之间存在连线，就说明它们可以搭配使用。这7个收集器大致可分为两大类

#### 1、串行收集器

使用单线程进行垃圾回收的收集器，每次回收时，串行收集器只有一个工作线程，对于并行能力较弱的计算机来说，串行收集器的专注性和独占性往往有更好的性能表现。串行收集器可以在新生代和老年代中使用，根据作用于不同的堆空间，分为新生代串行收集器和老年代收集器。

```
使用配置如下：

-XX:+UseSerialGC :年轻串行（Serial），老年串行（Serial Old）
```

- Serial收集器：

Serial收集器是一个新生代收集器，单线程执行，使用复制算法。它在进行垃圾收集时，必须暂停其他所有的工作线程(用户线程)。是Jvm client模式下默认的新生代收集器。对于限定单个CPU的环境来说，Serial收集器由于没有线程交互的开销，专心做垃圾收集自然可以获得最高的单线程收集效率。

![structure](https://github.com/tangyibo/tangyibo.github.io/blob/master/_posts/imgs/gc_1.png?raw=true)

- Serial Old收集器

1、Serial收集器的老年代版本，它同样是一个单线程收集器，使用“标记-整理”算法。

2、主要意义也是在于给Client模式下的虚拟机使用。

3、如果在Server模式下，那么它主要还有两大用途：

一种用途是在JDK 1.5以及之前的版本中与Parallel Scavenge收集器搭配使用，另一种用途就是作为CMS收集器的后备预案，在并发收集发生Concurrent Mode Failure时使用。

![structure](https://github.com/tangyibo/tangyibo.github.io/blob/master/_posts/imgs/gc_2.png?raw=true)

#### 2、并行收集器

- ParNew收集器

1、Serial收集器的多线程版本

2、单CPU不如Serial，因为存在线程交互的开销

```
使用：
-XX:+UseParNewGC 新生代并行（ParNew），老年代串行（Serial Old）

-XX:ParallelGCThreads=n 设置并行收集器收集时使用的CPU数。并行收集线程数。一般最好和计算机的CPU相当
```

![structure](https://github.com/tangyibo/tangyibo.github.io/blob/master/_posts/imgs/gc_3.png?raw=true)

-  Parallel Scavenge收集器

```
使用：
-XX:+UseParallelGC 
```

新生代使用并行回收收集器，老年代使用串行收集器

1、吞吐量优先”收集器

2、新生代收集器，复制算法，并行的多线程收集器

3、目标是达到一个可控制的吞吐量（Throughput）。

4、吞吐量=运行用户代码时间/（运行用户代码时间+垃圾收集时间），虚拟机总共运行了100分钟，其中垃圾收集花掉1分钟，那吞吐量就是99%。

5、两个参数用于精确控制吞吐量:

```
使用：
-XX：MaxGCPauseMillis 是控制最大垃圾收集停顿时间

-XX：GCTimeRatio 直接设置吞吐量大小

-XX：+UseAdaptiveSizePolicy 动态设置新生代大小、Eden与Survivor区的比例、晋升老年代对象年龄
```

6、并行（Parallel）：指多条垃圾收集线程并行工作，但此时用户线程仍然处于等待状态。

7、并发（Concurrent）：指用户线程与垃圾收集线程同时执行（但不一定是并行的，可能会交替执行），用户程序在继续运行，而垃圾收集程序运行于另一个CPU上。

- Parallel Old收集器

```
使用：
-XX:+UseParallelOldGC 
```

新生代和老年代都使用并行回收收集器

1、Parallel Scavenge收集器的老年代版本，使用多线程和“标记-整理”算法。

2、在注重吞吐量以及CPU资源敏感的场合，都可以优先考虑Parallel Scavenge加Parallel Old收集器。

![structure](https://github.com/tangyibo/tangyibo.github.io/blob/master/_posts/imgs/gc_4.png?raw=true)

- CMS收集器

1、以获取最短回收停顿时间为目标的收集器。

2、非常符合互联网站或者B/S系统的服务端上，重视服务的响应速度，希望系统停顿时间最短的应用

3、基于“标记—清除”算法实现的

4、CMS收集器的内存回收过程是与用户线程一起并发执行的

5、它的运作过程分为4个步骤，包括：

初始标记，“Stop The World”，只是标记一下GC Roots能直接关联到的对象，速度很快

并发标记，并发标记阶段就是进行GC RootsTracing的过程

重新标记，Stop The World”，是为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，但远比并发标记的时间短

并发清除（CMS concurrent sweep）

6、优点：并发收集、低停顿

7、缺点：

对CPU资源非常敏感。
无法处理浮动垃圾，可能出现“Concurrent Mode Failure”失败而导致另一次Full GC的产生。
一款基于“标记—清除”算法实现的收集器

```
-XX:+UseConcMarkSweepGC 应用CMS收集器

-XX:ConcGCThreads 设置并发线程数量

-XX:CMSInitiatingOccupancyFraction 设置当老年代空间实用率达到百分比值时进行一次cms回收,默认为68，当老年代的空间使用率达到68%的时候，会执行CMS回收

如果内存使用率增长的很快，在CMS执行的过程中，已经出现了内存不足的情况，此时CMS回收就会失败，虚拟机将启动老年代串行回收器进行垃圾回收，这回导致应用程序中断，直到垃圾回收完成后才会正常工作，这个过程GC的停顿时间可能较长，所以该值需要根据实际情况设置。

-XX:+UseCMSCompactAtFullCollection 设置cms在垃圾收集完成后进行一次内存碎片整理

-XX:CMSFullGCsBeforeCompaction 设定进行多少次cms回收后，进行一次内存压缩。
```

![structure](https://github.com/tangyibo/tangyibo.github.io/blob/master/_posts/imgs/gc_5.png?raw=true)

- G1（Garbage-First）收集器

1、当今收集器技术发展的最前沿成果之一

2、G1是一款面向服务端应用的垃圾收集器。

3、优点：

并行与并发：充分利用多CPU、多核环境下的硬件优势

分代收集：不需要其他收集器配合就能独立管理整个GC堆

空间整合：“标记—整理”算法实现的收集器，局部上基于“复制”算法不会产生内存空间碎片

可预测的停顿：能让使用者明确指定在一个长度为M毫秒的时间片段内，消耗在垃圾收集上的时间不得超过N毫秒

4、G1收集器的运作大致可划分为以下几个步骤：

初始标记：标记一下GC Roots能直接关联到的对象，需要停顿线程，但耗时很短

并发标记：是从GC Root开始对堆中对象进行可达性分析，找出存活的对象，这阶段耗时较长，但可与用户程序并发执行

最终标记：修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录

筛选回收：对各个Region的回收价值和成本进行排序，根据用户所期望的GC停顿时间来制定回收计划

```
-XX:+UserG1Gc 应用G1收集器

-XX:MaxGCPauseMillis 指定最大停顿时间

-XX:ParallelGCThreads 设置并行回收的线程数量
```

![structure](https://github.com/tangyibo/tangyibo.github.io/blob/master/_posts/imgs/gc_6.png?raw=true)
