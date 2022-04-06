---
layout: post
title: 网页加载过程分析
categories: web
description: 分析网页加载过程
keywords: web, tcp, ip, tcpip, http, HTTP
---

浏览器采用 HTTP 协议访问网页并获取数据加载展示过程的各个阶段技术分析。

> 从输入 URL 到页面加载完成发生了什么？

## DNS 解析

- Domain Name System，是一种组织成域层次结构的计算机和网络服务命名系统，它用于TCP/IP网络，它所提供的服务是用来将主机名和域名转换为IP地址的工作

### 关于本地缓存

- `%SystemRoot%\system32\drivers\etc\hosts` & `/etc/hosts`，优先级最高
- `ipconfig /flushdns` & `source /etc/hosts` 刷新 dns 缓存

### DNS 解析流程

1. 在浏览器中输入 qq.com 域名，浏览器会先检测浏览器的缓存内是否有此记录，有则直接访问，若没有则操作系统会先检查自己本地的 hosts 文件是否有这个网址映射关系，如果有，就先调用这个IP地址映射，完成域名解析。
2. 如果 hosts 里没有这个域名的映射，则查找本地 DNS 解析器缓存，是否有这个网址映射关系，如果有，直接返回，完成域名解析。
3. 如果 hosts 与本地 DNS 解析器缓存都没有相应的网址映射关系，首先会找 TCP/IP 参数中设置的首选 DNS 服务器，在此我们叫它本地 DNS 服务器，此服务器收到查询时，如果要查询的域名，包含在本地配置区域资源中，则返回解析结果给客户机，完成域名解析，此解析**具有权威性**。
4. 如果要查询的域名，不由本地 DNS 服务器区域解析，但该服务器已缓存了此网址映射关系，则调用这个IP地址映射，完成域名解析，此解析**不具有权威性**。
5. 如果本地 DNS 服务器本地区域文件与缓存解析都失效，则根据本地 DNS 服务器的设置（是否设置转发器）进行查询，如果未用转发模式，本地 DNS 就把请求发至 13 台根 DNS，根 DNS 服务器收到请求后会判断这个域名（.com）是谁来授权管理，并会返回一个负责该顶级域名服务器的一个 IP。本地 DNS 服务器收到 IP 信息后，将会联系负责 `.com` 域的这台服务器。这台负责 `.com` 域的服务器收到请求后，如果自己无法解析，它就会找一个管理 `.com` 域的下一级 DNS 服务器地址(qq.com)给本地 DNS 服务器。当本地 DNS 服务器收到这个地址后，就会找 `qq.com` 域服务器，重复上面的动作，进行查询，直至找到 `www.qq.com` 主机。
6. 如果用的是转发模式，此 DNS 服务器就会把请求转发至上一级 DNS 服务器，由上一级服务器进行解析，上一级服务器如果不能解析，或找根 DNS 或把转请求转至上上级，以此循环。不管是本地 DNS 服务器用是是转发，还是根提示，最后都是把结果返回给本地 DNS 服务器，由此 DNS 服务器再返回给客户机。
7. 可以使用 `nslookup <xx.com>` 或者 `dig +trace <xx.com>` 分析解析过程

![image](/images/posts/dns_analysis.jpg)

### 关于 DNS 加速

- 可使用 cdn 加速，例如可以根据每台机器的负载量，该机器离用户地理位置的距离等返回给不同用户不同 IP 地址。这种过程就是 DNS 负载均衡，又叫做 DNS 重定向。大家耳熟能详的 CDN(Content Delivery Network)就是利用 DNS 的重定向技术，DNS 服务器会返回一个跟用户最接近的点的 IP 地址给用户，CDN 节点的服务器负责响应用户的请求，提供所需的内容。
- DNS 预获取，可以在 html head 部分加上

```html
<meta http-equiv="x-dns-prefetch-control" content="on">
<link rel="dns-prefetch" href="//t11.baidu.com"/>
```

且尽量放在网页前面，推荐放在 `<meta charset="UTF-8">` 后面。

## TCP 连接

- 广播、拆包解包、合并包、路由表、NAT、TCP 传输层路由
- 三次握手四次挥手
- DDOS攻击之 SYN_Flood
- HTTP 报文格式
- HTTP HTTPS 分析比对
- HTTPS 中间人、RSA 秘钥协商、AES 加解密
- 负载均衡 LVS(tcp)/反向代理(7层，Nginx)/f5(硬件)

## 发送请求

连接建立后则可以发起 HTTP 请求

- HTTPS 的 TLS/SSL 握手

## 服务器处理请求并返回 HTTP 响应报文

Apache/CGI/Node.JS/Tomcat/weblogic/jboss/iis 等等接受到后交由后端 php/java/c#、Python 等各种语言服务进行处理，返回数据。

### WebSocket


## 浏览器解析渲染页面

### 常见加速策略

- CDN 访问加速、分布式文件系统的内容分发同步
- 浏览器对同一个域名的并发连接数限制

解决方式可以使用多个域名加大并发量，但是过多的散布会导致 DNS 解析上付出额外的代价，所以一般也是控制在 2-4 之间。**这里常见的一个性能小坑是没有机制去确保 URL 的哈希一致性（即同一个静态资源应该被哈希到同一个域名下），而导致资源被多次下载。**

- 使用二级域名作为cookie-free domains
- css sprite（css内同时加载资源太多导致 js 等资源下载延迟影响界面加载）
- gzip  js/css 的 minify、compress
- `304 Not Modified` VS `200 OK (from cache)`

Entity Tag、Last Modified、If Modified Since，可以参考此文： [阿里云存储如何让浏览器始终以200 (from cache)缓存图片](https://www.zhihu.com/question/28725359 "阿里云存储如何让浏览器始终以200 (from cache)缓存图片")以及此文：[配置错误产生的差距：200 OK (FROM CACHE) 与 304 NOT MODIFIED](http://div.io/topic/854 "为什么有的缓存是 200 OK (from cache)，有的缓存是 304 Not Modified 呢？很简单，看运维是否移除了 Entity Tag。移除了，就总是 200 OK (from cache)。没有移除，就两者会交替出现。")以及 RFC 协议内容：[Hypertext Transfer Protocol (HTTP/1.1): Caching](https://tools.ietf.org/html/rfc7234 "Hypertext Transfer Protocol (HTTP/1.1): Caching")，附教程一个：[Caching Tutorial](https://www.mnot.net/cache_docs/ "Caching Tutorial for Web Authors and Webmasters")

- 按需加载，下拉屏幕自动获取

### 页面访问

- PV、UV、大数据用户分析
- 具体系统后台业务

## 连接结束

Webkit 内核解析 V8 引擎
