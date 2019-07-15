---
layout: post
title: Linux 内核参数
categories: Linux
description: Linux 内核参数
keywords: linux, disk, hard drive, 硬盘, 磁盘, 存储
---

Linux 系统的内核参数是其高性能的关键。

## 内核参数查看与修改

### /proc/sys 目录

`/proc/sys/` 目录是 Linux 内核在启动后生成的伪目录，存放了当前系统中开启的所有内核参数，修改后的参数值仅在当次运行中生效，系统重启后会回滚历史值，一般用于临时性的验证修改的效果。若需永久性修改，则需要修改 `sysctl.conf`

```shell
cat /proc/sys/net/ipv4/tcp_tw_recycle
echo "0" > /proc/sys/net/ipv4/tcp_tw_recycle 将 net.ipv4.tcp_tw_recycle 的值修改为 0。
```

### sysctl.conf 文件

```shell
/sbin/sysctl -w net.ipv4.tcp_tw_recycle="0"
# 或者 vi /etc/sysctl.conf 修改 /etc/sysctl.conf 文件中的参数。
/sbin/sysctl -p # 使配置生效。
```

注意：调整内核参数后内核处于不稳定状态，请务必重启实例。

## 内核网络参数调整

参考 [Linux 实例常用内核网络参数介绍与常见问题处理](https://help.aliyun.com/knowledge_detail/41334.html)

### Linux 实例 NAT 哈希表满导致丢包

**描述**：Linux 实例出现间歇性丢包，无法连接实例，通过 tracert、mtr 等工具排查，外部网络未见异常。同时，如下图所示，在 `/var/log/message` 系统日志中重复出现大量（table full, dropping packet.）错误信息。

```shell
# 系统日志报错：
Feb  6 16:05:07 i-*** kernel: nf_conntrack: table full, dropping packet.
Feb  6 16:05:07 i-*** kernel: nf_conntrack: table full, dropping packet.
Feb  6 16:05:07 i-*** kernel: nf_conntrack: table full, dropping packet.
Feb  6 16:05:07 i-*** kernel: nf_conntrack: table full, dropping packet.
```

**分析**：`ip_conntrack` 是 Linux 系统内 NAT 的一个跟踪连接条目的模块，使用一个哈希表记录 TCP 协议 `established connection` 记录，当这个哈希表满了的时候，便会导致错误。Linux 系统会开辟一个空间用来维护每一个 TCP 链接，这个空间的大小与 nf_conntrack_buckets、nf_conntrack_max 相关，后者的默认值是前者的 4 倍，而前者在系统启动后无法修改，所以一般都是建议调大 nf_conntrack_max。由于系统维护连接比较消耗内存，请在系统空闲和内存充足的情况下调大 nf_conntrack_max，且根据系统的情况而定。

**解决**：使用如下命令处理

```shell
vi /etc/sysctl.conf
# 修改哈希表项最大值参数
net.netfilter.nf_conntrack_max = 655350
# 修改超时参数，默认情况下 timeout 是 432000（秒）
net.netfilter.nf_conntrack_tcp_timeout_established = 1200
sysctl -p # 使配置生效
```

### Time wait bucket table overflow 报错

**问题**：Linux 实例 `/var/log/message` 日志全是类似 `kernel: TCP: time wait bucket table overflow` 的报错信息，提示 `time wait bucket table` 溢出，如下：

```shell
Feb 18 12:28:38 i-*** kernel: TCP: time wait bucket table overflow
Feb 18 12:28:44 i-*** kernel: printk: 227 messages suppressed.
Feb 18 12:28:44 i-*** kernel: TCP: time wait bucket table overflow
Feb 18 12:28:52 i-*** kernel: printk: 121 messages suppressed.
Feb 18 12:28:52 i-*** kernel: TCP: time wait bucket table overflow
Feb 18 12:28:53 i-*** kernel: printk: 351 messages suppressed.
Feb 18 12:28:53 i-*** kernel: TCP: time wait bucket table overflow
Feb 18 12:28:59 i-*** kernel: printk: 319 messages suppressed.
```

`netstat -ant|grep TIME_WAIT|wc -l`，统计处于 `TIME_WAIT` 状态的 TCP 连接数很多。

**分析**：参数 `net.ipv4.tcp_max_tw_buckets` 可以调整内核中管理 `TIME_WAIT` 状态的数量，当实例中处于 `TIME_WAIT` 及需要转换为 `TIME_WAIT` 状态连接数之和超过了 `net.ipv4.tcp_max_tw_buckets` 参数值时，message 日志中将报错，同时内核关闭超出参数值的部分 TCP 连接。需要根据实际情况适当调高 `net.ipv4.tcp_max_tw_buckets`，同时从业务层面去改进 TCP 连接。

**解决**：执行以下命令

```shell
netstat -anp |grep tcp |wc -l # 统计 TCP 连接数。
vi /etc/sysctl.conf
net.ipv4.tcp_max_tw_buckets=5000
# sysctl -p #使配置生效。
```

### Linux 实例中 FIN_WAIT2 状态的 TCP 链接过多

**问题**：`FIN_WAIT2` 状态的 TCP 链接过多

**原因**：HTTP 服务中，Server 由于某种原因会主动关闭连接，例如 KEEPALIVE 超时的情况下。作为主动关闭连接的 Server 就会进入 `FIN_WAIT2` 状态。TCP/IP 协议栈中，存在半连接的概念，`FIN_WAIT2` 状态不算做超时，如果 Client 不关闭，`FIN_WAIT_2` 状态将保持到系统重启，越来越多的 `FIN_WAIT_2` 状态会致使内核 Crash。建议调小 `net.ipv4.tcp_fin_timeout` 参数，减少这个数值以便加快系统关闭处于 `FIN_WAIT2` 状态的 TCP 连接。

**解决**：执行命令 `vi /etc/sysctl.conf`，修改或加入以下内容：

```shell
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
```

执行命令 `sysctl -p` 使配置生效。

注意：由于 `FIN_WAIT2` 状态的 TCP 连接会进入 `TIME_WAIT` 状态，请同时参阅上述的 time wait bucket table overflow 报错。

### Linux 实例中出现大量 CLOSE_WAIT 状态的 TCP 连接

**问题**：执行命令 `netstat -atn|grep CLOSE_WAIT|wc -l` 发现当前系统中处于 `CLOSE_WAIT` 状态的 TCP 连接非常多。

**原因**：关闭 TCP 连接时，TCP 连接的两端都可以发起关闭连接的请求，若对端发起了关闭连接，但本地没有关闭连接，那么该连接就会处于 CLOSE_WAIT 状态。虽然该连接已经处于半开状态，但是已经无法和对端通信，需要及时的释放掉该链接。建议从业务层面及时判断某个连接是否已经被对端关闭，即在程序逻辑中对连接及时关闭检查。

**解决**：编程语言中对应的读、写函数一般包含了检测 CLOSE_WAIT TCP 连接功能，例如：

- Java 语言
  - 通过 read 方法来判断 I/O 。当 read 方法返回 -1 时则表示已经到达末尾。
  - 通过 close 方法关闭该链接。
- C 语言
  - 检查 read 的返回值。
  - 若等于 0 则可以关闭该连接。
  - 若小于 0 则查看 errno，若不是 AGAIN 则同样可以关闭连接。

### 客户端配置 NAT 后无法访问远端服务器

**问题**：客户端配置 NAT 后无法访问远端服务器，抓包检测发现远端对客户端发送的 SYN 包没有响应。

**原因**：若远端服务器的内核参数 `net.ipv4.tcp_tw_recycle` 和 `net.ipv4.tcp_timestamps` 的值都为 `1`，则远端服务器会检查每一个报文中的时间戳（Timestamp），若 Timestamp 不是递增的关系，远端服务器就不会响应这个报文。配置 NAT 后，远端服务器看到来自不同的客户端的源 IP 相同，但 NAT 前每一台客户端的时间可能会有偏差，报文中的 Timestamp 就不是递增的情况。

**解决**：远端服务器修改参数 `net.ipv4.tcp_tw_recycle` 为 `0`；远端服务器为 PaaS 服务时无法直接修改内核参数，需要在客户端上修改参数 `net.ipv4.tcp_tw_recycle` 和 `net.ipv4.tcp_timestamps` 为 `0`。

### 相关参数整理

|参数|说明|
|:---|:---|
|`net.ipv4.tcp_max_syn_backlog`| 该参数决定了系统中处于 `SYN_RECV` 状态的 TCP 连接数量。`SYN_RECV` 状态指的是当系统收到 SYN 后，作了 SYN+ACK 响应后等待对方回复三次握手阶段中的最后一个 ACK 的阶段。|
|`net.ipv4.tcp_syncookies` |该参数表示是否打开 TCP 同步标签（`SYN_COOKIES`），内核必须开启并编译 CONFIG_SYN_COOKIES，`SYN_COOKIES` 可以防止一个套接字在有过多试图连接到达时引起过载。默认值 0 表示关闭。当该参数被设置为 1 且 `SYN_RECV` 队列满了之后，内核会对 SYN 包的回复做一定的修改，即，在响应的 SYN+ACK 包中，初始的序列号是由源 IP + Port、目的 IP + Port 及时间这五个参数共同计算出一个值组成精心组装的 TCP 包。由于 ACK 包中确认的序列号并不是之前计算出的值，恶意攻击者无法响应或误判，而请求者会根据收到的 SYN+ACK 包做正确的响应。启用 `net.ipv4.tcp_syncookies` 后，会忽略 `net.ipv4.tcp_max_syn_backlog`。|
|`net.ipv4.tcp_synack_retries`|该参数指明了处于 `SYN_RECV` 状态时重传 SYN+ACK 包的次数。|
|`net.ipv4.tcp_abort_on_overflow`|设置该参数为 1 时，当系统在短时间内收到了大量的请求，而相关的应用程序未能处理时，就会发送 Reset 包直接终止这些链接。建议通过优化应用程序的效率来提高处理能力，而不是简单地 Reset。默认值： 0。|
|`net.core.somaxconn`|该参数定义了系统中每一个端口最大的监听队列的长度，是个全局参数。该参数和 `net.ipv4.tcp_max_syn_backlog` 有关联，后者指的是还在三次握手的半连接的上限，该参数指的是处于 `ESTABLISHED` 的数量上限。若您的 ECS 实例业务负载很高，则有必要调高该参数。listen 函数中的参数 backlog 同样是指明监听的端口处于 `ESTABLISHED` 的数量上限，当 backlog 大于 `net.core.somaxconn` 时，以 `net.core.somaxconn` 参数为准。|
|`net.core.netdev_max_backlog`|当内核处理速度比网卡接收速度慢时，这部分多出来的包就会被保存在网卡的接收队列上，而该参数说明了这个队列的数量上限。|