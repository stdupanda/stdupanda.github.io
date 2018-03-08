---
layout: post
title: mysql 去除重复记录
categories: Database
description: mysql 去除重复记录
keywords: database, mysql
---

整理 mysql 清理重复数据

---

`school_name` 为存在重复的列, `school_id` 是主键.

```sql

-- 删除重复记录,保存Id最小的一条
delete FROM test
 WHERE school_name in (SELECT school_name
                         FROM test
                        GROUP BY school_name
                       HAVING COUNT(*) > 1)
   and school_id not in (select min(school_id)
                           from test
                          group by school_id
                         having count(*) > 1);

```