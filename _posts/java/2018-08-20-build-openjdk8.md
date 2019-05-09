---
layout: post
title: 编译 openjdk8
categories: Java
description: 编译 openjdk8
keywords: Java, java, jdk, openjdk
---

整理手动编译 openjdk8 的流程。

## 前言

一开始是想着编译 jdk7 来着，但是看着 [openjdk7 编译流程](http://hg.openjdk.java.net/jdk7u/jdk7u/raw-file/tip/README-builds.html "openjdk7-build-readme") 发现，流程太繁琐。后来看了 [openjdk8 的编译流程 ](http://hg.openjdk.java.net/jdk8u/jdk8u/raw-file/tip/README-builds.html "openjdk8-build-readme") 简单了很多，于是决定进行 `openjdk8` 的编译。

refer to 官网:

> The build is now a "configure && make" style build

这就很舒服了对吧。

## 准备

`centos 6.10`, `openjdk7`

## 开始

### 使用 `Mercurial` 下载源码

```shell
yum install hg
hg clone http://hg.openjdk.java.net/jdk8/jdk8
cd jdk8
sh get_source.sh
# 在各个目录执行 hg status 命令
bash ./make/scripts/hgforest.sh status
```

下载完 openjdk8 的源码， `du -sh` 大概有 800M 的大小。压缩成 tar 包后大概是 300M。

注： *因为网络原因导致的源码下载失败问题，建议在阿里云机器进行操作，速度较快。*

### build

> Be sure the GNU make utility is version 3.81 or newer, e.g. run "make -version"

> Building JDK 8 requires use of a version of JDK 7 that is at Update 7 or newer. JDK 8 developers should not use JDK 8 as the boot JDK, to ensure that JDK 8 dependencies are not introduced into the parts of the system that are built with JDK 7.

```shell
# make 版本需高于 3.81
make -version
# 执行 configure 操作。可以在 configure 文件中查看对应的默认参数配置信息。
bash ./configure
# 可能会提示安装一些编译依赖，yum install 安装即可，如： yum install ccache
# 开始 make
make all
```

[自定义 `configure `参数](http://hg.openjdk.java.net/jdk8u/jdk8u/raw-file/tip/README-builds.html#configure "configure配置")

最后编译结果就在 jdk8 的 build 路径下。
