---
layout: post
title: 没用过的<%!%>标签
categories: Jsp
description: 没用过的<%!%>标签
keywords: Jsp, java
---

因为业务需要，需要把部分代码放到jsp里方便部署。在迁移代码的时候，突然发现无法把一个方法放到jsp的 `<%...%>` 代码片里，上网查了下，可以方法到 `<%!...%>` 里，记录下各种方式吧。

# <%!...%>

```

<%!

//1、可定义方法
public String outMethod(){
	return "outMethod";	
}
//2、可定义static方法
public static String outStaticMethod(){
	return "outStaticMethod";
}
//3、可定义static属性
public static int count = 0;

//4、不可以使用out对象
%>

```

# <%...%>

```

<%
//1、不可定义方法
//2、不可定义static方法
//3、不可定义static属性

//4、可以使用out对象
out.print(outMethod());
out.print(outStaticMethod());
out.print(count);
%>

```