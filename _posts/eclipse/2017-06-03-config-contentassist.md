---
layout: post
title: Eclipse 设置 自动提示
categories: Eclipse
description: Eclipse 设置 自动提示
keywords: shortkey, eclipse, 模板, Eclipse
---

在使用 eclipse 的代码提示快捷键时怪怪的。

正常情况下应该是按下快捷键后会弹出待选列表，结果现在是直接补全代码，不弹出选择框了。

解决方法如下：

`Prefrences` -> `General` -> `Keys` -> 选中 `Content Assist` -> 修改 `When` -> 改为 `Editing text`

保存之后再试了一下，舒服了。
