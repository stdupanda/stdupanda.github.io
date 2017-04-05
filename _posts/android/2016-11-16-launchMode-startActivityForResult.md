---
layout: post
title: startActivityForResult+onActivityResult
categories: Android
description: startActivityForResult启动新的Activity时onActivityResult立刻执行
keywords: android, Android, launchMode
---

在测试项目时发现一个问题，具体描述如下：

A 中调用了 `startActivityForResult` 启动 B，B 还未 `finish` 返回数据，A中的 `onActivityResult` 事件就立刻执行了。

# 分析问题

经过多方查找，终于定位在如下：

在 `AndroidManifest.xml` 文件中，若 Activity A 配置 `android:launchMode="singleInstance"` ，则会出现此问题。

当初是为了解决点击通知栏 `notification` 跳转到 `Activity` 时重复创建 `Activity` 实例的问题， 结果现在出了个坑。

# 解决问题

A、B 的配置改为 `android:launchMode="standard"` 即可。