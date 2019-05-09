---
layout: post
title: java 读取大文件
categories: Java
description: java 读取大文件
keywords: Java, java, big, file
---

整理 java 操作大文件的方法。

## 内存中操作

```java
Files.readLines(new File(path), Charsets.UTF_8);// guava
FileUtils.readLines(new File(path));// commons-io
```

需注意的是方式需要考虑内存限制，否则会出现 `OutOfMemoryError`

## `java.util.Scanner`

使用 `java.util.Scanner` 类扫描文件的内容，一行一行连续地读取， 这种方案将会遍历文件中的所有行——允许对每一行进行处理，而**不保持对它的引用**。

## `org.apache.commons.io.LineIterator`

```java
LineIterator it = FileUtils.lineIterator(theFile, "UTF-8");
try {
    while (it.hasNext()) {
        String line = it.nextLine();
        // do something with line
    }
} finally {
    LineIterator.closeQuietly(it);
}
```

此方法对内存占用也很少。

## `java.io.RandomAccessFile` 和 `NIO FileChannel`
