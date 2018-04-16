---
layout: post
title: maven 常用命令
categories: Maven
description: maven 常用命令
keywords: maven, java, Eclipse
---

整理汇总 maven 常用命令

# 自定义国内源

修改 `~/.m2/setting.xml` 文件, 内容如下:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

    <pluginGroups />
    <proxies />
    <servers />
    <mirrors>
        <mirror>
            <id>alimaven</id>
            <name>aliyun maven</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
            <mirrorOf>central</mirrorOf><!-- 此处需注意，可设置为 * 强制走此镜像 -->
        </mirror>
    </mirrors>
    <localRepository>D:/apache-maven-repository</localRepository>
</settings>
```

# 常用命令

语法: `mvn [options] [<goal(s)>] [<phase(s)>]`

创建项目: `mvn archetype:generate -DgroupId=xx -DartifactId=xx -DarchetypeArtifactId=maven-archetype-quickstart`

archetypeArtifactId 类型一般有: `maven-archetype-webapp`, `maven-archetype-quickstart`,

| 命令 | 解释 |
|:------------------|:------------------|
| 创建操作|
| `mvn archetype:generate ...`           | 生成 maven 工程, 参考前文 |
| `mvn eclipse:eclipse`                  | 转换成 eclipse 工程 |
| `mvn idea:idea`                        | 转换成 idea 工程 |
| `mvn -h`                               | 帮助 |
| `mvn --version`                        | 查看版本 |
| 测试 |                               
| `mvn test`                             | 测试 |
| 编译打包 |                           
| `mvn clean`                            | 清空 |
| `mvn package`                          | 打包 |
| `mvn package -DskipTests`              | 打包, 不执行测试用例, 会编译测试用例至 `target/test-classes` |
| `mvn package -Dmaven.test.skip=true`   | 打包, 不执行测试用例, 不编译测试用例 |
| `mvn clean package`                    | 等价于先 `mvn clean` 再 `mvn package` |
| `mvn install`                          | 打包到本地仓库 |
| `mvn dependency:sources`               | 下载源码包 |
| `mvn dependency:tree`                  | 打印依赖 |
| 运行 |                               
| `mvn jetty:run`                        | 需配置 jetty 插件 |
