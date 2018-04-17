---
layout: post
title: SpringCloud 系列 00 一个简单的 Spring Cloud Eureka Server 工程
categories: SpringCloud
description: SpringCloud 系列
keywords: Java, spring, springboot, java, boot, SpringCloud, cloud, eureka
---

基本的 Spring Cloud Eureka Server 工程, 单实例.

# 介绍

Home page: [https://github.com/spring-projects/spring-boot](https://github.com/spring-projects/spring-boot "https://github.com/spring-projects/spring-boot")

> Spring Boot makes it easy to create Spring-powered, production-grade applications and services with absolute minimum fuss. It takes an opinionated view of the Spring platform so that new and existing users can quickly get to the bits they need.

> You can use Spring Boot to create stand-alone Java applications that can be started using java -jar or more traditional WAR deployments. We also provide a command line tool that runs spring scripts.

> Our primary goals are:

> Provide a radically faster and widely accessible getting started experience for all Spring development

> Be opinionated out of the box, but get out of the way quickly as requirements start to diverge from the defaults

> Provide a range of non-functional features that are common to large classes of projects (e.g. embedded servers, security, metrics, health checks, externalized configuration)

> Absolutely no code generation and no requirement for XML configuration

以上摘自官网介绍.

按照 Spring Boot 的**规范**进行开发, 效率真是提升了很多.

> only add one `@EnableAutoConfiguration` recommend that you add it to your primary `@Configuration` class.

# 工程结构

## 建议结构

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

> `@SpringBootApplication`  annotation  is  equivalent  to  using  `@Configuration`, `@EnableAutoConfiguration` and `@ComponentScan` with their default attributes.

## 实际结构

```
sb00
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── cn
    │   │       └── xz
    │   │           ├── Application.java
    │   │           └── ctrl
    │   │               └── DemoController.java
    │   └── resources
    │       ├── application.properties-
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
    <artifactId>sb00</artifactId>
    <packaging>jar</packaging>
    <version>0.0.1-SNAPSHOT</version>
    <name>sb00</name>
    <description>simple sb demo</description>
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
		<!-- https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/ -->
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
        <finalName>sb00</finalName>
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

- 继承自 `spring-boot-starter-parent`
- 启用了 `devtools`, `test`

需要注意的是, 单独继承了 `spring-boot-starter-parent` 是不够的, 还必须显式声明 `<dependency>` 某一个父类中的依赖, 才可以在工程中使用具体的功能, 不建议修改依赖模块的版本号.

## application.yml

可以使用  [在线 properties 转 yaml 工具](https://www.bejson.com/devtools/properties2yaml/ "https://www.bejson.com/devtools/properties2yaml/") 实现 `.properties` 与 `.yml` 文件的转换.

```yaml
logging:
  file: ./sb00.log
  level:
    root: info
    cn: debug
server:
  port: 8080
```

- tomcat 端口
- log 文件路径
- 自定义日志级别

## Spring Boot 入口

```java
package cn.xz;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## Controller 类

```java
package cn.xz.ctrl;

import java.util.Random;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class DemoController {

    private static final Logger log = LoggerFactory.getLogger(DemoController.class);

    private String getString() {
        int num = new Random().nextInt(10);
        String str = "";
        for (int i = 0; i < num; i++) {
            str += "<br/>";
        }
        return "ok !" + str + System.currentTimeMillis();
    }

    @RequestMapping("/hello")
    public String index() {
        log.debug("{}", System.nanoTime());
        return "Hello World" + getString();
    }
}
```

# 运行

## 在 IDE 中运行

- `IntelliJ IDEA` 中可以直接识别出 Spring Boot 工程.
- `Eclipse` 建议安装 `Spring Tool Suite`, `Ansi Console`, `DevStyle` 等插件, 然后可以右键 `Run As -> Spring Boot App`.

## maven 运行

使用 `Spring Boot Maven Plugin`. 官网链接: [https://docs.spring.io/spring-boot/docs/current/maven-plugin/usage.html](https://docs.spring.io/spring-boot/docs/current/maven-plugin/usage.html "https://docs.spring.io/spring-boot/docs/current/maven-plugin/usage.html")

> The plugin provides several goals to work with a Spring Boot application:

> - `repackage`: create a jar or war file that is auto-executable. It can replace the regular artifact or can be attached to the build lifecycle with a separate classifier.

> - `run`: run your Spring Boot application with several options to pass parameters to it.
> - `start` and `stop`: integrate your Spring Boot application to the integration-test phase so that the application starts before it.
> - `build-info`: generate a build information that can be used by the Actuator.

```shell
# 直接运行 run,start,stop,repackage,build-info,help
mvn springboot:run
# 打包
mvn clean package
java -jar xx_app.jar
java -jar xx_app.jar --debug
# 也可以打包成 executeable jar 设置为 Linux 服务.
```

具体代码参见 [https://github.com/stdupanda/spring-boot-demo](https://github.com/stdupanda/spring-boot-demo "https://github.com/stdupanda/spring-boot-demo")