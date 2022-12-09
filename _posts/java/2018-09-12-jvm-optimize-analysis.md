---
layout: post
title: jvm 问题分析和优化
categories: Java
description: jvm 问题分析和优化
keywords: Java, java, jdk, openjdk, JVM, gc
---

整理常见 JVM 异常问题缘由、排查定位技巧、相关工具用法。

置顶区：

[1] Oracle 官方帮助文档入口：[Java Platform, Standard Edition Documentation](https://docs.oracle.com/en/java/javase/index.html)、官方问题排查手册[Troubleshooting Guide](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/)

[2] java 命令行参数（遇到奇怪的 `-XX` 时可以参考这里）： [java command line options](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html)

[3] Oracle 官方文档： [JVM 优化 guide](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/toc.html)

[4] G1 优化 guide： [点击打开默认是 G1 介绍部分](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/g1_gc.html#garbage_first_garbage_collection)、G1 介绍[The Garbage-First Garbage Collector](https://www.oracle.com/technetwork/java/javase/tech/g1-intro-jsp-135488.html)

[5] 为什么 heap 总有一部分内存“不被使用”： [HotSpot JVM throwing OOM even when there is memory available](https://blogs.oracle.com/poonam/hotspot-jvm-throwing-oom-even-when-there-is-memory-available-v2)

[6] G1 回收器里的永久代 [About G1 Garbage Collector, Permanent Generation and Metaspace](https://blogs.oracle.com/poonam/about-g1-garbage-collector%2c-permanent-generation-and-metaspace)

[7] [Getting Started with the G1 Garbage Collector](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/G1GettingStarted/index.html)

## JVM 参数

通过适当的设置 JVM 参数，有利于系统优化以及故障问题分析定位。

- 基本参数

|参数|说明|
|:---|:---|
|`-XshowSettings:all`||
|`-XX:+PrintCommandLineFlags`||
|`-javaagent:<path>`||
|`-X`|打印 `-X` 相关参数的帮助信息|
|`-XX:ErrorFile=filename`|默认是 `./hs_err_pid%p.log`|
|`-XX:+PrintFlagsInitial`|打印各个参数默认值|
|`-XX:+PrintFlagsFinal`|打印出各个运行时生效的选项配置|

- 某些 experimental 参数需要先解锁才可以配置

`-XX:+UnlockExperimentalVMOptions`

- 类加载相关参数

|参数|说明|
|:---|:---|
|`-Xbootclasspath[/a或/p]:path`|append 或 prepend|
|`-XX:+TraceClassLoading`||
|`-XX:+TraceClassResolution`||
|`-XX:+TraceClassUnloading`||

- 常用 G1 参数

|参数|说明|
|:---|:---|
|`-XX:+DisableExplicitGC`|`System.gc()`|
|`-Xnoclassgc`|不允许 gc|
|`-XX:InitialHeapSize=6m`(`-Xms6m`)|initial size (in bytes) of the heap,建议和 `-Xmx80m`/`-XX:MaxHeapSize=80m` 设置相同大小|
|`-XX:MaxHeapSize=80m`(`-Xmx80m`)| maximum size (in bytes) of the memory allocation pool，建议和 `-XX:InitialHeapSize` 设置相同大小|
|`-Xss1m`| thread stack size (in bytes) 同 `-XX:ThreadStackSize=1m` 64 位 linux 默认是 1m|
|`-Xmn6G`|initial and maximum size (in bytes) of the heap for the young generation。 建议是 between a half and a quarter of the overall heap size|
|`-XX:MaxDirectMemorySize=6m`| maximum total size (in bytes) of the New I/O (the java.nio package) direct-buffer allocations.|
|`-XX:MaxMetaspaceSize=50m -XX:MetaspaceSize=50m`| Sets the maximum amount of native memory that can be allocated for class metadata. By default, the size is not limited. The amount of metadata for an application depends on the application itself, other running applications, and the amount of memory available on the system. 可以将初始值和最大值设置为相同大小减少动态分配时间 |
|`-XX:MaxPermSize=size`|deprecated in JDK 8, and superseded by the -XX:MaxMetaspaceSize option.|

- gc 日志记录相关

|参数|说明|
|:---|:---|
|`-XX:+PrintGCDetails`||
|`-XX:+PrintHeapAtGC`|每次gc(yonggc,fullgc) 都会输出gc前后堆详情（Eden区域、from区、to区、old区、Metaspace区 、Class space等）|
|`-verbose:gc`|每次gc(yonggc,fullgc) 输出前后结果信息|
|`-Xloggc:/home/log/gc-%t.log`|将 GC 日志记录到指定文件中|
|`-XX:+PrintGCDateStamps`|记录详细时间，如：`2014-02-28T23:58:42.314+0800`|
|`-XX:+PrintGCTimeStamps`|记录时间戳|
|`-XX:+PrintGCApplicationStoppedTime`|输出gc引起的停顿时间|
|`-XX:+PrintReferenceGC`|打印强软虚引用清理结果|
|`-XX:+PrintTenuringDistribution`|打印分代情况|
|`-XX:+UseGCLogFileRotation`|注意重启后日志文件的影响|
|`-XX:NumberOfGCLogFiles=6`||
|`-XX:GCLogFileSize=6M`||

推荐阅读文档：[https://blog.gceasy.io/2016/11/15/rotating-gc-log-files/](https://blog.gceasy.io/2016/11/15/rotating-gc-log-files/)

GC 日志分析工具有：[GCeasy](https://gceasy.io/)、[GCViewer](https://github.com/chewiebug/GCViewer)

- Serviceability 相关参数

点击进入[官方文档入口](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABFJDIC "Oracle jdk 文档入口")

|参数|说明|
|:---|:---|
|`-XX:+HeapDumpOnOutOfMemoryError`||
|`-XX:HeapDumpPath=path`|默认是 `/java_pid%p.hprof`|
|`-XX:LogFile=path`|默认是 `./hotspot.log`|

## **G1 相关设置**

官网详情可参看 [java启动参数详解](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BABFAFAE)

重点介绍下 G1 回收器。这应该是较长时间内的主力配置了。

`-XX:+UseG1GC` 开启 G1 收集器。

### Important Defaults

|Option and Default Value|Option|
|:---|:---|
|`-XX:G1HeapRegionSize=n`|Sets the size of a G1 region. The value will be a power of two and can range from 1 MB to 32 MB. The goal is to have around 2048 regions based on the minimum Java heap size 不超过 2048 个 region|
|`-XX:MaxGCPauseMillis=200`|Sets a target value for desired maximum pause time. The default value is 200 milliseconds. 目标值，不保证达到|
|`-XX:G1NewSizePercent=5`|Sets the percentage of the heap to use as the minimum for the young generation size. The default value is 5 percent of your Java heap. This is an experimental flag. See How to Unlock Experimental VM Flags for an example. This setting replaces the -XX:DefaultMinNewGenPercent setting.|
|`-XX:G1MaxNewSizePercent=60`|Sets the percentage of the heap size to use as the maximum for young generation size. The default value is 60 percent of your Java heap. This is an experimental flag. See How to Unlock Experimental VM Flags for an example. This setting replaces the -XX:DefaultMaxNewGenPercent setting.|
|`-XX:ParallelGCThreads=n`|Sets the value of the STW worker threads. Sets the value of n to the number of logical processors. The value of n is the same as the number of logical processors up to a value of 8. If there are more than eight logical processors, sets the value of n to approximately 5/8 of the logical processors. This works in most cases except for larger SPARC systems where the value of n can be approximately 5/16 of the logical processors.|
|`-XX:ConcGCThreads=n`|Sets the number of parallel marking threads. Sets n to approximately 1/4 of the number of parallel garbage collection threads (ParallelGCThreads).|
|`-XX:InitiatingHeapOccupancyPercent=45`|Sets the Java heap occupancy threshold that triggers a marking cycle. The default occupancy is 45 percent of the entire Java heap.总堆内存占用百分比达到此值后执行gc；0则一直循环gc，默认为45|
|`-XX:G1MixedGCLiveThresholdPercent=85`|Sets the occupancy threshold for an old region to be included in a mixed garbage collection cycle. The default occupancy is 85 percent. This is an experimental flag. See How to Unlock Experimental VM Flags for an example. This setting replaces the -XX:G1OldCSetRegionLiveThresholdPercent setting.|
|`-XX:G1HeapWastePercent=5`|Sets the percentage of heap that you are willing to waste. The Java HotSpot VM does not initiate the mixed garbage collection cycle when the reclaimable percentage is less than the heap waste percentage. The default is 5 percent.|
|`-XX:G1MixedGCCountTarget=8`|Sets the target number of mixed garbage collections after a marking cycle to collect old regions with at most G1MixedGCLIveThresholdPercent live data. The default is 8 mixed garbage collections. The goal for mixed collections is to be within this target number.|
|`-XX:G1OldCSetRegionThresholdPercent=10`|Sets an upper limit on the number of old regions to be collected during a mixed garbage collection cycle. The default is 10 percent of the Java heap.|
|`-XX:G1ReservePercent=10`|Sets the percentage of reserve memory to keep free so as to reduce the risk of to-space overflows. The default is 10 percent. When you increase or decrease the percentage, make sure to adjust the total Java heap by the same amount.|
|`-XX:+ParallelRefProcEnabled`|并行处理Reference，加快处理速度，缩短耗时|
|`-XX:GCPauseIntervalMillis=200`|target for collection time space|

### Recommendations

When you evaluate and fine-tune G1 GC, keep the following recommendations in mind:

- Young Generation Size

**Avoid explicitly setting young generation size** with the `-Xmn` option or any or other **related** option such as `-XX:NewRatio`. Fixing the size of the young generation overrides the target pause-time goal.

- Pause Time Goals

When you evaluate or tune any garbage collection, there is always a latency versus throughput trade-off.

The G1 GC is an incremental garbage collector with uniform pauses, but also more overhead on the application threads. The throughput goal for the G1 GC is 90 percent application time and 10 percent garbage collection time. Compare this to the Java HotSpot VM parallel collector. The throughput goal of the parallel collector is 99 percent application time and 1 percent garbage collection time. Therefore, when you evaluate the G1 GC for throughput, relax your pause time target. Setting too aggressive a goal indicates that you are willing to bear an increase in garbage collection overhead, which has a direct effect on throughput. When you evaluate the G1 GC for latency, you set your desired (soft) real-time goal, and the G1 GC will try to meet it. As a side effect, throughput may suffer. See the section Pause Time Goal in Garbage-First Garbage Collector for additional information.

- Taming Mixed Garbage Collections

Experiment with the following options when you tune mixed garbage collections. See the "Important Defaults" above for information about these options:

> `-XX:InitiatingHeapOccupancyPercent`: Use to change the marking threshold.
>
> `-XX:G1MixedGCLiveThresholdPercent` and `-XX:G1HeapWastePercent`: Use to change the mixed garbage collection decisions.
>
> `-XX:G1MixedGCCountTarget` and `-XX:G1OldCSetRegionThresholdPercent`: Use to adjust the CSet for old regions.

当要求为 low latency 的时候，尝试仅赋值 `-XX:MaxGCPauseMillis=3` 和 `-XX:GCPauseIntervalMillis=1000`，保证 `Xms=Xmx`，观察效果。

## JVM 问题排查整理

### CPU利用率高

```bash
# 1. 输入命令后按 P 按照进程CPU使用率排序，找到进程对应的 pid，如 666
top -c
# 2. 继续按 P 找到占用CPU最多的线程 pid 如 10804
top -Hp 666
# 3. 将线程 pid 转为 16 进制数字
printf "%x\n" 10804
# 4. 查看 jvm 堆栈，找到对应问题线程，及其正在执行的代码
jstack 666 | grep '0x2a34' -C5 --color
```

### G1 full gc 问题

#### GC concurrent-mark-abort

mix gc 之前老年代就被填满，可以尝试增加堆大小，调整周期，修改线程数 `-XX:ConcGCThreads`

#### G1 日志 Overflow and Exhausted Log Messages

启用 G1 后 gc 日志中出现  `to-space exhausted` 或者 `to-space overflow` 问题，即晋升失败。可能是由于：G1 完成标记后开始混合式垃圾回收清理 s1，但在 s1 清理出足够空间前内存就被耗尽；大对象分配失败；

- Increase `-XX:G1ReservePercent` option (and the total heap accordingly) to increase the amount of reserve memory for"to-space".
- Start the marking cycle earlier by reducing the value of -XX:InitiatingHeapOccupancyPercent.
- Increase `-XX:ConcGCThreads` option to increase the number of parallel marking threads.

### OOM 问题

- 大量创建对象

```java
// 使用如下命令启动：
// java -verbose:gc -Xmx10M -Xms10M -XX:+PrintGCDetails -XX:+HeapDumpOnOutOfMemoryError -XX:SurvivorRatio=8 OOMTest
public class OOMTest {
    public static void main(String[] args) {
        List<Object> list = new ArrayList<>();
        for(;;) {
            list.add(new Object());
        }
    }
}
```

- 动态代理创建对象
- 方法区溢出
  - 程序使用了 CGLib 字节码增强和动态语言
  - 大量 JSP 或动态产生 JSP 文件的应用（JSP 第一次运行时需要编译为 Java 类）
  - 基于 OSGi 的应用（即使是同一个类文件，被不同的加载器加载也会视为不同的类）

```java
/**
 * VM Args：-XX：PermSize=10M-XX：MaxPermSize=10M
 *
 * @author zzm
 */
public class JavaMethodAreaOOM {
    public static void main(String[] args) {
        while (true) {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(OOMObject.class);
            enhancer.setUseCache(false);
            enhancer.setCallback(new MethodInterceptor() {
                public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy)
                    throws Throwable {
                    return proxy.invokeSuper(obj, args);
                }
            });
            enhancer.create();
        }
    }

    static class OOMObject {
    }
}
```

- direct memory 问题
  - 使用了 `native` 方法导致的内存溢出
  - metaspace 不断申请内存，导致**被 OS kill**

比如调用了 `nio` 包内的类，此时的内存 dump 很小，且没有明显异常。

- linux swap 和 GC 同时发生

SWAP 和 GC 同时发生会导致 GC 时间很长，JVM 严重卡顿，极端的情况下会导致服务崩溃。原因如下：

> JVM 进行 GC 时要对已用堆内存进行遍历；若此时堆的一部分被交换到 SWAP 中，遍历这部分数据的时候就需要将其交换回内存；但由于内存空间不足，就需要把内存中堆的另外一部分换到 SWAP 中去；极端情况下可能会把整个堆分区轮流往 SWAP 写一遍。
>
> Linux 对 SWAP 的回收是滞后的，我们就会看到大量 SWAP 占用。

此类问题可尝试用减小堆大小或者增加物理内存等方式解决；部署 Java 服务的 Linux 要避免使用 SWAP；

## Large Page

Also known as huge pages, large pages are memory pages that are significantly larger than the standard memory page size (which varies depending on the processor and operating system). Large pages optimize processor Translation-Lookaside Buffers.

A Translation-Lookaside Buffer (TLB) is a page translation cache that holds the most-recently used virtual-to-physical address translations. TLB is a scarce system resource. A TLB miss can be costly as the processor must then read from the hierarchical page table, which may require multiple memory accesses. By using a larger memory page size, a single TLB entry can represent a larger memory range. There will be less pressure on TLB, and memory-intensive applications may have better performance.

However, large pages page memory can negatively affect system performance. For example, when a large mount of memory is pinned by an application, it may create a shortage of regular memory and cause excessive paging in other applications and slow down the entire system. Also, a system that has been up for a long time could produce excessive fragmentation, which could make it impossible to reserve enough large page memory. When this happens, either the OS or JVM reverts to using regular pages.

## 常用 jdk 自带工具

下面简单列一下常见的分析工具。

|命令|用途|简单用法|
|:--|:---|:---|
| jps | 列出 jvm 进程| `jps -lvm` |
| jstack | 线程分析 | `jstack -F -l <pid> > x.tdump` |
| jmap | 内存分析 | `jmap -dump:format=b,file=x.hprof <pid> -F` |
| jinfo | 查看 jvm 信息 | `jinfo -sysprops <pid>` |
| jstat | jvm 统计信息监控 | `jstat -gc <pid> 3s 6` |

- 常用 jdk 图形化工具
  - jvisualvm 官方主推工具
  - jconsole 状态监控
  - jmc 状态监控

### jstat

> 使用格式： `jstat -<option> [-t] [-h<lines>] <vmid> [<interval> [<count>]]`

详情参看 [https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html "Oracle文档链接")

其中 option 分类如下：

|option|display|
|:--|:--|
|class| class loader|
|compiler|  Java HotSpot VM Just-in-Time compiler|
|gc| garbage collected heap|
|gccapacity|  generations and their corresponding spaces|
|gccause|  (same as -gcutil), with the cause of the last and current (when applicable) garbage collection events|
|gcnew|  new generation|
|gcnewcapacity| new generations and its corresponding spaces|
|gcold| old generation and metaspace statistics|
|gcoldcapacity| sizes of the old generation|
|gcmetacapacity| sizes of the metaspace|
|gcutil| summary about garbage collection statistics|
|printcompilation| Java HotSpot VM compilation method statistics|

下面整理常用的 option 及结果对照。

- `-gcutil` 输出结果含义：

|field|desc|
|:--|:--|
|S0  | Survivor space 0 utilization as a percentage of the space's current capacity |
|S1  | Survivor space 1 utilization as a percentage of the space's current capacity |
|E   | Eden space utilization as a percentage of the space's current capacity |
|O   | Old space utilization as a percentage of the space's current capacity |
|M   | Metaspace utilization as a percentage of the space's current capacity |
|CCS | Compressed class space utilization as a percentage |
|YGC | Number of young generation GC events |
|YGCT| Young generation garbage collection time |
|FGC | Number of full GC events |
|FGCT| Full garbage collection time |
|GCT | Total garbage collection time |

- `-gccause` 的输出是在 `-gcutil` 基础上增加两列：

|field|desc|
|:--|:--|
|LGCC| Cause of last garbage collection|
|GCC | Cause of current garbage collection|

- `-gc` 输出结果含义：

|field|desc|
|:--|:--|
| S0C  | Current survivor space 0 capacity (kB) |
| S1C  | Current survivor space 1 capacity (kB) |
| S0U  | Survivor space 0 utilization (kB) |
| S1U  | Survivor space 1 utilization (kB) |
| EC   | Current eden space capacity (kB) |
| EU   | Eden space utilization (kB) |
| OC   | Current old space capacity (kB) |
| OU   | Old space utilization (kB) |
| MC   | Metaspace capacity (kB) |
| MU   | Metacspace utilization (kB) |
| CCSC | Compressed class space capacity (kB) |
| CCSU | Compressed class space used (kB) |
| YGC  | Number of young generation garbage collection events |
| YGCT | Young generation garbage collection time |
| FGC  | Number of full GC events |
| FGCT | Full garbage collection time |
| GCT  | Total garbage collection time |

### jinfo

> 使用格式 jinfo [ option ] pid

详情参看 [https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jinfo.html](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jinfo.html)

```bash
jinfo <pid>                   # Prints both command-line flags and system property name-value pairs.
jinfo <pid> -sysprops         # Prints Java system properties as name-value pairs.
jinfo <pid> -flag name        # Prints the name and value of the specified command-line flag.
jinfo <pid> -flag [+|-]name   # enables or disables the specified Boolean command-line flag.
jinfo <pid> -flag name=value  # Sets the specified command-line flag to the specified value.
```

### jmap

详情参看 [https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jmap.html](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jmap.html)

```bash
# Prints a heap summary of the garbage collection used, the head configuration, and generation-wise heap usage.
# In addition, the number and size of interned Strings are printed.
jmap -heap <pid>
jmap -F -dump:[live,] format=b,file=filename <pid>
```

### jstack

详情参看 [https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstack.html](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstack.html)

```bash
jstack -F -l <pid> > t.log
```
