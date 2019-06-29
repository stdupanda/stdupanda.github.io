---
layout: post
title: java 并发基础 02 锁
categories: Java
description: java 并发基础 02 锁
keywords: java, 并发, concurrent, lock, 锁
---

整理 java 体系下的锁。

## 锁状态的分类

> 在Java SE 1.6 中，锁一共有 4 种状态，级别从低到高依次是：无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态，这几个状态会随着竞争情况逐渐升级，但不能降级。
>
> Java 对象头包括两个部分，一部分是存储对象自身的运行时数据，官方称为 Mark Word；另一部分是类型指针，即是对象指向它的类的元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。
>
> 其中 Mark Word 存储对象的 HashCode、分代年龄和**锁标记位**。

### 偏向锁

> 当一个线程获取锁时会在对象头和栈帧中的锁记录里存储锁偏向的线程 ID，以后该线程在进入和退出同步块时不需要进行 CAS 操作来加锁和解锁，只需简单地测试一下对象头的 Mark Word 里是否存储着指向当前线程的偏向锁。如果测试成功，表示线程已经获得了锁。如果测试失败，则需要再测试一下 Mark Word 中偏向锁的标识是否设置成 1（表示当前是偏向锁）：如果没有设置，则使用 CAS 竞争锁；如果设置了，则尝试使用 CAS 将对象头的偏向锁指向当前线程。
>
> **当其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁**。
>
> 偏向锁可以提高**有同步但无竞争**的程序性能。它同样是一个带有效益权衡（TradeOff）性质的优化，也就是说，它并不一定总是对程序运行有利，如果程序中**大多数的锁总是被多个不同的线程访问**，那偏向模式就是多余的。在具体问题具体分析的前提下，有时候使用参数 `-XX:-UseBiasedLocking` 来禁止偏向锁优化反而可以提升性能。

### 轻量级锁

> 轻量级锁目的是**在没有多线程竞争的前提下，减少传统的重量级锁使用操作系统互斥量产生的性能消耗**。
>
> 虚拟机尝试使用 CAS 进行加锁操作，若成功则说明此对象处于轻量级锁定状态；失败则继续检查，若当前线程是否已拥有对象锁，尝试使用 CAS 更新锁状态，进入自旋模式。
>
> 解锁时尝试使用 CAS 进行解锁，若成功说明没有竞争；若失败**则膨胀成重量级锁**。
>
> 轻量级锁能提升程序同步性能的依据是“对于绝大部分的锁，在整个同步周期内都是不存在竞争的”，这是一个经验数据。如果没有竞争，轻量级锁使用 CAS 操作避免了使用互斥量的开销，但如果存在锁竞争，除了互斥量的开销外，还额外发生了 CAS 操作，因此在有竞争的情况下，轻量级锁会比传统的重量级锁更慢。

### 重量级锁

> 重量级锁依赖于操作系统的互斥量（mutex）实现， 该操作会导致进程从用户态与内核态之间的切换， 开销较大。

## 锁的优化技术

虚拟机层面的优化只是尽最大努力的尝试，在编码时保证业务代码的稳健性才是最可行的办法。

### 自旋锁与自适应自旋

> - 多核 cpu 并行处理多个竞争锁的线程时，可以让后面请求锁的线程“稍等一下”，但不放弃处理器的执行时间，看看持有锁的线程是否很快就会释放锁。为了让线程等待，我们只需让线程执行一个忙循环（自旋），这项技术就是所谓的自旋锁。
>
> - 自旋等待本身虽然避免了线程切换的开销，但它是要占用处理器时间的，因此要求锁被占用的时间很短，否则自旋的线程只会白白消耗处理器资源，会带来性能上的浪费。因此，自旋等待的时间必须要有一定的限度，如果自旋超过了限定的次数仍然没有成功获得锁，就应当使用传统的方式去挂起线程了。自旋次数的默认值是 10 次，可以使用参数 `-XX:PreBlockSpin` 来更改。
>
> - JDK  1.6 中引入了**自适应的自旋锁**。自适应意味着自旋的时间不再固定了，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。如果在同一个锁对象上，自旋等待刚刚成功获得过锁，并且持有锁的线程正在运行中，那么虚拟机就会认为这次自旋也很有可能再次成功，进而它将**允许自旋等待持续相对更长的时间**，比如 100 个循环；如果对于某个锁，自旋很少成功获得过，那在以后要获取这个锁时将可能**省略掉自旋过程**（直接使用重量级锁），以避免浪费处理器资源。有了自适应自旋，随着程序运行和性能监控信息的不断完善，虚拟机对程序锁的状况预测就会越来越准确，虚拟机就会变得越来越“聪明”了。

### 锁消除

如果在编译过程中虚拟机认为同步代码内的资源不存在共享数据竞争，则会取消掉锁。(即：在一段代码中，堆上的所有数据都不会逃逸出去而被其他线程访问到)

### 锁粗化

> 如果一系列的连续操作都对同一个对象反复加锁和解锁，甚至加锁操作是出现在循环体中的，那即使没有线程竞争，频繁地进行互斥同步操作也会导致不必要的性能损耗。
>
> 如果虚拟机探测到有这样一串零碎的操作都对同一个对象加锁，将会**把加锁同步的范围扩展（粗化）到整个操作序列的外部**，这样**只需要加锁一次**就可以了。

## jdk 层的锁实现

在 Lock 接口出现之前，Java 程序是靠 synchronized 关键字实现锁功能的，而 Java SE 5 之后，并发包中新增了 Lock 接口（以及相关实现类）用来实现锁功能，它提供了与 synchronized 关键字类似的同步功能，只是在使用时需要显式地获取和释放锁。虽然它缺少了 synchronized 隐式获取释放锁的便捷性，但是却拥有了锁获取与释放的可操作性、可中断的获取锁以及超时获取锁等多种 synchronized 关键字所不具备的同步特性。

使用 synchronized 关键字将会隐式地获取锁，但是它将锁的获取和释放固化了，也就是先获取再释放。这种方式简化了同步的管理，可是扩展性没有显式的锁获取和释放来的好。例如，针对一个场景，先获得锁 A，然后再获取锁 B，当锁 B 获得后，释放锁 A 同时获取锁 C，当锁 C 获得后，再释放 B 同时获取锁 D，以此类推。这种场景下，synchronized 关键字就不那么容易实现了，而使用 Lock 却容易许多。

### Lock 接口

基本使用方式如下：

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
|:---|:---|
| `lock()` |获取锁，直至获取成功后才从此方法返回|
| `lockInterruptibly()` |获取锁后可以被中断，会抛出异常|
| `tryLock()` |尝试阻塞地获取锁，调用此方法后立即返回，返回是否获取到了锁|
| `newConditon()` |获取等待通知组件，该组件和当前的锁绑定，当前线程只有获取了锁，才能调用该组件的 `wait()` 方法，而调用后，当前线程将释放锁|

#### 队列同步器 AQS

队列同步器 `AbstractQueuedSynchronizer`（以下简称同步器），是用来构建锁或者其他同步组件的基础框架，它使用了一个 int 成员变量表示同步状态，通过内置的 FIFO 队列（CLH lock queue）来完成资源获取线程的排队工作，并发包的作者 Doug Lea 期望它能够成为实现大部分同步需求的基础。

同步器的主要使用方式是继承，**子类通过继承同步器并实现它的抽象方法来管理同步状态**，在抽象方法的现过程中免不了要对同步状态进行更改，这时就需要使用同步器提供的 3 个方法（`getState()`、`setState(int newState)` 和 `compareAndSetState(intexpect, int update)`）来进行操作，因为它们能够保证状态的改变是安全的。

AQS 的子类推荐被定义为自定义同步组件的**静态内部类**，同步器自身没有实现任何同步接口，它仅是定义了若干**同步状态获取和释放**的方法来供自定义同步组件使用，同步器既可以支持**独占式**地获取同步状态，也可以支持**共享式**地获取同步状态，这样就可方便实现不同类型的同步组件（如ReentrantLock、ReentrantReadWriteLock 和 CountDownLatch 等）。

同步器是实现锁（也可以是任意同步组件）的关键，在锁实现中聚合同步器，利用同步器实现锁的语义。可以这样理解二者之间的关系：

- 锁是面向使用者的，它定义了使用者与锁交互的接口（比如可以允许两个线程行访问），隐藏了实现细节
- 同步器面向的是锁的实现者，它简化了锁的实现方式，屏蔽了同步状态管理、线程的排队、等待与唤醒等底层操作。

锁和同步器隔离了使用者和实现者所需关注的领域。

同步器的设计是基于**模板方法模式**的，也就是说，使用者需要继承同步器并重写指定的方法，随后将同步器组合在自定义同步组件的实现中，并调用同步器提供的模板方法，而这些模板方法将会调用使用者重写的方法。

重写同步器指定的方法时，需要使用同步器提供的如下 3 个方法来访问或修改同步状态。

|方法|含义|
|:---|:---|
|`getState()`|获取当前同步状态|
|`setState(int newState)`|设置当前同步状态|
|`compareAndSetState(int expect,int update)`|使用CAS设置当前状态，该方法能够保证状态设置的原子性|

同步器可重写的方法包括：

- `tryAqurire(int arg)`
- `tryRelease(int arg)`
- `tryAqurireShared(int arg)`
- `tryReleaseShared(int arg)`
- `isHeldExclusively()`

实现自定义同步组件时，将要调用 AQS 提供的几个模板方法：

- `acquire(int arg)`
- `acquireInterruptibly(int arg)`
- `tryAcquireNanos(int arg, long nanos)`
- `acquireShared(int arg)`
- `acquireSharedInterruptibly(int arg)`
- `tryAcquireSharedNanos(int arg, long nanos)`
- `release(int arg)`
- `releaseShared(int arg)`
- `getQueuedThreads()`

上述几个模板方法分 3 类：**独占式获取与释放**同步状态、**共享式获取与释放**同步状态和**查询同步队列中的等待线程情况**。自定义同步组件将使用同步器提供的模板方法来实现自己的同步语义。

```java
// 类似 AQS 的 lock 工具
class FIFOMutex {
  private final AtomicBoolean locked = new AtomicBoolean(false);
  private final Queue<Thread> waiters = new ConcurrentLinkedQueue<Thread>();

  public void lock() {
    boolean wasInterrupted = false;
    Thread current = Thread.currentThread();
    waiters.add(current);

    // Block while not first in queue or cannot acquire lock
    while (waiters.peek() != current || !locked.compareAndSet(false, true)) {
      LockSupport.park(this);
      if (Thread.interrupted()) // ignore interrupts while waiting
      wasInterrupted = true;
    }

    waiters.remove();
    if (wasInterrupted)          // reassert interrupt status on exit
      current.interrupt();
  }

  public void unlock() {
    locked.set(false);
    LockSupport.unpark(waiters.peek());
  }
}
```

#### 重入锁

重入锁 ReentrantLock，顾名思义，就是支持重进入的锁，它表示该锁能够支持一个线程对资源的重复加锁。除此之外，该锁的还支持获取锁时的公平和非公性选择。synchronized 关键字**隐式的支持重入**，比如一个 synchronized 修饰的递归方法，在方法执行时，执行线程在获取了锁之后仍能连续次地获得该锁。ReentrantLock 虽然没能像 synchronized 关键字一样支持隐式的重进入，但是在调用 `lock()` 方法时，已经获取到锁的线程能够再次调用 `lock()` 方法获取锁而不被阻塞。

> 这里提到一个锁获取的公平性问题，如果在绝对时间上，先对进行获取的请求一定先被满足，那么这个锁是公平的，反之，是不公平的。公平的获取锁，也就是等待时间最长的线程最优先获取锁，也可以说锁获取是顺序。ReentrantLock 提供了一个构造函数，能够控制锁是否是公平的。事实上，公平的锁机制往往没有非公平的效率高，但是，并不是任何场景都是以 TPS 作为唯的指标，公平锁能够减少“饥饿”发生的概率，等待越久的请求越是能够得到优先满足。

#### 读写锁

ReadWriteLock 接口仅定义了获取读锁和写锁的两个方法，即 `readLock()`、`writeLock()`，而其实现—ReentrantReadWriteLock，除了接口方法之外，还提了一些便于外界**监控其内部工作状态**的方法，这些方法以及描述如下表所示。

|方法名|描述|
|---|---|
| `getReadLockCount()`|返回当前读锁被获取的次数|
| `getWriteLockCount()`|返回当前线程获取读锁的次数|
| `isWriteLocked()`|判断写锁是否被获取|
| `getWriteHoldCount()`|返回当前写锁被获取的次数|

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

读写锁同样依赖自定义同步器来实现同步功能，而读写状态就是其同步器的同步状态。回想 ReentrantLock 中自定义同步器的实现，同步状态表示锁被一个线重复获取的次数，而读写锁的自定义同步器需要在同步状态（一个整型变量）上维护多个读线程和一个写线程的状态，使得该状态的设计成为读写锁实现的关键。

> 如果在一个整型变量上维护多种状态，就一定需要“**按位切割使用**”这个变量，读写锁将变量切分成了两个部分，高 16 位表示读，低 16 位表示写。读写锁是如何迅速确定读和写各自的状态呢？答案是通过位运算。假设当前同步状态值为 S，写状态等于 S&0x0000FFFF（将高 16 位全部抹去），读状态等于 S>>>16（无符号 0 右移 16 位）。当写状态增加 1 时，等于 S+1，当读状态增加 1 时，等于 S+(1<<16)，也就是 S+0x00010000。
>
> 根据状态的划分能得出一个推论：S 不等于 0 时，当写状态（S&0x0000FFFF）等于 0 时，则读状态（S>>>16）大于 0，即读锁已被获取。

写锁是一个支持重进入的**排它锁**。如果当前线程已经获取了写锁，则增加写状态。如果当前线程在获取写锁时，读锁已经被获取（读状态不为0）或者该线程是已经获取写锁的线程，则当前线程进入等待状态。

读锁是一个支持重进入的**共享锁**，它能够被多个线程同时获取，在没有其他写线程访问（或者写状态为0）时，读锁总会被成功地获取，而所做的也只是（线安全的）增加读状态。如果当前线程已经获取了读锁，则增加读状态。如果当前线程在获取读锁时，写锁已被其他线程获取，则进入等待状态。获取读锁的实从 Java 5 到 Java 6 变得复杂许多，主要原因是新增了一些功能，例如 `getReadHoldCount()` 方法，作用是返回当前线程获取读锁的次数。读状态是所有线程获取读锁次数的总和，而每个线程各自获取读锁的次数只能选择保存在 ThreadLocal 中，由线程自身维护，这使获取读锁的实现变得复杂。

- 锁降级

锁降级指的是**写锁降级成为读锁**。如果当前线程拥有写锁，然后将其释放，最后再获取读锁，这种分段完成的过程不能称之为锁降级。锁降级是指**把持住（已拥有的）写锁，再获取到读锁，随后释放（先前拥有的）写锁**的过程。

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
> RentrantReadWriteLock 不支持锁升级（把持读锁、获取写锁，最后释放读锁的过程）。目的也是保证数据可见性，如果读锁已被多个线程获取，其中任意线成功获取了写锁并更新了数据，则其更新对其他获取到读锁的线程是不可见的。

### LockSupport

`LockSupport` 定义了一组的公共静态方法，这些方法提供了最基本的线程阻塞和唤醒功能，而 `LockSupport` 也成为构建同步组件的基础工具。`LockSupport` 定义一组以 park 开头的方法用来阻塞当前线程，以及 `unpark(Thread thread)` 方法来唤醒一个被阻塞的线程。Park 有停车的意思，假设线程为车辆，那么 park 方法代表着停车，而 unpark 方法则是指车辆启动离开，这些方法以及述如下所示。

|方法|说明|
|---|---|
|park()|阻塞当前线程，被中断或调用unpark()时会返回|
|parkNano(long)|超时阻塞返回|
|parkUntil(long)|超时阻塞|
|unPark(Thread t)|唤醒处于阻塞状态的线程t|

> 在 Java 6 中，LockSupport 增加了 `park(Object blocker)`、`parkNanos(Object blocker,long nanos)` 和 `parkUntil(Object blocker,longdeadline)` 3个方法，用于实现阻塞当前线程的功能，其中参数 `blocker` 是用来标识当前线程在等待的对象（以下称为阻塞对象），该对象主要用**问题排查和系统监控**。*建议使用新增的这些方法以便定位详细问题*。
>
> 在 Java 5 之前，当线程阻塞（使用 synchronized 关键字）在一个对象上时，通过线程 dump 能够查看到该线程的阻塞对象，方便问题定位；而 Java 5 推出的 Lock 等并发工具时却遗漏了这一点，致使在线程 dump 时无法提供阻塞对象的信息。**因此，在 Java 6 中， LockSupport 新增了上述 3 个含有阻塞对象的 park 方法，用以替代原有的 park 方法**。

### Lock 与 synchronized 区别

总结下 `Lock` 的优点：

|特性|描述|
|:---|:---|
|尝试非阻塞地获取锁|尝试获取锁，若此时锁没被其他线程获取到则成功获取并持有锁|
|能被中断地获取锁|与 `synchronized` 不同，获取到锁的线程能响应中断，即获取到锁的线程被中断时异常将会被抛出，同时锁会被释放|
|超时获取锁|若超时仍未获取到锁则返回|

### Condition 接口

Condition 实例本质上与 lock 对象是绑定的，使用 `lock.newCondition()` 即可获取 lock 的 condition 实例。

如下是一个支持存取的有界缓冲区实例。当缓冲区为空时 take 方法将会阻塞，直到缓冲区内有数据；当缓冲区满时 put 方法将会阻塞，直到缓冲区内有空闲空间。程序将会把 put 线程们和 take 线程们放在分开的 wait-sets 以实现单次只通知一个线程。可以使用两个 condition 实例完成上述要求。

> 注： The `java.util.concurrent.ArrayBlockingQueue` class provides this functionality, so there is no reason to implement this sample usage class.

```java
// jdk 文档内示例
class BoundedBuffer {
  final Lock lock = new ReentrantLock();
  final Condition notFull = lock.newCondition();
  final Condition notEmpty = lock.newCondition();

  final Object[] items = new Object[100];
  int putptr,
  takeptr,
  count;

  public void put(Object x) throws InterruptedException {
    lock.lock();
    try {
      while (count == items.length) notFull.await();
      items[putptr] = x;
      if (++putptr == items.length) putptr = 0; ++count;
      notEmpty.signal();
    } finally {
      lock.unlock();
    }
  }

  public Object take() throws InterruptedException {
    lock.lock();
    try {
      while (count == 0) notEmpty.await();
      Object x = items[takeptr];
      if (++takeptr == items.length) takeptr = 0; --count;
      notFull.signal();
      return x;
    } finally {
      lock.unlock();
    }
  }
}
```

Condition 定义了**等待/通知**两种类型的方法，当前线程调用这些方法时，需要提前获取到 Condition 对象关联的锁。Condition  对象是由 Lock 对象（调用 Lock 对象的 `newCondition()` 方法）创建出来的，换句话说，Condition 是依赖 Lock 对象的。

Condition 的使用方式比较简单，需要注意**在调用方法前获取锁**，使用方式如下所示。

```java
//...
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();
public void conditionWait() throws InterruptedException {
  lock.lock();
  try {
    condition.await();
  } finally {
    lock.unlock();
  }
}
public void conditionSignal() throws InterruptedException {
  lock.lock();
  try {
    condition.signal();
  } finally {
    lock.unlock();
  }
}
```

> 如代码所示，一般都会将 Condition 对象作为成员变量。
>
> - 当调用 `await()` 方法后，当前线程会释放锁并在此等待，而其他线程调用 Condition 对象的 `signal()` 方法，通知当前程后，当前线程才从 `await()` 方法返回，并且在返回前已经获取了锁。
>
> - 获取一个 Condition 必须通过 Lock 的 `newCondition()` 方法。
>
> ConditionObject 是同步器 AbstractQueuedSynchronizer 的内部类，因为 Condition 的操作需要获取相关联的锁，所以作为同步器的内部类也较为合理。每个 Condition 对象都包含着一个队列（以下称为等待队列），该队列是 Condition 对象实现等待/通知功能的关键。
>
> 下面将分析 Condition 的实现，主要包括：等待队列、等待和通知。
>
- 等待队列是一个 FIFO 的队列，在队列中的每个节点都包含了一个线程引用，该线程就是在 Condition 对象上等待的线程，如果一个线程调用了 `Condition.await()`方法，那么该线程将会释放锁、构造成节点加入等待队列并进入等待状态。事实上，节点的定义复用了同步器中节点的定义，也就是说，同步队列和等待队中节点类型都是同步器的静态内部类 AbstractQueuedSynchronizer.Node。

- 一个 Condition 包含一个等待队列，Condition 拥有首节点（firstWaiter）和尾节点（lastWaiter）。当前线程调用 `Condition.await()` 方法，将会以当前线程构造节点，并将节点从尾部加入等待队列。

- 调用 Condition 的 `await()` 方法（或者以 await 开头的方法），会使当前线程进入等待队列并释放锁，同时线程状态变为等待状态。当从 `await()` 方法返回时，当前线程一定获取了 Condition 相关联的锁。如果从队列（同步队列和等待队列）的角度看 `await()` 方法，当调用 `await()` 方法时，相当于同步队列的首节点（获取锁的节点）移动到 Condition 的等待队列中。

```java
public final void await() throws InterruptedException {
  if (Thread.interrupted())
    throw new InterruptedException();
  // 当前线程加入等待队列
  Node node = addConditionWaiter();
  // 释放同步状态，也就是释放锁
  int savedState = fullyRelease(node);
  int interruptMode = 0;
  while (!isOnSyncQueue(node)) {
    LockSupport.park(this);
    if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
      break;
  }
  if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
    interruptMode = REINTERRUPT;
  if (node.nextWaiter != null)
    unlinkCancelledWaiters();
  if (interruptMode != 0)
    reportInterruptAfterWait(interruptMode);
}
```

> 调用 Condition的 `signal()` 方法，将会唤醒在等待队列中等待时间最长的节点（首节点），在唤醒节点之前，会将节点移到同步队列中。调用该方法的前置条件是当前线程必须获取了锁，可以看到 `signal()` 方法进行了 `isHeldExclusively()` 检查，也就是当前线程必须是获取了锁的线程。接着获取等待队列的首节点，将其移动到同步队列并使用 LockSupport 唤醒节点中的线程。
>
> 通过调用同步器的 `enq(Node node)` 方法，等待队列中的头节点线程安全地移动到同步队列。当节点移动到同步队列后，当前线程再使用 LockSupport 唤醒该节点的线程。被唤醒后的线程，从 `await()` 方法中的while循环中退出（`isOnSyncQueue(Node node)` 方法返回 true，节点已经在同步队列中），进而调用同步器的 `acquireQueued()` 方法加入到获取同步状态的竞争中。成功获取同步状态（或者说锁）之后被唤醒的线程将从先前调用的 `await()` 方法返回，此时该线程已经成功地获取了锁。
>
> Condition 的 `signalAll()` 方法，相当于对等待队列中的每个节点均执行一 `signal()` 方法，效果就是将等待队列中所有节点全部移动到同步队列中，并唤醒每个节点的线程。

---

 (゜-゜)つロ *参考并致谢《Java并发编程的艺术》《深入理解Java虚拟机》*
