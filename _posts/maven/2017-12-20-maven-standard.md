---
layout: post
title: maven 标准结构
categories: Maven
description: maven 标准结构
keywords: maven, java
---

整理汇总 maven 工程标准结构

|  path              | desc |
|:-------------------|:-----------------------------|
| `src/main/java`      | Application/Library sources
| `src/main/resources` | Application/Library resources
| `src/main/filters`   | Resource filter files
| `src/main/webapp`    | Web application sources
| `src/test/java`      | Test sources
| `src/test/resources` | Test resources
| `src/test/filters`   | Test resource filter files
| `src/it`             | Integration Tests (primarily for plugins)
| `src/assembly`       | Assembly descriptors
| `src/site`           | Site
| `LICENSE.txt`        | Project's license
| `NOTICE.txt`         | Notices and attributions required by libraries that the project depends on
| `README.txt`         | Project's readme


参考官网 [maven-in-five-minutes](http://maven.apache.org/guides/getting-started/maven-in-five-minutes.html "maven-in-five-minutes"), [introduction-to-the-standard-directory-layout](http://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html "introduction-to-the-standard-directory-layout")