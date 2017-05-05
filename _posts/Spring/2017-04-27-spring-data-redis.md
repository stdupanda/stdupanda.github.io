---
layout: post
title: Spring Data Redis 集成 jedis 操作 redis 内存管理
categories: Spring
description: Spring 简介
keywords: Spring, redis, Redis
---

使用 Spring Data Redis 配合 jedis 操作 redis 进行编程，以及一个 DAO 接口。

# 相关配置

## 配置 `maven`

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

## 配置 `spring.xml`

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

# 业务中调用

## `jedis` 方式

```java
Jedis jedis = jedisPool.getResource();
jedis.select(Integer.parseInt(DB_INDEX));
jedis.exists(key);
jedis.get(key);
String result = jedis.set(key, value);
```

## `spring-data-redis` 方式

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;
import org.springframework.stereotype.Component;

@Component
public class InfoDAO {

    @Autowired
    private RedisTemplate<String, UserInfo> redisTemplate;

    public UserInfo get(String id) {
        ValueOperations<String, UserInfo> opsForValue = redisTemplate.opsForValue();
        return opsForValue.get(id);
    }

    public void setUserLogonInfo(String key, UserInfo userInfo) {
        ValueOperations<String, UserInfo> opsForValue = redisTemplate.opsForValue();
        opsForValue.set(key, userInfo);
    }
}
```

或者

```java
import java.io.Serializable;
import java.util.concurrent.TimeUnit;

import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.ValueOperations;

@Component
public class CommonRedisDao {

    @Autowired
    private RedisTemplate<String, Serializable> redisTemplate;

    public Object get(String key) {
        ValueOperations<String, Serializable> valueOperations =
                (ValueOperations<String, Serializable>) getRedisTemplate().opsForValue();
        Object v = valueOperations.get(key);
        return v;
    }

    public void set(String key,Serializable val, int expireDays){
        ValueOperations<String, Serializable> opsForValue = getRedisTemplate().opsForValue();
        set(key, val, opsForValue, expireDays);
    }

    private void set(String key,Serializable val, ValueOperations<String, Serializable> opsForValue, int days){
        if (0 > days) {
            days = Constants.MAX_LOGIN_DAYS;
        }
        opsForValue.set(key, val, days, TimeUnit.DAYS);
    }
}
```

或者泛型方式

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
