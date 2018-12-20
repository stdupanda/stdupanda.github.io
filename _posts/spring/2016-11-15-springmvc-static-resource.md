---
layout: post
title: SpringMVC对静态资源的处理
categories: Spring
description: SpringMVC对静态资源的处理
keywords: SpringMVC, java, JavaWeb, Spring, SSM
---

整理之前项目中SpringMVC对静态资源的处理问题，之前为了时间进度转而使用了 `.do` 的配置，绕开了这个问题，今天进行分析解决整理，记录下来方便后续查看。

# 问题场景

`web.xml`中配置了 SpringMVC 的 `org.springframework.web.servlet.DispatcherServlet` ，在html界面中访问静态资源文件时会出现404问题，同时 SpringMVC 后台警告 `No mapping found for HTTP request with URI [/xxx/xxx.js] in DispatcherServlet with name 'springmvc'-[org.springframework.web.servlet.DispatcherServlet.noHandlerFound`


# 两种经典解决办法

**注：**务必在`web.xml`中 `org.springframework.web.servlet.DispatcherServlet` <servlet>设置中增加 `<load-on-startup>` 并设置为0或较小的值，使其拥有较高的优先级。

## 使用 `<mvc:default-servlet-handler />`

在`springMVC-servlet.xml`中配置`<mvc:default-servlet-handler />`后，会在Spring MVC上下文中定义一个`org.springframework.web.servlet.resource.DefaultServletHttpRequestHandler`，对进入`DispatcherServlet`的URL进行筛查，如果发现是静态资源的请求，就将该请求转由Web应用服务器默认的Servlet处理，如果不是静态资源的请求，才由`DispatcherServlet`继续处理。

- 一般Web应用服务器(包括Tomcat, Jetty, Glassfish, JBoss, Resin, WebLogic, WebSphere)默认的Servlet名称是"default"，因此`DefaultServletHttpRequestHandler`可以找到它，只需要如下配置：

`<mvc:default-servlet-handler />`

- 如果你使用的Web应用服务器的默认Servlet名称不是"default"，则需要通过`default-servlet-name`属性显示指定：

`<mvc:default-servlet-handler default-servlet-name="所使用的Web服务器默认使用的Servlet名称" />`


## 使用 `<mvc:resources />`

`<mvc:default-servlet-handler />`将静态资源的处理经由Spring MVC框架交回Web应用服务器处理。而`<mvc:resources />`更进一步，由Spring MVC框架自己处理静态资源，并添加一些有用的附加值功能。

首先，`<mvc:resources />`允许静态资源放在任何地方，如WEB-INF目录下、类路径下等，你甚至可以将JavaScript等静态文件打到JAR包中。通过location属性指定静态资源的位置，由于location属性是Resources类型，因此可以使用诸如"classpath:"等的资源前缀指定资源位置。传统Web容器的静态资源只能放在Web容器的根路径下，`<mvc:resources />`完全打破了这个限制。

其次，`<mvc:resources />`依据当前著名的Page Speed、YSlow等浏览器优化原则对静态资源提供优化。你可以通过`cacheSeconds`属性指定静态资源在浏览器端的缓存时间，一般可将该时间设置为一年，以充分利用浏览器端的缓存。在输出静态资源时，会根据配置设置好响应报文头的Expires 和 Cache-Control值。

在接收到静态资源的获取请求时，会检查请求头的Last-Modified值，如果静态资源没有发生变化，则直接返回303相应状态码，提示客户端使用浏览器缓存的数据，而非将静态资源的内容输出到客户端，以充分节省带宽，提高程序性能。


- 在`springMVC-servlet`中添加如下配置：

`<mvc:resources location="/,classpath:/META-INF/publicResources/" mapping="/resources/**"/>`

以上配置将Web根路径"/"及类路径下 `/META-INF/publicResources/` 的目录映射为`/resources`路径。假设Web根路径下拥有images、js这两个资源目录,在images下面有bg.gif图片，在js下面有test.js文件，则可以通过 `/resources/images/bg.gif` 和 `/resources/js/test.js` 访问这二个静态资源。

假设WebRoot还拥有 `images/bg1.gif` 及 `js/test1.js`，则也可以在网页中通过 `/resources/images/bg1.gif` 及 `/resources/js/test1.js` 进行引用。


- 简单些的配置

`<mvc:resources mapping="/images/**" location="/images/" />`

使用`<mvc:resources/>`元素把`images/**`映射到`ResourceHttpRequestHandler`进行处理，`location`指定静态资源的位置.
