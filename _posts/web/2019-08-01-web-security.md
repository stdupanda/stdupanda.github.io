---
layout: post
title: 网页安全问题整理
categories: web
description: 分析网页加载过程
keywords: web, security, xss, csrf, sql, http
---

常见的 web 安全问题介绍。

## 简介

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
|数据存储|数据库|URL|URL|
|输出来源|后端服务|后端服务|前段 js 脚本|
|输出位置|http 响应报文|http 响应报文|动态构建的 DOM 节点|

- CSRF
- 点击劫持
- URL 跳转

### 服务端

- SQL 注入
- 命令注入
  - 调用可执行命令的系统函数
  - 函数入参无校验
- 文件注入
  - webshell
  - 文件上传、下载无校验

## 工具篇

- 推荐的 Firefox 插件
  - FireBug
  - HackBar
  - AdvancedCookieManager
  - ProxySwitcher
- 推荐的抓包工具
  - Burpsuit
  - Charles
  - Fiddler
- 使用 `御剑` 扫描敏感文件
- 使用 python 脚本编写
- 漏扫工具
  - AWVS
  - Netsparker
  - Appscan
  - Sqlmap
  - 相辅相成，灵活使用，总结思路
- 相关网站
  - fofa
  - shodan
  - zoomeye
- 综合训练工具
  - DVWA

## SDL 开发安全流程

- 培训
  - 核心安全培训
- 需求
  - 安全需求分析
  - 质量要求、bug数量要求
  - 安全和隐私风险评估
- 设计
  - 设计需求分析
  - 减小攻击面
  - 威胁建模
- 实施
  - 使用指定工具
  - 弃用不安全函数
  - 静态分析
- 验证
  - 动态分析
  - 模糊测试
  - 威胁模型和攻击面评估
- 发布
  - 事件响应计划
  - 最终安全评析
  - 发布存档
- 响应
