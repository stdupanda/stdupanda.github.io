---
layout: post
title: centos 最小化安装后各种配置
categories: Linux
description: centos 最小化安装后各种配置
keywords: linux
---

本文记录最小化安装 centos 后的各种问题解决。

## 网络连接异常

> 建议直接使用 `setup` 命令去修改配置。
>
> `yum install setuptool ntsysv system-config-network-tui iptables`

注意：新版本的 `setup` 内不再包括网卡配置，可以使用命令：**`nmtui`** (即： network manager tool ui) 来进行配置。

### 修改网络配置

修改 `/etc/sysconfig/network-scripts/` 的，网卡名字是 `ifcfg-xxx`。

```shell
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
# 配置自动启动
ONBOOT=yes
```

以上是 `dhcp` 的方式。若需指定静态地址则：

```shell
IPADDR=192.168.1.x
NETMASK=255.255.255.0
NETWORK=192.168.1.0
GATEWAY=
BROADCAST=
```

### 配置 DNS

`vim /etc/resolv.conf`

```bash
nameserver 202.96.134.133
nameserver 8.8.8.8
```

### 重启网络服务

```bash
/etc/init.d/network restart
service network restart
systemctl restart network
systemctl status network -l
```

重启服务再 `ip addr`查看下配置是否已生效。
