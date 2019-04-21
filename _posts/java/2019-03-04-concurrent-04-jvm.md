---
layout: post
title: java并发基础04-jvm
categories: Java
description: java并发基础04-jvm
keywords: Java, java, jdk, openjdk, concurrent, lock, jvm
---

整理 jvm 相关内容。

JVM 以一个进程(Process)的身份运行在Linux系统上，了解Linux与进程的内存关系，是理解JVM与Linux内存的关系的基础。

# 1. Linux 内存空间

Linux 的内存空间由物理内存和swap(位于磁盘)两个部分组成。整个内存空间被划分成内核内存(Kernel Space)、用户内存(User Space)。

内核内存是 Linux 自身使用的内存空间，主要提供给程序调度、内存分配、连接硬件资源等程序逻辑使用。用户内存是提供给各个进程主要空间，Linux给各个进程提供相同的虚拟内存空间；这使得进程之间相互独立，互不干扰。实现的方法是采用虚拟内存技术：给每一个进程一定虚拟内存空间，而只有当虚拟内存实 际被使用时，才分配物理内存。

JVM本质就是一个进程，因此其内存空间(也称之为运行时数据区，注意与JMM的区别)也有进程的一般特点。但是，JVM又不是一个普通的进程，其在内存空间上有许多崭新的特点，主要原因有两 个：1.JVM将许多本来属于操作系统管理范畴的东西，移植到了JVM内部，目的在于减少系统调用的次数；2. Java NIO，目的在于减少用于读写IO的系统调用的开销。

JVM进程与普通进程内存模型比较如下图:

![image](https://github.com/stdupanda/stdupanda.github.io/raw/master/images/posts/process_jvm.jpg)

需要说明的是，这个模型的并不是JVM内存使用的精确模型，更侧重于从操作系统的角度而省略了一些JVM的内部细节(尽管也很重要)。下面从用户内存和内核内存两个方面讲解JVM进程的内存特点。

## 1.1. 用户内存

> 永久代本质上是Java程序的代码区和数据区。Java程序中类(class)，会被加载到整个区域的不同数据结构中去，包括常量池、域、方法数据、方法体、构造函数、以及类中的专用方法、实例初始化、接口初始化等。这个区域对于操作系统来说，是堆的一个部分；而对于Java程序来 说，这是容纳程序本身及静态资源的空间，使得JVM能够解释执行Java程序。
>
> 其次是新生代和老年代。新生代和老年代才是Java程序真正使用的堆空间，主要用于内存对象的存储；但是其管理方式和普通进程有本质的区别。
>
> 普通进程在运行时给内存对象分配空间时，会触发一次分配内存空间的系统调用，由操作系统的线程根据对象的大小分配好空间后返回；同时程序释放对象时，也会触发一次系统调用，通知操作系统对象所占用的空间已经可以回收。
>
> JVM对内存的使用和一般进程不同。JVM向操作系统申请一整段内存区域(具体大小可以在JVM参数调节)作为Java程序的堆(分为新生代和老年代)； 当Java程序申请内存空间时，JVM将在这段空间中按所需大小分配给Java程序，并且Java程序不负责通知JVM何时可以释放这个对象的空间，垃圾对象内存空间的回收由JVM进行。
>
> JVM的内存管理方式的优点是显而易见的，包括：第一，减少系统调用的次数。JVM在给Java程序分配内存空间时不需要操作系统干预，仅仅在Java堆大小变化时需要向操作系统申请内存或通知回收，而普通程序每次内存空间的分配回收都需要系统调用参与；第二，减少内存泄漏，普通程序没有(或者没有及时)通知操作系统内存空间的释放是内存泄漏的重要原因之一，而由JVM统一管理，可以避免程序员带来的内存泄漏问题。
>
> 最后是未使用区，未使用区是分配新内存空间的预备区域。对于普通进程来说，这个区域被可用于堆和栈空间的申请及释放，每次堆内存分配都会使用这个区 域，因此大小变动频繁；对于JVM进程来说，调整堆大小及线程栈时会使用该区域，而堆大小一般较少调整，因此大小相对稳定。操作系统会动态调整这个区域的 大小，并且这个区域通常并没有被分配实际的物理内存，只是允许进程在这个区域申请堆或栈空间。

## 1.2. 内核内存

应用程序通常不直接和内核内存打交道，内核内存由操作系统进行管理和使用；不过随着Linux对性能的关注及改进，一些新的特性使得应用程序可以使用内核内存，或者是映射到内核空间。Java NIO正是在这种背景下诞生的，其充分利用了Linux系统的新特性，提升了Java程序的IO性能。

Linux和Java NIO在内核内存上开辟空间给程序使用，主要是减少不要的复制，以减少IO操作系统调用的开销。

## 1.3. jvm内存划分

通过上面的分析，省略比较小的区域，可以总结JVM占用的内存：

JVM内存 ≈ Java永久代 ＋ Java堆(新生代和老年代) ＋ 线程栈＋ Java NIO

SWAP和GC同时发生会导致GC时间很长，JVM严重卡顿，极端的情况下会导致服务崩溃。原因如下：JVM进行GC时，时需要对相应堆分区的已用 内存进行遍历；假如GC的时候，有堆的一部分内容被交换到SWAP中，遍历到这部分的时候就需要将其交换回内存，同时由于内存空间不足，就需要把内存中堆的另外一部分换到SWAP中去；于是在遍历堆分区的过程中，(极端情况下)会把整个堆分区轮流往SWAP写一遍。Linux对SWAP的回收是滞后的，我们就会看到大量SWAP占用。此类问题可尝试用减小堆大小或者增加物理内存等方式解决。

部署Java服务的Linux系统，在内存分配上，需要避免SWAP的使用；具体如何分配需要综合考虑不同场景下JVM对Java永久代 、Java堆(新生代和老年代)、线程栈、Java NIO所使用内存的需求。

# 2. jvm 内存区域划分

Java虚拟机在执行Java程序的过程中会把它所管理的内存划分为若干个不同的数据区域。这些区域都有各自的用途，以及创建和销毁的时间，有的区域随着虚拟机进程的启动而存在，有些区域则依赖用户线程的启动和结束而建立和销毁。根据《Java虚拟机规范(JavaSE 7版)》的规定，Java虚拟机所管理的内存将会包括以下几个运行时数据区域：

![image](https://github.com/stdupanda/stdupanda.github.io/raw/master/images/posts/jvm_runtime_region.png)

## 2.1. 程序计数器 Program Counter Register

程序计数器(Program Counter Register)是一块较小的内存空间，可以看作是当前线程所执行的字节码的位置指示器。按照虚拟机规范，字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，以完成程序功能。

Java虚拟机的多线程是通过线程轮流切换并分配处理器执行时间的方式来实现的，在任何一个确定的时刻，一个处理器(对于多核处理器来说是一个内核)都只会执行一条线程中的指令。因此，为了线程切换后能恢复到正确的执行位置，每条线程都需要有一个独立的程序计数器，各条线程之间计数器互不影响，独立存储，我们称这类内存区域为“**线程私有**” 的内存。

> 如果线程正在执行的是一个Java方法，这个计数器记录的是正在执行的虚拟机字节码指令的地址；如果正在执行的是Native方法，这个计数器值则为空(Undefined)。此内存区域是唯一一个在Java虚拟机规范中**没有规定任何OutOfMemoryError情况**的区域。

## 2.2. 虚拟机栈 VM Stack

虚拟机栈的栈元素是栈帧，当有一个方法被调用时，代表这个方法的栈帧入栈；当这个方法返回时，其栈帧出栈。因此，虚拟机栈中栈帧的入栈顺序就是方法调用顺序。什么是栈帧呢？栈帧可以理解为一个方法的运行空间。它主要由两部分构成，一部分是**局部变量表**，方法中定义的局部变量以及方法的参数就存放在这张表中；另一部分是**操作数栈**，用来存放操作数。

与程序计数器一样，Java虚拟机栈(Java Virtual Machine Stacks)也是**线程私有**的，它的生命周期与线程相同。虚拟机栈描述的是**Java方法执行的内存模型**：每个方法在执行的同时都会创建一个栈帧(Stack Frame，方法运行时的基础数据结构)用于存储**局部变量表、操作数栈、动态链接、方法出口**等信息。每一个方法从调用直至执行完成的过程，就对应着一个栈帧在虚拟机栈中入栈到出栈的过程。

经常有人把Java内存区分为堆内存(Heap)和栈内存(Stack)，这种分法比较粗糙，Java内存区域的划分实际上远比这复杂。这种划分方式的流行只能说明大多数程序员最关注的、与对象内存分配关系最密切的内存区域是这两块。其中所指的“堆”笔者在后面会专门讲述，而所指的“栈”就是现在讲的虚拟机栈，或者说是虚拟机栈中局部变量表部分。

局部变量表存放了**编译期可知**的各种**基本数据类型**(boolean、byte、char、short、int、float、long、double)、**对象引用**(reference类型，它不等同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置)和**returnAddress类型**(指向了一条字节码指令的地址)。其中64位长度的long和double类型的数据会占用2个局部变量空间(Slot)，其余的数据类型只占用1个。局部变量表所需的内存空间在编译期间完成分配，当进入一个方法时，这个方法需要在帧中分配多大的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。

在Java虚拟机规范中，对这个区域规定了两种异常状况：如果**线程请求的栈深度大于虚拟机所允许的深度**，将抛出 `StackOverflowError` 异常；如果虚拟机栈可以动态扩展(当前大部分的Java虚拟机都可动态扩展，只不过Java虚拟机规范中也允许固定长度的虚拟机栈)，如果**扩展时无法申请到足够的内存**，就会抛出 `OutOfMemoryError` 异常。

## 2.3. 本地方法栈 Method Stack

本地方法栈(Native Method Stack)与虚拟机栈所发挥的作用是非常相似的，它们之间的区别不过是虚拟机栈为虚拟机执行**Java方法(也就是字节码)服务**，而本地方法栈则为虚拟机使用到的**Native方法**服务。在虚拟机规范中对本地方法栈中方法使用的语言、使用方式与数据结构并没有强制规定，因此具体的虚拟机可以自由实现它。甚至有的虚拟机(譬如Sun HotSpot虚拟机)**直接就把本地方法栈和虚拟机栈合二为一**。与虚拟机栈一样，本地方法栈区域也会抛出StackOverflowError和OutOfMemoryError异常。

## 2.4. 堆 Heap

Java堆(Java Heap)*被所有线程共享*，在虚拟机启动时创建，是占比最大的内存区域。唯一目的就是存放对象实例，几乎所有的对象实例都在这里分配内存：

> The heap is the runtime data area from which memory for all class instances and arrays is allocated.

Java堆是垃圾收集器管理的主要区域，因此很多时候也被称做“GC堆”(GarbageCollected Heap)。从内存回收的角度来看，由于现在收集器基本都采用分代收集算法，所以Java堆中还可以细分为：新生代和老年代；再细致一点的有Eden空间、From Survivor空间、To Survivor空间等。从内存分配的角度来看，线程共享的Java堆中可能划分出多个线程私有的分配缓冲区(Thread Local Allocation Buffer,TLAB)。不过无论如何划分，都与存放内容无关，无论哪个区域，存储的都仍然是对象实例，进一步划分的目的是为了更好地回收内存，或者更快地分配内存。根据Java虚拟机规范的规定，Java堆可以处于物理上不连续的内存空间中，只要逻辑上是连续的即可，就像我们的磁盘空间一样。在实现时，既可以实现成固定大小的，也可以是可扩展的，不过当前主流的虚拟机都是按照可扩展来实现的(通过-Xmx和-Xms控制)。如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError异常。

## 2.5. 方法区 Method Area

方法区(Method Area)与Java堆一样，是*各个线程共享*的内存区域，它用于存储已被虚拟机加载的`类信息、常量、静态变量、即时编译器编译后的代码`等数据。

有时候方法区又被称为“**永久代**”(Permanent Generation)，原因是因为HotSpot虚拟机的设计团队选择把GC分代收集扩展至方法区，或者说使用永久代来实现方法区而已，这样HotSpot的垃圾收集器可以像管理Java堆一样管理这部分内存，能够省去专门为方法区编写内存管理代码的工作。对于其他虚拟机(如BEA JRockit、IBM J9等)来说是不存在永久代的概念的。本质上两者并不等价。

Java虚拟机规范对方法区的限制非常宽松，除了和Java堆一样不需要连续的内存和可以选择固定大小或者可扩展外，还可以选择**不实现**垃圾收集。相对而言，垃圾收集行为在这个区域是比较少出现的，但并非数据进入了方法区就如永久代的名字一样“永久”存在了。这区域的内存回收目标主要是针对**常量池**的回收和**对类型的卸载**，一般来说，这个区域的回收“成绩”比较难以令人满意，尤其是类型的卸载，条件相当苛刻，但是这部分区域的回收确实是必要的。

注：此处需记录下 `String#intern()` 问题。了解清楚 `String常量池`(jdk启动时已经初始化了一部分字符串) 和 `堆内String对象`(堆内存内是否存在新创建的字符串对象) 的分析。

在jdk1.7之前，字符串常量存储在方法区的PermGen Space，这会导致大量的性能问题和OOM。在jdk1.7之后，字符串常量重新被移到了堆中。

根据Java虚拟机规范的规定，当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常。

可以通过 -XX:MaxPermSize 设定永久代最大可分配的内存空间，默认大小是64M(64位JVM由于指针膨胀，默认是85M)

方法区和永久代：

> 方法区是运行时数据区的一部分，是jvm规范中的一部分，并不是实际的实现；永久带是实现层面的东西，永久代里面存的东西基本上就是方法区规定的那些东西，且并不是所有的jvm中都有永久代，只存在于hotspot jvm中，并且只存在于jdk7和之前的版本中，jdk8中已经彻底移除了永久代，jdk8中引入了一个新的内存区域叫metaspace。

因此，我们可以说，永久带是方法区的一种实现，当然，在hotspot jdk8中metaspace可以看成是方法区的一种实现。

### 2.5.1. 运行时常量池

运行时常量池(Runtime Constant Pool)是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池(Constant Pool Table)，用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。

运行时常量池相对于Class文件常量池的另外一个重要特征是具备动态性，Java语言并不要求常量一定只有编译期才能产生，也就是并非预置入Class文件中常量池的内容才能进入方法区运行时常量池，运行期间也可能将新的常量放入池中，这种特性被开发人员利用得比较多的便是String类的intern()方法。既然运行时常量池是方法区的一部分，自然受到方法区内存的限制，当常量池无法再申请到内存时会抛出OutOfMemoryError异常。

## 2.6. jdk8 后的 jvm 结构调整

jdk8开始，HotSpot VM 移除永久代PermGen，改为MetaSpace元数据空间；

> With the advent of JDK8, we no longer have the PermGen. No, the metadata information is not gone, just that the space where it was held is no longer contiguous to the Java heap. The metadata has now moved to **native memory** to an area known as the “Metaspace”.
>
> The move to Metaspace was necessary since the PermGen was really hard to tune. There was a possibility that the metadata could move with every full garbage collection. Also, it was difficult to size the PermGen since the size depended on a lot of factors such as the total number of classes, the size of the constant pools, size of methods, etc.
>
> Additionally, each garbage collector in HotSpot needed specialized code for dealing with metadata in the PermGen. Detaching metadata from PermGen not only allows the seamless management of Metaspace, but also allows for improvements such as simplification of full garbage collections and future concurrent de-allocation of class metadata.

https://www.infoq.com/articles/Java-PERMGEN-Removed

![image](https://github.com/stdupanda/stdupanda.github.io/raw/master/images/posts/permgen_to_metadata.jpg)

由于元数据区使用本地内存，建议用户限制元数据区大小，或者增大机器内存；同时仍要加强系统质量监控管理。

### 2.6.1. 永久代取消原因

在JDK8之前的HotSpot VM，永久代是一片连续的堆空间。永久代的垃圾收集是和老年代(old generation)捆绑在一起的，因此无论谁满了，都会触发永久代和老年代的垃圾收集。一个明显的问题是，当JVM加载的类信息容量超过了参数-XX：MaxPermSize设定的值时，应用将会报OOM的错误。

使用G1，PermGen仅仅在FullGC(stop-the-word,STW)时才会被收集。G1仅仅在PermGen满了或者应用分配内存的速度比G1并发垃圾收集速度快的时候才触发FullGC。PermGen中类的元数据信息在每次FullGC的时候可能会被收集，但成绩很难令人满意。而且应该为PermGen分配多大的空间很难确定，因为PermSize的大小依赖于很多因素，比如JVM加载的class的总数，常量池的大小，方法的大小等。

另一种更说法是为了利于和 Oracle 的 JRocket 合并。

### 2.6.2. meta元数据区

`java -XX:+PrintFlagsInitial|grep space` 查看meta区大小

- -XX:MetaspaceSize

class metadata的初始空间配额，以bytes为单位，达到该值就会触发垃圾收集进行类型卸载，同时GC会对该值进行调整：如果释放了大量的空间，就适当的降低该值；如果释放了很少的空间，那么在不超过MaxMetaspaceSize(如果设置了的话)，适当的提高该值。

- -XX：MaxMetaspaceSize

可以为class metadata分配的最大空间。默认是没有限制的。但还是建议大家设置这个参数，可能会因为没有限制而导致metaspace被无止境使用(一般是内存泄漏)而被OS Kill。这个参数会限制metaspace被committed的内存大小，会保证committed的内存不会超过这个值，一旦超过就会触发GC，这里要注意和MaxPermSize的区别，MaxMetaspaceSize并不会在jvm启动的时候分配一块这么大的内存出来，而MaxPermSize是会分配一块这么大的内存的。

- -XX：MinMetaspaceFreeRatio

在GC之后，最小的Metaspace剩余空间容量的百分比，减少为class metadata分配空间导致的垃圾收集

- -XX:MaxMetaspaceFreeRatio

在GC之后，最大的Metaspace剩余空间容量的百分比，减少为class metadata释放空间导致的垃圾收集

#### 2.6.2.1. jstat 分析定位

可以使用 `jstat` 查看 metaspace字段。

M，CCS，MC，MU，CCSC，CCSU，MCMN，MCMX，CCSMN，CCSMX

| 项目 | 说明|
|:---|:--|
| `M`     | Metaspace - Percent Used |
| `CCS`   | Compressed Class Space - Percent Used |
| `MC`    | Metaspace Capacity - Current |
| `MU`    | Metaspae Used |
| `CCSC`  | Compressed Class Space Capacity - Current |
| `CCSU`  | Compressed Class Space Used |
| `MCMN`  | Metaspace Capacity - Minimum |
| `MCMX`  | Metaspace Capacity - Maximum |
| `CCSMN` | Compressed Class Space Capacity - Minimum |
| `CCSMX` |Compressed Class Space Capacity - Maximum |

[https://docs.oracle.com/javase/7/docs/technotes/tools/share/jstat.html](https://docs.oracle.com/javase/7/docs/technotes/tools/share/jstat.html "Oracle文档链接")

# 3. JVM 编程相关流程

此部分包括了对象在jvm中的细节问题。

## 3.1. 对象内存结构

在HotSpot虚拟机中，对象在内存中存储的布局可以分为3块区域：对象头(Header)、实例数据(Instance Data)和对齐填充(Padding)。

### 3.1.1. 对象头

HotSpot虚拟机的对象头包括两部分信息，第一部分用于存储对象自身的运行时数据，如哈希码(HashCode)、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等，官方称它为“Mark Word”。

对象头的另外一部分是类型指针，即对象指向它的类元数据的指针，虚拟机通过这个类型指针来确定这个对象是哪个类的实例。另外，如果对象是一个Java数组，那在对象头中还必须有一块用于记录数组长度的数据。

### 3.1.2. 实例数据

实例数据部分是对象真正存储的有效信息，也是在程序代码中所定义的各种类型的字段内容，并且相同宽度的字段总是被分配到一起。

### 3.1.3. 对齐填充

对齐填充部分非必须，无特定含义，它仅仅起着占位符的作用。由于HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说，就是对象的大小必须是8字节的整数倍。而对象头部分正好是8字节的倍数(1倍或者2倍)，因此，当对象实例数据部分没有对齐时，系统需要通过对齐填充部分来补全对象头

## 3.2. 类加载流程

> 虚拟机把描述类的数据从 class 文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的 Java 类型，这就是虚拟机的类加载机制。

### 3.2.1. 运行期加载机制

java 运行期加载机制牺牲了少许性能开销，但为自身带来了很大的灵活性。例如，面向接口编程时运行期指定实现类、自定义ClassLoader加载二进制字节流对应的类、jsp/osgi/cglib 等，都是其灵活性的体现。

### 3.2.2. 类加载过程的各个阶段

加载(Loading)、验证(Verification)、准备(Preparation)、解析(Resolution)、初始化(Initialization)、使用(Using)和卸载(Unloading)7个阶段。

![image](https://github.com/stdupanda/stdupanda.github.io/raw/master/images/posts/class_load_process.png)

#### 3.2.2.1. 类的加载

1. 通过一个类的 Binary Name(全限定名)来获取定义此类的二进制字节流。
2. 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。
3. 在内存中生成一个代表这个类的 `java.lang.Class` 对象，作为方法区这个类的各种数据的访问入口。

比如从 zip/jar/war/ear 包、网络、运行时计算、动态生成类 里获取类的二进制流；

需要说明的是， `java.lang.reflect.Proxy` 就是通过 `ProxyGenerator.generateProxyClass` 来为特定接口生成形式为“*$Proxy”的代理类的二进制字节流。此处不在详细描述开源项目 `cglib(Byte Code Generation Library)`。

加载阶段完成后，jvm就将二进制字节流按照格式要求存储在方法区内，并实例化一个 `java.lang.Class` 类的对象。

#### 3.2.2.2. 验证

文件格式验证、元数据验证、字节码验证、符号引用验证。

##### 3.2.2.2.1. 文件格式验证

1. 文件头 `0xCAFEBABE`
2. 主次版本号是否支持
3. 常量类型是否支持
4. 常量定义是否正确
5. ... 更多的验证细节不再列举

##### 3.2.2.2.2. 元数据验证

1. 类的集成关系
2. 抽象类、接口实现类是否已实现方法
3. 子类与父类是否存在矛盾(语法语义问题)

##### 3.2.2.2.3. 字节码验证

验证程序语义是否合法，是否符合逻辑。例如类型赋值、转换是否合理。

##### 3.2.2.2.4. 符合引用验证

> 符号引用验证的目的是确保解析动作能正常执行，如果无法通过符号引用验证，那么将会抛出一个 java.lang.IncompatibleClassChangeError 异常的子类，如 java.lang.IllegalAccessError、ava.lang.NoSuchFieldError、java.lang.NoSuchMethodError 等。

1. 符号引用中通过字符串描述的全限定名是否能找到对应的类
2. 在指定类中是否存在符合方法的字段描述符以及简单名称所描述的方法和字段。
3. 符号引用中的类、字段、方法的访问性(private、protected、public、default)是否可被当前类访问。

#### 3.2.2.3. 准备

为类变量(`static` 修饰的静态成员变量)分配内存并设置初始值(`final static` 类型此时会初始化为对应值)。等到了类初始化阶段会被实例化为具体值。

#### 3.2.2.4. 解析

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。

#### 3.2.2.5. 初始化

初始化阶段是执行类构造器＜clinit＞()方法的过程。

> ＜clinit＞()方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块(static{}块)中的语句合并产生的。虚拟机会保证在子类的＜clinit＞()方法执行之前，父类的＜clinit＞()方法已经执行完毕。

> ＜clinit＞()方法对于类或接口来说并不是必需的，如果一个类中没有静态语句块，也没有对变量的赋值操作，那么编译器可以不为这个类生成＜clinit＞()方法。

## 3.3. 对象创建流程

虚拟机遇到一条new指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查其代表的类是否已被加载、解析和初始化过。如果没有，那必须先执行相应的类加载过程。在类加载检查通过后，接下来虚拟机将为新生对象分配内存。对象所需内存的大小在类加载完成后便可完全确定，为对象分配空间的任务等同于把一块确定大小的内存从堆中划分出来。

> 假设Java堆中内存是绝对规整的，所有用过的内存都放在一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲空间那边挪动一段与对象大小相等的距离，这种分配方式称为“指针碰撞”（Bump the Pointer）。如果Java堆中的内存并不是规整的，已使用的内存和空闲的内存相互交错，那就没有办法简单地进行指针碰撞了，虚拟机就必须维护一个列表，记录上哪些内存块是可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录，这种分配方式称为“空闲列表”（Free  List）。

> 执行new指令之后会接着执行＜init＞方法，把对象按照程序员的意愿进行初始化，这样一个真正可用的对象才算完全产生出来。

小结： 本文主要整理了 java 常用并发框架知识点。

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

美团技术团队 http://www.open-open.com/lib/view/open1420814127390.html

周志明 《深入理解java虚拟机：JVM高级特性与最佳实践》