---
layout: post
title: java 并发基础03并发类
categories: Java
description: java 并发基础03并发类
keywords: Java, java, jdk, openjdk, concurrent, lock
---

整理总结常用 java 并发类库。

# 并发容器和框架

## 并发操作 `map`

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

## 并发队列

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
> 访问者的公平性是使用可重入锁实现的，代码如下。
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

# 原子操作类

# 并发工具类

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