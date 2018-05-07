---
layout: post
title: SpringBoot 系列 08 自定义 logback 扩展
categories: SpringBoot
description: SpringBoot 系列
keywords: Java, spring, springboot, java, boot
---

自定义 Spring Boot 项目的 logback 设置，按天滚动文件。

# 介绍

Spring Boot 默认使用的是 `logback` 作为日志实现，我们在代码中可以放心使用 `slf4j` 进行日志记录。

# 默认可配置项

参考官方文档，可以简单的进行如下配置 `application.yml`:

```yml
logging:
  file: ./sb07.log
  level:
    root: info
    com.xxx: debug
  pattern:
      console:  xxx
      file: xxx
      level: xxx
```

# 自定义扩展 `logback`

在 `src/main/resource` 路径下，创建 `logback-spring.xml` 文件。此时不建议设置 `loggin.file` 等配置。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>

    <springProperty scope="context" name="springAppName" source="spring.application.name" />

    <!-- Example for logging into the build folder of your project -->
    <property name="LOG_FILE" value="${springAppName}_log"/>​

    <property name="LAYOUT_PATTERN"
              value="[%d{yyyy-MM-dd HH:mm:ss.SSS}][%-5level][%thread] %logger:%method-%-4line | %msg%n"/>

    <!-- You can override this to have a custom pattern -->
    <property name="CONSOLE_LOG_PATTERN"
              value="%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"/>

    <!-- Appender to log to console -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <!-- Minimum logging level to be presented in the console logs-->
            <level>DEBUG</level>
        </filter>
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>utf8</charset>
        </encoder>
    </appender>

    <appender name="file" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <File>${LOG_FILE}.log</File>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>DEBUG</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <FileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.log</FileNamePattern>
            <maxHistory>3650</maxHistory>
            <cleanHistoryOnStart>false</cleanHistoryOnStart>
        </rollingPolicy>
        <append>true</append>
        <encoder>
            <pattern>${LAYOUT_PATTERN}</pattern>
        </encoder>
    </appender>

<!-- Appender to log to file in a JSON format -->
    <appender name="logstash" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}.json</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE}.json.%d{yyyy-MM-dd}</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <timestamp>
                    <timeZone>UTC</timeZone>
                </timestamp>
                <pattern>
                    <pattern>
                        {
                        "severity": "%level",
                        "service": "${springAppName:-}",
                        "trace": "%X{X-B3-TraceId:-}",
                        "span": "%X{X-B3-SpanId:-}",
                        "parent": "%X{X-B3-ParentSpanId:-}",
                        "exportable": "%X{X-Span-Export:-}",
                        "pid": "${PID:-}",
                        "thread": "%thread",
                        "class": "%logger{40}",
                        "rest": "%message"
                        }
                    </pattern>
                </pattern>
            </providers>
        </encoder>
    </appender>

    <root level="info">
        <appender-ref ref="console"/>
        <appender-ref ref="file"/>
        <appender-ref ref="logstash"/>
    </root>
    <logger name="com.xxx" additivity="true">
        <level value="info"/>
        <appender-ref ref="file"/>
    </logger>
    <logger name="cn.xz" level="debug"/>
    <logger name="com.apache.ibatis" level="TRACE"/>
    <logger name="java.sql.Connection" level="DEBUG"/>
    <logger name="java.sql.Statement" level="DEBUG"/>
    <logger name="java.sql.PreparedStatement" level="DEBUG"/>
</configuration>
```

其中包括了 `json` 格式的日志文件，用于 `logstah` 采集日志进行日志检索分析。

需要注意设置 `spring.application.name` 属性。


具体代码参见 [https://github.com/stdupanda/spring-boot-demo](https://github.com/stdupanda/spring-boot-demo "https://github.com/stdupanda/spring-boot-demo")
