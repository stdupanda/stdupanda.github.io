---
layout: post
title: tomcat远程调试
categories: JavaWeb
description: tomcat远程调试
keywords: tomcat, JavaWeb
---

项目在开发机器上运行正常，但是在生产 linux 服务器上运行总是异常，无法定位是 jvm 或者其他问题导致的，只能远程 debug 一下分析看看。

## JPDA

JPDA(Java Platform Debugger Architecture) 是 Java 平台调试体系结构的缩写，通过 JPDA 提供的 API，开发人员可以方便灵活的搭建 Java 调试应用程序。JPDA 主要由三个部分组成：Java 虚拟机工具接口（JVMTI），Java 调试线协议（JDWP），以及 Java 调试接口（JDI）。而像Eclipse和IDEA这种开发工具提供的图形界面的调试工具，其实就是实现了JDI。

### catlina.sh

tomcat使用如下方式进行启动jpda：

`./catalina.sh jpda start`

默认情况下，远程调试的默认端口为8000，可以通过JPDA_ADDRESS进行配置，指定自定义的端口，另外，还有两个可以配置的参数

- JPDA_TRANSPORT：即调试器和虚拟机之间数据的传输方式，默认值是dt_socket

- JPDA_SUSPEND：即JVM启动后是否立即挂起，默认是n

`CATALINA_OPTS="-server -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8899"`

一般不用配置上述参数，使用默认的 `./catalina.sh jpda start` 即可。

注意**看下系统防火墙是否开启对应端口号**

### eclipse

`Debug Configurations,Remote Java Application,Project,Host,Port,Source`，在项目源码中加上断点，访问断点所在代码即可进入断点 debug 了。

### 其他方式

如果使用 `./catalina.sh jpda start` 方式启动不生效，可以尝试：在tomcat启动参数JAVA_OPTS里增加：

```shell
-agentlib:jdwp=transport=dt_socket,address=8088,suspend=n,server=y
```

然后在 idea 中新建 debug 配置，指定 host 和端口号，然后启动即可。

如果端口连接失败且确认网络，尝试修改为 `address=*:8088` 后重试。

> Before Java 5.0, use -Xdebug and -Xrunjdwp arguments. These options will still work in later versions, but it will run in interpreted mode instead of JIT, which will be slower.
