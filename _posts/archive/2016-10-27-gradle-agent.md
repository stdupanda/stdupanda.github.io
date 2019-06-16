---
layout: post
title: 修改Gradle使用国内源
categories: 归档
description: 修改Gradle使用国内源
keywords: android, Android, studio
---

由于需要使用 Gradle 仓库中心的项目，下载速度又比较慢，网上查询了下，发现清一色的全是 oschina 的库，但是人家 [2015年06月29日停止服务](https://www.oschina.net/news/63762/maven-oschina-paused) 了，于是想到了类似网易、新浪、阿里、腾讯等大佬的国内镜像，真找到了：

# 使用阿里云的Maven镜像仓库

在 **`project-level`** 的 `build.gradle`中修改如下：

```
allprojects {
    repositories {
        //jcenter()
        //maven{ url 'http://maven.oschina.net/content/groups/public/'}
        maven{ url 'http://maven.aliyun.com/nexus/content/groups/public/'}
    }
}
```

爽多了啊有木有