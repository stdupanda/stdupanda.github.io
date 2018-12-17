---
layout: post
title: 详解 ClassLoader 类加载器
categories: Java
description: 详解 jvm 类加载器
keywords: Java, java, jdk, openjdk
---

整理总结 `jvm` 类加载器机制。

# 前言

对于 Class 对象大家应该不陌生， 然而当说到编译器处理 `.java` 文件后形成的 `.class` 字节码文件时， 按需加载策略是怎样的呢？ `Class` 对象是如何创建又存储在哪儿呢？

熟悉 jvm 加载处理 class 的流程有助于掌握整个 jvm 的运行机制， 因此写篇文件记录一下。

# 字节码文件的编译和加载

## 编译流程

整体流程就是：

`.java` -> `javac` -> `.class`

详细的编译过程如下：

`.java` -> 词法、语法、语义分析器 -> 抽象语法树 -> 字节码生成器 -> 代码生成器 -> 字节码

## 加载流程

`.class` -> ClassLoader -> 字节码校验器 -> 解释器 -> OS

# JVM 基础结构

如图：

![image](https://github.com/stdupanda/stdupanda.github.io/raw/master/images/posts/jvm.jpg)

# 类加载器 ClassLoader

ClassLoader 负责将字节码内容转换成内存形式的 Class 对象。当类加载器将 `.class` 文件装载完成后，jvm 内将会形成一个对应的元信息对象， 即 `Class<T>` 类。字节码可以来自于磁盘文件 *.class，也可以是 jar 包里的 *.class，也可以来自远程服务器提供的字节流，字节码的本质就是一个字节数组 `byte[]`，它有特定的复杂的内部格式，有很多复杂的加密技术就是在此基础上实现的。

每个 `Class<T>` 类内部都有一个对应的 `ClassLoader` 对象。源码如下：

```java
public final class Class<T> implements java.io.Serializable,
                              GenericDeclaration,
                              Type,
                              AnnotatedElement {
    //...

    // Initialized in JVM not by private constructor
    // This field is filtered from reflection access, i.e. getDeclaredField
    // will throw NoSuchFieldException
    private final ClassLoader classLoader;

    //...
}
```

## `ClassLoader` 类别

JVM 中内置了三个重要的 ClassLoader，分别是 Bootstrap ClassLoader、ExtClassLoader 和 AppClassLoader。

### Bootstrap ClassLoader

`Bootstrap ClassLoader` 是由底层代码实现并被嵌入到了 JVM 中，我们将它称之为「根加载器」，当 JVM 启动时，`Bootstrap ClassLoader` 也随着启动，负责加载 JVM 运行时核心类，这些类位于 `JAVA_HOME/lib/rt.jar` 文件中，我们常用内置库都在里面，比如 `java.util.*`、`java.io.*`、`java.nio.*`、`java.lang.*` 等等。可以用指令 `-Xbootclasspath` 指定自定义路径。

### `ExtClassLoader`

`ExtClassLoader` 位于 jdk 的 `rt.jar` 内 `sun.misc.Launcher$ExtClassLoader`，继承自 `URLClassLoader`, 负责加载 JVM 扩展类，比如 swing 系列、内置的 js 引擎、xml 解析器 等等，这些库名通常以 javax 开头，它们的 jar 包位于 `JAVA_HOME/lib/ext/*.jar` 中，有很多 jar 包。可以用指令 `-D java.ext.dirs` 自定义指定路径。

### `AppClassLoader`

`AppClassLoader` 位于 jdk 的 `rt.jar` 内 `sun.misc.Launcher$AppClassLoader`， 继承自 `URLClassLoader`, 是直接面向我们用户的类加载器，它会加载 Classpath 环境变量里定义的路径中的 jar 包和目录。我们自己编写的代码以及使用的第三方 jar 包通常都是由它来加载的。`AppClassLoader` 可以由 `ClassLoader` 类提供的静态方法 getSystemClassLoader() 得到，，当我们的 main 方法执行的时候，这第一个用户类的加载器就是 AppClassLoader。

### 介绍一下 `URLClassLoader`

jdk 内置了一个 `URLClassLoader`，用户只需要传递规范的网络路径给构造器，就可以使用 `URLClassLoader` 来加载对应类库了。`URLClassLoader` 不但可以加载本地路径的类库，还可以加载远程类库，取决于构造器中不同的地址形式。`ExtClassLoader` 和 `AppClassLoader` 都是 `URLClassLoader` 的子类，它们都是从本地文件系统里加载类库。

## `ClassLoader` 运行机制

### 校验加载顺序

![image](https://github.com/stdupanda/stdupanda.github.io/raw/master/images/posts/classloader_order.png)

### 双亲委派机制

JVM 在加载类时默认采用的是双亲委派机制。通俗的说，就是某个特定的类加载器在接到加载类请求时，首先将加载任务委托给父类加载器，依次递归，如果父类加载器可以完成，则返回，否则自己尝试加载。

双亲委派机制是在 ClassLoader 的 loadClass 方法中实现的，标准扩展类加载器和系统类加载器都遵循了双亲委派机制，因为他们都继承了 ClassLoader，而且没有重写 loadClass 方法。

请看 jdk1.8 `ClassLoader` 的如下代码

```java
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }

            if (c == null) {
                // If still not found, then invoke findClass in order
                // to find the class.
                long t1 = System.nanoTime();
                c = findClass(name);

                // this is the defining class loader; record the stats
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}
```

**`Bootstrap ClassLoader` 是在找不到 parent 的时候才会被委派，而 `AppClassLoader` 的 parent 会被指定为 ExtensionClassLoader，而 `ExtensionClassLoder` 的 parent 则为 null。**

> `ClassLoader` 中的构造函数中，parent 默认是通过 `getSystemClassLoader()` 方法获取到的系统构造类加载器。
>
> AppClassLoader 的构造函数中传入了 parent ClassLoader，这个构造函数是 `protect` 的，在 AppClassLoader 的 getAppClassLoader 静态方法里被调用，而这个静态方法需要一个 ClassLoader 作为 AppClassLoader 的 parent。再看 Launcher 的构造方法，这个参数是 Launcher.ExtClassLoader.getExtClassLoader() 生成的，也就是说，在构造 AppClassLoader 时传入的是 ExtClassLoader。
>
> `ExtClassLoader` 的构造函数很容易看，parent 直接传入的是 `null`。

## 用户自定义类加载器和线程上下文类加载器

此处不介绍用户自定义类加载器。

### 线程上下文类加载器

线程上下文类加载器，可通过方法 `getContextClassLoader()` 和 `setContextClassLoader()` 获取和设置。在没有指定线程上下文类加载器的情况下，线程将继承父线程的上下文类加载器。Java 应用运行的初始线程的上下文类加载器是应用类加载器，所以在不指定的情况下就默认是应用类加载器。

为什么要有线程类加载器这个概念呢？

假设当前只有双亲委派模型，那么并不能解决 Java 应用开发中遇到的全部的类加载问题。Java 提供了很多服务提供接口 SPI (Service Provider Interface)，允许第三方为这些接口提供实现。比如说 JDBC，JCE，JAXP，JNDI 等等。这些 SPI 接口由 Java 核心库提供，而这些 SPI 接口的实现代码很可能位于 java 应用所依赖的 jar 包，可通过类路径 classpath 找到。问题在于，SPI 的接口是 Java 核心库的一部分，是由启动类加载器实现的，而 SPI 的实现类一般是由系统类加载器加载的。启动类加载器无法加载 SPI 的实现，也没法交给它的子类系统类加载器来加载实现类。

使用线程上下文类加载器，可以在执行线程中抛弃双亲委派加载模式，转而采用线程上下文类加载器来加载需要的类，这样就可以显式地指定类加载器。大部分的 java application 如 jboss，tomcat 都是采用 `contextClassLoader` 来处理 web 服务。还有一些采用 hot swap 的框架，也是采用线程上下文类加载器。

对于运行在 Java EE容器中的 Web 应用来说，类加载器的实现方式与一般的 Java 应用有所不同。不同的 Web 容器的实现方式也会有所不同。以 Apache Tomcat 来说，每个 Web 应用都有一个对应的类加载器实例。该类加载器也使用代理模式，所不同的是**它首先尝试去加载某个类，如果找不到再代理给父类加载器**。这与一般类加载器的顺序是相反的。这是 Java Servlet 规范中的推荐做法，其目的是*使得 Web 应用自己的类的优先级高于 Web 容器提供的类*。这种代理模式的一个例外是：Java 核心库的类是不在查找范围之内的。这也是为了保证 Java 核心库的类型安全。

1. 每个 Web 应用自己的 Java 类文件和使用的库的 jar 包，分别放在 `WEB-INF/classes` 和 `WEB-INF/lib` 目录下面。

2. 多个应用共享的 Java 类文件和 jar 包，分别放在 Web 容器指定的由所有 Web 应用共享的目录下面。

3. 当出现找不到类的错误时，检查当前类的类加载器和当前线程的上下文类加载器是否正确。

下面举例分析一下 tomcat 内部加载不同 webapp 服务时涉及到的线程上下文类加载器问题。

在 `org.apache.catalina.startup.Bootstrap` 内有如下逻辑：

```java
/**
* Initialize daemon.
* @throws Exception Fatal initialization error
*/
public void init() throws Exception {

    initClassLoaders();

    Thread.currentThread().setContextClassLoader(catalinaLoader);

    SecurityClassLoad.securityClassLoad(catalinaLoader);

    // Load our startup class and call its process() method
    if (log.isDebugEnabled())
        log.debug("Loading startup class");
    Class<?> startupClass = catalinaLoader.loadClass("org.apache.catalina.startup.Catalina");
    Object startupInstance = startupClass.getConstructor().newInstance();

    // Set the shared extensions class loader
    if (log.isDebugEnabled())
        log.debug("Setting startup class properties");
    String methodName = "setParentClassLoader";
    Class<?> paramTypes[] = new Class[1];
    paramTypes[0] = Class.forName("java.lang.ClassLoader");
    Object paramValues[] = new Object[1];
    paramValues[0] = sharedLoader;
    Method method =
        startupInstance.getClass().getMethod(methodName, paramTypes);
    method.invoke(startupInstance, paramValues);

    catalinaDaemon = startupInstance;

}
private void initClassLoaders() {
    try {
        commonLoader = createClassLoader("common", null);
        if( commonLoader == null ) {
            // no config file, default to this loader - we might be in a 'single' env.
            commonLoader=this.getClass().getClassLoader();
        }
        catalinaLoader = createClassLoader("server", commonLoader);
        sharedLoader = createClassLoader("shared", commonLoader);
    } catch (Throwable t) {
        handleThrowable(t);
        log.error("Class loader creation threw exception", t);
        System.exit(1);
    }
}
```

启动时创建了不同类型的类加载目的如下：

1. 对于各个webapp中的 class 和 lib，需要相互隔离，不能出现一个应用中加载的类库会影响另一个应用的情况；可以联系到 tomcat 自身的 lib 路径作用。

2. 第二个原因则是与 jvm 一样的安全性问题。使用单独的类加载器去装载 tomcat 自身的类库，以免其他恶意或无意的破坏；

3. 热部署。

![image](https://github.com/stdupanda/stdupanda.github.io/raw/master/images/posts/classloader_tomcat.png)