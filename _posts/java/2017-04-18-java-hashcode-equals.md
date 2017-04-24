---
layout: post
title: Java 中 hashcode & equals
categories: Java
description: Java 中的hashcode & equals分析
keywords: Java, 集合
---

为 java 集合框架做基础，先掌握 `hashCode()` 和 `equals()` 接口。

# equals  方法定义

Object类中默认的实现方式是：return this == obj，也就是说，只有this 和 obj引用同一个对象，才会返回true。
而我们往往需要用equals来判断 2个对象是否等价，而非验证他们的唯一性。这样我们在实现自己的类时，就要重写equals.

> 判定 equals() 原则如下：
 
> - 对于任何非 null 对象引用值 x， x.equals(x) 应返回 true。
> - 对于任何非 null 对象引用值 x、y，x.equals(y) 应返回 true 当且仅当 y.equals(x) 返回 true. 
> - 对于任何非 null 对象引用值 x、y、z，若 x.equals(y) 返回 true 且 y.equals(z) 返回 true, 则 x.equals(z) 应返回 true. 
> - 对于任何非 null 对象引用值 x、y，在对象未改变的情况下 x.equals(y) 的返回值应永远返回 true 或永远返回 false。
> - 对于任何非 null 对象引用值 x, x.equals(null) 应永远返回 false. 
> -对于任何非 null 对象引用值 x、y，当且仅当 x and y 引用同一个对象 (x == y has the value true)时才返回 true

> **Note that it is generally necessary to override the hashCode() method whenever this method is overridden, so as to maintain the general contract for the hashCode method, which states that equal objects must have equal hash codes.**

## 一种错误

有些程序员使用下面的第二种写法替代第一种比较运行时类的写法。应该避免这样做。

```java
// 1 ok
if ((obj == null) || (obj.getClass() != this.getClass())) {
    return false;
}
// 2 error
if (!(obj instanceof Test)) {
    return false; // avoid 避免！
}
```
它违反了公约中的对称原则。

例如：假设Dog扩展了Aminal类。

```java
dog instanceof Animal // 得到true

animal instanceof Dog // 得到false
```
这就会导致

animal.equls(dog) 返回true
dog.equals(animal) 返回false

仅当Test类没有子类的时候，这样做才能保证是正确的。

# hashCode 方法定义

> Returns a hash code value for the object. This method is supported for the benefit of hash tables such as those provided by java.util.HashMap. 返回该对象的哈希值。该方法为使用哈希表提升性能，例如java.util.Hashtable 提供的哈希表。

> The general contract of hashCode is: 

> - Whenever it is invoked on the same object more than once during an execution of a Java application, the hashCode method must consistently return the same integer, provided no information used in equals comparisons on the object is modified. This integer need not remain consistent from one execution of an application to another execution of the same application. 同一个对象每次调用 hashCode 方法都返回同一个整数值，前提是将对象进行 equals 比较时所用的信息没有被修改。一个对象的哈希值在程序一次运行中不变，不同运行过程中允许不一样。
> - If two objects are equal according to the equals(Object) method, then calling the hashCode method on each of the two objects must produce the same integer result. 若两个对象的 equals() 方法返回 true，则两个对象的哈希值一定相同
> - It is not required that if two objects are unequal according to the java.lang.Object.equals(java.lang.Object) method, then calling the hashCode method on each of the two objects must produce distinct integer results. However, the programmer should be aware that producing distinct integer results for unequal objects may improve the performance of hash tables. 若两个对象调用 equals(Object)返回 false 那么两个对象的哈希值并不一定不同。但是应该注意：为不相等的对象生成不同的哈希值可以提升哈希表性能。

> As much as is reasonably practical, the hashCode method defined by class Object does return distinct integers for distinct objects. (This is typically implemented by converting the internal address of the object into an integer, but this implementation technique is not required by the JavaTM programming language.)由 Object 类定义的 hashCode()  方法确实会针对不同的对象返回不同的整数。（这一般是通过将该对象的内部地址转换成一个整数来实现的，但是 JavaTM 编程语言不需要这种实现技巧。）

# equals() & hashCode()

整理如下：

- `hashCode()` 方便利用哈希表快速查找对象，用于在 `散列存储结构存储`（例如 `HashMap`、`HashTable`）中确定对象的存储地址的。

- 若两个对象进行 `equals()` 操作返回为 `true`，则这两个对象的 `hashCode()` 返回结果一定相同。

- 若两个对象的 `hashCode()` 一样，并不一定保证两个对象进行 `equals()` 操作返回为 `true`，只能说明这两个对象在散列存储结构中(如Hashtable)被“存放在一个篮子里”。

- **若对象的 `equals()` 方法被重写，那么对象的 `hashCode()` 也要重写，否则就会违反上面提到的第2点。当此对象做 `Map` 类中的 `Key` 时，两个 `equals()` 为 `true` 的对象其获取的 `value` 都是同一个，才能符合实际。**

# 集合 & equals() & hashCode()

将对象放入到集合中时，首先判断要放入对象的 hashCode 值与集合中的任意一个元素的 hashCode 值是否相等，如果不相等直接将该对象放入集合中。如果 hashCode 值相等，然后再通过 equals()  方法判断要放入对象与集合中的任意一个对象是否相等，如果 equals() 判断不相等，直接将该元素放入到集合中，否则不放入。

# hashCode() 编写指导

在编写hashCode时，你需要考虑的是，最终的hash是个int值，而不能溢出。不同的对象的hash码应该尽量不同，避免hash冲突。

引用自[博客](http://www.cnblogs.com/lulipro/p/5628750.html "原文")

定义一个int类型的变量 hash,初始化为 7。

接下来让你认为重要的字段（equals中衡量相等的字段）参入散列运，算每一个重要字段都会产生一个hash分量，为最终的hash值做出贡献（影响）

运算方法参考表

| 重要字段var的类型     | 他生成的hash分量   |
| ---------------------:|:-------------   :  |
| byte, char, short,int | (int) var
| long                  |(int)(var ^ (var >>> 32))
| boolean               | var ? 1 : 0
| float                 | Float.floatToIntBits(var)
| double                | long bits = Double.doubleToLongBits(var);
| double                | 分量 = (int)(bits ^ (bits >>> 32));
| 引用类型              | (null == var ? 0 : var.hashCode())

# 分布式应用 & 哈希码

** 在分布式应用中不要使用哈希码 **

一个远程对象可能与本地对象有不同的哈希码，即使这两个对象是相等的。

此外，你应该意识到从一个版本到另一个版本哈希码的功能实现可能会更改。因此您的代码不应该依赖于任何特定的哈希码值。例如，你不应该使用哈希码来持久化状态。下次你运行程序的时候，“相同”对象的哈希码可能不同。
最好的建议可能是：完全不使用哈希码，除非你自己创造了基于哈希的算法。