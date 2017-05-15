---
layout: post
title: git 提交时的换行符问题
categories: git
description: git 常用命令总结
keywords: Python, Ubuntu, psutil
---

window 下使用 `git` 命令提交时总是被提示：

`warning: LF will be replaced by CRLF in xxx`
`The file will have its original line endings in your working directory.`

肯定是因为 windows 和 linux 的换行符处理不一致导致的，网上确认了下，
