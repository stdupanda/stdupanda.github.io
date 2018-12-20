---
layout: post
title: SpringBoot 系列 04 Mybatis 多数据源配置
categories: SpringBoot
description: SpringBoot 系列
keywords: Java, spring, springboot, java, boot, mybatis, datasource
---

一个集成了 多数据源 Mybatis 的 Spring Boot 工程.

# 工程结构

## 实际结构

```
sb04
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── cn
    │   │       └── xz
    │   │           ├── Application.java
    │   │           ├── ctrl
    │   │           │   └── DemoController.java
    │   │           ├── dao
    │   │           │   ├── CloudUserInfoMapper.java
    │   │           │   └── CustomMapper.java
    │   │           ├── dao2
    │   │           │   └── CustomMapper2.java
    │   │           ├── Datasource01Config.java
    │   │           ├── Datasource02Config.java
    │   │           └── entity
    │   │               ├── CloudUserInfo.java
    │   │               └── CloudUserInfoExample.java
    │   └── resources
    │       ├── application.yml
    │       ├── mapper
    │       │   └── CloudUserInfoMapper.xml
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
    <artifactId>sb04</artifactId>
    <packaging>jar</packaging>
    <version>0.0.1-SNAPSHOT</version>
    <name>sb04</name>
    <description>mybatis multi datasource</description>
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
            <artifactId>spring-boot-devtools</artifactId>
            <optional>true</optional>
        </dependency><!-- will be disabled where using "java -jar ..." -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
    <build>
        <finalName>sb04</finalName>
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
  file: ./sb04.log
  level:
    root: info
    cn: debug
    org.mybatis: debug
server:
  port: 8084
spring:
  datasource:
    datasource01:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://192.168.1.241:3306/hce_guiyang?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&useSSL=false
      username: root
      password: xxx
    datasource02:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://192.168.1.241:3306/hce_guiyang?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&useSSL=false
      username: root
      password: xxx
mybatis:
  config-location: classpath:mybatis-config.xml
  mapper-locations: classpath:mapper/*.xml
```

- tomcat 端口
- log 文件路径
- 自定义日志级别
- DataSource 配置
- MyBatis 配置

## Spring Boot 入口

```java
package cn.xz;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.transaction.annotation.EnableTransactionManagement;

@SpringBootApplication
// @MapperScan("cn.xz.dao")多数据源配置单独位于相应配置类中
@EnableTransactionManagement
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## 数据源配置

```java
package cn.xz;

import javax.sql.DataSource;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

@Configuration
@MapperScan(basePackages = "cn.xz.dao", sqlSessionTemplateRef = "datasource01SqlSessionTemplate")
public class Datasource01Config {

    @Primary
    @Bean(name = "datasource01")
    @ConfigurationProperties(prefix = "spring.datasource.datasource01")
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Primary
    @Bean(name = "datasource01TransactionManager")
    public DataSourceTransactionManager testTransactionManager(@Qualifier("datasource01") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Primary
    @Bean(name = "datasource01SqlSessionFactory")
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("datasource01") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/*.xml"));
        return bean.getObject();
    }

    @Primary
    @Bean(name = "datasource01SqlSessionTemplate")
    public SqlSessionTemplate testSqlSessionTemplate(
            @Qualifier("datasource01SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```

```java
package cn.xz;

import javax.sql.DataSource;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

@Configuration
@MapperScan(basePackages = "cn.xz.dao2", sqlSessionTemplateRef = "datasource02SqlSessionTemplate")
public class Datasource02Config {

    // @Primary
    @Bean(name = "datasource02")
    @ConfigurationProperties(prefix = "spring.datasource.datasource02")
    public DataSource testDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "datasource02TransactionManager")
    public DataSourceTransactionManager testTransactionManager(@Qualifier("datasource02") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "datasource02SqlSessionFactory")
    public SqlSessionFactory testSqlSessionFactory(@Qualifier("datasource02") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/*.xml"));
        return bean.getObject();
    }

    @Bean(name = "datasource02SqlSessionTemplate")
    public SqlSessionTemplate testSqlSessionTemplate(
            @Qualifier("datasource02SqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}

```


## 测试用例

```java
package cn.xz;

import static org.hamcrest.CoreMatchers.containsString;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.context.WebApplicationContext;

import cn.xz.dao.CloudUserInfoMapper;
import cn.xz.dao2.CustomMapper2;
import cn.xz.entity.CloudUserInfo;

@RunWith(SpringRunner.class)
@SpringBootTest
public class ApplicationTests {

    private static final Logger log = LoggerFactory.getLogger(ApplicationTests.class);

    @Autowired
    private WebApplicationContext webApplicationContext;
    @Autowired
    private CloudUserInfoMapper userInfoMapper;
    @Autowired
    private CustomMapper2 customMapper2;

    private MockMvc mvc;

    @Before
    public void setUp() throws Exception {
        mvc = MockMvcBuilders.webAppContextSetup(this.webApplicationContext).build();
    }

    @Test
    @Transactional(transactionManager = "datasource01TransactionManager")
    public void contextLoads() throws Exception {
        log.info("mybatis multi datasource test.");
        // mybatis
        CloudUserInfo userInfo = userInfoMapper.selectByPrimaryKey("13007817442");
        log.info("userInfo: {}", userInfo);
        log.info("datasource 2 test : {}", customMapper2.test());
        log.info("ok");
        mvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON)).andExpect(status().isOk())
                .andExpect(content().string(containsString("Hello World")));
    }

}
```

具体代码参见 [https://github.com/stdupanda/spring-boot-demo](https://github.com/stdupanda/spring-boot-demo "https://github.com/stdupanda/spring-boot-demo")