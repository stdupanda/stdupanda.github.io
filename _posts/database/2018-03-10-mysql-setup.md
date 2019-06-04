---
layout: post
title: MySQL 常见运维问题整理
categories: Database
description: MySQL 常见运维问题整理
keywords: database, mysql, MySQL
---

整理 MySQL 安装部署运维问题解决优化记录。

## 安装

平台为 centos7

### rpm 包方式安装

```shell
yum install wget
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.21-1.el6.x86_64.rpm-bundle.tar
wget http://mirrors.ustc.edu.cn/mysql-ftp/Downloads/MySQL-5.7/mysql-5.7.21-1.el6.x86_64.rpm-bundle.tar
#解压到某个路径，安装全部的 rpm 包
yum install mysql-*.rpm
```

### yum 库方式安装

```shell
# 可参考官网切换版本 https://dev.mysql.com/downloads/repo/yum/
wget https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
rpm -ivh mysql57-community-release-el7-9.noarch.rpm
yum install mysql-server
systemctl start mysqld.service
```

### 配置

常用配置如下：

#### 初次登录密码

安装过程若无提示密码则尝试查看日志： `grep password /var/log/mysqld.log` 即可。

#### 允许远程登录

```sql
-- 修改表
use mysql;
pdate user set host = '%' where user = 'root';
select host, user from user;
-- GRANT授权
GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
FLUSH   PRIVILEGES;
GRANT ALL PRIVILEGES ON dk.* TO 'myuser'@'192.168.1.3' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
FLUSH   PRIVILEGES;
```

### 修改密码

```sql
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('newpass');
FLUSH PRIVILEGES;
-- 或者该
use mysql;
UPDATE user SET Password = PASSWORD('newpass') WHERE user = 'root';
```

#### 丢失密码启动

```shell
mysqld_safe --skip-grant-tables&
```

## 调优

常见优化策略：

### 最大连接数

```sql
-- 临时解决方案，重启失效
SET GLOBAL max_connections = 2000;
SHOW VARIABLES LIKE "max_connections";
```

需要先确认 `linux` 系统本身的 `ulimit -n`，然后修改配置文件：

```shell
max_connections=12345
```

重启 MySQL 即可。
