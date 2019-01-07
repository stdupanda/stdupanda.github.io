---
layout: post
title: java 并发基础02锁
categories: Java
description: java 并发基础02锁
keywords: Java, java, jdk, openjdk, concurrent, lock
---

整理总结 java 线程基础功能点。

# 锁的分类及简介

## 介绍

>  锁是用来控制多个线程访问共享资源的方式，一般来说，一个锁能够防止多个线程同时访问共享资源（但是有些锁可以允许多个线程并发的访问共享资源，比如读写锁）。在Lock接口出现之前，Java程序是靠synchronized关键字实现锁功能的，而Java SE 5之后，并发包中新增了Lock接口（以及相关实现类）用来实现锁功能，它提供了与synchronized关键字类似的同步功能，只是在使用时需要显式地获取和释放锁。虽然它缺少了（通过synchronized块或者方法所提供的）隐式获取释放锁的便捷性，但是却拥有了锁获取与释放的可操作性、可中断的获取锁以及超时取锁等多种synchronized关键字所不具备的同步特性。使用synchronized关键字将会隐式地获取锁，但是它将锁的获取和释放固化了，也就是先获取再释放。然，这种方式简化了同步的管理，可是扩展性没有显示的锁获取和释放来的好。例如，针对一个场景，手把手进行锁获取和释放，先获得锁A，然后再获取锁，当锁B获得后，释放锁A同时获取锁C，当锁C获得后，再释放B同时获取锁D，以此类推。这种场景下，synchronized关键字就不那么容易实现了，而使用Lock却容易许多。

## 区别

总结下 `Lock` 的优点：

|特性|描述|
|:---|:---|
|尝试非阻塞地获取锁|尝试获取锁，若此时锁没被其他线程获取到则成功获取并持有锁|
|能被中断地获取锁|与 `Synchronized` 不同，获取到锁的线程能响应中断，即获取到锁的线程被中断时异常将会被抛出，同时锁会被释放|
|超时获取锁|若超时仍未获取到锁则返回|

# 详细介绍

## `Lock` 接口

### 基本使用

```java
// Lock 接口的基本使用方法
Lock l = ...;
l.lock();
try {
  // access the resource protected by this lock
} finally {
  l.unlock();
}
```

详细接口描述请看 jdk 文档。下面简单写几个：

|方法|描述|
|lock()|获取锁，直至获取成功后才从此方法返回|
|lockInterruptibly()|获取锁后可以被中断，会抛出异常|
|tryLock()|尝试阻塞地获取锁，调用此方法后立即返回，返回是否获取到了锁|
|newConditon()|获取等待通知组件，该组件和当前的锁绑定，当前线程只有获取了锁，才能调用该组件的`wait()`方法，而调用后，当前线程将释放锁|

### 队列同步器

> 队列同步器AbstractQueuedSynchronizer（以下简称同步器），是用来构建锁或者其他同步组件的基础框架，它使用了一个int成员变量表示同步状态，通过置的FIFO队列来完成资源获取线程的排队工作，并发包的作者（Doug Lea）期望它能够成为实现大部分同步需求的基础。同步器的主要使用方式是继承，子类通过继承同步器并实现它的抽象方法来管理同步状态，在抽象方法的现过程中免不了要对同步状态进行更改，这时就需要使用同步器提供的3个方法（getState()、setState(int newState)和compareAndSetState(intexpect,int update)）来进行操作，因为它们能够保证状态的改变是安全的。子类推荐被定义为自定义同步组件的静态内部类，同步器自身没有实现任何同步接口，它仅是定义了若干同步状态获取和释放的方法来供自定义同步组件使用，同步器既可以支持独占式地获取同步状态，也可以支持共享式地获取同步状态，这样就可方便实现不同类型的同步组件（ReentrantLock、ReentrantReadWriteLock和CountDownLatch等）。同步器是实现锁（也可以是任意同步组件）的关键，在锁实现中聚合同步器，利用同步器实现锁的语义。可以这样理解二者之间的关系：锁是面向使用者的，它定义了使用者与锁交互的接口（比如可以允许两个线程行访问），隐藏了实现细节；同步器面向的是锁的实现者，它简化了锁的实现方式，屏蔽了同步状态管理、线程的排队、等待与唤醒等底层操作。锁和同步器好地隔离了使用者和实现者所需关注的领域。

>  同步器的设计是基于模板方法模式的，也就是说，使用者需要继承同步器并重写指定的方法，随后将同步器组合在自定义同步组件的实现中，并调用同步器提供的模板方法，而这些模板方法将会调用使用者重写的方法。
>
> 重写同步器指定的方法时，需要使用同步器提供的如下3个方法来访问或修改同步状态。
>
> - ·getState()：获取当前同步状态。
> - ·setState(int newState)：设置当前同步状态。
> - ·compareAndSetState(int expect,int update)：使用CAS设置当前状态，该方法能够保证状态
设置的原子性。

同步器可重写的方法包括：

- `tryAqurire(int arg)`
- `tryRelease(int arg)`
- `tryAqurireShared(int arg)`
- `tryReleaseShared(int arg)`
- `isHeldExclusively()`

实现自定义同步组件时，将要调用 AQS 提供的几个模板方法：

- aquire(int arg)
- aquireInterruptibly(int arg)
- tryAquireNanos(int arg, long nanos)
- aquireShared(int arg)
- aquireSharedInterrupitbly(int arg)
- tryAquireSharedNanos(int arg, long nanos)
- release(int arg)
- releaseShared(int arg)
- getQueuedThreads()

> 上述几个模板方法分3类：独占式获取与释放同步状态、共享式获取与释放
同步状态和查询同步队列中的等待线程情况。自定义同步组件将使用同步器提供的模板方法
来实现自己的同步语义。
---

### 重入锁

> 重入锁ReentrantLock，顾名思义，就是支持重进入的锁，它表示该锁能够支持一个线程对资源的重复加锁。除此之外，该锁的还支持获取锁时的公平和非公性选择。回忆在同步器一节中的示例（Mutex），同时考虑如下场景：当一个线程调用Mutex的lock()方法获取锁之后，如果再次调用lock()方法，则该线程将被自己所阻塞，原因是Mutex在实现tryAcquire(int acquires)方法时没有考虑占有锁的线程再次获取锁的场景，而在调用tryAcquire(intacquires)方法时返回了false，导致该线程被阻塞。简单地说，Mutex是个不支持重进入的锁。而synchronized关键字隐式的支持重进入，比如一个synchronized修饰的递归方法，在方法执行时，执行线程在获取了锁之后仍能连续次地获得该锁，而不像Mutex由于获取了锁，而在下一次获取锁时出现阻塞自己的情况。ReentrantLock虽然没能像synchronized关键字一样支持隐式的重进入但是在调用lock()方法时，已经获取到锁的线程，能够再次调用lock()方法获取锁而不被阻塞。这里提到一个锁获取的公平性问题，如果在绝对时间上，先对进行获取的请求一定先被满足，那么这个锁是公平的，反之，是不公平的。公平的获取锁，也就是等待时间最长的线程最优先获取锁，也可以说锁获取是顺序。ReentrantLock提供了一个构造函数，能够控制锁是否是公平的。事实上，公平的锁机制往往没有非公平的效率高，但是，并不是任何场景都是以TPS作为唯的指标，公平锁能够减少“饥饿”发生的概率，等待越久的请求越是能够得到优先满足。

### 读写锁

> ReadWriteLock仅定义了获取读锁和写锁的两个方法，即readLock()方法和writeLock()方法，而其实现——ReentrantReadWriteLock，除了接口方法之外，还提了一些便于外界监控其内部工作状态的方法，这些方法以及描述如下表所示。

|方法名|描述|
|---|---|
|getReadLockCount()|返回当前读锁被获取的次数|
|getWriteLockCount()|返回当前线程获取读锁的次数|
|isWriteLocked()|判断写锁是否被获取|
|getWriteHoldCount()|返回当前写锁被获取的次数|

下面是一个读写锁的例子：

```java
public class Cache {
  static Map<String, Object> map = new HashMap<String, Object>();
  static ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
  static Lock r = rwl.readLock();
  static Lock w = rwl.writeLock();
  // 获取一个key对应的value
  public static final Object get(String key) {
    r.lock();
    try {
      return map.get(key);
    } finally {
      r.unlock();
    }
  }
  // 设置key对应的value，并返回旧的value
  public static final Object put(String key, Object value) {
    w.lock();
    try {
      return map.put(key, value);
    } finally {
      w.unlock();
    }
  }
  // 清空所有的内容
  public static final void clear() {
    w.lock();
    try {
      map.clear();
    } finally {
      w.unlock();
    }
  }
}
```

> 读写锁同样依赖自定义同步器来实现同步功能，而读写状态就是其同步器的同步状态。回想ReentrantLock中自定义同步器的实现，同步状态表示锁被一个线重复获取的次数，而读写锁的自定义同步器需要在同步状态（一个整型变量）上维护多个读线程和一个写线程的状态，使得该状态的设计成为读写锁实现的关。
>
>  如果在一个整型变量上维护多种状态，就一定需要“按位切割使用”这个变量，读写锁将变量切分成了两个部分，高16位表示读，低16位表示写。读写锁是如何迅速确定读和写各自的状态呢？答案是通过位运算。假设当前同步状态值为S，写状态等于S&0x0000FFFF（将高16位全部抹去），读状态等于S>>>16（无符号0右移16位）。当写状态增加1时，等于S+1，当读状态增加1时，等于S+(1<<16)，也就是S+0x00010000。
>
> 根据状态的划分能得出一个推论：S不等于0时，当写状态（S&0x0000FFFF）等于0时，则读
状态（S>>>16）大于0，即读锁已被获取。
>
> 写锁是一个支持重进入的排它锁。如果当前线程已经获取了写锁，则增加写状态。如果当前线程在获取写锁时，读锁已经被获取（读状态不为0）或者该线程是已经获取写锁的线程，则当前线程进入等待状态。
>
> 读锁是一个支持重进入的共享锁，它能够被多个线程同时获取，在没有其他写线程访问（或者写状态为0）时，读锁总会被成功地获取，而所做的也只是（线安全的）增加读状态。如果当前线程已经获取了读锁，则增加读状态。如果当前线程在获取读锁时，写锁已被其他线程获取，则进入等待状态。获取读锁的实从Java 5到Java 6变得复杂许多，主要原因是新增了一些功能，例如getReadHoldCount()方法，作用是返回当前线程获取读锁的次数。读状态是所有线程获取读锁次数的总和而每个线程各自获取读锁的次数只能选择保存在ThreadLocal中，由线程自身维护，这使获取读锁的实现变得复杂。
>
>  **锁降级** 指的是写锁降级成为读锁。如果当前线程拥有写锁，然后将其释放，最后再获取读锁，这种分段完成的过程不能称之为锁降级。锁降级是指把持住（当拥有的）写锁，再获取到读锁，随后释放（先前拥有的）写锁的过程。

```java
// 锁降级示例
public void processData() {
  readLock.lock();
  if (!update) {
    // 必须先释放读锁
    readLock.unlock();
    // 锁降级从写锁获取到开始
    writeLock.lock();
    try {
    if (!update) {
      // 准备数据的流程（略）
      update = true;
    }
    readLock.lock();
    } finally {
      writeLock.unlock();
    }
    // 锁降级完成，写锁降级为读锁
  }
  try {
    // 使用数据的流程（略）
  } finally {
    readLock.unlock();
  }
}
```

> 锁降级中读锁的获取是否必要呢？答案是必要的。主要是为了保证数据的可见性，如果当前线程不获取读锁而是直接释放写锁，假设此刻另一个线程（记作线T）获取了写锁并修改了数据，那么当前线程无法感知线程T的数据更新。如果当前线程获取读锁，即遵循锁降级的步骤，则线程T将会被阻塞，直到当前线程用数据并释放读锁之后，线程T才能获取写锁进行数据更新。
>
>  RentrantReadWriteLock不支持锁升级（把持读锁、获取写锁，最后释放读锁的过程）。目的也是保证数据可见性，如果读锁已被多个线程获取，其中任意线成功获取了写锁并更新了数据，则其更新对其他获取到读锁的线程是不可见的。

### LockSupport



小结： 本文主要整理了 java 锁的基础功能点，是 java 并发的基础。

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

 (゜-゜)つロ *参考并致谢《Java并发编程的艺术》*