---
layout: post
title: Eclipse 底部 Problems 视图错误提示
categories: Eclipse
description: Eclipse 底部 Problems 视图错误提示
keywords: eclipse, Eclipse
---

在使用 eclipse 打开之前的项目后，`Problems` 视图总是提示类似的错误：


`Java compiler level does not match the version of the installed Java project facet.`

`Target runtime Apache Tomcat v6.0 is not defined. xxx	Unknown	Faceted Project Problem`


解决方法如下：

打开工程路径的 `.settings` 文件夹，打开 `org.eclipse.wst.common.project.facet.core.xml` 文件，把和问题相关的行删除。然后再在 eclipse 中刷新下工程即可。
