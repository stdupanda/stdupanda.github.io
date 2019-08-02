---
layout: post
title: 网页安全问题整理
categories: web
description: 分析网页加载过程
keywords: web, security, xss, csrf, sql, http
---

整理常见的 web 安全问题。

## 简介

web 服务在公共的网络环境下存在很多的潜在的或者暴露的问题。常见的 web 安全问题可以简单分为前段和后端。

### http 报文结构

- 请求报文
  - 请求行
  - 请求头
    - cookie 凭证
    - refrer 流量统计、防盗链、仿 CSRF
- 响应报文
  - 状态行
  - 消息报头
  - 响应正文
    - 响应 cookie

### 搜索引擎使用技巧

- 搜索关键字
  - hacked by
- 高级搜索
  - intitle:keyword
  - intext:keyword
  - site:domain

## 网站被 hack 后的问题

- 被埋入恶意暗链
  - 医疗、游戏、色情、博彩类暗链
- 恶意利用 SEO 优化

## 常见 web 安全问题

### 客户端

- XSS
  - 存储型
  - 反射型
  - DOM 型

|XSS 类型|存储型|反射型|DOM型|
|:---:|:---:|:---:|:---:|
|触发过程|1、黑客构建的XSS脚本<br/>2、访问携带XSS脚本的页面|正常用户访问携带XSS脚本的URL|正常用户访问携带XSS脚本的URL|
|数据存储|
|输出来源|
|输出位置|

- CSRF
- 点击劫持
- URL 跳转

### 服务端

- SQL 注入
- 命令注入
- 文件注入
  - webshell

简单总结：
