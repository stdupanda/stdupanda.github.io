---
layout: post
title: 整理 Java Agent 探针技术
categories: Java
description: 整理 Java Agent 探针技术
keywords: java, javaagent, agent, 探针
---

官方文档入口： [Collections Framework Overview](https://docs.oracle.com/javase/8/docs/api/java/lang/instrument/package-summary.html)

## 前言

在 JDK1.5 中引入了 `java.lang.instrument` 工具包以实现动态修改 Java 字节码，其机制是字节码被 JVM 执行之前获取这些字节码信息并通过字节码转换器对这些字节码进行修改。这类似是一种虚拟机级别的 AOP 实现。

`java.lang.instrument` 是基于 JVMTI 机制实现的。

JVMTI（Java Virtual Machine Tool Interface）是一套由 Java 虚拟机提供的了一套代理程序机制，可以支持第三方工具程序以代理的方式连接和访问 JVM。JVMTI 的功能非常丰富，包括虚拟机中线程、内存/堆/栈，类/方法/变量，事件/定时器处理等等。使用 JVMTI 一个基本的方式就是设置回调函数，在某些事件发生的时候触发并作出相应的动作，这些事件包括虚拟机初始化、开始运行、结束，类的加载，方法出入，线程始末等等。

## 加载方式

分为静态和动态两中加载方式。

### 静态加载

流程如下：

#### 编写类

自定义类中必须包括 `premain` 方法，且满足如下两种格式：

```java
public static void premain(String agentArgs, Instrumentation inst)
public static void premain(String agentArgs)
```

JVM 优先加载第一个方法，加载成功则忽略第二个方法；若没有第一种方法则加载第二个方法。

`agentArgs` 是一个字符串；`inst` 是 `java.lang.instrument.Instrumentation` 的实例，由 JVM 自动传入。接口中集中了几乎所有的功能方法，例如类定义的转换和操作等等。

#### 编写配置

jar 包内需要定义 `MANIFEST.MF` 文件，格式如下：

```text
Premain-Class: 包含 premain 方法的类（类的全路径名）
Agent-Class: 包含 agentmain 方法的类（类的全路径名）
Boot-Class-Path: 设置引导类加载器搜索的路径列表。查找类的特定于平台的机制失败后，引导类加载器会搜索这些路径。按列出的顺序搜索路径。列表中的路径由一个或多个空格分开。路径使用分层 URI 的路径组件语法。如果该路径以斜杠字符（“/”）开头，则为绝对路径，否则为相对路径。相对路径根据代理 JAR 文件的绝对路径解析。忽略格式不正确的路径和不存在的路径。如果代理是在 VM 启动之后某一时刻启动的，则忽略不表示 JAR 文件的路径。（可选）
Can-Redefine-Classes: true表示能重定义此代理所需的类，默认值为 false（可选）
Can-Retransform-Classes: true 表示能重转换此代理所需的类，默认值为 false （可选）
Can-Set-Native-Method-Prefix: true表示能设置此代理所需的本机方法前缀，默认值为 false（可选）

```

最后必须要有一个空行。

#### 运行

通过命令行方式启动：

```bash
java -javaagent:agent1.jar -javaagent:agent2.jar -jar app.jar
```

可以添加多个 `-javaagent` 并按定义的顺序生效。

### 动态加载

更多请查看官方文档 [Module jdk.attach](https://docs.oracle.com/en/java/javase/11/docs/api/jdk.attach/module-summary.html)

```java
VirtualMachine jvm = VirtualMachine.attach(jvmPid);
jvm.loadAgent(agentFile.getAbsolutePath());
jvm.detach();
```
