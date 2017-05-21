---
layout: post
title: SpringMVC 的 mock 单元测试
categories: SpringMVC
description: SpringMVC 的 mock 单元测试
keywords: SpringMVC, mock, test
---

整理下 `Spring` 下的单元测试方案。

# 关于单元测试

一般是在模块开发完成之后进行单元测试，给定一个输入，测试是否能得到预期的结果。

## mock 框架

- EasyMock [link](http://easymock.org/ "link")
- Mockito [link](http://easymock.org/ "link")

这两个都是专业的 java 测试框架。

# spring-test

下面介绍 spring 自带的 test 框架

## 示例

```java
package cn.xz.test;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.MediaType;
import org.springframework.test.annotation.Rollback;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.result.MockMvcResultHandlers;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration(value = "src/main/webapp")
@ContextConfiguration(locations = { "classpath:config/spring.xml", "classpath:config/spring_mvc.xml" })
@Rollback(true)
// 4.2已废弃@TransactionConfiguration(transactionManager = "transactionManager",
// defaultRollback = true)
public class UserTest {

    @Autowired
    private WebApplicationContext webApplicationContext;

    private MockMvc mockMvc;

    private Logger logger = LoggerFactory.getLogger(getClass());

    @Before
    public void before() {
        mockMvc = MockMvcBuilders.webAppContextSetup(this.webApplicationContext).build();
    }

    @Test
    public void get() throws Exception {
        // mockMvc.perform(fileUpload("/doc").file("a1",
        // "ABC".getBytes("UTF-8")));
        this.mockMvc.perform(MockMvcRequestBuilders.get("/test").accept(MediaType.APPLICATION_JSON_UTF8))
                .andDo(MockMvcResultHandlers.print()).andExpect(MockMvcResultMatchers.status().isOk())
                // .andExpect(MockMvcResultMatchers.view().name("user/info"))//
                .andExpect(MockMvcResultMatchers.content().contentType("application/json;charset=UTF-8"))
                // .andExpect(MockMvcResultMatchers.jsonPath("$.name").value("Lee"))
                .andReturn();
        logger.info(" getok!");

    }

    @Test
    public void post() throws Exception {
        this.mockMvc
                .perform(MockMvcRequestBuilders.post("/test/{id}", 42)
                        .accept(MediaType.parseMediaType("application/json;charset=UTF-8")))
                .andDo(MockMvcResultHandlers.print()).andExpect(MockMvcResultMatchers.status().isOk())
                // .andExpect(MockMvcResultMatchers.view().name("user/info"))//
                .andExpect(MockMvcResultMatchers.content().contentType("application/json;charset=UTF-8"))
                // .andExpect(MockMvcResultMatchers.jsonPath("$.name").value("Lee"))
                .andReturn();
        logger.info("post ok!");
    }

    @Test
    public void postJson() throws Exception {
        String json = "{user:1,name:2}";
        mockMvc.perform(MockMvcRequestBuilders.post("/test/json{}", 12).contentType(MediaType.APPLICATION_JSON_UTF8)
                .content(json).accept(MediaType.parseMediaType("application/json;charset=UTF-8")))
                .andDo(MockMvcResultHandlers.print()).andExpect(MockMvcResultMatchers.status().isOk())
                // .andExpect(MockMvcResultMatchers.view().name("user/info"))//
                .andExpect(MockMvcResultMatchers.content().contentType("application/json;charset=UTF-8"))
                // .andExpect(MockMvcResultMatchers.jsonPath("$.name").value("Lee"))
                .andReturn();
        logger.info("post json ok!");
    }
}

```

## maven 配置

```xml
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <slf4j.version>1.7.25</slf4j.version>
    <cxf.version>3.1.1</cxf.version>
</properties>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId><!-- 在引入BOM之后，引入其他Spring依赖时，都无需指定版本 -->
            <artifactId>spring-framework-bom</artifactId>
            <version>4.3.7.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-test</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
    </dependency>

    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.1.0</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core -->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.8.2</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-api -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>${slf4j.version}</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.slf4j/slf4j-log4j12 -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.25</version>
    </dependency>

    <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-core -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-core</artifactId>
        <version>2.7.9</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.7.9</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/commons-codec/commons-codec -->
    <dependency>
        <groupId>commons-codec</groupId>
        <artifactId>commons-codec</artifactId>
        <version>1.10</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.5</version>
    </dependency>
    <!-- https://mvnrepository.com/artifact/commons-fileupload/commons-fileupload -->
    <dependency>
        <groupId>commons-fileupload</groupId>
        <artifactId>commons-fileupload</artifactId>
        <version>1.3</version>
    </dependency>
</dependencies>
```
