---
layout: post
title: java 并发编程01内存可见性问题
categories: 并发编程
description: java 并发编程时的内存可见性问题
keywords: java, concurrent, memory
---

简述 java 并发编程时内存可见性的问题。

# 程序示例

```java
package 多线程;

public class Demo001内存可见性 {
    public static void main(String[] args) {
        System.out.println("===开始===");
        ThreadDemo demo = new ThreadDemo();
        new Thread(demo).start();
        while (true) {
            if (1 == demo.getFlag()) {
                System.out.println(System.nanoTime() + Thread.currentThread().getName() + ":flag is " + demo.getFlag());
                break;
            }
        }
        System.out.println(System.nanoTime() + "===结束===");
    }
}

// inner class
class ThreadDemo implements Runnable {

    // private volatile int flag = 0;
    private int flag = 0;

    @Override
    public void run() {
        try {
            Thread.sleep(100);// 故意设置一定时间的延迟，防止此线程在 main 函数 if 判断之前结束
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        flag = 1;
        System.out.println(System.nanoTime() + Thread.currentThread().getName() + ":set flag to 1 ok");
    }

    public int getFlag() {
        return flag;
    }
}
```

# 运行效果

以上程序会持续运行不退出，因为内存可见性的问题，多个线程操作同一个对象的同一个变量时不能同步的问题，导致程序无法跳出循环。

# 关于 spring 框架的  `ThreadLocal`

> 按照传统经验，如果某个对象是非线程安全的，在多线程环境下，对对象的访问必须采用synchronized进行线程同步。但模板类并未采用线程同步机制，因为线程同步会降低并发性，影响系统性能。此外，通过代码同步解决线程安全的挑战性很大，可能会增强好几倍的实现难度。那么模板类究竟仰仗何种魔法神功，可以在无须线程同步的情况下就化解线程安全的难题呢？答案就是ThreadLocal！

> ThreadLocal 在 Spring 中发挥着重要的作用，在管理 request 作用域的 Bean、事务管理、任务调度、AOP 等模块都出现了它们的身影，起着举足轻重的作用。要想了解 Spring 事务管理的底层技术，ThreadLocal 是必须攻克的山头堡垒。

> ThreadLocal，顾名思义，它不是一个线程，而是 **线程的一个本地化对象**。 **当工作于多线程中的对象使用 ThreadLocal 维护变量时，ThreadLocal 为每个使用该变量的线程分配一个独立的变量副本。所以每一个线程都可以独立地改变自己的副本，而不会影响其他线程所对应的副本。从线程的角度看，这个变量就像是线程的本地变量，这也是类名中 “Local” 所要表达的意思**。

关于 ThreadLocal 更多参考 [http://www.iteye.com/topic/1123824](http://www.iteye.com/topic/1123824 "http://www.iteye.com/topic/1123824")
