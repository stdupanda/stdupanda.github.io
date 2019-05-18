---
layout: post
title: TCP/IP 协议整理
categories: web
description: TCP/IP 协议的三次握手、四次挥手具体内容
keywords: web, tcp, ip, tcpip, shake, wave
---

整理 TCP/IP 常见问题。

## TCP Header Format

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

## TCP Connection State

- LISTEN 

represents waiting for a connection request from any remoteTCP and port.

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

## TCP Connection State Diagram

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

## Control Bits

- URG:  Urgent Pointer field significant
- ACK:  Acknowledgment field significant
- PSH:  Push Function
- RST:  Reset the connection
- SYN:  Synchronize sequence numbers
- FIN:  No more data from sender


## Sequence Number

The sequence number of the first data octet in this segment (except when SYN is present). If SYN is present the sequence number is the initial sequence number (ISN) and the first data octet is ISN+1.

## Acknowledgment Number

If the ACK control bit is set this field contains the value of the next sequence number the sender of the segment is expecting to receive.  Once a connection is established this is always sent.


## 标志位 ACK SYN FIN

|标志位|含义|
|:------|:------|
|ACK|确认位，只有 ACK=1 时 ack 才起作用。正常通讯时 ACK 为1（第一次发起请求时因为没有需要确认接收的数据所以 ACk 为0）|
|SYN|同步位，用于在建立连接时同步序号。初始建立连接时并无历史接收数据，所以 ack 也无法设置，此时按照正常的机制就无法运行了，SYN 的作用就是来解决此问题：当接收端收到 SYN=1 的报文时就会直接将 ack 设置为接收到的 seq+1 值，注意这里的值并不是校验后设置的，而是根据 SYN 直接设置的，这样正常的机制就可以运行了，所以 SYN 叫做同部位。需注意的是，SYN 在前两次握手时都为1，因为通信的双方都需要设置一个初始值。|
|FIN|终止位，用于在数据传输完毕后释放连接|


# 三次握手 & 四次挥手

## 流程图

![image](https://github.com/stdupanda/stdupanda.github.io/raw/master/images/posts/tcp_hand_shake_wave.png)

## 注意点

- TCP 传输为全双工模式，client 与 server 是对等的，可以同时传输数据，连接和关闭都需要双方同时进行。
- 三次握手中前两次可以保证服务端可以正确接收并放回请求，后两次可以保证客户端可以正确接收并返回请求，且在三次握手的过程中还使用了 SYN 标志位初始化了双方的 ack 值。四次挥手就是双方分别发送 FIN 标识来关闭连接并让对方确认。
- 若发出一次握手后就不回应第三次握手，服务器端会认为是第二次握手的数据传输失败，就会再次发送第二次握手报文，默认会一直发送 5 次，第五次后还收不到第三次的握手报文则丢弃请求，这就是 DDOS 攻击中的 SYN Flood 攻击。
- UDP 协议是无连接的，和 TCP 相比减少了沟通时间。
- HTTP 协议底层传输默认是基于可靠的 TCP，出于效率 Google 制定了一套基于 UDP 的 QUIC(Quick UDP Internet Connection)协议，但未广泛使用。
