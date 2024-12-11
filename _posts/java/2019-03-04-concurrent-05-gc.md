---
layout: post
title: java 并发基础 05 GC
categories: Java
description: java 并发基础 05 GC
keywords: java, 并发, concurrent, lock, jvm, gc, 垃圾回收
---

JVM 以一个进程的身份运行在操作系统上，掌握 JVM 的 GC 机制，有利于更加深入掌握 java 语言。

本文主要讲解 JVM 的垃圾回收机制。关于 JVM 组成请[点击查看](/concurrent-04-jvm)。

## 垃圾回收判定

垃圾回收是 JVM 内存管理的核心部分。

### 对象引用类型

java 一共有强、软、弱、虚四种引用类型。区别如下：

|引用类型|描述|
|:---|:---|
| 强引用|对象的默认引用类型|
| 软引用 SoftReference |GC 时被标记，下次 GC 时若堆内存空间不足，则对象会被回收。|
| 弱引用 WeakReference | GC 发生时对象会直接被回收。|
| 虚引用 PhantomReference | 完全不影响 GC。|

### 对象回收判定

如何判断一个对象是否可以被回收呢？可以使用如下两种算法：

#### 引用计数法

> 给对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就加 1；当引用失效时，计数器值就减 1；任何时刻计数器为 0 的对象就是不可能再被使用的。

存在问题如下：

> 很难解决对象之间相互循环引用的问题。

#### 可达性分析算法

`Reachability Analysis` 算法的基本思路就是通过一系列的称为 “GC Roots” 的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链（Reference Chain），当一个对象到 GC Roots 没有任何引用链相连（用图论的话来说，就是**从 GC Roots 到这个对象不可达**）时，则证明此对象是不可用的。

### GC 逃逸

某个对象被判定要 GC 后会被放置在 F-Queue 队列之中，JVM 会建立一个低优先级的 Finalizer 线程去访问这个队列，触发对象的 `finalize()` 方法，但并不一定会等待方法结束（是为了避免队列中其他对象一直等待被处理，甚至导致 GC 操作崩溃）。

Finalizer 线程访问 F-Queue 队列后会标记其中的对象，之后就会被回收。

> 如果对象要在finalize（）中成功拯救自己——只要重新与引用链上的任何一个对象建立关联即可，譬如把自己（this关键字）赋值给某个类变量或者对象的成员变量，那在第二次标记时它将被移除出“即将回收”的集合；如果对象这时候还没有逃脱，那基本上它就真的被回收了。
>
> ```java
> /**
> *此代码演示了两点：
> *1.对象可以在被GC时自我拯救。
> *2.这种自救的机会只有一次，因为一个对象的finalize()方法最多只会被系统自动调用一次
> *@author zzm
> */
> public class FinalizeEscapeGC {
>     public static FinalizeEscapeGC SAVE_HOOK = null;
>
>     public void isAlive() {
>         System.out.println("yes,i am still alive：)");
>     }
>
>     @Override
>     protected void finalize() throws Throwable {
>         super.finalize();
>         System.out.println("finalize mehtod executed!");
>         FinalizeEscapeGC.SAVE_HOOK = this;
>     }
>
>     public static void main(String[] args) throws Throwable {
>         SAVE_HOOK = new FinalizeEscapeGC();
>         //对象第一次成功拯救自己
>         SAVE_HOOK = null;
>         System.gc();
>         //因为finalize方法优先级很低，所以暂停 0.5 秒以等待它
>         Thread.sleep(500);
>
>         if (SAVE_HOOK != null) {
>             SAVE_HOOK.isAlive();
>         } else {
>             System.out.println("no,i am dead：(");
>         }
>
>         //下面这段代码与上面的完全相同，但是这次自救却失败了
>         SAVE_HOOK = null;
>         System.gc();
>         //因为finalize方法优先级很低，所以暂停0.5秒以等待它
>         Thread.sleep(500);
>
>         if (SAVE_HOOK != null) {
>             SAVE_HOOK.isAlive();
>         } else {
>             System.out.println("no,i am dead：(");
>         }
>     }
> }
>
> ```

值得注意的地方是，任何一个对象的 `finalize()` 方法都**只会被系统自动调用一次**，下一次回收时对象的 `finalize()` 方法不会被再次执行，对象最终还是会被回收。

### 对象分配策略

- 对象优先在 eden 分配
- 大对象直接进入 tenured gen
- 长期存活对象将进入 tenured gen

默认 `-XX:MaxTenuringThreshold=15`

- 动态对象年龄判定

如果 Survivor 空间中相同年龄所有对象大小的总和大于 Survivor 空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到 MaxTenuringThreshold 中要求的年龄。

### GC 类型及触发原因

- Minor GC 新生代垃圾回收
  - eden 内存不足
  - 发生了 Full GC
- Full GC(Major GC) 老年代回收
  - tenured 内存不足
  - survivor 移动到老年代失败
  - old 内最大连续内存空间不足
  - 手动调用 `System.gc()`

## 垃圾回收算法

下面介绍常用的垃圾回收算法。

### 标记-清除算法

“标记-清除”（Mark-Sweep）算法是**最基础**的收集算法，算法分为“标记”和“清除”两个阶段：

- 标记：标记出所有要被回收的对象
- 清除：在标记完成后统一回收所有被标记的对象

标记-清除算法主要不足在效率和空间消耗两方面：

- 效率问题
  - 标记和清除两个过程的效率都不高
- 空间问题
  - 标记清除之后会产生**大量不连续的内存碎片**
  - 可能导致无法给较大对象分配**连续内存**而被迫**提前触发 GC**

### 复制算法

“复制”（Copying）算法主要是解决效率问题。过程如下：

- 将可用内存等分为 A B 两块
- 只在 A 部分内分配对象
- 当 A 用尽后，将还存活着的对象复制到 B，然后**把整个 A 清空**

这样一来内存分配时也就不用考虑内存碎片等复杂情况，只要移动堆顶指针，按顺序分配内存即可。实现简单，运行高效。但是显然这种算法的代价很大，直接将内存缩小为了原来的一半。

需要注意的是，现代商业 JVM 大部分都是基于复制算法来回收的新生代，但并不是按照 1:1 的比例分配内存，默认是 8:1:1(Eden:Survivor:“浪费空间”，eden:s0:s1)，即 90% 的新生代空间是可用的，10% 用来做复制操作。

### 标记-整理算法

标记-整理（Mark-Compact）算法适合应用在老年代这种空间宝贵的分区。

此算法的标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存，这样就避免了大量内存碎片的产生。

- 标记：标记出所有需要回收的对象
- 整理：将所有存活的对象移动到一端，然后将边界外的内存清理。

### 分代收集算法

现代商业虚拟机都是采用“分代收集”（Generational  Collection）算法，根据对象生存周期将内存划分成不同区域，比如新生代和老年代，然后根据不同分区采用合适的算法：

- 新生代： GC 时会清理大量对象且 GC 频率高，适合采用复制算法；
- 老年代： GC 频率较低且存在大量对象，就适合采用标记-整理算法。

### HotSpot 实现

HotSpot 会在特定的位置，即安全点（Safepoint)，让线程停顿下来开始 GC。Safepoint 的选定既不能太少以致于让 GC 等待时间太长，也不能过于频繁以致于过分增大运行时的负荷。

当线程处于 Waiting 状态或者 Blocked 状态，这时候线程无法响应 JVM 的中断请求，无法到达安全点挂起，对于这种情况，就需要安全区域（Safe Region）来解决。

安全区域是指在一段代码片段之中，引用关系不会发生变化，在这个区域中的任意地方开始 GC 都是安全的。

在线程执行到 Safe Region 中的代码时，若这段时间里 JVM 要发起 GC 就不用管标识自己为 Safe Region 状态的线程了。在线程要离开 Safe Region 时，它要检查系统是否已经完成了根节点枚举（或者是整个 GC 过程），如果完成了，那线程就继续执行，否则它就必须等待。

## 垃圾回收器

垃圾收集器是使用垃圾收集算法进行内存回收的具体实现。

不同的厂商、不同版本的虚拟机所提供的垃圾收集器都可能会有很大差别，并且一般都会提供参数供用户根据自己的应用特点和要求组合出各个年代所使用的收集器。

如下是各个垃圾回收器搭配说明：

![各垃圾回收器搭配说明](/images/posts/different_gc.png)

连线的两个收集器说明可以搭配使用。

### 新生代回收器

下面是新生代常用的回收器。

#### Serial

Serial 收集器是最基本、最早的收集器。

- 采用单线程的方式工作。
- GC 时必须暂停其他所有的工作线程，直到 GC 结束。（“Stop The World”）

可以给 Client 模式下的虚拟机使用。

#### ParNew

ParNew 收集器其实就是 Serial 收集器的多线程版本。

- 是许多 Server 模式下的虚拟机中首选的新生代收集器
- 目前只有它能与CMS收集器配合工作。

#### Parallel Scavenge

- 使用复制算法、并行的多线程收集器（这一点上类似 ParNew）
- 吞吐量优先
- 允许设置最大垃圾收集停顿时间、吞吐量大小。

所谓吞吐量就是CPU用于运行用户代码的时间与CPU总消耗时间的比值，即吞吐量=运行用户代码时间/（运行用户代码时间+垃圾收集时间），虚拟机总共运行了100分钟，其中垃圾收集花掉1分钟，那吞吐量就是99%。

### 老年代回收器

下面是老年代常用的回收器。

#### Serial Old

Serial  Old 是 Serial 收集器的老年代版本。

- 是一个单线程收集器
- 使用“标记-整理”算法。

可以给 Client 模式下的虚拟机使用。

#### Parallel Old

- 是 Parallel Scavenge 收集器的老年代版本，使用多线程和“标记-整理”算法。
- 在注重吞吐量以及CPU资源敏感的场合，可以使用 Parallel Scavenge 加 Parallel Old 收集器。

#### CMS

CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器，适合要求服务快速响应、GC 停顿时间短的系统。

- 基于标记-清除算法
- 多线程实现并发标记与并发清除。
- 默认会消耗较多 CPU 资源
- 会有较多内存碎片

### G1 收集器

> `-XX:+UseG1GC`
>
> Enables the use of the garbage-first (G1) garbage collector. It is a server-style garbage collector, targeted for multiprocessor machines with a large amount of RAM. It meets GC pause time goals with high probability, while maintaining good throughput. The G1 collector is recommended for applications requiring large heaps (sizes of around 6 GB or larger) with limited GC latency requirements (stable and predictable pause time below 0.5 seconds).
>
> By default, this option is disabled and the collector is chosen automatically based on the configuration of the machine and type of the JVM.

（详见 [https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABFAFAE](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABFAFAE)）

G1（Garbage-First）收集器具备如下特点：

- 并行与并发
  - 充分利用多 CPU 来缩短 Stop-The-World 停顿的时间
  - 甚至能通过并发的方式避免某些 GC 造成的业务线程停顿

- 分代收集
  - 仍采用分代概念
  - 管理整个 GC 堆
  - 优化对象标记

- 空间整合
  - 基于“标记—整理”算法与“复制”算法实现，避免了产生内存空间碎片，收集后能提供规整的可用内存，这样在分配大对象时就不会因为无法找到连续内存空间而提前触发下一次 GC。

- GC 停顿可预测

允许指定在 M 毫秒内 GC 最多消耗 N 毫秒。

G1 将整个堆内存等分为多个独立区域（Region），新生代和老年代不再是物理隔离的，是一部分 Region（不需要连续）的集合。

G1 收集器之所以能建立可预测的停顿时间模型，是因为它可以有计划地避免在整个 Java 堆中进行全区域的垃圾收集。

G1 跟踪各个 Region 里面的垃圾堆积的价值大小（回收所获得的空间大小以及回收所需时间的经验值），并在后台维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region（这也就是 Garbage-First 名称的来由）。这种使用 Region 划分内存空间以及有优先级的区域回收方式，保证了 G1 收集器在有限的时间内可以获取尽可能高的收集效率。

Young 区的回收仍旧采用 STW 的方式进行，通过将对象从一个区域复制到另一个区域实现垃圾回收，避免了内存碎片；

若一个对象空间超过了 Region 的一半则会被分配到老年代；若其为短期存在则将其存放在 Humongous 区，此过程可能会启动 FullGC 来分配连续的 Humongous 区。

- 提供了 3 种模式
  - young gc
    - Eden 空间不足时，将 Eden 内数据移动到 Survivor 或晋升到 Old
    - 会 STW
  - mixed gc
    - 对象晋升到老年代速度太快时（回收整个新生代以及一部分老年代）
    - 三色标记法
  - full gc ： 老年代空间不足时（是一种 serial gc）

建议看这里：[Getting Started with the G1 Garbage Collector](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/G1GettingStarted/index.html)



### ZGC 收集器

自 JDK 11 开始支持，是一个可扩展的低延迟回收器，其特点如下：

- 暂停时间不超过 10 毫秒
- 暂停时间不会随堆或实时设置大小而增加
- 处理堆范围从几百 M 到几 TB。

具体原理说明如下：

- GC 线程和应用线程并行
- 使用 Colored Pointer 和 Load Barrier 实现并发执行
- 灵活的划分 Region 进行堆内存清理
  - 会动态地调整 Region 大小，不同大小的对象分配到不同 Page。
- 将活着的对象都移动到另一个 Region，整个回收掉原来的 Region。
  - 先停顿并标记 GC Roots 到达的对象
  - 并发地递归标记这些对象可达的其他对象
  - 移动对象并压缩清理
- 递增式清理
  - 每次清理一部分
  - 保证 GC 停顿时间小
- 目前不分代
  - 增大 GC 堆大小以提高吞吐

小结： 本文主要整理了 JVM 相关知识点。

***参考：***

周志明 《深入理解java虚拟机：JVM高级特性与最佳实践》

美团技术团队： [Linux与JVM的内存关系分析](http://www.open-open.com/lib/view/open1420814127390.html "Linux与JVM的内存关系分析")
