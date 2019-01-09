---
layout: post
title: java 并发基础03并发类
categories: Java
description: java 并发基础03并发类
keywords: Java, java, jdk, openjdk, concurrent, lock
---

整理总结常用 java 并发类库。

# 并发容器和框架

## 并发下 `map` 的选择

### `HashMap` 和 `HashSet` 的问题

> HashMap在并发执行put操作时会引起死循环，是因为多线程会导致HashMap的Entry链表形成环形数据结构，一旦形成环形数据结构，Entry的next节点永远为空，就会产生死循环获取Entry。

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

对应的 jdk `put` 源码：

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