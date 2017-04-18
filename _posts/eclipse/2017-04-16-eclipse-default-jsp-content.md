---
layout: post
title: eclipse中修改jsp模板
categories: Eclipse
description: eclipse maven 打包可执行 jar 包
keywords: jsp, eclipse, 模板, Eclipse
---

eclipse 中新建 jsp 文件时默认生成的内容太少不实用，把 Myeclipse 里的内容导入到 eclipse 的 jsp 模板即可。

`Prefrences` -> `Web` -> `JSP Files`,修改 `Encoding` 为 `UTF-8`

`Prefrences` -> `Web` -> `JSP Files` -> `Editor` -> `Templates` -> `New JSP File(html)`, 选择 `Edit`, 然后输入下面的内容。

```jsp
<%@ page language="java" contentType="text/html; charset=${encoding}" pageEncoding="${encoding}"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>
<html>
  <head>
    <base href="<%=basePath%>">
    <meta http-equiv="Content-Type" content="text/html; charset=${encoding}">
    <meta http-equiv="pragma" content="no-cache">
    <meta http-equiv="cache-control" content="no-cache">
    <meta http-equiv="expires" content="0">    
    <meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
    <meta http-equiv="description" content="This is my page">
    <!--
    <link rel="stylesheet" type="text/css" href="styles.css">
    -->
    <title>Insert title here</title>
  </head>
  <body>
    ${cursor}
  </body>
</html>
```