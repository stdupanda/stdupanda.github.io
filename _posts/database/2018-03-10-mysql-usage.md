---
layout: post
title: MySQL 使用整理
categories: Database
description: MySQL 使用整理
keywords: database, mysql, MySQL
---

整理记录 MySQL 常见问题。

## 安装配置

平台为 centos7

### 安装

```shell
# rpm 包安装
yum install wget
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.21-1.el6.x86_64.rpm-bundle.tar
wget http://mirrors.ustc.edu.cn/mysql-ftp/Downloads/MySQL-5.7/mysql-5.7.21-1.el6.x86_64.rpm-bundle.tar
#解压到某个路径，安装全部的 rpm 包
yum install mysql-*.rpm

# yum 安装
# 可参考官网切换版本 https://dev.mysql.com/downloads/repo/yum/
wget https://dev.mysql.com/get/mysql57-community-release-el7-9.noarch.rpm
rpm -ivh mysql57-community-release-el7-9.noarch.rpm
yum install mysql-server
systemctl start mysqld.service
```
### 账户登录

初次登录时，若安装过程中无提示密码则尝试查看日志： `grep password /var/log/mysqld.log` 即可。

```sql
-- 允许 root 远程登录
-- 修改表
use mysql;
update user set host = '%' where user = 'root';
select host, user from user;
-- GRANT授权
GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
FLUSH   PRIVILEGES;
GRANT ALL PRIVILEGES ON dk.* TO 'myuser'@'192.168.1.3' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
FLUSH   PRIVILEGES;

-- 修改密码
SET GLOBAL validate_password_policy=0;
SET GLOBAL validate_password_length=1;
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('newpass');
FLUSH PRIVILEGES;
-- 或者
use mysql;
UPDATE user SET Password = PASSWORD('newpass') WHERE user = 'root';
```

```shell
# 丢失密码启动
mysqld_safe --skip-grant-tables &
```

### 调优

#### 最大连接数

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

## 使用问题

### sql脚本里有双引号

比如使用 Navicat 导出的 sql 里有双引号，在导入其他库时出现问题。

```sql
SET SESSION SQL_MODE=ANSI_QUOTES;
```

## 统计/筛选

### 无限级联查询

MySQL 8 之前：

```sql
/*
-- 表结构：
CREATE TABLE `tb` (
`id`  int(11) NOT NULL AUTO_INCREMENT ,
`pid`  int(11) NOT NULL ,
`id_info`  varchar(8) NOT NULL DEFAULT '' ,
PRIMARY KEY (`id`)
)
*/
-- 查询语句：
SELECT
  id,
  pid
FROM
  (SELECT * FROM tb ORDER BY pid, id) s,
  (SELECT @pv :=0) t
WHERE
  FIND_IN_SET(pid, @pv)
AND LENGTH(@pv := concat(@pv, ',', id));
```

MySQL8 开始支持递归查询：

[MySQL Recursive CTE (Common Table Expressions)](https://www.geeksforgeeks.org/mysql-recursive-cte-common-table-expressions/)、[SQK Fiddle](http://sqlfiddle.com/#!9/d74210/1)、[How to create a MySQL hierarchical recursive query](https://stackoverflow.com/questions/20215744/how-to-create-a-mysql-hierarchical-recursive-query)、[A Definitive Guide To MySQL Recursive CTE](http://www.mysqltutorial.org/mysql-recursive-cte/)

```sql
/* 表 products 结构如下：
id | name        | parent_id
19 | category1   | 0
20 | category2   | 19
21 | category3   | 20
22 | category4   | 21
*/
-- 查询语句如下：
WITH RECURSIVE cte (id, name, parent_id) as (
  SELECT     id,
             name,
             parent_id
  FROM       products
  WHERE      parent_id = 0
  UNION ALL
  SELECT     p.id,
             p.name,
             p.parent_id
  FROM       products p
  INNER JOIN cte
          ON p.parent_id = cte.id
)
SELECT * FROM cte;
```

### 按天/月/星期等统计

```sql
SELECT DATE_FORMAT(create_time,'%Y%m%d') days,COUNT(caseid) count FROM tc_case GROUP BY days;
SELECT DATE_FORMAT(create_time,'%Y%u') weeks,COUNT(caseid) count FROM tc_case GROUP BY weeks;
SELECT DATE_FORMAT(create_time,'%Y%m') months,COUNT(caseid) count FROM tc_case GROUP BY months;
```

整理 MySQL 清理重复数据

## 去重

`school_name` 为存在重复的列, `school_id` 是主键.

```sql

-- 单列去重,保存 id 最小的一条
DELETE FROM test
 WHERE school_name IN (SELECT school_name
                         FROM test
                        GROUP BY school_name
                       HAVING COUNT(*) > 1)
   AND school_id NOT IN (SELECT MIN(school_id)
                           FROM test
                          GROUP BY school_id
                         HAVING COUNT(*) > 1);
```

## 多列去重

```sql
-- 多列去重
DELETE FROM test
 WHERE id NOT IN (SELECT name, email, max(id)
                    FROM test
                   GROUP BY name, email
                  HAVING id IS NOT NULL);
-- 新思路
DELETE p1 FROM TABLE p1, TABLE p2
 WHERE p1.name = p2.name
   AND p1.email = p2.email
   AND p1.id < p2.id;
```

https://www.zhihu.com/question/33189744

```sql
-- 查找表中多余的重复记录（多个字段）
select * from vitae a
where (a.peopleId,a.seq) in (select peopleId,seq from vitae group by peopleId,seq having count(*) > 1)

-- 删除表中多余的重复记录（多个字段），只留有rowid最小的记录
DELETE FROM vitae a
WHERE (a.peopleId,a.seq) IN (SELECT peopleId,seq FROM vitae GROUP BY peopleId,seq HAVING COUNT(*) > 1)
AND rowid NOT IN (SELECT MIN(rowid) FROM vitae GROUP BY peopleId,seq HAVING COUNT(*)>1)
```

http://www.jb51.net/article/23964.htm
