---
layout: post
title: java 并发基础 (゜-゜)つロ 01 线程
categories: Java
description: java 并发基础 (゜-゜)つロ 01 线程
keywords: Java, java, jdk, openjdk
---

整理总结 `jvm` 类加载器机制。

# 线程简介

## 简介

> 现代操作系统在运行一个程序时，会为其创建一个进程。例如，启动一个Java程序，操作系统就会创建一个Java进程。现代操作系统调度的最小单元是线程，也叫轻量级进程（Light Weight Process），在一个进程里可以创建多个线程，这些线程都拥有各自的计数器、堆栈和局部变量等属性，并且能够访问共享的内存变量。处理器在这些线程上高速切换，让使用者感觉
到这些线程在同时执行。
> 一个Java程序从main()方法开始执行，然后按照既定的代码逻辑执行，看似没有其他线程
参与，但实际上Java程序天生就是多线程程序，因为执行main()方法的是一个名称为main的线程。

如下代码可以打印一个普通的 java 进程包括哪些线程。

```java
public class MultiThread{
  public static void main(String[] args) {
    // 获取Java线程管理MXBean
    ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
    // 不需要获取同步的monitor和synchronizer信息，仅获取线程和线程堆栈信息
    ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
    // 遍历线程信息，仅打印线程ID和线程名称信息
    for (ThreadInfo threadInfo : threadInfos) {
      System.out.println("[" + threadInfo.getThreadId() + "] " + threadInfo.
      getThreadName());
    }
  }
}
```

## 线程状态

|状态|说明|
|:---|:---|
|NEW|初始状态|
|RUNNABLE|运行状态(就绪+运行中)|
|BLOCKED|阻塞于锁|
|WAITING|等待状态，表示当前线程需要等待其他线程进行通知或中断|
|TIMED_WAITING|超时等待，在指定时间内返回|
|TERMINATED|已终止|

参考 jdk1.8 doc，详解如下：

### `BLOCKED`

> Thread state for a thread blocked waiting for a monitor lock.
>
> A thread in the blocked state is waiting for a monitor lock to enter a synchronized block/method or reenter a synchronized block/method after calling `Object.wait`.

### `WAITING`

> Thread state for a waiting thread. A thread is in the waiting state due to calling one of the following methods:
>
> - `Object.wait` with no timeout
> - `Thread.join` with no timeout
> - `LockSupport.park`
>
> A thread in the waiting state is waiting for another thread to perform a particular action. For example, a thread that has called `Object.wait()` on an object is waiting for another thread to call `Object.notify()` or `Object.notifyAll()` on that object. A thread that has called  `Thread.join()` is waiting for a specified thread to terminate.

### `TIMED_WAITING`

> Thread state for a waiting thread with a specified waiting time.
>
> A thread is in the timed waiting state due to calling one of the following methods with a specified positive waiting time:
>
> - `Thread.sleep`
> - `Object.wait` with timeout
> - `Thread.join` with timeout
> - `LockSupport.parkNanos`
> - `LockSupport.parkUntil`

### 状态转换总结

图示如下:

![image](https://github.com/stdupanda/stdupanda.github.io/raw/master/images/posts/thread_state.png)

> 特别说明，关于 `BLOCKED` 状态，阻塞状态是线程阻塞在进入 `synchronized` 关键字修饰的方法或代码块（获取锁）时的状态，但是阻塞在 `java.concurrent` 包中 `Lock` 接口的线程状态却是等待状态，因为 `java.concurrent`包中Lock接口对于阻塞的实现均使用了 `LockSupport` 类中的相关方法。

# 基本操作

## 中断

线程的一个标志位，表示一个运行中的线程是否*被其他线程进行了中断操作*。举例对于运行的线程t，任意线程通过调用 t.interrupt() 来对 t 进行中断操作。

通过 `t.isInterrupted()` 来判断 t 是否被中断，也可以调用 `t.isInterrupted(true)` 和 `Thread.interrupted()` 对 t 的中断标志位进行复位。

jdk 在很多类中，都是先清除中断标志位，然后抛出 `InterruptedException`；此时再调用 `isInterrupted()` 方法就会返回 false。

## 安全操作线程

不允许使用已过时的 `suspend()/resume()/stop()` 方法来操作线程；暂停和恢复操作应改为使用*等待/通知*机制进行实现。

安全停止线程示例如下：

```java
private static class Runner implements Runnable {
    private long i;
    private volatile boolean on = true;

    @Override
    public void run() {
        while (on && !Thread.currentThread().isInterrupted()){
            i++;
        }
        System.out.println("Count i = " + i);
    }
    public void cancel() {
        on = false;
    }
}
```

上述例子中，进行线程中断或者调用 `cancel()` 方法均可以优雅结束线程。

## 线程间通信

### volatile 和 synchronized

使用 volatile 修饰变量、使用 synchronized 修饰方法都是通过*共享内存*作为中介实现的线程间通信方式；

> 对于 synchronized 关键字，无论是修饰方法还是代码段，最终在 class 文件内都是操作的一个 monitor 对象监视器，对应的jvm指令即 moniterenter、monitorexit；
>
> 任意线程对Object（Object由synchronized保护）的访问，首先要获得 Object 的监视器。如果获取失败，线程进入同步队列，线程状态变为 BLOCKED。当访问 Object 的前驱（获得了锁的线程）释放了锁，则该释放操作唤醒阻塞在同步队列中的线程，使其重新尝试对监视器的获取。

![image](https://github.com/stdupanda/stdupanda.github.io/raw/master/images/posts/object_monitor.png)

### 等待通知机制

#### 基本方法

实现“等待通知”机制的相关方法(以下假设当前对象为o，执行方法的当前线程为t)

|方法名|描述|
|:---|:---|
|`wait()`|当t获取o的对象锁之后执行此方法会让t进入等待状态（同时释放o的对象锁）直至其他线程进行通知唤醒|
|`wait(long)`|等待一段时间后没通知就返回|
|`wait(long,int)`|对超时时间更细粒度的控制|
|`notify()`|t获取了o的对象锁后通知一个在o上等待的线程使其从obj.wait()方法返回|
|`notifyAll()`|通知所有等待在obj上的线程|

以上仅为简述，**必须详细查看jdk文档的javadoc文档及实例！**

> 1）使用 `wait()`、`notify()` 和 `notifyAll()` 时需要先对调用对象加锁。
>
> 2）调用 `wait()` 方法后，线程状态由 `RUNNING` 变为 `WAITING`，并将当前线程放置到对象的等待队列。
>
> 3）`notify()` 或 `notifyAll()` 方法调用后，等待线程依旧不会从 `wait()` 返回，需要调用 `notify()` 或 `notifyAll()` 的线程释放锁之后，等待线程才有机会从 `wait()` 返回。
>
> 4）`notify()` 方法将等待队列中的一个等待线程从等待队列中移到同步队列中，而 `notifyAll()`  方法则是将等待队列中所有的线程全部移到同步队列，被移动的线程状态由 `WAITING` 变为 `BLOCKED`。
>
> 5）从 `wait()` 方法返回的前提是获得了调用对象的锁。
>
> 从上述细节中可以看到，等待/通知机制依托于同步机制，其目的就是确保等待线程从 `wait()` 方法返回时**能够感知到通知线程对变量做出的修改**。

![image](https://github.com/stdupanda/stdupanda.github.io/raw/master/images/posts/thread_wait_notify.png)

> 在图中，WaitThread 首先获取了对象的锁，然后调用对象的 `wait()` 方法，从而放弃了锁并进入了对象的等待队列 WaitQueue 中，进入等待状态。由于 WaitThread 释放了对象的锁，NotifyThread 随后获取了对象的锁，并调用对象的 `notify()` 方法，将 WaitThread 从 WaitQueue 移到 SynchronizedQueue 中，此时 WaitThread 的状态变为阻塞状态。NotifyThread 释放了锁之后，WaitThread 再次获取到锁并从 wait() 方法返回继续执行。

#### 流程总结

等待通知，即消费者生产者，遵循特定的原则。

##### 消费者流程

- 获取对象的锁
- 若条件不满足则调用对象的`wait()`方法，被通知后仍要检查条件
- 条件满足则执行对应逻辑

```java
synchronized(obj){
  while(条件不满足){
    obj.wait();
  }
  // 对应的处理逻辑
}
```

##### 生产者流程

- 获得对象的锁
- 改变条件
- 通知所有在对象上等待的线程

```java
synchronized(obj){
  // 改变条件
    obj.notifyAll();
  // 对应的处理逻辑
}
```

### thread.join()

若线程A执行了 `t.join()`，则A会等待 t 线程执行完后才会从 `t.join` 出返回。类似的方法还包括 `join(long)`,`join(long,int)` 具备超时返回。

需要知道的是， `join()` 实际实现还是调用的 `wait()` 方法。

### threadlocal

# 经典实例

## 等待超时

## 线程池

创建方式

---

 (゜-゜)つロ *参考并致谢《Java并发编程的艺术》*