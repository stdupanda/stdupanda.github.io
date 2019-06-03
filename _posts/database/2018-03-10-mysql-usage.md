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
