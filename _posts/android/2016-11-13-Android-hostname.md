---
layout: post
title: 修改Android手机主机名
categories: 归档
description: 修改Android手机主机名
keywords: android, Android, 手机, 主机
---

调整android网络主机名

# 定位配置文件

一般是/system目录下的build.prop文件，其他的奇葩ROM再具体分析。

# 调整配置

build.prop文件的末尾添加一行net.hostname=XXXX

修改此文件需要Root权限

# 保存文件并重启手机

然后主机名就可以生效了。