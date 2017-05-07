---
layout: post
title: git 常用命令总结
categories: git
description: git 常用命令总结
keywords: Python, Ubuntu, psutil
---

常用的 git 命令总结。

| command                                            | desc |
|:---------------------------------------------------|:------------|
| git clone git@xxx.git                              | 从远程主机克隆一个版本库 |
| git branch                                         | 查看本地分支               |
| git branch -a                                      | 查看本地+远程分支 |
| git branch testing                                 | 本地创建 testing 分支         |
| git checkout develop                               | 切换到 develop 分支  |
| git status                                         | 查看本地仓库当前状态  |
| git add <path>                                     | 将工作文件修改提交到本地暂存区  |
| git add -A(--all)                                  | statges **All**       |
| git add .                                          | stages new and modified,without deleted      |
| git add -u                                         | stages deleted and modified,without new       |
| git push -u origin develop                         | 推送本地库当前分支到origin远程仓库的develop远程分支        |
| git rm <file>                                      | 从版本库中删除一个文件       |
| git tag                                            | 列出现有标签      |
| git tag 20170428.v.1.2                             | 添加一个轻量级标签        |
| git show                                           |  查看此标签信息       |



                                                                                                                                                                                                                                                             
