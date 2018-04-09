---
layout: post
title: mysql 去除重复记录
categories: Database
description: mysql 去除重复记录
keywords: database, mysql
---

整理 mysql 清理重复数据

# 单列去重
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

# 多列去重

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
