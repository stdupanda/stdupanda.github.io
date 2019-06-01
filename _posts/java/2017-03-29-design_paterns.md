---
layout: post
title: java Design Patterns 设计模式整理
categories: Java
description: java Design Patterns 设计模式
keywords: Java, Design Patterns, 设计模式
---

## 单例模式 singleton

```java
public class Tool {
    private volatile static Tool instance;

    private Tool(){}

    public static Tool getInstance() {
        if (null == instance) {
            synchronized {
                if (null == instance) {
                    instance = new Tool();
                }
            }
        }
        return instance;
    }
}
```
