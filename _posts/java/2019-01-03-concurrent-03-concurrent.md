---
layout: post
title: java 并发基础03并发类
categories: Java
description: java 并发基础03并发类
keywords: Java, java, jdk, openjdk, concurrent, lock
---

整理总结常用 java 并发类库。

# 并发容器和框架

## 并发 `map`

### `HashMap` 和 `HashSet` 的问题

> HashMap在并发执行put操作时会引起死循环，是因为多线程会导致HashMap的Entry链表形成环形数据结构，一旦形成环形数据结构，Entry的next节点永远为空，就会产生死循环获取Entry，举例如下：

```java
final HashMap<String, String> map = new HashMap<String, String>(2);
Thread t = new Thread(new Runnable() {

  @Override
  public void run() {
    for (int i = 0; i < 10000; i++) {
	  new Thread(new Runnable() {
          @Override
          public void run() {
            map.put(UUID.randomUUID().toString(), "");
          }
        }, "ftf" + i).start();
      }
  }
}, "ftf");
t.start();
t.join();
```

对应的 jdk1.7 的 `java.util.HashMap#put()` 源码：

```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {// 并发时会导致链表结构被破坏，造成死循环
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
```
> HashTable容器使用synchronized来保证线程安全，但在线程竞争激烈的情况下HashTable的效率非常低下。因为当一个线程访问HashTable的同步方法，其线程也访问HashTable的同步方法时，会进入阻塞或轮询状态。如线程1使用put进行元素添加，线程2不但不能使用put方法添加元素，也不能使用get方法获取元素，所以竞争越激烈效率越低。

> HashTable容器在竞争激烈的并发环境下表现出效率低下的原因是所有访问HashTable的线程都必须竞争同一把锁，假如容器里有多把锁，每一把锁用于锁器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效提高并发访问效率，这就是ConcurrentHashap所使用的锁分段技术。首先将数据分成一段一段地存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能其他线程访问。

### `ConcurrenHashMap`

> ConcurrentHashMap的锁分段技术可有效提升并发访问率。HashTable容器在竞争激烈的并发环境下表现出效率低下的原因是所有访问HashTable的线程都必竞争同一把锁，假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，而可以有效提高并发访问效率，这就是ConcurrentHashMap所使用的锁分段技术。首先将数据分成一段一段地存储，然后给每一段数据配一把锁，当一个线占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。

#### 结构

> ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。Segment是一种可重入锁（ReentrantLock），在ConcurrentHashMap里扮演锁的角色。HashEntry则用于存储键值对数据。一个ConcurrentHashMap里包含一个Segment数组。Segment的结构和HashMap类似，是一种数组和链表结构。一个Segmen里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素，每个Segment守护着一个HashEntry数组里的元素，当对HashEntry数组的数据进行修改，必须首先获得与它对应的Segment锁。

#### 初始化

> ConcurrentHashMap初始化方法是通过initialCapacity、loadFactor和concurrencyLevel等几个参数来初始化segment数组、段偏移量segmentShift、段掩segmentMask和每个segment里的HashEntry数组来实现的。


#### 锁分段定位

> 既然ConcurrentHashMap使用分段锁Segment来保护不同段的数据，那么在插入和获取元素的时候，必须先通过散列算法定位到Segment。可以看到ConcurrntHashMap会首先使用Wang/Jenkins hash的变种算法对元素的hashCode进行一次再散列。

> ```java
> private static int hash(int h) {
>   h += (h << 15) ^ 0xffffcd7d;
>   h ^= (h >>> 10);
>   h += (h << 3);
>   h ^= (h >>> 6);
>   h += (h << 2) + (h << 14);
>   return h ^ (h >>> 16);
> }
> ```

> 之所以进行再散列，目的是减少散列冲突，使元素能够均匀地分布在不同的Segment上，从而提高容器的存取效率。假如散列的质量差到极点，那么所有的素都在一个Segment中，不仅存取元素缓慢，分段锁也会失去意义。

## 并发 `queue`

> 在并发编程中，有时候需要使用线程安全的队列。如果要实现一个线程安全的队列有两种方式：一种是使用阻塞算法，另一种是使用非阻塞算法。使用阻算法的队列可以用一个锁（入队和出队用同一把锁）或两个锁（入队和出队用不同的锁）等方式来实现。非阻塞的实现方式则可以使用循环CAS的方式来实现。

### `ConcurrentLinkedQueue`

> ConcurrentLinkedQueue是一个基于链接节点的无界线程安全队列，它采用先进先出的规则对节点进行排序，当我们添加一个元素的时候，它会添加到队列尾部；当我们获取一个元素时，它会返回队列头部的元素。它采用了“wait-free”算法（即CAS算法）来实现，该算法在Michael&Scott算法上进行了一些修改。
>
> ConcurrentLinkedQueue由head节点和tail节点组成，每个节点（Node）由节点元素（item）和指向下一个节点（next）的引用组成，节点与节点之间就是通过这个next关联起来，从而组成一张链表结构的队列。默认情况下head节点存储的元素为空，tail节点等于head节点。

### 阻塞队列

> 阻塞队列（BlockingQueue）是一个支持两个附加操作的队列。这两个附加的操作支持阻塞的插入和移除方法。
>
> 1）支持阻塞的插入方法：意思是当队列满时，队列会阻塞插入元素的线程，直到队列不满。
>
> 2）支持阻塞的移除方法：意思是在队列为空时，获取元素的线程会等待队列变为非空。
>
> 阻塞队列常用于生产者和消费者的场景，生产者是向队列里添加元素的线程，消费者是从队列里取元素的线程。阻塞队列就是生产者用来存放元素、消费者用来获取元素的容器。

> JDK 7提供了7个阻塞队列，如下。
>
> - ArrayBlockingQueue：一个由数组结构组成的有界阻塞队列。
> - LinkedBlockingQueue：一个由链表结构组成的有界阻塞队列。
> - PriorityBlockingQueue：一个支持优先级排序的无界阻塞队列。
> - DelayQueue：一个使用优先级队列实现的无界阻塞队列。
> - SynchronousQueue：一个不存储元素的阻塞队列。
> - LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。
> - LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

> 1.ArrayBlockingQueue
> 
> ArrayBlockingQueue是一个用数组实现的有界阻塞队列。此队列按照先进先出（FIFO）的原则对元素进行排序。默认情况下不保证线程公平的访问队列，所谓公平访问队列是指阻塞的线程，可以按照阻塞的先后顺序访问队列，即先阻塞线程先访问队列。非公平性是对先等待的线程是非公平的，当队列可用时，阻塞的线程都可以争夺访问队列的资格，有可能先阻塞的线程最后才访问队列。为了保证公平性，通常会降低吞吐量。我们可以使用以下代码创建一个公平的阻塞队列。

> ```java
> ArrayBlockingQueue fairQueue = new ArrayBlockingQueue(1000,true);
> ```
>
> 访问者的公平性是使用可重入锁实现的，代码如下。
>
> ```java
> public ArrayBlockingQueue(int capacity, boolean fair) {
>   if (capacity <= 0)
>     throw new IllegalArgumentException();
>   this.items = new Object[capacity];
>   lock = new ReentrantLock(fair);
>   notEmpty = lock.newCondition();
>   notFull = lock.newCondition();
> }
> ```
> 
> 2.LinkedBlockingQueue
> 
> LinkedBlockingQueue是一个用链表实现的有界阻塞队列。此队列的默认和最大长度为Integer.MAX_VALUE。此队列按照先进先出的原则对元素进行排序。
> 
> 3.PriorityBlockingQueue
> 
> PriorityBlockingQueue是一个支持优先级的无界阻塞队列。默认情况下元素采取自然顺序升序排列。也可以自定义类实现compareTo()方法来指定元素排序规则，或者初始化PriorityBlockingQueue时，指定构造参数Comparator来对元素进行排序。需要注意的是不能保证同优先级元素的顺序。
> 
> 4.DelayQueue
> 
> DelayQueue是一个支持延时获取元素的无界阻塞队列。队列使用PriorityQueue来实现。队列中的元素必须实现Delayed接口，在创建元素时可以指定多久才能从队列中获取当前元素。只有在延迟期满时才能从队列中提取元素。DelayQueue非常有用，可以将DelayQueue运用在以下应用场景。
> 
> ·缓存系统的设计：可以用DelayQueue保存缓存元素的有效期，使用一个线程循环查询
> DelayQueue，一旦能从DelayQueue中获取元素时，表示缓存有效期到了。
> 
> ·定时任务调度：使用DelayQueue保存当天将会执行的任务和执行时间，一旦从
> DelayQueue中获取到任务就开始执行，比如TimerQueue就是使用DelayQueue实现的。
> 
> 5.SynchronousQueue
> 
> SynchronousQueue是一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素。它支持公平访问队列。默认情况下线程采用非公平性策略访问队列。使用以下构造方法可以创建公平性访问的SynchronousQueue，如果设置为true，则等待的线程会采用先进先出的顺序访问队列。
> 
> ```java
> public SynchronousQueue(boolean fair) {
>   transferer = fair new TransferQueue() : new TransferStack();
> }
> ```
> 
> SynchronousQueue可以看成是一个传球手，负责把生产者线程处理的数据直接传递给消费者线程。队列本身并不存储任何元素，非常适合传递性场景。SynchronousQueue的吞吐量高于LinkedBlockingQueue和ArrayBlockingQueue。
> 
> 6.LinkedTransferQueue
> 
> LinkedTransferQueue是一个由链表结构组成的无界阻塞TransferQueue队列。相对于其他阻塞队列，LinkedTransferQueue多了tryTransfer和transfer方法。
> 
> 7.LinkedBlockingDeque
> 
> LinkedBlockingDeque是一个由链表结构组成的双向阻塞队列。所谓双向队列指的是可以从队列的两端插入和移出元素。双向队列因为多了一个操作队列的入口，在多线程同时入队时，也就减少了一半的竞争。相比其他的阻塞队列，LinkedBlockingDeque多了addFirst、addLast、offerFirst、offerLast、peekFirst和peekLast等方法，以First单词结尾的方法，表示插入、获取（peek）或移除双端队列的第一个元素。以Last单词结尾的方法，表示插入、获取或移除双端队列的最后一个元素。另外，插入方法add等同于addLast，移除方法remove等效于removeFirst。但是take方法却等同于takeFirst，不知道是不是JDK的bug，使用时还是用带有First和Last后缀的方法更清楚。在初始化LinkedBlockingDeque时可以设置容量防止其过度膨胀。另外，双向阻塞队列可以运用在“工作窃取”模式中。

## `Fork/Join` 框架

> Fork/Join框架是Java 7提供的一个用于并行执行任务的框架，是一个把大任务分割成若干个小任务，最终汇总每个小任务结果后得到大任务结果的框架。

### 工作窃取算法

> 工作窃取（work-stealing）算法是指某个线程从其他队列里窃取任务来执行。那么，为什么需要使用工作窃取算法呢？假如我们需要做一个比较大的任务，可以把这个任务分割为若干互不依赖的子任务，为了减少线程间的竞争，把这些子任务分别放到不同的队列里，并为每个队列创建一个单独的线程来执行队列里的任务，线程和队列一一对应。比如A线程负责处理A队列里的任务。但是，有的线程会先把自己队列里的任务干完，而其他线程对应的队列里还有任务等待处理。干完活的线程与其等着，不如去帮其他线程干活，于是它就去其他线程的队列里窃取一个任务来执行。而在这时它们会访问同一个队列，所以为了减少窃取任务线程和被窃取任务线程之间的竞争，通常会使用 **双端队列** ，被窃取任务线程永远从双端队列的**头部**拿任务执行，而窃取任务的线程永远从双端队列的**尾部**拿任务执行。
> 
> - 优点
> 
> 充分利用线程并发操作执行任务，减少线程间的竞争切换；
> 
> - 缺点
> 
> 某些情况下还是存在竞争，比如双端队列中只有一个任务时。并且此算法会消耗更多的系统资源，比如创建多个线程，多个双端队列。

### 使用流程

1. 分割任务
2. 执行任务
3. 合并结果

> ```java
> import java.util.concurrent.ForkJoinPool;
> import java.util.concurrent.Future;
> import java.util.concurrent.RecursiveTask;
> 
> public class CountTask extends RecursiveTask<Integer> {
>     private static final long serialVersionUID = 310195418127112037L;
> 
>     private static final int THRESHOLD = 2;// 阈值
>     private int start;
>     private int end;
> 
>     public CountTask(int start, int end) {
>         this.start = start;
>         this.end = end;
>     }
> 
>     @Override
>     protected Integer compute() {
>         int sum = 0;
>         // 如果任务足够小就计算任务
>         boolean canCompute = (end - start) <= THRESHOLD;
>         if (canCompute) {
>             for (int i = start; i <= end; i++) {
>                 sum += i;
>             }
>         } else {
>             // 如果任务大于阈值，就分裂成两个子任务计算
>             int middle = (start + end) / 2;
>             CountTask leftTask = new CountTask(start, middle);
>             CountTask rightTask = new CountTask(middle + 1, end);
>             // 执行子任务
>             leftTask.fork();
>             rightTask.fork();
>             // 等待子任务执行完，并得到其结果
>             int leftResult = leftTask.join();
>             int rightResult = rightTask.join();
>             // 合并子任务
>             sum = leftResult + rightResult;
> 
>             // 检查任务是否正常完成
>             if(leftTask.isCompletedAbnormally()) {
>                 System.out.println(leftTask.getException());
>             }
>         }
>         return sum;
>     }
> 
>     public static void main(String[] args) {
>         ForkJoinPool forkJoinPool = new ForkJoinPool();
>         // 生成一个计算任务，负责计算1+2+3+4
>         CountTask task = new CountTask(1, 4);
>         // 执行一个任务
>         Future<Integer> result = forkJoinPool.submit(task);
>         try {
>             System.out.println(result.get());
>         } catch (Exception e) {
>             e.printStackTrace();
>         }
>     }
> }
> ```

# 原子操作类

> Java从JDK 1.5开始提供了 `java.util.concurrent.atomic` 包（以下简称Atomic包），这个包中的原子操作类提供了一种用法简单、性能高效、线程安全地更新一个变量的方式。因为变量的类型有很多种，所以在Atomic包里一共提供了13个类，属于4种类型的原子更新方式，分别是原子更新**基本类型**、原子更新**数组**、原子更新**引用**和原子更新**属性（字段）**。Atomic包里的类基本都是使用`Unsafe`实现的包装类。

## 原子更新基本类型类

`AtomicBoolean`, `AtomicInteger`, `AtomicLong`

## 原子更新数组

`AtomicIntegerArray`, `AtomicLongArray`, `AtomicIntegerArray`, `AtomicReferenceArray`

## 原子更新引用类型

要原子更新多个变量，就要使用这个原子更新引用类型提供的类。

> - `AtomicReference` 原子更新引用类型
> - `AtomicReferenceFieldUpdater` 原子更新引用类型里的字段
> - `AtomicMarkableReference` 原子更新带有标记位的引用类型。可以原子更新一个布尔类型的标记位和引用类型
> 
> ```java
> import java.util.concurrent.atomic.AtomicReference;
> 
> public class AtomicRefrenceTest {
>     public static AtomicReference<User> atomicReference = new AtomicReference<User>();
> 
>     public static void main(String[] args) {
>         User user1 = new User("user1");
>         atomicReference.set(user1);
>         System.out.println(atomicReference.get());
>         User user2 = new User("user2");
>         atomicReference.compareAndSet(user1, user2);
>         System.out.println(atomicReference.get());
>     }
>     
>     static class User {
>         private String name;
> 
>         public User(String name) {
>             this.name = name;
>         }
>         public String getName() {
>             return name;
>         }
> 
>         public void setName(String name) {
>             this.name = name;
>         }
> 
>         @Override
>         public String toString() {
>             return "User [name=" + name + "]";
>         }
>     }
> }
> ```

## 原子更新字段类

解决 ABA 问题

`AtomicIntegerFieldUpdater`
`AtomicLongFieldUpdater`
`AtomicStampedReference`

> 要想原子地更新字段类需要两步。第一步，因为原子更新字段类都是抽象类，每次使用的时候必须使用静态方法`newUpdater()`创建一个更新器，并且需要设置想要更新的类和属性。第二步，更新类的字段（属性）必须使用 `public volatile` 修饰符。
>
> 以上3个类提供的方法几乎一样，所以仅以`AstomicIntegerFieldUpdater`为例进行讲解
>
> ```java
> public class AtomicIntegerFieldUpdaterTest {
>   // 创建原子更新器，并设置需要更新的对象类和对象的属性
>   private static AtomicIntegerFieldUpdater<User> a = AtomicIntegerFieldUpdater.newUpdater(User.class, "old");
>   public static void main(String[] args) {
>     // 设置柯南的年龄是10岁
>     User conan = new User("conan"， 10);
>     // 柯南长了一岁，但是仍然会输出旧的年龄
>     System.out.println(a.getAndIncrement(conan));
>     // 输出柯南现在的年龄
>     System.out.println(a.get(conan));
>   }
> 
>   public static class User {
>     private String name;
>     public volatile int old;
>     public User(String name， int old) {
>       this.name = name;
>       this.old = old;
>     }
>     public String getName() {
>       return name;
>     }
>     public int getOld() {
>       return old;
>     }
>   }
> }
> ```

# 并发工具类

下面整理 jdk 提供的常用并发工具类。

## `CountDownLatch` 等待多线程完成

```java
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.TimeUnit;

public class CountDownLatchTest {
    // 等待 2 个线程完成再继续
    static CountDownLatch latch = new CountDownLatch(2);
    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(1);
                latch.countDown();
            }
        }).start();
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(2);
                latch.countDown();
            }
        }).start();
        try {
//          latch.await();//不限定超时
            latch.await(3, TimeUnit.MINUTES);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

```

## `CyclicBarrier` 同步屏障

> 每个线程在到达一个屏障点（同步点）时都等待，一直到所有的线程都到达了这个屏障点后，屏障才打开，让所有这些线程一起返回。
>
> 单个线程调用 `CyclicBarrier#await()` 方法，表明其到达了屏障，然后处于阻塞状态，直至所有线程都到达屏障才放行。

> ```java
> import java.util.concurrent.BrokenBarrierException;
> import java.util.concurrent.CyclicBarrier;
> 
> public class CyclicBarrierTest {
>     static CyclicBarrier barrier = new CyclicBarrier(4);
> //    static CyclicBarrier barrier2 = new CyclicBarrier(4, new Runnable() {
> //        @Override
> //        public void run() {
> //            System.out.println("all finished. 此处可以汇总各个线程的结果");
> //        }
> //    });
> 
>     public static void main(String[] args) {
>         for (int i = 0; i < 3; i++) {
>             final int t = i;
>             new Thread(new Runnable() {
>                 @Override
>                 public void run() {
>                     try {
>                         System.out.println(t);
>                         barrier.await();
>                     } catch (InterruptedException | BrokenBarrierException e) {
>                         e.printStackTrace();
>                     }
>                 }
>             }).start();
>         }
>         try {
>             barrier.await();
>         } catch (InterruptedException | BrokenBarrierException e) {
>             e.printStackTrace();
>         }
>         System.out.println("ok");
>     }
> }
> 
> ```

相比于 `CountDownLatch`, `CyclicBarrier` 更为灵活，而且前者的计数器只能使用一次，后者可以使用 `reset()` 重置，也可以获取阻塞的线程数量等。

## `Semaphore` 并发线程控制

> 控制同时访问特定线程的线程数量，保证合理使用公共资源，适用于流控调度等场景。
> 
> ```java
> public class SemaphoreTest {
>   private static final int THREAD_COUNT = 30;
>   private static ExecutorServicethreadPool = Executors.newFixedThreadPool(THREAD_COUNT);
>   private static Semaphore s = new Semaphore(10);
>   public static void main(String[] args) {
>     for (inti = 0; i< THREAD_COUNT; i++) {
>       threadPool.execute(new Runnable() {
>         @Override
>         public void run() {
>           try {
>             s.acquire();
>             System.out.println("save data");
>             s.release();
>           } catch (InterruptedException e) {
>               e.printStackTrace();
>           }
>         }
>       });
>     }
>     threadPool.shutdown();
>   }
> }
> ```

## `Exchanger` 线程间数据交换

> Exchanger（交换者）是一个用于线程间协作的工具类。Exchanger用于进行线程间的数据交换。它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。这两个线程通过exchange方法交换数据，如果第一个线程先执行exchange()方法，它会一直等待第二个线程也执行exchange方法，当两个线程都到达同步点时，这两个线程就可以交换数据，将本线程生产出来的数据传递给对方。
>
> Exchanger可以用于遗传算法，遗传算法里需要选出两个人作为交配对象，这时候会交换两人的数据，并使用交叉规则得出2个交配结果。Exchanger也可以用于校对工作，比如我们需要将纸制银行流水通过人工的方式录入成电子银行流水，为了避免错误，采用AB岗两人进行录入，录入到Excel之后，系统需要加载这两个Excel，并对两个Excel数据进行校对，看看是否录入一致，代码如下：
>
> ```java
> public class ExchangerTest {
>     private static final Exchanger<String> exgr = new Exchanger<String>();
> 
>     private static ExecutorService threadPool = Executors.newFixedThreadPool(2);
> 
>     public static void main(String[] args) {
>         threadPool.execute(new Runnable() {
>             @Override
>             public void run() {
>                 try {
>                     String A = "银行流水A";// A录入银行流水数据
>                     exgr.exchange(A);
>                 } catch (InterruptedException e) {
>                 }
>             }
>         });
>         threadPool.execute(new Runnable() {
>             @Override
>             public void run() {
>                 try {
>                     String B = "银行流水B";// B录入银行流水数据
>                     String A = exgr.exchange("B");
>                     System.out.println("A和B数据是否一致：" + A.equals(B) + "，A录入的是：" + A + "，B录入是：" + B);
>                 } catch (InterruptedException e) {
>                 }
>             }
>         });
>         threadPool.shutdown();
>     }
> }
> ```
> 如果两个线程有一个没有执行exchange()方法，则会一直等待，如果担心有特殊情况发生，避免一直等待，可以使用exchange（V x，longtimeout，TimeUnit unit）设置最大等待时长。


# 线程池

# `Executor` 框架





---

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

 (゜-゜)つロ *参考并致谢《Java并发编程的艺术》*