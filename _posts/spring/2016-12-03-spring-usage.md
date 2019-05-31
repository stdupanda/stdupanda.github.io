---
layout: post
title: Spring 使用整理
categories: Spring
description: Spring 使用
keywords: Spring, java
---

This article records the spring framework usages that are frequently used in usual programs.

## 集成 redis

- 配置 `maven`

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <!-- 在引入BOM之后，引入其他Spring依赖时，都无需指定版本 -->
            <groupId>org.springframework</groupId>
            <artifactId>spring-framework-bom</artifactId>
            <version>4.3.7.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
<dependencies>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-redis</artifactId>
        <version>1.7.8.RELEASE</version>
    </dependency>
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
        <version>2.7.2</version>
    </dependency>
</dependencies>
```

- 配置 bean xml

```xml
<bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <property name="maxIdle" value="300" />
    <property name="maxTotal" value="600" />
    <property name="maxWaitMillis" value="1000" />
    <property name="testOnBorrow" value="true" />
</bean>
<bean id="jedisPool" class="redis.clients.jedis.JedisPool">
    <constructor-arg index="0" ref="poolConfig" />
    <constructor-arg index="1" value="${redis_ip}" />
    <constructor-arg index="2" value="${redis_port}" />
    <constructor-arg index="3" value="${redis_timeout}" />
</bean>
<bean id="connFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
    <property name="hostName" value="127.0.0.1" />
    <property name="port" value="6379" />
    <property name="poolConfig" ref="poolConfig" />
</bean>
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
    <property name="connectionFactory" ref="connFactory" />
    <property name="defaultSerializer">
        <bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />
    </property>
    <property name="valueSerializer">
        <!-- 用 json 格式将 object 序列化字节存入 redis -->
        <bean class="org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer" />
        <!--用 jvm 方式将 object 序列化字节存入 redis
        <bean class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer"
            /> -->
    </property>
</bean>
```

- 业务中调用

`jedis` 方式：

```java
Jedis jedis = jedisPool.getResource();
jedis.select(Integer.parseInt(DB_INDEX));
jedis.exists(key);
jedis.get(key);
String result = jedis.set(key, value);
```

`spring-data-redis` 方式：

```java
import java.util.concurrent.TimeUnit;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.stereotype.Component;

@Component
public class RedisBaseDao<K, V> {

    @Autowired
    private RedisTemplate<K, V> redisTemplate;

    public V get(K key) {
        ValueOperations<K, V> valueOperations = (ValueOperations<K, V>) redisTemplate.opsForValue();
        V v = valueOperations.get(key);
        return v;
    }

    public void set(K key, V v, int days){
        ValueOperations<K, V> opsForValue = redisTemplate.opsForValue();
        opsForValue.set(key, v, days, TimeUnit.DAYS);
    }
}
```

## mock 测试

- EasyMock [link](http://easymock.org/ "link")
- Mockito [link](http://easymock.org/ "link")

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

## 静态资源映射

未进行静态资源映射配置可能会导致 css/js 等无法访问。

需要在 `web.xml` 内，`org.springframework.web.servlet.DispatcherServlet` 对应 `<servlet>` 节点里增加 `<load-on-startup>` 并设置为 0 或较小的值，使其拥有较高的优先级。

- 使用 `<mvc:default-servlet-handler />`
  - 会将静态资源的请求转给 web 容器默认的 Servlet 处理
  - 特殊 web 容器可以设置 `default-servlet-name` 指定其值
- 使用 `<mvc:resources />`
  - 由 Spring MVC 框架自己处理静态资源，并添加一些有用的附加值功能。

首先，`<mvc:resources />`允许静态资源放在任何地方，如 WEB-INF 目录下、类路径下等，你甚至可以将 js/css 等静态文件打到 jar 包中，然后通过 location 属性指定静态资源的位置。

其次，`<mvc:resources />`依据当前著名的 Page Speed、YSlow 等浏览器优化原则对静态资源提供优化。你可以通过`cacheSeconds` 属性指定静态资源在浏览器端的缓存时间，一般可将该时间设置为一年，以充分利用浏览器端的缓存。在输出静态资源时，会根据配置设置好响应报文头的 `Expires` 和 `Cache-Control` 值。

在接收到静态资源的获取请求时，会检查请求头的 `Last-Modified` 值，如果静态资源没有发生变化，则直接返回 `303` 响应状态码，提示客户端使用浏览器缓存的数据，而非将静态资源的内容输出到客户端，以充分节省带宽，提高程序性能。

比如 `<mvc:resources location="/,classpath:/META-INF/static/" mapping="/resources/**"/>` 即可以将根路径 `/` 及类路径下 `/META-INF/static/` 的目录映射为 `/resources` 路径。

## IE 11 接收 json 数据后提示下载

去掉 `<mvc:annotation-driven />`，使用如下配置：

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