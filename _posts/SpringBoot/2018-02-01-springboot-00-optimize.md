---
layout: post
title: SpringBoot 系列 00 关于优化
categories: SpringBoot
description: SpringBoot 系列
keywords: Java, spring, springboot, java, boot
---

逐渐补充日常使用中发现的需要优化的点, 整理记载.

# 启动入口

使用 `-Ddebug` 选项启动项目, 然后分析 `Positive matches` 中的选项, 然后修改启动类的注解为如下:

```java
@Configuration
@Import({
        DispatcherServletAutoConfiguration.class,
        EmbeddedServletContainerAutoConfiguration.class,
        ErrorMvcAutoConfiguration.class,
        HttpEncodingAutoConfiguration.class,
        HttpMessageConvertersAutoConfiguration.class,
        JacksonAutoConfiguration.class,
        JmxAutoConfiguration.class,
        MultipartAutoConfiguration.class,
        ServerPropertiesAutoConfiguration.class,
        PropertyPlaceholderAutoConfiguration.class,
        ThymeleafAutoConfiguration.class,
        WebMvcAutoConfiguration.class,
        WebSocketAutoConfiguration.class,
})
public class SampleWebUiApplication {
```

`@SpringBootApplication` 需要扫描判断各个模块, 因此会增加启动时间. 改为显示指定可以加速.

参考 https://segmentfault.com/a/1190000004252885 https://yq.aliyun.com/articles/2360
