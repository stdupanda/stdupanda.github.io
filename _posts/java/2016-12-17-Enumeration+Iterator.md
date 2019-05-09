---
layout: post
title: Enumeration和Iterator
categories: Java
description: Enumeration和Iterator
keywords: Java
---

整理了 Enumeration 和 Iterator 相比的一些不同之处：

Iterator-迭代器 在功能上可以完全替代 Enumeration-枚举 。后者目前还保留在Java标准库里纯粹是为了兼容老API（例如Hashtable、Vector、Stack等老的collection类型）。

Iterator相比Enumeration有以下区别：

- 前者的方法名比后者简明扼要；

- 前者添加了一个可选的remove()方法（“可选”意味着一个实现Iterator接口的类可以选择不实现remove()方法）；

- 前者是“fail-fast”的——如果它在遍历过程中，底下的容器发生了结构变化（例如add或者remove了元素），则它会抛出ConcurrentModificationException；后者没有这种检查机制；

- 前者可以配合 `Iterable<E>` 接口用于Java 5的for-each循环中。

Enumeration速度是Iterator的2倍，同时占用更少的内存，但是，Iterator远远比Enumeration安全，因为其他线程不能够修改正在被iterator遍历的集合里面的对象。同时，Iterator允许调用者删除底层集合里面的元素，这对Enumeration来说是不可能的。

**调用next()方法后才可以调用remove()方法,而且每次调用next()后最多只能调用一次remove()方法,否则抛出IllegalStateException异常.**

使用Iterator来遍历集合时，应使用Iterator的remove()方法来删除集合中的元素，使用集合的remove()方法将抛出ConcurrentModificationException异常。

|`Enumeration`|`Iterator`|
|:--|:--|
|兼容老API|专门替代前者|
|—|允许删除元素|
|—|方法名称优化|

作者：RednaxelaFX
链接：https://www.zhihu.com/question/42961567/answer/95050993
来源：知乎
著作权归作者所有，转载请联系作者获得授权。
