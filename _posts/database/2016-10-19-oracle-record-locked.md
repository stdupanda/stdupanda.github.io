---
layout: post
title: oracle record is locked by another user
categories: Database
description: record is locked by another user
keywords: database, oracle
---

今天连接一个远程oracle服务器操作的比较快但是网比较慢，结果在更新语句时提示 `record is locked by another user`,记录下解决办法。

---

# 查看锁

```sql

SELECT
	t2.username,
	t2. SID,
	t2.serial#,
	t2.logon_time
FROM
	v$locked_object t1,
	v$session t2
WHERE
	t1.session_id = t2. SID
ORDER BY
	t2.logon_time;

```

# 将所有的锁kill掉

```sql

alter system kill session 'sid,serial#'

```

环境 Oracle11G R2