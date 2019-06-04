---
layout: post
title: mysql 使用整理
categories: Database
description: mysql 使用整理
keywords: database, mysql
---

整理记录 mysql 常见问题。

## 使用问题

### `sql` 文件内使用双引号出现问题

比如使用 Navicat 导出的 sql 里有双引号，在导入其他库时出现问题。

```sql
SET SESSION SQL_MODE=ANSI_QUOTES;
```

## 统计/筛选

### 无限级联查询

MySQL8 之前：

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

MySQL8开始支持递归查询：

[MySQL | Recursive CTE (Common Table Expressions)](https://www.geeksforgeeks.org/mysql-recursive-cte-common-table-expressions/)、[SQK Fiddle](http://sqlfiddle.com/#!9/d74210/1)、[How to create a MySQL hierarchical recursive query](https://stackoverflow.com/questions/20215744/how-to-create-a-mysql-hierarchical-recursive-query)、[A Definitive Guide To MySQL Recursive CTE](http://www.mysqltutorial.org/mysql-recursive-cte/)

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
select DATE_FORMAT(create_time,'%Y%m%d') days,count(caseid) count from tc_case group by days;
select DATE_FORMAT(create_time,'%Y%u') weeks,count(caseid) count from tc_case group by weeks;
select DATE_FORMAT(create_time,'%Y%m') months,count(caseid) count from tc_case group by months;
```

整理 mysql 清理重复数据

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
delete from test
 where id not in (select name, email, max(id)
                    from test
                   group by name, email
                  having id is not null);
-- 新思路
DELETE p1 from TABLE p1, TABLE p2
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
delete from vitae a
where (a.peopleId,a.seq) in (select peopleId,seq from vitae group by peopleId,seq having count(*) > 1)
and rowid not in (select min(rowid) from vitae group by peopleId,seq having count(*)>1)
```

http://www.jb51.net/article/23964.htm
