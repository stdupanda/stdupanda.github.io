---
layout: post
title: SpringBoot 系列 03 MQ
categories: SpringBoot
description: SpringBoot 系列
keywords: Java, spring, springboot, java, boot, mq, rabbitmq
---

一个集成了 MQ 的 Spring Boot 工程.

# 工程结构

## 实际结构

```
sb03
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── cn
    │   │       └── xz
    │   │           ├── Application.java
    │   │           ├── config
    │   │           │   └── RabbitMQConfig.java
    │   │           ├── ctrl
    │   │           │   └── DemoController.java
    │   │           ├── mq
    │   │           │   └── other
    │   │           │       └── MQReceiver.java
    │   │           └── service
    │   │               └── MQService.java
    │   └── resources
    │       └── application.yml
    └── test
        └── java
            └── cn
                └── xz
                    └── ApplicationTests.java
```
# 代码实例

## maven 配置

打包方式为 `jar`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.xz</groupId>
    <artifactId>sb03</artifactId>
    <packaging>jar</packaging>
    <version>0.0.1-SNAPSHOT</version>
    <name>sb03</name>
    <description>sb with rabbitmq</description>
    <url>http://maven.apache.org</url>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.9.RELEASE</version>
        <relativePath /><!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency><!-- will be disabled where using "java -jar ..." -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <finalName>sb03</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <executable>true</executable>
                    <fork>true</fork>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                    <archive>
                        <!-- rm MavenDescriptor -->
                        <addMavenDescriptor>false</addMavenDescriptor>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## application.yml

可以使用  [在线 properties 转 yaml 工具](https://www.bejson.com/devtools/properties2yaml/ "https://www.bejson.com/devtools/properties2yaml/") 实现 `.properties` 与 `.yml` 文件的转换.

```yaml
logging:
  file: ./sb03.log
  level:
    root: info
    cn: debug
spring:
  rabbitmq:
    host: 172.16.100.156
    port: 5672
    username: rabbitmq
    password: rabbitmq
server:
  port: 8083
```

- tomcat 端口
- log 文件路径
- 自定义日志级别
- mq 配置

## Spring Boot 入口

```java
package cn.xz;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## MQ 配置

```java
package cn.xz.config;

import org.springframework.amqp.core.Queue;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitMQConfig {
    public static final String MQ_QUEUE_NAME = "test_queue_666";

    @Bean
    public Queue queue() {
        return new Queue(MQ_QUEUE_NAME, true, false, false);
    }
}
```
## MQ 接收

```java
package cn.xz.mq.other;

import java.util.List;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Service;

import cn.xz.config.RabbitMQConfig;

@Service
@RabbitListener(queues = RabbitMQConfig.MQ_QUEUE_NAME)
public class MQReceiver {

    private static final Logger log = LoggerFactory.getLogger(MQReceiver.class);

    @RabbitHandler
    public void process(String message) {// 只能接收 String 类型的消息
        log.debug("recv: {}", message);
    }

    @RabbitHandler
    public void process(List<String> message) {// 必须单独声明对应的接收类型，否则无法接收
        log.debug("recv: {}", message);
    }
}
```

## MQ 发送

```java
package cn.xz.service;

import java.util.Arrays;
import java.util.List;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.AmqpTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import cn.xz.config.RabbitMQConfig;

@Service
public class MQService {

    private static final Logger log = LoggerFactory.getLogger(MQService.class);

    @Autowired
    private AmqpTemplate amqpTemplate;

    public void sendQueue(String exchange_key, String queue_key, Object object) {
        amqpTemplate.convertAndSend(exchange_key, queue_key, object);
    }

    public void sendQueue(String queue_key, Object object) {
        amqpTemplate.convertAndSend(queue_key, object);
    }

    public void test() {
        log.debug("mq test begin.");
        String msg = "hello" + System.currentTimeMillis();
        sendQueue(RabbitMQConfig.MQ_QUEUE_NAME, msg);
        log.debug("sent str: {}", msg);
        char[] charArray = Long.toString(System.currentTimeMillis()).toCharArray();
        String[] strArr = new String[charArray.length];
        for (int i = 0; i < charArray.length; i++) {
            strArr[i] = String.valueOf(charArray[i]);
        }
        List<String> asList = Arrays.asList(strArr);
        log.debug("sent obj: {}", asList);
        sendQueue(RabbitMQConfig.MQ_QUEUE_NAME, asList);
        log.debug("mq test end  .");
    }
}
```

具体代码参见 [https://github.com/stdupanda/spring-boot-demo](https://github.com/stdupanda/spring-boot-demo "https://github.com/stdupanda/spring-boot-demo")