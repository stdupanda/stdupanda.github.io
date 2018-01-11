---
layout: post
title: SpringBoot 系列 01 SSM-Redis-schedule-properties
categories: SpringBoot
description: SpringBoot 系列
keywords: Java, spring, springboot, java, boot
---

一个集成了 SSM-Redis-schedule-properties 的 Spring Boot 工程.

# 工程结构

## 实际结构

```
sb01/
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── cn
    │   │       └── xz
    │   │           ├── Application.java
    │   │           ├── cache
    │   │           │   └── RedisBaseDao.java
    │   │           ├── context
    │   │           │   └── SpringContextHelper.java
    │   │           ├── ctrl
    │   │           │   └── DemoController.java
    │   │           ├── dao
    │   │           │   ├── CloudUserInfoMapper.java
    │   │           │   └── CustomMapper.java
    │   │           ├── entity
    │   │           │   ├── CloudUserInfo.java
    │   │           │   └── CloudUserInfoExample.java
    │   │           ├── MyProperties.java
    │   │           ├── schedule
    │   │           │   └── MySchedule.java
    │   │           └── WebConfig.java
    │   └── resources
    │       ├── application.yml
    │       ├── mapper
    │       │   └── CloudUserInfoMapper.xml
    │       ├── my.properties
    │       └── mybatis-config.xml
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
    <artifactId>sb01</artifactId>
    <packaging>jar</packaging>
    <version>0.0.1-SNAPSHOT</version>
    <name>sb01</name>
    <description>properties, mybatis page, redis,schedule, addMavenDescriptor</description>
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
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.1</version>
        </dependency>
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper</artifactId>
            <version>4.1.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <finalName>sb01</finalName>
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
  file: ./sb01.log
  level:
    root: info
    cn: debug
    org.mybatis: debug
server:
  port: 8081
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://192.168.1.241:3306/hce_guiyang?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&useSSL=false
    username: root
    password: Cec@123456
  redis:
    host: 192.168.1.233
    port: 6379
    pool:
      max-active: -1
      max-idle: 6
    timeout: 60000
    database: 1
mybatis:
  config-location: classpath:mybatis-config.xml
  mapper-locations: classpath:mapper/*.xml
```

- tomcat 端口
- log 文件路径
- 自定义日志级别
- DataSource 配置
- MyBatis 配置
- Redis 配置

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
@EnableScheduling
@MapperScan("cn.xz.dao")
@EnableConfigurationProperties
@EnableTransactionManagement
public class Application {

    // 创建事务管理器1
    // @Bean(name = "txManager1")
    // public PlatformTransactionManager txManager(DataSource dataSource) {
    // return new DataSourceTransactionManager(dataSource);
    // }

    // 创建事务管理器2
    // @Bean(name = "txManager2")
    // public PlatformTransactionManager txManager2(EntityManagerFactory factory) {
    // return new JpaTransactionManager(factory);
    // }

    // 实现接口 TransactionManagementConfigurer 方法，其返回值代表在拥有多个事务管理器的情况下默认使用的事务管理器
    // @Override
    // public PlatformTransactionManager annotationDrivenTransactionManager() {
    // return txManager2;
    // }

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## 自定义配置

```java
package cn.xz;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

@Component
@PropertySource(value = { "classpath:my.properties" })
public class MyProperties {

    @Value("${my.config1}")
    private String config1;
    @Value("${my.config1}")
    private String config2;

    public String getConfig1() {
        return config1;
    }

    public String getConfig2() {
        return config2;
    }
}
```
## Redis

```java
package cn.xz.cache;

import java.util.concurrent.TimeUnit;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.stereotype.Repository;

@Repository
public class RedisBaseDao<V> {

    @Autowired
    private RedisTemplate<String, V> redisTemplate;

    public void remove(String key) {
        redisTemplate.delete(key);
    }

    public V get(String key) {
        ValueOperations<String, V> valueOperations = redisTemplate.opsForValue();
        V v = valueOperations.get(key);
        return v;
    }

    public void set(String key, V v, int days) {
        ValueOperations<String, V> opsForValue = redisTemplate.opsForValue();
        opsForValue.set(key, v, days, TimeUnit.DAYS);
    }
}
```

## Schedule

```java
package cn.xz.schedule;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class MySchedule {

    private static final Logger log = LoggerFactory.getLogger(MySchedule.class);

    @Scheduled(cron = "0/30 * * * * ?")
    public void aaa() {
        log.debug("MySchedule @Scheduled {}", System.nanoTime());
    }
}

```

具体代码参见 [https://github.com/stdupanda/spring-boot-demo](https://github.com/stdupanda/spring-boot-demo "https://github.com/stdupanda/spring-boot-demo")