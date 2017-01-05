---
layout: post
title: centos7 网卡配置
categories: linux
description: centos7 网卡配置
keywords: linux
---

今天安装了 cengos7 打算安装 mysql 测试下，之前都选择图形化界面，今天选择了最小化安装，开机没有图形化界面设置网络的入口，只能修改配置文件了。

# 找到配置文件

一般是是位于 `/etc/sysconfig/network-scripts/` 路径，网卡名字是 `ifcfg-ethxxxxxx` 或者 `ifcfg-enp0s3xxxx`，使用 vim 打开配置文件进行修改。

```
TYPE=Ethernet
BOOTPROTO=dhcp
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3
UUID=a51b993f-89b1-40ef-be1b-0e0f41576fad
DEVICE=enp0s3
ONBOOT=yes
```

配置 DNS

`vim /etc/resolv.conf `

```
nameserver 202.96.134.133
nameserver 8.8.8.8
```

# 重启网卡

`/etc/init.d/network restart`

`service network restart`

------
