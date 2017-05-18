---
layout: post
title: maven 集成 jetty 插件
categories: Maven
description: maven 集成 jetty 插件
keywords: maven, jetty, java
---

配置 `jetty` 插件，节省 部署到 `tomcat` 的时间。

# 修改 `pom.xml`

```xml
<build>
    <finalName>xz</finalName>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.2</version>
            <configuration>
                <source>1.7</source>
                <target>1.7</target>
            </configuration>
        </plugin>
        <plugin>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-maven-plugin</artifactId>
            <version>9.3.6.v20151106</version>
            <configuration>
                <httpConnector>
                    <port>80</port>
                </httpConnector>
                <scanIntervalSeconds>3</scanIntervalSeconds>
                <webApp>
                    <contextPath>/xz</contextPath>
                </webApp>
            </configuration>
        </plugin>
    </plugins>
</build>
```

# 调整运行配置

注意相关节点的配置，然后右键项目，

- `maven` -> `update project`
- `run as` -> `Maven Build`
- 配置 `Main` -> `Goals`，输入 `jetty:run -Djetty.port=8080`


之后就可以用 jetty 启动项目了。
