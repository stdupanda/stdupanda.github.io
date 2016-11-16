---
layout: page
title: 关于
description: 不断学习不断进步
keywords: 关于
comments: false
menu: 关于
permalink: /about/
---

用 github 的博客记录自己工作和生活的点点滴滴


## 时常记住

* 但行好事，莫问前程
* 傲慢与偏见
* 静坐独思己过，闲谈莫论人非
* 努力改变人生
* 修合无人见，存心有天知

## 联系

* GitHub：[@stdupanda](https://github.com/stdupanda)
* 博客：[{{ site.title }}]({{ site.url }})
* QQ：笑口常开 1197591680

## Skill Keywords

#### Software Engineer Keywords
<div class="btn-inline">
    {% for keyword in site.skill_software_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>

#### Mobile Developer Keywords
<div class="btn-inline">
    {% for keyword in site.skill_mobile_app_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>

#### Windows Developer Keywords
<div class="btn-inline">
    {% for keyword in site.skill_windows_keywords %}
    <button class="btn btn-outline" type="button">{{ keyword }}</button>
    {% endfor %}
</div>

---

[jekyll博客主题们](http://jekyllthemes.org/)

Fored from [mzlogin](https://github.com/mzlogin/mzlogin.github.io) 2016-09-19, 基于 [DONGChuan](http://dongchuan.github.io/) 修改。

其中 Wiki 部分保留原作者内容。