---
layout: post
title: SpringBoot 系列 10 Spring Boot Admin 集成(server端，未注册到eureka)
categories: SpringBoot
description: SpringBoot 系列
keywords: Java, spring, springboot, java, boot, mybatis
---

一个集成了 Spring Boot Admin server 端的 Spring Boot 工程.

参考文档 http://codecentric.github.io/spring-boot-admin/1.5.7

# Actuator

|HTTP方法	|路径	|描述|	鉴权|
|:--|:--|:--|
|GET|	/autoconfig	|查看自动配置的使用情况|	true|
|GET|	/configprops	|查看配置属性，包括默认配置	|true|
|GET|	/beans	|查看bean及其关系列表	|true|
|GET|	/dump	|打印线程栈	|true|
|GET|	/env	|查看所有环境变量	|true|
|GET|	/env/{name}	|查看具体变量值	|true|
|GET|	/health	|查看应用健康指标|	false|
|GET|	/info	|查看应用信息	|false|
|GET|	/mappings	|查看所有url映射|	true|
|GET|	/metrics	|查看应用基本指标|	true|
|GET|	/metrics/{name}	|查看具体指标|	true|
|POST| 	/shutdown	|关闭应用	|true|
|GET|   /trace	|查看基本追踪信息|	true|
|
# 代码实例

## maven 配置

打包方式为 `jar`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>cn.xz</groupId>
    <artifactId>sb10</artifactId>
    <packaging>jar</packaging>
    <version>0.0.1-SNAPSHOT</version>
    <name>sb10</name>
    <description>sba</description>
    <url>http://maven.apache.org</url>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.12.RELEASE</version>
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
            <groupId>de.codecentric</groupId>
            <artifactId>spring-boot-admin-starter-server</artifactId>
            <version>1.5.7</version>
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
        <finalName>sb10</finalName>
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
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>build-info</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

## application.yml

可以使用  [在线 properties 转 yaml 工具](https://www.bejson.com/devtools/properties2yaml/ "https://www.bejson.com/devtools/properties2yaml/") 实现 `.properties` 与 `.yml` 文件的转换.

```yaml
logging:
  file: ./log_sb10.log
  level:
    root: info
    cn: debug
spring:
  application:
    name: sb10_SBA
server:
  port: 8010
```

## Spring Boot 集成

接收消息：

```java
package cn.xz;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import de.codecentric.boot.admin.config.EnableAdminServer;

@SpringBootApplication
@EnableAdminServer
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

```

具体代码参见 [https://github.com/stdupanda/spring-boot-demo](https://github.com/stdupanda/spring-boot-demo "https://github.com/stdupanda/spring-boot-demo")
