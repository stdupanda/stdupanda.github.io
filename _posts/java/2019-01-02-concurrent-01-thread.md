---
layout: post
title: java并发基础01-线程
categories: Java
description: java并发基础01-线程
keywords: java, 并发, concurrent, lock, 线程, thread
---

整理 java 并发编程之线程相关功能点。

## 线程简介

### 概念

> 现代操作系统在运行一个程序时，会为其创建一个进程。现代操作系统调度的最小单元是线程，也叫轻量级进程（Light Weight Process），在一个进程里可以创建多个线程，这些线程都拥有各自的计数器、堆栈和局部变量等属性，并且能够访问共享的内存变量。处理器在这些线程上高速切换，让使用者感觉到这些线程在同时执行。
>
> 一个 Java 程序由 main 线程负责执行 `main()` 方法开始执行。

如下代码可以打印一个普通的 java 进程包括哪些线程。

```java
public class MultiThread {
  public static void main(String[] args) {
    // 获取Java线程管理MXBean
    ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
    // 不需要获取同步的monitor和synchronizer信息，仅获取线程和线程堆栈信息
    ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
    // 遍历线程信息，仅打印线程ID和线程名称信息
    for (ThreadInfo threadInfo : threadInfos) {
      System.out.println("[" + threadInfo.getThreadId() + "] " + threadInfo.getThreadName());
    }
  }
}
```

### 线程的实现

- 使用内核线程实现

内核线程（Kernel-Level Thread, KLT）即直接由操作系统内核支持的线程，由内核来完成线程切换，内核通过操纵调度器对线程进行调度，并负责将线程的任务映射到各个处理器上。

程序一般不会直接去使用内核线程，而是去使用内核线程的一种高级接口——轻量级进程（Light Weight Process, LWP），也就是通常意义上所讲的线程，每个轻量级进程都由一个内核线程支持，即**一对一的线程模型**。

由于内核线程的支持，每个轻量级进程都成为一个独立的调度单元，即使有一个轻量级进程在系统调用中阻塞了，也不会影响整个进程继续工作，但是轻量级进程具有它的局限性：首先，由于是基于内核线程实现的，所以各种线程操作，如创建、析构及同步，都需要进行系统调用。而系统调用的代价相对较高，需要在用户态和内核态中来回切换。其次，每个轻量级进程都需要有一个内核线程的支持，因此轻量级进程要消耗一定的内核资源（如内核线程的栈空间），也就是说一个系统支持轻量级进程的数量是有限的。

- 使用用户线程实现

用户线程的建立、同步、销毁和调度完全在用户态中完成，不需要内核的帮助。如果程序实现得当，这种线程不需要切换到内核态，因此操作可以是非常快速且低消耗的，也可以支持规模更大的线程数量，部分高性能数据库中的多线程就是由用户线程实现的。这种进程与用户线程之间 1：N 的关系称为一对多的线程模型，

使用用户线程的优势在于不需要系统内核支援，劣势也在于没有系统内核的支援，所有的线程操作都需要用户程序自己处理。线程的创建、切换和调度都是需要考虑的问题，而且由于操作系统只把处理器资源分配到进程，那诸如“阻塞如何处理”、“多处理器系统中如何将线程映射到其他处理器上”这类问题解决起来将会异常困难，甚至不可能完成。

早期的 java 曾经采用过此模式。

- 使用用户线程+轻量级进程混合实现

在这种混合实现下，既存在用户线程，也存在轻量级进程。用户线程还是完全建立在用户空间中，因此用户线程的创建、切换、析构等操作依然廉价，并且可以支持大规模的用户线程并发。而操作系统提供支持的轻量级进程则作为用户线程和内核线程之间的桥梁，这样可以使用内核提供的线程调度功能及处理器映射，并且用户线程的系统调用要通过轻量级线程来完成，大大降低了整个进程被完全阻塞的风险。在这种混合模式中，用户线程与轻量级进程的数量比是不定的，即为 N：M 的关系，这种就是多对多的线程模型。

许多 UNIX 系列的操作系统，如 Solaris、HP-UX 等都提供了 N：M 的线程模型实现。

### 上下文切换损耗

> CPU通过给每个线程分配CPU时间片来实现线程切换机制，也就是通过时间片分配算法过来循环执行任务，当前任务执行一个时间片后会切换到下一个任务，在切换前会保存上一个任务的状态，以便下次切换回这个任务时，可以再加载这个任务的状态。所以任务从保存到再加载的过程就是一次上下文切换。

在 Linux 上可以通过 `vmstat` 查看上下文切换次数。

```shell
vmstat 1
# 结果集中的 CS (context switch) 即为上下文切换次数
```

### 如何避免上下文切换

- 无锁并发编程

多线程竞争锁时会引起上下文切换，所以多线程处理数据时，可以用一些办法来避免使用锁，如将数据的 ID 按照 Hash 算法取模分段，不同的线程处理不同段的数据。

- CAS 算法

Java 的 Atomic 包使用 CAS 算法来更新数据，而不需要加锁。

- 使用最少线程

避免创建不需要的线程，比如任务很少，但是创建了很多线程来处理，这样会造成大量线程都处于等待状态。

- 使用协程

在单线程里实现多任务的调度，并在单线程里维持多个任务间的切换。

### 重要方法

- wait

> Causes the current thread to wait until either another thread invokes the notify() method or the notifyAll() method for this object, or a specified amount of time has elapsed.
>
> **The current thread must own this object's monitor.**
>
> This method causes the current thread (call it T) to place itself in the **wait set** for this object and then to **relinquish any and all synchronization claims on this object**. Thread T becomes disabled for thread scheduling purposes and lies dormant until one of four things happens:
>
> 1. Some other thread invokes the notify method for this object and thread T happens to be arbitrarily chosen as the thread to be awakened.
>
> 2. Some other thread invokes the notifyAll method for this object.
>
> 3. Some other thread interrupts thread T.
>
> 4. The specified amount of real time has elapsed, more or less. If timeout is zero, however, then real time is not taken into consideration and the thread simply waits until notified.
>
> The thread T is then removed from the **wait set** for this object and re-enabled for thread scheduling. It then competes in the usual manner with other threads for the right to synchronize on the object; once it has gained control of the object, all its synchronization claims on the object are restored to the status quo ante - that is, to the situation as of the time that the wait method was invoked. Thread T then returns from the invocation of the wait method. Thus, on return from the wait method, the synchronization state of the object and of thread T is exactly as it was when the wait method was invoked.
>
> A thread can also wake up without being notified, interrupted, or timing out, a so-called spurious wakeup. While this will rarely occur in practice, applications must guard against it by testing for the condition that should have caused the thread to be awakened, and continuing to wait if the condition is not satisfied. In other words, **waits should always occur in loops**, like this one:
>
> ```java
> synchronized (obj) {
>     while (<condition does not hold>)
>         obj.wait(timeout);
>     ... // Perform action appropriate to condition
> }
> ```
>
> (For more information on this topic, see Section 3.2.3 in Doug Lea's "Concurrent Programming in Java (Second Edition)" (Addison-Wesley, 2000), or Item 50 in Joshua Bloch's "Effective Java Programming Language Guide" (Addison-Wesley, 2001).
>
> If the current thread is interrupted by any thread before or while it is waiting, then an InterruptedException is thrown. This exception is not thrown until the lock status of this object has been restored as described above.
>
> Note that the wait method, as it places the current thread into the wait set for this object, unlocks only this object; any other objects on which the current thread may be synchronized remain locked while the thread waits.
>
> This method should only be called by a thread that is the owner of this object's monitor. See the **notify** method for a description of the ways in which a thread can become the owner of a monitor.

- notify

> Wakes up a single thread that is waiting on this object's monitor. If any threads are waiting on this object, one of them is chosen to be awakened. The choice is **arbitrary肆意** and occurs at the discretion自行决定 of the implementation. A thread waits on an object's monitor by calling one of the wait methods.
>
> The awakened thread will not be able to proceed **until the current thread relinquishes放弃 the lock on this object**. The awakened thread will compete in the usual manner with any other threads that might be actively competing to synchronize on this object; for example, the awakened thread enjoys no reliable privilege or disadvantage in being the next thread to lock this object.
>
> This method should only be called by a thread that is **the owner of this object's monitor**. A thread becomes the owner of the object's monitor in one of three ways:
>
> - By executing a synchronized instance method of that object.
> - By executing the body of a synchronized statement that synchronizes on the object.
> - For objects of type Class, by executing a synchronized static method of that class.
>
> Only one thread at a time can own an object's monitor.

- notifyAll

> Wakes up all threads that are waiting on this object's monitor. A thread waits on an object's monitor by calling one of the wait methods.
>
> The awakened threads will not be able to proceed **until the current thread relinquishes the lock on this object**. The awakened threads will compete in the usual manner with any other threads that might be actively competing to synchronize on this object; for example, the awakened threads enjoy no reliable privilege or disadvantage in being the next thread to lock this object.
>
> This method should only be called by a thread that is **the owner of this object's monitor**. See the notify method for a description of the ways in which a thread can become the owner of a monitor.

- sleep

> Causes the currently executing thread to sleep (temporarily cease execution) for the specified number of milliseconds, subject to the precision and accuracy of system timers and schedulers. The thread **does not lose ownership of any monitors**

- `wait` & `sleep`

| wait|sleep|
|:--|:--|
| 释放锁 | 持有锁 |
| 需先获取锁 |—|
| 需在loop中判断条件是否满足 |—|

### 线程状态

|状态|说明|
|:---|:---|
|NEW|初始状态|
|RUNNABLE|运行状态(就绪+运行中)|
|BLOCKED|阻塞于锁|
|WAITING|等待状态，表示当前线程需要等待其他线程进行通知或中断|
|TIMED_WAITING|超时等待，在指定时间内返回|
|TERMINATED|已终止|

参考 jdk1.8 源码，详解如下：

#### `BLOCKED`

> Thread state for a thread blocked waiting for a monitor lock.
>
> A thread in the blocked state is waiting for a monitor lock to enter a synchronized block/method or reenter a synchronized block/method after calling `Object.wait`.

#### `WAITING`

> Thread state for a waiting thread. A thread is in the waiting state due to calling one of the following methods:
>
> - `Object.wait` with no timeout
> - `Thread.join` with no timeout
> - `LockSupport.park`
>
> A thread in the waiting state is waiting for another thread to perform a particular action.
>
> For example, a thread that has called `Object.wait()` on an object is waiting for another thread to call `Object.notify()` or `Object.notifyAll()` on that object. A thread that has called  `Thread.join()` is waiting for a specified thread to terminate.

#### `TIMED_WAITING`

> Thread state for a waiting thread with a specified waiting time.
>
> A thread is in the timed waiting state due to calling one of the following methods with a specified positive waiting time:
>
> - `Thread.sleep`
> - `Object.wait` with timeout
> - `Thread.join` with timeout
> - `LockSupport.parkNanos`
> - `LockSupport.parkUntil`

#### 状态转换总结

图示如下:

![image](/images/posts/thread_state.png)

> 特别说明，关于 `BLOCKED` 状态，阻塞状态是线程阻塞在进入 `synchronized` 关键字修饰的方法或代码块（获取锁）时的状态，但是阻塞在 `java.concurrent` 包中 `Lock` 接口的线程状态却是等待状态，因为 `java.concurrent`包中 `Lock` 接口对于阻塞的实现均使用了 `LockSupport` 类中的相关方法。

## 基本操作

### 中断

线程的一个标志位，表示一个运行中的线程是否**被其他线程进行了中断操作**。举例对于运行的线程 t，任意线程通过调用 t.interrupt() 来对 t 进行中断操作。

通过 `t.isInterrupted()` 来判断 t 是否被中断，也可以调用 `t.isInterrupted(true)` 和 `Thread.interrupted()` 对 t 的中断标志位进行复位。

jdk 在很多类中，都是**先清除中断标志位，然后抛出 `InterruptedException`**；此时再调用 `isInterrupted()` 方法就会返回 false。

### 安全操作线程

禁止使用已过时的 `suspend()/resume()/stop()` 方法来操作线程；暂停和恢复操作应改为使用**等待/通知**机制进行实现。

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

### 线程间通信

#### volatile 和 synchronized

使用 volatile 修饰变量、使用 synchronized 修饰方法都是通过 **共享内存** 作为中介实现的线程间通信方式；

#### `synchronized` 实现原理

> 对于 synchronized 关键字，无论是修饰方法还是代码段，最终在 class 文件内都是操作的一个 monitor 对象监视器，对应的 jvm 指令即 `moniterenter`、`monitorexit`；
>
> 任意线程对 Object（Object由synchronized保护）的访问，首先要获得 Object 的监视器。如果获取失败，线程进入同步队列，线程状态变为 BLOCKED。当访问 Object 的前驱（获得了锁的线程）释放了锁，则该释放操作唤醒阻塞在同步队列中的线程，使其重新尝试对监视器的获取。

![image](/images/posts/object_monitor.png)

#### 等待通知机制

##### 相关基本方法

实现“等待通知”机制的相关方法(以下假设当前对象为 o，执行方法的当前线程为 t)

|方法名|描述|
|:---|:---|
|`wait()`|当t获取o的对象锁之后执行此方法会让t进入等待状态（同时释放o的对象锁）直至其他线程进行通知唤醒|
|`wait(long)`|等待一段时间后没通知就返回|
|`wait(long,int)`|对超时时间更细粒度的控制|
|`notify()`|t获取了o的对象锁后通知一个在o上等待的线程使其从obj.wait()方法返回|
|`notifyAll()`|通知所有等待在o的对象锁上的线程|

以上仅为简述，**必须详细查看 jdk 文档的注释及实例！**

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

![image](/images/posts/thread_wait_notify.png)

> 在图中，WaitThread 首先获取了对象的锁，然后调用对象的 `wait()` 方法，从而放弃了锁并进入了对象的等待队列 WaitQueue 中，进入等待状态。由于 WaitThread 释放了对象的锁，NotifyThread 随后获取了对象的锁，并调用对象的 `notify()` 方法，将 WaitThread 从 WaitQueue 移到 SynchronizedQueue 中，此时 WaitThread 的状态变为阻塞状态。NotifyThread 释放了锁之后，WaitThread 再次获取到锁并从 wait() 方法返回继续执行。

##### 流程总结

等待通知，即消费者生产者，遵循特定的原则。

###### 消费者流程

- 获取对象的锁
- 若条件不满足则调用对象的`wait()`方法，被通知后仍要检查条件
- 条件满足则执行对应逻辑

```java
synchronized(obj){
  while(条件判断){
    obj.wait();
  }
  // 对应的处理逻辑
}
```

###### 生产者流程

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

#### thread.join()

若线程 A 执行了 `t.join()`，则 A 会等待 t 线程执行完后才会从 `t.join` 出返回。类似具备超时返回的方法还包括 `join(long)`,`join(long,int)` 等。

> 需要知道的是， `join()` 的具体实现还是调用的 `wait()` 方法， `join()` 方法是被 `synchronized` 修饰的，也就意味着调用 `t.join()` 时已经获取到了 `t` 的对象锁。

#### `Threadlocal`

jdk文档中对 `ThreadLocal` 的描述：

> Each thread holds an implicit reference to its copy of a thread-local variable as long as the thread is alive and the ThreadLocal instance is accessible; after a thread goes away, all of its copies of thread-local instances are subject to garbage collection (unless other references to these copies exist).

每个线程内部有一个 `ThreadLocalMap` 类型的成员变量，用于保存线程私有的 `ThreadLocal<T>` 对象，T 类实例对象值存储在 `Entry[]` 数组中，key 为 `ThreadLocal<T>` 对象，通过其成员变量与 `0x61c88647` 累加运算得出 `Entry[]` 数组的 index。需要说明的是，`Entry` 类是继承自 `WeakReference<ThreadLocal<?>>`，目的是为了优化系统 GC。也就是说 key 会被 gc，但值可能不会被 gc。

> 和 `HashMap` 的最大的不同在于，ThreadLocalMap 结构非常简单，没有 next 引用，也就是说 ThreadLocalMap 中解决 Hash 冲突的方式并非链表的方式，而是采用**线性探测**的方式，所谓线性探测，就是根据初始 key 的 hashcode 值确定元素在 table 数组中的位置，如果发现这个位置上已经有其他 key 值的元素被占用，则利用**固定的算法**寻找一定步长的下个位置，依次判断，直至找到能够存放的位置。
>
> ThreadLocalMap 解决 Hash 冲突的方式就是简单的步长加 1 或减 1，寻找下一个相邻的位置。
>
> ```java
> /**
>  * Increment i modulo len.
>  */
> private static int nextIndex(int i, int len) {
>     return ((i + 1 < len) ? i + 1 : 0);
> }
>
> /**
>  * Decrement i modulo len.
>  */
> private static int prevIndex(int i, int len) {
>     return ((i - 1 >= 0) ? i - 1 : len - 1);
> }
> ```
>
> 显然 ThreadLocalMap 采用线性探测的方式解决 Hash 冲突的效率很低，如果**有大量不同的 ThreadLocal 对象放入 map 中时更容易发送冲突，或者发生二次冲突**。
>
> 所以这里引出的良好建议是：每个线程只存一个变量，这样的话所有的线程存放到 map 中的 Key 都是相同的 ThreadLocal。如果一个线程要保存多个变量，就需要创建多个 ThreadLocal，多个 ThreadLocal 放入 Map 中就容易造成 Hash 冲突。
>
> 由于 ThreadLocalMap 的 **key 是弱引用，而 value 是强引用**。这就导致了一个问题，ThreadLocal 在没有外部对象强引用时，发生 GC 时弱引用 Key 会被回收，而 Value 不会回收，如果创建 ThreadLocal 的线程一直持续运行，那么这个 Entry 对象中的 value 就有可能一直得不到回收，发生内存泄露。
>
> 既然 Key 是弱引用，那么我们要做的事，就是在调用 ThreadLocal 的 `get()`、`set()` 方法时完成后再调用 **`remove()`** 方法，将 Entry 节点和 Map 的引用关系移除，这样整个 Entry 对象在 GC Roots 分析后就变成不可达了，下次 GC 的时候就可以被回收。否则就有可能发生内存泄露。

#### `ThreadlocalRandom`

`ThreadlocalRandom` 避免了多线程并发时争夺导致性能下降；`Random` 内部是采用了 CAS 实现了并发安全。

用法为 `ThreadLocalRandom.current().nextX(...) (where X is Int, Long, etc).`

## 经典实例

### 等待超时

```java
// 对当前对象加锁
public synchronized Object get(long mills) throws InterruptedException {
  long future = System.currentTimeMillis() + mills;
  long remaining = mills;// 等待持续时间
  // 当超时大于0并且result返回值不满足要求
  while ((result == null) && remaining > 0) {
    wait(remaining);
    remaining = future - System.currentTimeMillis();
  }
  return result;
}
```

### 线程池

请参考开源 jdbc 连接池的实现代码分析。

---

小结： 本文主要整理了 java 线程基础功能点，是 java 并发的基础。

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

 (゜-゜)つロ *参考并致谢《Java并发编程的艺术》《深入理解Java虚拟机》*