---
layout: post
title: java并发基础04-JVM
categories: Java
description: java并发基础04-JVM
keywords: java, 并发, concurrent, lock, jvm, 对象, 加载
---

JVM 以一个进程的身份运行在操作系统上，掌握 JVM 内部组成与运行机制，是深入掌握 java 的基础。

本文主要介绍 JVM 的组成。关于 GC 部分请[点击查看](https://stdupanda.github.io/2019/03/04/concurrent-05-gc)。

## JVM 内存区域划分

Java 虚拟机在执行 Java 程序的过程中会把它所管理的内存划分为若干个不同的数据区域。

根据《Java 虚拟机规范(JavaSE 7版)》的规定，Java 虚拟机所管理的内存将会包括以下几个运行时数据区域：

![image](/images/posts/jvm_runtime_region.png)

> JVM 的内存管理方式的优点如下：
>
> 第一，减少系统调用的次数。JVM 在给 Java 程序分配内存空间时不需要操作系统干预，仅仅在 Java 堆大小变化时需要向操作系统申请内存或通知回收，而普通程序每次内存空间的分配回收都需要系统调用参与；
>
> 第二，由 JVM 统一管理内存，减少内存泄漏问题。

### 程序计数器

程序计数器(Program Counter Register)是一块较小的内存空间，可以看作是当前线程所执行的字节码的位置指示器。按照虚拟机规范，字节码解释器工作时通过改变这个计数器的值来选取下一条需要执行的字节码指令，从而完成程序功能。

Java 虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器（对于多核处理器来说是一个内核）都只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储，也就是说程序计数器是“**线程私有**” 的。

> 如果线程正在执行的是一个 Java 方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是 native 方法，这个计数器值则为空(Undefined)。
>
> 程序计数器是唯一一个在 Java 虚拟机规范中**没有规定任何 OutOfMemoryError 情况**的区域。

### 虚拟机栈

虚拟机栈（VM Stack）的栈元素是栈帧，当有一个方法被调用时，代表这个方法的栈帧入栈；当这个方法返回时，其栈帧出栈。因此，虚拟机栈中栈帧的入栈顺序就是方法调用顺序。

什么是栈帧呢？栈帧可以理解为**一个方法的运行空间**。它主要由两部分构成：

- 一部分是**局部变量表**，方法中定义的局部变量以及方法的参数就存放在这张表中；
- 另一部分是**操作数栈**，用来存放操作数。

与程序计数器一样，Java 虚拟机栈(Java Virtual Machine Stacks)也是**线程私有**的，它的生命周期与线程相同。

虚拟机栈描述的是 **Java 方法执行的内存模型**：每个方法在执行的同时都会创建一个栈帧(Stack Frame，方法运行时的基础数据结构)用于存储**局部变量表、操作数栈、动态链接、方法出口**等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。

> 经常有人把 Java 内存区分为堆内存(Heap)和栈内存(Stack)，这种分法比较粗糙，Java 内存区域的划分实际上远比这复杂。这种划分方式的流行只能说明大多数程序员最关注的、与对象内存分配关系最密切的内存区域是这两块。

局部变量表存放了**编译期可知**的各种：

- **基本数据类型**(boolean、byte、char、short、int、float、long、double)
- **对象引用**(reference 类型，它不等同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置)
- **returnAddress 类型**(指向了一条字节码指令的地址)。

其中 64 位长度的 long 和 double 类型的数据会占用 2 个局部变量空间(Slot)，其余的数据类型只占用 1 个。局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。

在 Java 虚拟机规范中，对这个区域规定了两种异常状况：

- 如果**线程请求的栈深度大于虚拟机所允许的深度**，将抛出 `StackOverflowError` 异常；
- 如果虚拟机栈可以动态扩展(当前大部分的Java虚拟机都可动态扩展，只不过 Java 虚拟机规范中也允许固定长度的虚拟机栈)，如果**扩展时无法申请到足够的内存**，就会抛出 `OutOfMemoryError` 异常。

### 本地方法栈

本地方法栈(Native Method Stack)与虚拟机栈所发挥的作用是非常相似的，它们之间的区别不过是虚拟机栈为虚拟机执行 **Java 方法(也就是字节码)服务**，而本地方法栈则为虚拟机使用到的 **Native 方法**服务。

在虚拟机规范中对本地方法栈中方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机(譬如 Sun HotSpot 虚拟机)**直接就把本地方法栈和虚拟机栈合二为一**。

与虚拟机栈一样，本地方法栈区域也会抛出 StackOverflowError 和 OutOfMemoryError 异常。

### 堆

Java 堆(Java Heap)*被所有线程共享*，在虚拟机启动时创建，是占比最大的内存区域。唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存：

> The heap is the runtime data area from which memory for all class instances and arrays is allocated.

Java 堆是垃圾收集器管理的主要区域，因此很多时候也被称做“GC堆”(GarbageCollected Heap)。

从内存回收的角度来看，由于现在收集器基本都采用分代收集算法，所以 Java 堆中还可以细分为：新生代和老年代；再细致一点的有 Eden 空间、From Survivor 空间、To Survivor 空间等。

从内存分配的角度来看，线程共享的 Java 堆中可能划分出多个线程私有的分配缓冲区(Thread Local Allocation Buffer,TLAB)。

根据 Java 虚拟机规范的规定，Java 堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的即可，就像我们的磁盘空间一样。在实现时，既可以实现成固定大小的，也可以是可扩展的，不过当前主流的虚拟机都是按照可扩展来实现的(通过 -Xmx 和 -Xms 控制)。

如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出 OutOfMemoryError 异常。

### 方法区

方法区(Method Area)与 Java 堆一样，是*各个线程共享*的内存区域，它用于存储已被虚拟机加载的`类信息、常量、静态变量、即时编译器编译后的代码`等数据。

有时候方法区又被称为“**永久代**”(Permanent Generation)，原因是因为 HotSpot 虚拟机的设计团队选择把 GC 分代收集扩展至方法区，或者说使用永久代来实现方法区而已，这样 HotSpot 的垃圾收集器可以像管理 Java 堆一样管理这部分内存，能够省去专门为方法区编写内存管理代码的工作。对于其他虚拟机(如BEA JRockit、IBM J9等)来说是不存在永久代的概念的。本质上两者并不等价。

Java 虚拟机规范对方法区的限制非常宽松，除了和 Java 堆一样不需要连续的内存和可以选择固定大小或者可扩展外，还可以选择**不实现**垃圾收集。相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入了方法区就如永久代的名字一样“永久”存在了。这区域的内存回收目标主要是针对**常量池**的回收和**对类型的卸载**，一般来说，这个区域的回收“成绩”比较难以令人满意，尤其是类型的卸载，条件相当苛刻，但是这部分区域的回收确实是必要的。

注：此处需记录一下 `String#intern()` 问题。了解清楚 `String 常量池`(jdk启动时已经初始化了一部分字符串) 和 `堆内 String对象`(JVM heap内是否存在新创建的字符串对象) 的分析。

在jdk1.7之前，字符串常量存储在方法区的 PermGen Space，这会导致大量的性能问题和 OOM。在 jdk1.7 之后，字符串常量重新被移到了堆中。

根据 Java 虚拟机规范的规定，当方法区无法满足内存分配需求时，将抛出 OutOfMemoryError 异常。

可以通过 -XX:MaxPermSize 设定永久代最大可分配的内存空间，默认大小是 64M(64 位 JVM 由于指针膨胀，默认是 85 M)

#### 方法区和永久代的区别

方法区和永久代的区别如下：

- 方法区是运行时数据区的一部分，是 JVM 规范中的一部分，并不是实际的实现；
- 永久代是实现层面的东西，永久代里面存的东西基本上就是方法区规定的那些东西
- 并不是所有的 JVM 中都有永久代，只存在于 Hot Spot JVM 中，并且只存在于 jdk7 和之前的版本中，jdk8 中已经彻底移除了永久代

因此，我们可以说，永久带是方法区的一种实现，当然，在 Hot Spot jdk8 中 metaspace 可以看成是方法区的一种实现。

#### 运行时常量池

运行时常量池(Runtime Constant Pool)是方法区的一部分，用于存放编译期生成的各种字面量和符号引用。

运行时常量池相对于 class 文件常量池的另外一个重要特征是具备动态性，Java 语言并不要求常量一定只有编译期才能产生，也就是并非预置入class 文件中常量池的内容才能进入方法区运行时常量池，运行期间也可能将新的常量放入池中，这种特性被开发人员利用得比较多的便是 `String#intern()` 方法。

由于运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出 OutOfMemoryError 异常。

## PermGen & Metaspace

### jdk8 开始取消 PermGen

jdk8 开始，HotSpot VM 移除永久代 `PermGen`，改为 `MetaSpace` 元数据空间；

> With the advent of JDK8, we no longer have the PermGen. No, the metadata information is not gone, just that the space where it was held is no longer contiguous to the Java heap. The metadata has now moved to **native memory** to an area known as the “Metaspace”.
>
> The move to Metaspace was necessary since the PermGen was really hard to tune. There was a possibility that the metadata could move with every full garbage collection. Also, it was difficult to size the PermGen since the size depended on a lot of factors such as the total number of classes, the size of the constant pools, size of methods, etc.
>
> Additionally, each garbage collector in HotSpot needed specialized code for dealing with metadata in the PermGen. Detaching metadata from PermGen not only allows the seamless management of Metaspace, but also allows for improvements such as simplification of full garbage collections and future concurrent de-allocation of class metadata.

更多可以阅读：

- [Where Has the Java PermGen Gone?](https://www.infoq.com/articles/Java-PERMGEN-Removed "Where Has the Java PermGen Gone?")
- [About G1 Garbage Collector, Permanent Generation and Metaspace](https://blogs.oracle.com/poonam/about-g1-garbage-collector%2c-permanent-generation-and-metaspace "About G1 Garbage Collector, Permanent Generation and Metaspace")

![image](/images/posts/permgen_to_metadata.jpg)

> **由于元数据区使用本地内存，建议用户限制元数据区大小，或者增大机器内存；同时仍要加强系统质量监控管理。**

- heap
  - young gen
    - eden
    - survivor
      - s0/from
      - s1/to
  - old/tenure gen
- metaspace

> only 'Eden' and 'From' space are available for allocations. 'To' is kept free to be used for copying surviving objects and is omitted while reporting the Young generation capacity.

### 永久代取消原因

- 永久代的垃圾收集是和老年代(old generation)捆绑在一起的，因此无论谁满了，都会触发永久代和老年代的垃圾收集。

一个明显的问题是，当 JVM 加载的类信息容量超过了参数 -XX：MaxPermSize 设定的值时，应用将会报 OOM 的错误。

- 使用 G1，PermGen 仅仅在 FullGC(stop-the-word,STW)时才会被收集。

G1 仅仅在 PermGen 满了或者G1 并发垃圾收集速度不够的时候才触发 FullGC。PermGen 中类的元数据信息在每次 FullGC 的时候可能会被收集，但成绩很难令人满意。而且应该为 PermGen 分配多大的空间很难确定，因为 PermSize 的大小依赖于很多因素，比如 JVM 加载的 class 总数，常量池的大小，方法的大小等。

另一种更说法是为了利于和 Oracle 的 JRocket 合并。

### metaspace 元数据区

`java -XX:+PrintFlagsInitial|grep space` 查看 meta 区大小

- -XX:MetaspaceSize=\<NNN>

class metadata 的初始空间配额，以 bytes 为单位，达到该值就会触发垃圾收集进行类型卸载，同时 GC 会对该值进行调整：如果释放了大量的空间，就适当的降低该值；如果释放了很少的空间，那么在不超过 MaxMetaspaceSize(如果设置了的话)，适当的提高该值。

> where \<NNN> is the initial amount of space(the initial high-water-mark) allocated for class metadata (in bytes) that may induce a garbage collection to unload classes. The amount is approximate. After the high-water-mark is first reached, the next high-water-mark is managed by the garbage collector

- -XX：MaxMetaspaceSize=\<NNN>

可以为 class metadata 分配的最大空间。默认是没有限制的。但还是建议大家设置这个参数，可能会因为没有限制而导致 metaspace 被无止境使用(一般是内存泄漏)而被 OS Kill。实际占用超过这个值就会触发 GC。（MaxMetaspaceSize 并不会在 JVM 启动的时候分配一块这么大的内存出来，而 MaxPermSize 是会分配一块这么大的内存的）

> where \<NNN> is the maximum amount of space to be allocated for class metadata (in bytes). This flag can be used to limit the amount of space allocated for class metadata. This value is approximate. By default there is no limit set.

- -XX：MinMetaspaceFreeRatio=\<NNN>

> where \<NNN> is the minimum percentage of class metadata capacity free after a GC to avoid an increase in the amount of space (high-water-mark) allocated for class metadata that will induce a garbage collection.

- -XX:MaxMetaspaceFreeRatio=\<NNN>

> where \<NNN> is the maximum percentage of class metadata capacity free after a GC to avoid a reduction in the amount of space (high-water-mark) allocated for class metadata that will induce a garbage collection.

## 对象内存结构

在HotSpot虚拟机中，对象在内存中存储的布局可以分为3块区域：对象头(Header)、实例数据(Instance Data)和对齐填充(Padding)。

- 对象头

HotSpot虚拟机的对象头包括两部分信息，第一部分用于存储对象自身的运行时数据，如哈希码(HashCode)、GC 分代年龄、锁状态标志、线程持有的锁、偏向线程 ID、偏向时间戳等，官方称它为“Mark Word”。

对象头的另外一部分是类型指针，即对象指向它的类元数据的指针，虚拟机通过这个类型指针来确定这个对象是哪个类的实例。另外，如果对象是一个 Java 数组，那在对象头中还必须有一块用于记录数组长度的数据。

- 实例数据

实例数据部分是对象真正存储的有效信息，也是在程序代码中所定义的各种类型的字段内容，并且相同宽度的字段总是被分配到一起。

- 对齐填充

对齐填充部分非必须，无特定含义，它仅仅起着占位符的作用。由于 HotSpot VM 的自动内存管理系统要求对象起始地址必须是 8 字节的整数倍，换句话说，就是对象的大小必须是 8 字节的整数倍。而对象头部分正好是8字节的倍数(1 倍或者 2 倍)，因此，当对象实例数据部分没有对齐时，系统需要通过对齐填充部分来补全对象头

## 类加载流程

> 虚拟机把描述类的数据从 class 文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的 Java 类型，这就是虚拟机的类加载机制。

### 运行期加载机制

java 运行期加载机制牺牲了少许性能开销，但为自身带来了很大的灵活性。例如，面向接口编程时运行期指定实现类、自定义 ClassLoader 加载二进制字节流对应的类、jsp/osgi/cglib 等，都是其灵活性的体现。

### 类加载流程的各个阶段

加载(Loading)、验证(Verification)、准备(Preparation)、解析(Resolution)、初始化(Initialization)、使用(Using)和卸载(Unloading)7个阶段。

![image](/images/posts/class_load_process.png)

- 加载

  1. 通过一个类的 Binary Name(全限定名)来获取定义此类的二进制字节流。
  2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
  3. 在内存中生成一个代表这个类的 `java.lang.Class` 对象，作为方法区这个类的各种数据的访问入口。

比如从 zip/jar/war/ear 包、网络、运行时计算、动态生成类 里获取类的二进制流；

需要说明的是， `java.lang.reflect.Proxy` 就是通过 `ProxyGenerator.generateProxyClass` 来为特定接口生成形式为“*$Proxy”的代理类的二进制字节流。此处不在详细描述开源项目 `cglib(Byte Code Generation Library)`。

加载阶段完成后，JVM 就将二进制字节流按照格式要求存储在方法区内，并实例化一个 `java.lang.Class` 类的对象。

- 验证

类的验证主要包括文件格式验证、元数据验证、字节码验证、符号引用验证。

**文件格式验证**

1. 文件头 `0xCAFEBABE` 占四个字节大小
2. 主次版本号是否支持
3. 常量类型是否支持
4. 常量定义是否正确
5. ... 更多的验证细节不再列举

**元数据验证**

1. 类的集成关系
2. 抽象类、接口实现类是否已实现方法
3. 子类与父类是否存在矛盾(语法语义问题)

**字节码验证**

验证程序语义是否合法，是否符合逻辑。例如类型赋值、转换是否合理。

**符合引用验证**

> 符号引用验证的目的是确保解析动作能正常执行，如果无法通过符号引用验证，那么将会抛出一个 `java.lang.IncompatibleClassChangeError` 异常的子类，如 `java.lang.IllegalAccessError`、`java.lang.NoSuchFieldError`、`java.lang.NoSuchMethodError` 等。

1. 符号引用中通过字符串描述的全限定名是否能找到对应的类
2. 在指定类中是否存在符合方法的字段描述符以及简单名称所描述的方法和字段。
3. 符号引用中的类、字段、方法的访问性(private、protected、public、default)是否可被当前类访问。

- 准备

为类变量(`static` 修饰的静态成员变量)分配内存并设置初始值(`final static` 类型此时会初始化为对应值)。等到了类初始化阶段会被实例化为具体值。

- 解析

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。

- 初始化

初始化阶段是执行类构造器 \<clinit>() 方法的过程。

> \<clinit>() 方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块(static{}块)中的语句合并产生的。虚拟机会保证在子类的 \<clinit>() 方法执行之前，父类的 \<clinit>()方法已经执行完毕。
> \<clinit>() 方法对于类或接口来说并不是必需的，如果一个类中没有静态语句块，也没有对变量的赋值操作，那么编译器可以不为这个类生成 \<clinit>() 方法。

## 对象创建流程

虚拟机遇到一条 `new` 指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查其代表的类是否已被加载、解析和初始化过。如果没有，那必须先执行相应的**类加载**过程。在类加载检查通过后，接下来虚拟机将为新生对象分配内存。对象所需内存的大小在类加载完成后便可完全确定，为对象分配空间的任务等同于把一块确定大小的内存从堆中划分出来。

> 假设 Java 堆中内存是绝对规整的，所有用过的内存都放在一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲空间那边挪动一段与对象大小相等的距离，这种分配方式称为“指针碰撞”（Bump the Pointer）；
>
> 如果 Java 堆中的内存并不是规整的，已使用的内存和空闲的内存相互交错，那就没有办法简单地进行指针碰撞了，虚拟机就必须维护一个列表，记录上哪些内存块是可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录，这种分配方式称为“空闲列表”（Free  List）。
>
> 执行 new 指令之后会接着执行 \<init> 方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。

小结： 本文主要整理了 JVM 相关知识点。

```java
//                .-~~~~~~~~~-._       _.-~~~~~~~~~-.
//            __.'              ~.   .~              `.__
//          .'//                  \./                  \\`.
//        .'//                     |                     \\`.
//      .'// .-~"""""""~~~~-._     |     _,-~~~~"""""""~-. \\`.
//    .'//.-"                 `-.  |  .-'                 "-.\\`.
//  .'//______.============-..   \ | /   ..-============.______\\`.
//.'______________________________\|/______________________________`.
```

***致谢：***
周志明 《深入理解java虚拟机：JVM高级特性与最佳实践》

美团技术团队： [Linux与JVM的内存关系分析](http://www.open-open.com/lib/view/open1420814127390.html "Linux与JVM的内存关系分析")
