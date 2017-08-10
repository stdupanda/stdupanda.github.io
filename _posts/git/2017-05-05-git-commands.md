---
layout: post
title: git 常用命令总结
categories: git
description: git 常用命令总结
keywords: Python, Ubuntu, psutil
---

常用的 git 命令总结。

| command                                              | desc |
|:-----------------------------------------------------|:------------|
| **配置部分** ||
| `git config <--global or --system or --local> -l`    | 查看<当前用户/系统/当前库>配置信息 |
| `git clone git@xxx.git`                              | 从远程主机克隆一个版本库 |
| `git remote -v`                                      | 查看 git 远程库地址 |
| `git remote show origin`                             | 查看远程库对应关系 |
| **分支** |
| `git branch`                                         | 查看本地分支               |
| `git branch -a`                                      | 查看本地+远程分支 |
| `git branch testing`                                 | 本地创建 testing 分支         |
| `git branch -d develop`                              | 删除 develop 分支 |
| `git checkout develop`                               | 切换到 develop 分支 |
| `git status`                                         | 查看本地仓库当前状态  |
| `git add <path>`                                     | 将工作文件修改提交到本地暂存区  |
| `git add -A(--all)`                                  | statges **All**       |
| `git add .`                                          | stages new and modified,without deleted      |
| `git add -u`                                         | stages deleted and modified,without new       |
| `git push -u origin develop`                         | 推送本地库当前分支到 origin 远程仓库的 develop 远程分支        |
| `git push -u origin :develop`                        | 删除 origin 远程仓库的 develop 远程分支        |
| `git rm <file>`                                      | 从版本库中删除一个文件       |
| `git reset HEAD file  `                              | 把暂存区的修改撤销掉（unstage）重新放回工作区       |
| `git reset --hard HEAD`                              | 彻底会退到某一个版本，本地源码也会变为上一个源码内容      |
| `git tag`                                            | 列出现有标签      |
| `git tag 20170428.v.1.2`                             | 添加一个轻量级标签        |
| `git push orign 20170428.v.1.2`                      | 推送标签到远程仓库        |
| `git tag -d 20170428.v.1.2`                          | 删除一个标签 |
| `git push orign :refs/tags/20170428.v.1.2`           | 删除远程仓库的标签        |
| `git merge testing`                                  | 把 testing 分支合并到当前分支,不会保留历史分支记录~~_不推荐_~~ |
| `git merge --no-ff testing`                          | 把 testing 分支合并到当前分支,并保存之前的分支历史**推荐用法** |
| `git show`                                           | 查看此标签信息       |
| `git pull`                                           | 同步更新最新代码到本地|
| `git log --graph`                  | 查看分支合并图 |
| `git log --graph --pretty=oneline --abbrev-commit` | 查看分支合并图 |

- `git merge` 冲突解决

合并时若有冲突回提示你 `Unmerged paths` 和 `Untracked files`，并会提示类似格式的冲突文件列表：
```
/xxx/xx.properties~HEAD
/xxx/xx.properties~develop
```
解决方式如下：

```
Git is a distributed version control system.
Git is free software distributed under the GPL.
Git has a mutable index called stage.
Git tracks changes of files.
<<<<<<< branch1
Creating a new branch is quick & simple.
=======
Creating a new branch is quick AND simple.
>>>>>>> branch2
```
在执行了 `git merge branch_name` 之后，合并时有冲突的文件在打开后就出现类似的内容，其中可以看到 `git` 用 `<<<<<<<`，`=======`，`>>>>>>>` 标记出不同分支的内容，我们修改如下后保存：

`Creating a new branch is quick and simple.`

然后提交：`git add xx.properties`,`git commit -m "conflict fixed"`,这样冲突就解决了。

`git log --graph --pretty=oneline --abbrev-commit` 查看分支合并情况。




然后应该分析如何处理其中的问题，进行处理，将 `git` 添加的这部分进行处理，

---

`git` 命令参考网站：

[git-scm中文版](https://git-scm.com/book/zh/v2 "https://git-scm.com/book/zh/v2")

[git-scm英文版](https://git-scm.com/book/en/v2 "https://git-scm.com/book/en/v2")
