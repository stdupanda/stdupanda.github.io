---
layout: post
title: Design Patterns
categories: Java
description: Design Patterns
keywords: Java, Design Patterns, 设计模式
---

## 单例模式 singleton

```java
//省略饿汉模式
//考虑并发的饱汉
public class Tool {
    private volatile static Tool instance;
    public static Tool getInstance() {
        if (null == instance) {
            synchronized (Tool.class) {
                if (null == instance) {
                    instance = new Tool();
                    instance.init();
                }
            }
        }
        return instance;
    }
}
```
