---
layout: post
title: 在 linux 上搭建 git 仓库
categories: git
description: 在 linux 上搭建 git 仓库并配置秘钥登录
keywords: linux, git, repo
---

整理在 linux 上搭建 git 仓库的流程，以及密钥登录的配置。

# 服务器操作

## 安装相关组件

```shell
yum install git git-tools # 建议使用发行版自带的包管理工具安装最新版本 git
git --version
```

## 初始化 linux 账户

```shell
id git
useradd git
passwd git
su git # 以 git 账户登录进行后续操作
cd /home/git
mkdir .ssh
touch .ssh/authorized_keys
chown -R git:git /home/git/.ssh # 确保 git 用户对版本库的权限
git init --bare test.git # 生成 test.git 文件夹，即版本库
```

## 修改 ssh 配置

`sudo vi /etc/ssh/sshd_config` 打开如下注释：

```shell
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile      .ssh/authorized_keys
AuthorizedKeysCommand none
AuthorizedKeysCommandRunAs nobody
```

`/etc/rc.d/init.d/sshd restart` 重启 sshd 服务。

## 导入客户端公钥

将客户端提供的公钥文件内容附加到 `/home/git/.ssh/authorized_keys` 文件中。文件内容一行即为一个客户端公钥。

# 客户端配置

## 生成 RSA 证书

`ssh-keygen -t rsa -b 4096` (`man ssh-keygen`)

此处不再赘述服务器证书的生成过程。需要保证 `id_rsa.pub` 文件内容已存在于服务器的 `authorized_keys` 文件中。

## clone 项目到本地

`git clone git@xxx.com:/home/git/test.git` or

`git clone ssh://git@xxx.com:3721/home/git/test.git`

流程正确无误的情况下即可不用输入任何密码就能把项目 clone 到本地，后续 push 也不用输入密码(若客户端私钥设置了密码则必须手动输入密码)。

# 服务端与客户端的文件权限

```shell
sudo chown -R git:git /home/.ssh/
sudo chmod -R 644 /home/.ssh/
sudo chmod 600 id_rsa
```
# 问题总结

## 已配置公钥但是 clone 仍需要输入密码

- 确认 git 版本库的权限是否正常
- 确认 客户端私钥是否有密码保护

## 如何取消客户端私钥密码？

`openssl rsa -in ~/.ssh/id_rsa -out ~/.ssh/id_rsa_new`

新生成的 id_rsa_new 即不存在密码保护了。

---
`git` 命令参考网站：

[git-scm中文版](https://git-scm.com/book/zh/v2 "https://git-scm.com/book/zh/v2")

[git-scm英文版](https://git-scm.com/book/en/v2 "https://git-scm.com/book/en/v2")
