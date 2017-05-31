---
layout: post
title: Eclipse 设置 jdk
categories: Eclipse
description: Eclipse 设置 jdk
keywords: shortkey, eclipse, 模板, Eclipse
---

机器上装了多个 `JDK`，但是 `Path` 中只能指定一个，现在记录修改手动指定 `Eclipse` 的 `JDK` 路径。

在其安装路径下找到 `eclipse.ini` 文件，首行加入如下：

```
-vm
D:/jdk1.7.0_45/bin/javaw.exe
```

保存即可。
