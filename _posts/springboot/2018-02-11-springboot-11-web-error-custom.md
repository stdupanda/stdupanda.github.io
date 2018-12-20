---
layout: post
title: SpringBoot 系列 11 Spring Boot web 错误界面自定义
categories: SpringBoot
description: SpringBoot 系列
keywords: Java, spring, springboot, java, boot, mybatis
---

自定义 400 等错误界面

参考官方文档 27.1 -> Error Handling 部分。

看到网上一水的写代码搞继承实现，感觉歪果仁肯定不会这么费劲去搞这一个小事。果断看文档。

按照文档，直接在 `src/main/resources/` 下新建 `public/error/`，可以放置一个 `4XX.html` 代表所有的客户端错误，也可以放置一个 `404.html` 代表 404 错误。

注意：优先使用已存在的 `404.html`, 否则匹配 `4xx.html`。


具体代码参见 [https://github.com/stdupanda/spring-boot-demo](https://github.com/stdupanda/spring-boot-demo "https://github.com/stdupanda/spring-boot-demo")
