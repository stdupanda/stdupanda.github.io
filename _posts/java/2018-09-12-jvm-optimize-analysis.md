---
layout: post
title: jvm 问题分析和优化
categories: Java
description: jvm问题分析和优化
keywords: Java, java, jdk, openjdk, JVM
---

整理常见 JVM 异常问题缘由、排查定位技巧、相关工具用法。

置顶区：

[1] Oracle 官方帮助文档入口：[Java Platform, Standard Edition Documentation](https://docs.oracle.com/en/java/javase/index.html)

## JVM 问题排查整理

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
  - 上面程序使用了的 CGLib 字节码增强和动态语言
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
                public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
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

JVM 进行 GC 时要对已用堆内存进行遍历；若此时堆的一部分被交换到 SWAP 中，遍历这部分数据的时候就需要将其交换回内存；

但由于内存空间不足，就需要把内存中堆的另外一部分换到 SWAP 中去；极端情况下可能会把整个堆分区轮流往 SWAP 写一遍。

Linux 对 SWAP 的回收是滞后的，我们就会看到大量 SWAP 占用。

此类问题可尝试用减小堆大小或者增加物理内存等方式解决；部署 Java 服务的 Linux 要避免SWAP的使用；

-XX:+DisableExplicitGC

## 常用 jdk 自带工具

下面简单列一下常见的分析工具。

|命令|用途|简单用法|
|:--|:---|:---|
| jps | 列出 jvm 进程| `jps -lvm` |
| jstack | 线程分析 | `jstack -F -l <pid> > x.jstack` |
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

`-gcutil` 输出结果含义：

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

`-gccause` 的输出是在 `-gcutil` 基础上增加两列：

|field|desc|
|:--|:--|
|LGCC| Cause of last garbage collection|
|GCC | Cause of current garbage collection|

`-gc` 输出结果含义：

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

```shell
jinfo <pid>                   # Prints both command-line flags and system property name-value pairs.
jinfo <pid> -sysprops         # Prints Java system properties as name-value pairs.
jinfo <pid> -flag name        # Prints the name and value of the specified command-line flag. 
jinfo <pid> -flag [+|-]name   # enables or disables the specified Boolean command-line flag.
jinfo <pid> -flag name=value  # Sets the specified command-line flag to the specified value.
```

### jmap

详情参看 [https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jmap.html](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jmap.html)

```shell
# Prints a heap summary of the garbage collection used, the head configuration, and generation-wise heap usage.
# In addition, the number and size of interned Strings are printed.
jmap -heap <pid> 
jmap -F -dump:[live,] format=b, file=filename <pid>
```

### jstack

详情参看 [https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstack.html](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstack.html)

```shell
jstack -F -l <pid> > t.log
```
