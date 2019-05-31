---
layout: post
title: Linux 系统参数优化
categories: Linux
description: Linux 系统参数优化
keywords: linux, tcp, jvm, optimize, 优化, 调参, 参数, centos
---

对于线上服务来说，最常用的运行平台就是 Linux 发行版了，因此经常需要对 Linux 系统的内核、网络、IO 等方面进行参数调整和优化，本文主要整理记录更新此部分内容。

ulimit -Sn 100000 # This will only work if hard limit is big enough.
sysctl -w fs.file-max=100000
/proc/swaps
transparent_hugepage(thp) 数据库或者 NoSQL 实例上都应当关闭
