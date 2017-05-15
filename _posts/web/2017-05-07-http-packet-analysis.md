---
layout: post
title: HTTP 报文格式分析
categories: web
description: 分析 HTTP 报文的结构
keywords: web, tcp, ip, tcpip, http, HTTP
---

HTTP 是应用层协议，在 TCP/IP 协议收到数据后需要按照 HTTP 协议进行解析。此文整理 HTTP 协议报文结构等。

# 报文分类

HTTP 报文分为请求报文 request message 和 响应报文 response message。

# 报文结构

请求报文和响应报文都包括三部分： `首行、头部、主体`

## 首行

请求报文和响应报文的首行差异如下：

### 请求报文首行

格式：`method/URL/HTTP version`

- `method` 主要有：`GET POST HEAD PUT DELETE OPTIONS` 等等
- `URL` 请求的地址
- `HTTP vesion` HTTP 协议版本，当前大部分是 1.1 版

请求报文首行举例：`GET/HTTP/1.1`

### 响应报文首行

格式：`HTTP version/status code/info`

- `HTTP version` HTTP 协议版本
- `status code` 响应状态码
- `info` 简短原因，可无

#### 状态码分类

- `1xx` 信息性
- `2xx` 成功，如 `200` 表示成功
- `3xx` 重定向，如 `301` 表示重定向
- `4xx` 客户端错误 如 `404` 表示找不到请求的资源
- `5xx` 服务器端错误，如 `500`表示服务端内部错误

较常见状态码：200, 204, 301, 302, 304, 400, 401, 403, 404, 422, 500

|code|meaning|
|--|---|
| 100 | Continue 初始的请求已经接受，客户应当继续发送请求的其余部分|
| 101 | Switching Protocols 服务器将遵从客户的请求转换到另外一种协议
| 200 | OK 一切正常，对GET和POST请求的应答文档跟在后面。
| 201 | Created 服务器已经创建了文档，Location头给出了它的URL。
| 202 | Accepted 已经接受请求，但处理尚未完成。
| 203 | Non-Authoritative Information 文档已经正常地返回，但一些应答头可能不正确，因为使用的是文档的拷贝
| 204 | No Content 没有新文档，浏览器应该继续显示原来的文档。如果用户定期地刷新页面，而Servlet可以确定用户文档足够新，这个状态代码是很有用的
| 205 | Reset Content 没有新的内容，但浏览器应该重置它所显示的内容。用来强制浏览器清除表单输入内容
| 206 | Partial Content 客户发送了一个带有Range头的GET请求，服务器完成了它
| 300 | Multiple Choices 客户请求的文档可以在多个位置找到，这些位置已经在返回的文档内列出。如果服务器要提出优先选择，则应该在Location应答头指明。
| 301 | Moved Permanently 客户请求的文档在其他地方，新的URL在Location头中给出，浏览器应该自动地访问新的URL。
| 302 | Found 类似于301，但新的URL应该被视为临时性的替代，而不是永久性的。
| 303 | See Other 类似于301/302，不同之处在于，如果原来的请求是POST，Location头指定的重定向目标文档应该通过GET提取
| 304 | Not Modified 客户端有缓冲的文档并发出了一个条件性的请求（一般是提供If-Modified-Since头表示客户只想比指定日期更新的文档）。服务器告诉客户，原来缓冲的文档还可以继续使用。
| 305 | Use Proxy 客户请求的文档应该通过Location头所指明的代理服务器提取
| 307 | Temporary Redirect 和302（Found）相同。许多浏览器会错误地响应302应答进行重定向，即使原来的请求是POST，即使它实际上只能在POST请求的应答是303时才能重定向。由于这个原因，HTTP 1.1新增了307，以便更加清除地区分几个状态代码：当出现303应答时，浏览器可以跟随重定向的GET和POST请求；如果是307应答，则浏览器只能跟随对GET请求的重定向。
| 400 | Bad Request 请求出现语法错误。
| 401 | Unauthorized 客户试图未经授权访问受密码保护的页面。应答中会包含一个WWW-Authenticate头，浏览器据此显示用户名字/密码对话框，然后在填写合适的Authorization头后再次发出请求。
| 403 | Forbidden 资源不可用。
| 404 | Not Found 无法找到指定位置的资源
| 405 | Method Not Allowed 请求方法（GET、POST、HEAD、Delete、PUT、TRACE等）对指定的资源不适用。
| 406 | Not Acceptable 指定的资源已经找到，但它的MIME类型和客户在Accpet头中所指定的不兼容
| 407 | Proxy Authentication Required 类似于401，表示客户必须先经过代理服务器的授权。
| 408 | Request Timeout 在服务器许可的等待时间内，客户一直没有发出任何请求。客户可以在以后重复同一请求。
| 409 | Conflict 通常和PUT请求有关。由于请求和资源的当前状态相冲突，因此请求不能成功。
| 410 | Gone 所请求的文档已经不再可用，而且服务器不知道应该重定向到哪一个地址。它和404的不同在于，返回407表示文档永久地离开了指定的位置，而404表示由于未知的原因文档不可用。
| 411 | Length Required 服务器不能处理请求，除非客户发送一个Content-Length头
| 412 | Precondition Failed 请求头中指定的一些前提条件失败
| 413 | Request Entity Too Large 目标文档的大小超过服务器当前愿意处理的大小。如果服务器认为自己能够稍后再处理该请求，则应该提供一个Retry-After头
| 414 | Request URI Too Long URI太长
| 416 | Requested Range Not Satisfiable 服务器不能满足客户在请求中指定的Range头
| 500 | Internal Server Error 服务器遇到了意料不到的情况，不能完成客户的请求
| 501 | Not Implemented 服务器不支持实现请求所需要的功能。例如，客户发出了一个服务器不支持的PUT请求
| 502 | Bad Gateway 服务器作为网关或者代理时，为了完成请求访问下一个服务器，但该服务器返回了非法的应答
| 503 | Service Unavailable 服务器由于维护或者负载过重未能应答。例如，Servlet可能在数据库连接池已满的情况下返回503。服务器返回503时可以提供一个Retry-After头
| 504 | Gateway Timeout 由作为代理或网关的服务器使用，表示不能及时地从远程服务器获得应答
| 505 | HTTP Version Not Supported 服务器不支持请求中所指明的HTTP版本

注意301和302区别

响应报文首行举例：`HTTP/1.1 200 OK`

## 头部 headers

保存一些键值对格式的属性，用冒号分隔，如：

```
Host: www.baidu.com
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.81 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Encoding: gzip, deflate, sdch, br
Accept-Language: zh-CN,zh;q=0.8
Cookie: BAIDUID=B2B5102089D00E736D0363BA9CF5870C:FG=1; BIDUPSID=B2B5102089D00E736D0363BA9CF5870C; PSTM=1486961211; Hm_lvt_12423ecbc0e2ca965d84259063d35238=1492074218; BD_UPN=12314353; MCITY=-131%3A
```

### `Content-Type`

`Content-Type` 表示后面的文档属于什么MIME类型。格式为： `Content-Type: [type]/[subtype]`。

常见格式包括：

| `Content-Type` | 文件扩展名 |
|:---------------|:----------|
|text/html|.html|
|text/xml|.xml|
|application/json|.json(JavaScript Object Notation)|
|application/octet-stream|.二进制流|

```java
response.setContentType("application/octet-stream");//设置文件类型
response.setHeader("Content-Disposition", "attachment;filename="+filename);//设置文件名
response.setHeader("Content-Length", fileLength);//设置下载文件大小
```

## 主体 body

请求报文的主体主要保存 `POST`类型的参数，响应报文的主体主要保存响应内容数据。
