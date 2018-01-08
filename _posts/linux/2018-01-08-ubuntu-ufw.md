---
layout: post
title: Ubuntu ufw 配置
categories: Linux
description: Ubuntu ufw 配置
keywords: linux, Ubuntu, ubuntu, ufw
---

整理下 Ubuntu 系统的防火墙 `ufw` 配置.

# 安装

一般是默认已安装好的. 安装命令如下:

`sudo apt-get install ufw`

# 使用 `ufw`

| command                         | desc |
|:--------------------------------|:------------|
| `ufw enable/disable` | 启用 / 禁用 `ufw` 服务 |
| `ufw status` | 查看各端口配置状态 |
| `ufw allow 8080` | 允许外部访问 8080 端口 |
| `ufw delete allow 8080` | 删除上一条命令的操作, 即不允许外部访问 8080 端口 |
| `ufw deny smtp` | 禁止外部访问 smtp 服务 |
| `ufw deny proto tcp from 10.0.0.0/8 to 192.168.0.1 port 22`| 拒绝所有的 TCP 流量从 10.0.0.0/8 到 192.168.0.1 地址的 22 端口 |
| `ufw allow from 192.168.0.0/16` | 允许来自 192.168.0.0/16 子网的请求 |