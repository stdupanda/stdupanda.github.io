---
layout: post
title: SpringBoot 系列 02 JPA
categories: SpringBoot
description: SpringBoot 系列
keywords: Java, spring, springboot, java, boot, jpa
---

一个集成了 JPA 的 Spring Boot 工程.

# 工程结构

## 实际结构

```
sb02
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── cn
    │   │       └── xz
    │   │           ├── Application.java
    │   │           ├── ctrl
    │   │           │   └── DemoController.java
    │   │           ├── entity
    │   │           │   └── CloudUserInfo.java
    │   │           ├── repository
    │   │           │   └── UserInfoRepository.java
    │   │           └── service
    │   │               ├── impl
    │   │               │   └── UserServiceImpl.java
    │   │               └── UserService.java
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
    <artifactId>sb02</artifactId>
    <packaging>jar</packaging>
    <version>0.0.1-SNAPSHOT</version>
    <name>sb02</name>
    <description>jpa</description>
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
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
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
        <finalName>sb02</finalName>
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
  file: ./sb02.log
  level:
    root: info
    cn: debug
    org.springframework.orm.jpa.JpaTransactionManager: debug
    org.hibernate.SQL: debug
#    org.hibernate.type.descriptor.sql: trace
server:
  port: 8082
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://192.168.1.241:3306/hce_guiyang?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&useSSL=false
    username: root
    password: xxx
# 配置 jpa, 采用 Hibernate 实现
  jpa:
    show-sql: true
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL5InnoDBDialect
        hbm2ddl:
          auto: update
```

- tomcat 端口
- log 文件路径
- 自定义日志级别
- DataSource 配置
- JPA 配置
- Hibernate 配置

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

## 实体类

这个实体类是使用 `Eclipse` 的 `Hibernate Tools` 插件生成的.

```java
package cn.xz.entity;
// Generated Jan 9, 2018 11:30:28 AM by Hibernate Tools 5.2.6.Final

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.Table;

/**
 * CloudUserInfo generated by hbm2java
 */
@Entity
@Table(name = "cloud_user_info", catalog = "hce_guiyang")
public class CloudUserInfo implements java.io.Serializable {

    /**
     * 
     */
    private static final long serialVersionUID = -6452663887770367038L;
    private String userId;
    private String imei;

    public CloudUserInfo() {
    }

    @Override
    public String toString() {
        return "CloudUserInfo [userId=" + userId + ", imei=" + imei + "]";
    }

    public CloudUserInfo(String userId, String onlineMoney) {
        this.userId = userId;
        this.onlineMoney = onlineMoney;
    }

    public CloudUserInfo(String userId, String imei) {
        this.userId = userId;
        this.imei = imei;
    }

    @Id

    @Column(name = "user_id", unique = true, nullable = false, length = 64)
    public String getUserId() {
        return this.userId;
    }

    public void setUserId(String userId) {
        this.userId = userId;
    }

    @Column(name = "imei", length = 64)
    public String getImei() {
        return this.imei;
    }

    public void setImei(String imei) {
        this.imei = imei;
    }
}
```

## JpaRepository 类

```java
package cn.xz.repository;

import java.util.List;

import org.springframework.data.jpa.repository.JpaRepository;

import cn.xz.entity.CloudUserInfo;

public interface UserInfoRepository extends JpaRepository<CloudUserInfo, String> {
    List<CloudUserInfo> findByUserId(String userId);

    List<CloudUserInfo> findByUserIdIsNotNull();
}
```

附注介绍 Spring Data JPA 规范.

### 直接使用默认方法

``` java 
@Test
public void test() throws Exception {
	User user=new User();
	userRepository.findAll();
	userRepository.findOne(1l);
	userRepository.save(user);
	userRepository.delete(user);
	userRepository.count();
	userRepository.exists(1l);
	// ...
}
```

### 自定义

**具体的关键字，使用方法和生产成SQL如下表所示**

| Keyword	| Sample	|JPQL snippet
|----     |---        |---
| And	|findByLastnameAndFirstname	|… where x.lastname = ?1 and x.firstname = ?2
| Or	|findByLastnameOrFirstname	|… where x.lastname = ?1 or x.firstname = ?2
| Is,Equals|	findByFirstnameIs,findByFirstnameEquals	|… where x.firstname = ?1
| Between	|findByStartDateBetween	|… where x.startDate between ?1 and ?2
| LessThan |	findByAgeLessThan	|… where x.age < ?1
| LessThanEqual|	findByAgeLessThanEqual	|… where x.age ⇐ ?1
| GreaterThan	|findByAgeGreaterThan	|… where x.age > ?1
| GreaterThanEqual|	findByAgeGreaterThanEqual	|… where x.age >= ?1
| After	|findByStartDateAfter	|… where x.startDate > ?1
| Before	|findByStartDateBefore	|… where x.startDate < ?1
| IsNull	|findByAgeIsNull	|… where x.age is null
| IsNotNull,NotNull|	findByAge(Is)NotNull	|… where x.age not null
| Like	|findByFirstnameLike	|… where x.firstname like ?1
| NotLike	|findByFirstnameNotLike	|… where x.firstname not like ?1
| StartingWith|	findByFirstnameStartingWith	|… where x.firstname like ?1 (parameter bound with appended %)
| EndingWith	|findByFirstnameEndingWith	|… where x.firstname like ?1 (parameter bound with prepended %)
| Containing	|findByFirstnameContaining	|… where x.firstname like ?1 (parameter bound wrapped in %)
| OrderBy	|findByAgeOrderByLastnameDesc	|… where x.age = ?1 order by x.lastname desc
| Not	|findByLastnameNot	|… where x.lastname <> ?1
| In	|findByAgeIn(Collection<Age> ages)	|… where x.age in ?1
| NotIn|	findByAgeNotIn(Collection<Age> age)	|… where x.age not in ?1
| TRUE|	findByActiveTrue()	|… where x.active = true
| FALSE|	findByActiveFalse()	|… where x.active = false
| IgnoreCase|	findByFirstnameIgnoreCase	|… where UPPER(x.firstame) = UPPER(?1)

### 分页与自定义

- Pageable 分页
- Example 自定义拼装 like 等查询条件

```java
package cn.xz.service.impl;

import java.util.List;
import java.util.UUID;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Example;
import org.springframework.data.domain.ExampleMatcher;
import org.springframework.data.domain.ExampleMatcher.GenericPropertyMatchers;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;
import org.springframework.data.domain.Sort.Direction;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import cn.xz.entity.CloudUserInfo;
import cn.xz.repository.UserInfoRepository;
import cn.xz.service.UserService;

@Service
@Transactional
public class UserServiceImpl implements UserService {

    private static final Logger log = LoggerFactory.getLogger(UserServiceImpl.class);

    @Autowired
    private UserInfoRepository userInfoRepository;

    @Override
    public void test() {
        log.debug("jpa test.");
        CloudUserInfo tmp = new CloudUserInfo();
        tmp.setUserId(userId);
        // ExampleMatcher init
        ExampleMatcher matcher = ExampleMatcher.matching() // 构建对象
                .withMatcher("userId", GenericPropertyMatchers.startsWith()) // 姓名采用“开始匹配”的方式查询
                .withIgnorePaths("focus"); // 忽略属性：是否关注。因为是基本类型，需要忽略掉
        Example<CloudUserInfo> example = Example.of(tmp, matcher);
        // jpa select by example with page
        List<CloudUserInfo> list2 = userInfoRepository.findAll(example, new Sort("userId"));// 经测试，排序条件不可以是 user_id
        log.debug("jpa example {}", list2);
        Sort sort = new Sort(Direction.DESC, "userId");
        Pageable pageable = new PageRequest(1, 10, sort);
        log.debug("jpa page1 {}", userInfoRepository.findAll(example, pageable));
        log.debug("jpa page2 {}", userInfoRepository.findAll(pageable));
    }
}
```

具体代码参见 [https://github.com/stdupanda/spring-boot-demo](https://github.com/stdupanda/spring-boot-demo "https://github.com/stdupanda/spring-boot-demo")

更多请参考: http://www.ityouknow.com/springboot/2016/08/20/springboot(%E4%BA%94)-spring-data-jpa%E7%9A%84%E4%BD%BF%E7%94%A8.html