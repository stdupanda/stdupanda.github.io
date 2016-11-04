---
layout: post
title: javascript常见问题整理
categories: javascript
description: 整理 javascript 的常见用法
keywords: javascript
---

工作中经常用到javascript，时不时的需要上网查询一些用法，统一整理一下，随用随更新。


# setTimeout setInterval

setTimeout( ) 是属于 window 的 method, 但我们都是略去 window 这顶层物件名称, 这是用来设定一个时间, 时间到了, 就会执行一个指定的 method。

```javascript
setTimeout(code,millisec);//code必需。要调用的函数后要执行的 JavaScript 代码串。millisec必需。在执行代码前需等待的毫秒数。
//setTimeout() 只执行 code 一次。如果要多次调用，请使用 setInterval() 或者让 code 自身再次调用 setTimeout()。

function demo(){
    //do sth
}
setTimeout(demo, 1000);
setTimeout("demo()", 1000);//推荐，可以传参数


setInterval(code,millisec[,"lang"])；//code必需。要调用的函数或要执行的代码串。millisec必须。周期性执行或调用 code 之间的时间间隔，以毫秒计。
var func=self.setInterval("clock()",50);
Window.clearInterval(func);从而取消对 code 的周期性执行的值。
```



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

# 选择器（重要）

## id,class,elemnt

[w3school-jquery_ref_selectors](http://www.w3school.com.cn/jquery/jquery_ref_selectors.asp "w3school-jquery")

```javascript
$("*");//所有元素
$("#lastname");//id="lastname" 的元素
$(".intro");//所有 class="intro" 的元素
$(".intro.demo");//所有 <p> 元素
$("p");//所有 class="intro" 且 class="demo" 的元素
```
## element:

```javascript
$("th,td,.intro");//所有带有匹配选择的元素
$("p:first");//第一个 <p> 元素
$("p:last");//最后一个 <p> 元素
$("tr:even");//所有偶数 <tr> 元素
$("tr:odd");//所有奇数 <tr> 元素
```

## s1,s2,s3

```javascript
$("th,td,.intro");//所有带有匹配选择的元素
jQuery("#div_ChgProcess_PropForm0").find("textarea,input,select").each(function(){
    jQuery(this).attr("disabled","disabled");
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
$("#id").empty();
$("#id").html("");//清空selectd列表
$("#id").append("<option value='Value'>Text</option>");
$("#id").prepend("<option value='0'>请选择</option>");
$("#id option:last").remove();
$("#id option[index='0']").remove();
$("#id option[value='3']").remove();
$("#id option[text='4']").remove();

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