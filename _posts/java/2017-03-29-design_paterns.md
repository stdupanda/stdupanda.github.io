---
layout: post
title: java Design Patterns 设计模式整理
categories: Java
description: java Design Patterns 设计模式
keywords: Java, Design Patterns, 设计模式
---

## 单例模式 singleton

https://mp.weixin.qq.com/s/9KV5CRyiLIn957ROr_KAZQ

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

public class ToolFactory {
    public static class Tool {
        void fun() {
        }
    }

    public static Tool getTool() {
        return InnerToolFactory.toolIns;
    }

    // 利用内部类延迟加载
    private static class InnerToolFactory {
        public static final Tool toolIns;

        static {
            System.out.println("初始化Tool实例");
            toolIns = new Tool();
        }
    }

    @Test
    public void test() {
        System.out.println("开始");
        Tool tool = ToolFactory.getTool();
        System.out.println(tool);
    }
}
```
