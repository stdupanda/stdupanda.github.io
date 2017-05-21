---
layout: post
title: SpringMVC返回 json 格式对象时在 IE11 异常
categories: SpringMVC
description: SpringMVC返回 json 格式对象时在 IE11 异常
keywords: SpringMVC, java, JavaWeb, Spring, SSM
---

今天准备整理下 `Spring` 下的单元测试方案。

在搭建 `Spring MVC` 环境出现一个问题，IE11 中处理返回的 json 格式时浏览器直接提示另存为，于是尝试解决下此问题。

# 问题场景

## 前提

控制器层使用的注解进行配置，在 `Spring` 的配置文件中采用了简单的 `<mvc:annotation-driven />` 进行了配置

## 结果

IE11 中处理返回的 json 数据时直接提示另存为，在其他浏览器未见。

# 问题分析

需要查看注解 `<mvc:annotation-driven />` 对应源码 `org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter`

# 解决办法

配置文件中禁用 `<mvc:annotation-driven />`，改为如下配置：

```xml
<bean
    class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping" />
<bean
    class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="messageConverters">
        <list>
            <bean
                class="org.springframework.http.converter.StringHttpMessageConverter">
                <property name="supportedMediaTypes">
                    <list>
                        <bean class="org.springframework.http.MediaType">
                            <constructor-arg index="0" value="text" />
                            <constructor-arg index="1" value="plain" />
                            <constructor-arg index="2" value="UTF-8" />
                        </bean>
                        <value>text/html;charset=UTF-8</value>
                    </list>
                </property>
            </bean>

            <!-- 解析返回json -->
            <bean
                class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                <property name="supportedMediaTypes">
                    <list>
                        <value>text/html;charset=UTF-8</value>
                        <value>text/plain;charset=UTF-8</value>
                        <value>application/json;charset=UTF-8</value>
                    </list>
                </property>
            </bean>
        </list>
    </property>
</bean>
```

若只配置第一个 `RequestMappingHandlerMapping` 则会提示 `no adapter for handler xxx:: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler` 错误。
