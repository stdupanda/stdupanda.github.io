---
layout: post
title: springboot 001
categories: SpringBoot
description: socket 相关整理
keywords: Java, socket, 套接字, nio, NIO, java
---

SpringBoot 个人整理

java -jar xx.jar --debug

only add one @EnableAutoConfiguration.

recommend that you add it to your primary @Configuration class.

@SpringBootApplication  annotation  is  equivalent  to  using  @Configuration,@EnableAutoConfiguration and @ComponentScan with their default attributes

> Tip

> We recommend that you follow Java’s recommended package naming conventions and use a reversed domain name (for example, com.example.project).

```
com
    +- example
        +- myproject
        +- Application.java
        |
        +- domain
        |    +- Customer.java
        |    +- CustomerRepository.java
        |
        +- service
        |    +- CustomerService.java
        |
        +- web
            +- CustomerController.java
```
```java
// Application.java
package com.example.myproject;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableAutoConfiguration
@ComponentScan
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

[从 1.4 开始 @ConfigurationProperties 去掉了 `locations` 属性 ](https://github.com/spring-projects/spring-boot/issues/6726 "https://github.com/spring-projects/spring-boot/issues/6726")