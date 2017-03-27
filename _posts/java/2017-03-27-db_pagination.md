---
layout: post
title: db分页总结
categories: Java
description: db分页总结
keywords: Java, mysql, oracle, 分页
---

- oracle

```sql

-- 分页查询格式：
SELECT *
  FROM (SELECT A.*, ROWNUM RN
          FROM (SELECT * FROM TABLE_NAME where 1=1 and 2=2) A
         WHERE ROWNUM <= 40)
 WHERE RN >= 21

 /*
其中最内层的查询SELECT * FROM TABLE_NAME表示不进行翻页的原始查询语句。ROWNUM <= 40和RN >= 21控制分页查询的每页的范围。

上面给出的这个分页查询语句，在大多数情况拥有较高的效率。分页的目的就是控制输出结果集大小，将结果尽快的返回。

在上面的分页查询语句中，这种考虑主要体现在WHERE ROWNUM <= 40这句上。
*/
```

- mysql

mysql的方式比较多

```
//传统方式，越往后分页 LIMIT 语句的偏移量就会越大，速度也会明显变慢
SELECT * FROM table LIMIT 5,10;  // 检索记录行 6-15
SELECT * FROM table LIMIT 95,-1; // 检索记录行 96-last.
SELECT * FROM table LIMIT 5;     //检索前 5 个记录行

//使用子查询优化
SELECT * FROM articles WHERE  id >=  (
	SELECT id FROM articles WHERE 1=1 ORDER BY id LIMIT 10000, 1
) LIMIT 10  


//以下是只提供上一页、下一页的链接形式
SELECT * FROM message WHERE id>1020 ORDER BY id ASC LIMIT 20;  //下一页
SELECT * FROM message WHERE id<1000 ORDER BY id DESC LIMIT 20; //上一页

//

```