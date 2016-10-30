---
layout: post
title: jquery常见用法整理
categories: jquery
description: 整理jquery的常见用法
keywords: jquery, javascript
---

工作中经常用到jquery，API之外时不时的需要上网查询一些用法，统一整理一下，随用随更新。


# 使用 noConflict 释放 $ 的控制

noConflict() 方法会释放会 $ 标识符的控制，这样其他脚本就可以使用它了。



- 通过全名替代简写的方式来使用 jQuery

```javascript
$.noConflict();
jQuery(document).ready(function(){
  jQuery("button").click(function(){
    jQuery("p").text("jQuery 仍在运行！");
  });
});
```

- 也可以创建自己的简写。noConflict() 可返回对 jQuery 的引用，您可以把它存入变量，以供稍后使用。请看这个例子：

```javascript
var jq = $.noConflict();
jq(document).ready(function(){
  jq("button").click(function(){
    jq("p").text("jQuery 仍在运行！");
  });
});
```

# select 相关操作

```javascript
$("#id option:selected").text();//获取选中项的文字-text
$("#id").find("option:selected").text();//获取选中项的文字-text
$("#id").val();//获取选中项的值-value
$("#id").find("option[text='val']").attr("selected",true);//设置文字为val的项选中 
$("#id").val("val");//设置值为 val 的项选中
$("#id")[0].selectedIndex = 1;//设置index为1的项选中
$("#id").trigger("change");//触发selectd的change事件
```

# css 样式相关操作

```javascript
$("#id").show();//表示display:block
$("#id").hide();//表示display:none
$("#id").toggle();//切换元素的可见状态。如果元素是可见的，切换为隐藏的；如果元素是隐藏的，切换为可见的。

$("#id").css('display','none');
$("#id").css('display','block');
$("#id")[0].style.display = 'none';

```

# disable 禁用

```javascript
//禁用
$('#id').attr("disabled",true);
$('#id').attr("disabled","disabled");

//恢复
$('#id').attr("disabled",false);
$('#id').removeAttr("disabled");
$('#id').attr("disabled","");
```