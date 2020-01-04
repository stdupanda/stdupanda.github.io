---
layout: post
title: TCP/IP 协议整理
categories: web
description: TCP/IP 协议的三次握手、四次挥手具体内容
keywords: web, tcp, ip, tcpip, shake, wave
---

整理 TCP/IP 常见问题。

## TCP 报文结构

### TCP Header Format

详细可参见  [RFC793](https://tools.ietf.org/html/rfc793 "RFC793")、[rfc-editor 上的 RFC793](https://www.rfc-editor.org/rfc/rfc793.txt)。

```
0                   1                   2                   3
0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### TCP Connection State

- LISTEN 

represents waiting for a connection request from any remote TCP and port.

- SYN-SENT

represents waiting for a matching connection request after having sent a connection request.

- SYN-RECEIVED 

represents waiting for a confirming connection request acknowledgment after having both received and sent a connection request.

- ESTABLISHED

represents an open connection, data received can be delivered to the user.  The normal state for the data transfer phase of the connection.

- FIN-WAIT-1

represents waiting for a connection termination request from the remote TCP, or an acknowledgment of the connection termination request previously sent.

- FIN-WAIT-2

represents waiting for a connection termination request from the remote TCP.

- CLOSE-WAIT

represents waiting for a connection termination request from the local user.

- CLOSING

represents waiting for a connection termination request acknowledgment from the remote TCP.

- LAST-ACK

represents waiting for an acknowledgment of the connection termination request previously sent to the remote TCP (which includes an acknowledgment of its connection termination request).

- TIME-WAIT

represents waiting for enough time to pass to be sure the remote TCP received the acknowledgment of its connection termination request.

- CLOSED

represents no connection state at all.

### TCP Connection State Diagram

```
                                    
                              +---------+ ---------\      active OPEN  
                              |  CLOSED |            \    -----------  
                              +---------+<---------\   \   create TCB  
                                |     ^              \   \  snd SYN    
                   passive OPEN |     |   CLOSE        \   \           
                   ------------ |     | ----------       \   \         
                    create TCB  |     | delete TCB         \   \       
                                V     |                      \   \     
                              +---------+            CLOSE    |    \   
                              |  LISTEN |          ---------- |     |  
                              +---------+          delete TCB |     |  
                   rcv SYN      |     |     SEND              |     |  
                  -----------   |     |    -------            |     V  
 +---------+      snd SYN,ACK  /       \   snd SYN          +---------+
 |         |<-----------------           ------------------>|         |
 |   SYN   |                    rcv SYN                     |   SYN   |
 |   RCVD  |<-----------------------------------------------|   SENT  |
 |         |                    snd ACK                     |         |
 |         |------------------           -------------------|         |
 +---------+   rcv ACK of SYN  \       /  rcv SYN,ACK       +---------+
   |           --------------   |     |   -----------                  
   |                  x         |     |     snd ACK                    
   |                            V     V                                
   |  CLOSE                   +---------+                              
   | -------                  |  ESTAB  |                              
   | snd FIN                  +---------+                              
   |                   CLOSE    |     |    rcv FIN                     
   V                  -------   |     |    -------                     
 +---------+          snd FIN  /       \   snd ACK          +---------+
 |  FIN    |<-----------------           ------------------>|  CLOSE  |
 | WAIT-1  |------------------                              |   WAIT  |
 +---------+          rcv FIN  \                            +---------+
   | rcv ACK of FIN   -------   |                            CLOSE  |  
   | --------------   snd ACK   |                           ------- |  
   V        x                   V                           snd FIN V  
 +---------+                  +---------+                   +---------+
 |FINWAIT-2|                  | CLOSING |                   | LAST-ACK|
 +---------+                  +---------+                   +---------+
   |                rcv ACK of FIN |                 rcv ACK of FIN |  
   |  rcv FIN       -------------- |    Timeout=2MSL -------------- |  
   |  -------              x       V    ------------        x       V  
    \ snd ACK                 +---------+delete TCB         +---------+
     ------------------------>|TIME WAIT|------------------>| CLOSED  |
                              +---------+                   +---------+
```

图片版请<a href="/images/posts/tcp_connection_state_diagram.png" target="_blank">点击这里查看</a>

### Control Bits

- URG:  Urgent Pointer field significant
- ACK:  Acknowledgment field significant
- PSH:  Push Function
- RST:  Reset the connection
- SYN:  Synchronize sequence numbers
- FIN:  No more data from sender

|标志位|名称|含义|
|:------|:------|:------|
|ACK|确认位|只有 ACK = 1 时 ack 才起作用。正常通讯时 ACK 为 1（第一次发起请求时因为没有需要确认接收的数据所以 ACk 为 0）|
|SYN|同步位|用于在建立连接时同步序号。<br/>初始建立连接时并无历史接收数据，所以 ack 也无法设置。当接收端收到 SYN = 1 的报文时就会直接将 ack 设置为接收到的 seq + 1，注意这里的值并不是校验后设置的，而是根据 SYN 直接设置的，所以 SYN 叫做同步位。<br/>需注意的是，SYN 在前两次握手时都为 1，因为通信的双方都需要设置一个初始值。|
|FIN|终止位|用于在数据传输完毕后释放连接|

### Sequence Number

The sequence number of the first data octet in this segment (except when SYN is present). If SYN is present the sequence number is the initial sequence number (ISN) and the first data octet is ISN+1.

### Acknowledgment Number

If the ACK control bit is set this field contains the value of the next sequence number the sender of the segment is expecting to receive.  Once a connection is established this is always sent.

## 连接建立和释放

先说一个概念，MSL（maximum segment lifetime）报文最大生存时间，超过这个时间就被丢弃；在一般的实现中取 30 秒，有些按照 RFC 793 规范采用 2 分钟。

### 流程图

![image](/images/posts/tcp_handshake_wave.png)

### 详细流程

关于可靠传输

- 可靠传输的工作原理
  - 停止等待协议
  - 连续 ARQ 协议
- 可靠传输的实现
  - 以字节为单位的滑动窗口
    - 未收到确认时，可以先把窗口内数据都发送出去
    - 已发送的数据在确认前均需保留以备重发
    - 发送窗口不可以超过接收窗口
    - 接收方采用累积确认方式（只需对按序到达的最后一个分组发送确认，即确认了所有分组）
  - 超时重传
  - 选择确认

关于流控：基于滑动窗口实现流量控制

- 拥塞控制
  - 慢开始，设置较小的发送窗口
  - 拥塞避免，逐渐增大发送窗口
  - 快重传，让发送方尽早知道报文丢失
  - 快恢复，重传丢失的报文

建立连接时的三次握手（三报文握手）

- 三次握手
  - RFC793 中对此简单说明：引入三次握手的主要原因是为了避免过时的重复连接在再次建连时造成的混乱
  - 前两次可以保证 B 可以正常收发消息，后两次可以保证 A 可以正常收发消息
  - 防止 B 接收到已失效的 A 的连接请求报文后建立连接，浪费资源
  - 指定初始化序列号
  - 若 A 故意不进行第三次通讯，B 会一直发送最多 5 次，打满半连接队列浪费自身资源。这就是 DDOS 攻击中的 SYN Flood 攻击。
  - 第三次握手就可以携带业务数据了

断开连接时的四次挥手

- 四次挥手
  - B 收到 A 的断开请求后响应一个确认报文；此时 A 已没有要发送的数据了；
  - A 经过 2MSL 后才进入到 CLOSED 状态；
    - 保证 A 发送的最后一个 ACK 报文能到达 B，否则 B 无法正常关闭
    - 保证 B 能来得及把所有数据发给 A
  - B 端有保活机制，2h（内核参数默认值） 未收到 A 数据后，会每隔 75s 发送最多 10 次检测报文后关闭连接

《UNIX 网络编程》 关于 2MSL 后进入 CLOSED状态的描述：

> Since the duration of the TIME_WAIT state is twice the MSL, this allows MSL seconds for packet in one direction to be lost, and another MSL seconds for the reply to be lost. By enforcing this rule, we are guaranteed that when we successfully establish a TCP connecton, all old duplicates from previous incarnations of the connection have expired in the network.

联系四次挥手过程，服务端大量 `time_wait` 状态的端口问题如何处理

- server 端 socket 编程时设置 `SO_REUSEADDR` 实现重用端口
- socket 编程时判断连接状态， read() 为 -1 的情况、try...catch 保证 close()
- 进行内核调整，参考另一篇文章： [Linux 内核参数](/_posts/linux/2018-07-01-linux-kernel-param.md)

简单整理如下：

```shell
net.ipv4.tcp_keepalive_time = 1200 
#表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为20分钟。

net.ipv4.ip_local_port_range = 1024 65000 
#表示用于向外连接的端口范围。缺省情况下很小：32768到61000，改为1024到65000。

net.ipv4.tcp_max_syn_backlog = 8192 
#表示SYN队列的长度，默认为1024，加大队列长度为8192，可以容纳更多等待连接的网络连接数。

net.ipv4.tcp_max_tw_buckets = 5000 
#表示系统同时保持TIME_WAIT套接字的最大数量，如果超过这个数字，TIME_WAIT套接字将立刻被清除并打印警告信息。
#默认为180000，改为5000。对于Apache、Nginx等服务器，上几行的参数可以很好地减少TIME_WAIT套接字数量，但是对于 Squid，效果却不大。此项参数可以控制TIME_WAIT套接字的最大数量，避免Squid服务器被大量的TIME_WAIT套接字拖死。
```

关于 TCP 报文的粘包拆包

- 粘包/拆包问题
  - 消息定长
  - 固定分隔符
  - 报头中设计报文体长度字段
  - netty 的解决方式
    - `LengthFieldBasedFrameDecoder`
    - `FixedLengthFrameDecoder`
    - `LineBasedFrameDecoder`/`DelimiterBaseFrameDecoder`
- UDP 协议是无连接的，和 TCP 相比减少了沟通时间。
- HTTP 协议底层传输默认是基于可靠的 TCP，出于效率 Google 制定了一套基于 UDP 的 QUIC(Quick UDP Internet Connection)协议，但未广泛使用。
