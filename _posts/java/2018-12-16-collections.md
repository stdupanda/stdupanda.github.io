---
layout: post
title: 整理 java 集合框架
categories: Java
description: 整理 java collections 集合框架
keywords: java, collection, 集合, hash
---

整理 java collections 集合框架

官方文档入口： [Collections Framework Overview](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html)

## 前言

java 集合框架的接口分为两类，Collection 以及 Map 接口。

各个集合接口类的命名方式均为： `<Implementation-style><Interface>` ，整理如下：

|Interface|Hash Table|Resizable Array|Balanced Tree|Linked List|Hash Table + Linked List|
|:---|:---|:---|:---|:---|:---|
|Set|HashSet||TreeSet||LinkedHashSet|
|List||ArrayList||LinkedList||
|Deque||ArrayDeque||LinkedList||
|Map|HashMap||TreeMap||LinkedHashMap|

## java.util.Collection

> `java.util.Collection` 有如下 descendants:

- `java.util.Set`
- `java.util.SortedSet`
- `java.util.NavigableSet`
- `java.util.Queue`
- `java.util.concurrent.BlockingQueue`
- `java.util.concurrent.TransferQueue`
- `java.util.Deque`
- `java.util.concurrent.BlockingDeque`

下面主要整理常用的比较重要的类。

> List 是元素有序的、可以重复、可以为 null 的集合。

- ArrayList
  - 使用动态数组实现
    - 初始初始容量 10
    - 当元素数量大于初始容量时进行扩容
    - int newCapacity = oldCapacity + (oldCapacity >> 1) 右移一位相当于除2，即为原来容量的 1.5 倍
  - 操作效率
    - 查询速度快，实现了 RandomAccess 接口
    - 在中间插入元素时，由于其余元素要后移导致性能比较差。
  - 非线程安全
  - 允许重复元素、null 元素。
- LinkedList
  - 基于双向链接实现类，存储前后节点耗费稍多的内存
  - 它表现上是一个有序的集合，但内存中其实是无序保存。
  - 操作效率
    - 查询速度慢
    - 插入的速度快
  - 非线程安全的集合。
  - 允许重复元素、null 元素
- Vector
  - 使用动态数组实现
    - 默认初始容量 10
    - 扩容规则：新容量 = 旧容量 + 扩容增量，默认 新容量 = 2 * 旧容量。
  - 使用了 synchronized 是一个线程安全的集合
  - 单线程下建议使用 ArrayList

> Set 是元素无序、不可重复的集合。

- HashSet
  - 内部使用 HashMap 实现
    - 根据对象的哈希值进行运算，得出元素在哈希表的位置
    - 通过 == 和 equals 判断是否存在
      - 若已存在则返回 false，不继续操作
  - 存取速度快
- LinkedHashSet
  - 使用 LinkedHashMap 实现
  - 继承自 HashSet，大部分操作调用父类实现
  - 元素有序
- TreeSet
  - 使用红-黑树的数据结构实现的，默认对元素进行自然排序（String）。

## java.util.Map

> collection interfaces which are based on `java.util.Map` are not true collections. However, these interfaces contain collection-view operations, which enable them to be manipulated as collections.
>
> The Map interface provides three collection views, which allow a map's contents to be viewed as a set of keys, collection of values, or set of key-value mappings.
>
> Map has the following offspring:

- `java.util.SortedMap`
- `java.util.NavigableMap`
- `java.util.concurrent.ConcurrentMap`
- `java.util.concurrent.ConcurrentNavigableMap`

下面主要整理常用的比较重要的类。

> Map 集合类用于存储键——值对。

- HashMap
  - 使用哈希表实现
    - 根据 key 的哈希值进行运算，得出元素在哈希表的位置
    - 默认大小为 16
    - 扩容为 2n
  - 组成
    - 1.7 版本基于 数组 + 链表
    - 1.8 版本基于 数组 + 链表/红黑树（链表长度大于 8 时转换为红黑树）
  - key 和 value 均允许为 null
  - 不保证顺序性
- LinkedHashMap
  - 继承自 HashMap
  - 增加双向链表，使得 key-value 有序
- ConcurrentHashMap
  - 组成
    - 1.7 采用锁分段数组+链表
    - 1.8 采用cas+数组+链表/红黑树
    - 默认初始大小 16
    - 默认负载因子 0.75
- TreeMap
  - 基于红黑树数据结构实现
    - 默认初始大小为 11
    - 默认扩容变为 2n+1
  - 会对元素的 key 进行排序存储
- Hashtable
  - 数组 + 链表组成
    - 默认容量 11
    - 默认复杂因子 0.75
  - key 和 value 均不允许为 null
  - 使用了 synchronized 是线程安全的
  - 并发效率低

## 并发集合类

并发操作集合接口包括：

- BlockingQueue
- TransferQueue
- BlockingDeque
- ConcurrentMap
- ConcurrentNavigableMap

如下是实现类：

- LinkedBlockingQueue
- ArrayBlockingQueue
- PriorityBlockingQueue
- DelayQueue
- SynchronousQueue
- LinkedBlockingDeque
- LinkedTransferQueue
- CopyOnWriteArrayList
- CopyOnWriteArraySet
- ConcurrentSkipListSet
- ConcurrentHashMap
- ConcurrentSkipListMap

主要整理如下：

- CopyOnWriteArrayList
  - 内部持有 ReentrantLock，更新时加锁
  - 内部数组为 volatile 的 array
  - 写操作
    - 写时复制出一个数组
    - 更新完成后将新数组赋值给 array
- ConcurrentHashMap
  - 见上面 Map 部分的总结
- CopyOnWriteArraySet
  - 内部采用 CopyOnWriteArrayList 实现
  - 本质是 Set，不允许重复元素

---
