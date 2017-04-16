---
layout: post
title: eclipse中用maven插件打包可执行jar
categories: Eclipse
description: eclipse maven 打包可执行 jar 包
keywords: maven, jar, java, Eclipse
---

需要注意的是 eclipse 新建工程 maven 项目时不选择 archetype，直接创建一个 simple 类型的。

然后修改 `pom.xml`

```xml
<!-- 开头部分的 -->
<packaging>jar</packaging><!-- 打包成jar -->
<name>qr_maker</name>


<!-- 配置打包begin -->
<build>
	<plugins>
		<plugin>
			<artifactId>maven-compiler-plugin</artifactId>
			<version>2.3.2</version>
			<configuration>
				<source>1.8</source>
				<target>1.8</target>
			</configuration>
		</plugin>
		<plugin>
			<artifactId> maven-assembly-plugin </artifactId>
			<configuration>
				<descriptorRefs>
					<descriptorRef>jar-with-dependencies</descriptorRef>
				</descriptorRefs>
				<archive>
					<manifest>
						<mainClass>cn.xz.qrmaker.view.MainWindow</mainClass>
					</manifest>
				</archive>
			</configuration>
			<executions>
				<execution>
					<id>make-assembly</id>
					<phase>package</phase>
					<goals>
						<goal>single</goal>
					</goals>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
<!-- 配置打包end -->
```

(注意 `<archive><manifest><mainClass>` 节点的配置)
之后 `maven` -> `update project`，然后就可以 `run as` -> `Maven install` ，之后就可以在 `target` 文件夹中生成两个 jar，一个包含依赖，一个不包含依赖
