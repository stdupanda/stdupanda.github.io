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


