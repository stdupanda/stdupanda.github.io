---
layout: post
title: linux sed
categories: Linux
description: linux sed
keywords: linux, sed
---

打包流程中需要利用 shell 脚本，将源码 `.java` 文件中的字符串进行替换，于是整理 sed 的使用方式。

# 目的介绍

`sed` 处理 java 文件修改内容，原文如下:

```java
public class AppVersion {
    private static final String GIT_COMMIT_ID = "";
    .....其他内容
}
```
需要将 GIT_COMMIT_ID 赋值，然后进行编译打包等操作。

# sed 简介

流式处理文件

# 记录
## 交互方式

`sed -i ` 直接修改文件

`sed -e` 控制台输出处理结果，不写入文件

因此是先 `sed -e` 再 `sed -i` 执行写入。

## 实际记录

```bash
sed -e 's/GIT_COMMIT_ID.*"/替换后的内容，允许空格/g' AppVersion.java
sed -e 's/GIT_COMMIT_ID[[:space:]]=.*;/替换后的内容，允许空格/g' AppVersion.java
```

## shell变量

表达式中包括shell变量时需要按照以下方式处理

```bash
sed "s/$a/$b/" filename
sed 's/'$a'/'$b'/' filename 
sed s/$a/$b/ filename
```