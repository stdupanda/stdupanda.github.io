---
layout: post
title: git 提交时的换行符问题
categories: git
description: git 常用命令总结
keywords: Python, Ubuntu, psutil
---

提交仓库时提示 `warning: LF will be replaced by CRLF in xxx`

git 仓库位于 Linux 系统，开发时是在 windows 系统上检出仓库进行开发，于是，window 下使用 `git` 命令提交时总是被提示：

`warning: LF will be replaced by CRLF in xxx`

`The file will have its original line endings in your working directory.`

肯定是因为 windows 和 linux 的换行符处理不一致导致的，网上确认了下，解决方式如下：

在Windows系统上，把 `core.autocrlf` 设置成 `true`，则签出代码时，LF 会被转换成 CRLF，提交时自动地把 CRLF 转换成 LF：

- 命令行修改用户配置

`$ git config --global core.autocrlf true`

- `eclipse` 中修改用户配置

`Team -> User Settings -> 配置 core.autocrlf true`

更多配置见下面：

- `git config --global core.autocrlf input` 在Windows系统上的签出文件中保留 CRLF，会在 Mac 和 Linux 系统上，包括仓库中保留 LF。

- `git config --global core.autocrlf false` 仅运行在Windows上

参考原文如下：

> **core.autocrlf**

> 假如你正在Windows上写程序，又或者你正在和其他人合作，他们在Windows上编程，而你却在其他系统上，在这些情况下，你可能会遇到行尾结束符问题。这是因为Windows使用回车和换行两个字符来结束一行，而Mac和Linux只使用换行一个字符。虽然这是小问题，但它会极大地扰乱跨平台协作。
> Git可以在你提交时自动地把行结束符CRLF转换成LF，而在签出代码时把LF转换成CRLF。用`core.autocrlf`来打开此项功能，如果是在Windows系统上，把它设置成`true`，这样当签出代码时，LF会被转换成CRLF：

> `$ git config --global core.autocrlf true`

> Linux或Mac系统使用LF作为行结束符，因此你不想 Git 在签出文件时进行自动的转换；当一个以CRLF为行结束符的文件不小心被引入时你肯定想进行修正，把core.autocrlf设置成input来告诉 Git 在提交时把CRLF转换成LF，签出时不转换：

> `$ git config --global core.autocrlf input`


> 这样会在Windows系统上的签出文件中保留CRLF，会在Mac和Linux系统上，包括仓库中保留LF。

> 如果你是Windows程序员，且正在开发仅运行在Windows上的项目，可以设置false取消此功能，把回车符记录在库中：

> `$ git config --global core.autocrlf false`

-----
注： carriage return；line feed

参考网站：[git-scm 配置 Git](https://git-scm.com/book/zh/v1/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-%E9%85%8D%E7%BD%AE-Git "https://git-scm.com/book/zh/v1/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-%E9%85%8D%E7%BD%AE-Git")
