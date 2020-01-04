---
layout: post
title: 分布式业务幂等性设计
categories: 架构
description: 分布式系统业务幂等性相关整理
keywords: 架构, architect, cache, 缓存, redis, lock, 分布式, 分布式事务, 事务, 数据库, 幂等
---

## 幂等性理论

如果一个业务多次执行和只执行一次的影响是相同的，那么这个业务就是幂等性的。

## 实现方案

多个方向、多种方案可以实现接口的幂等，需要结合实际场景来决定具体策略。

### 唯一索引

在表中设计一个字段作为`唯一索引`实现插入去重。当然根据业务也可以采用`联合`唯一索引，当然列越少越好。

### 使用 `token`

典型例子就是防止表单重新提交。

- 后端生成一个带有效期的 `token` 返回前端
- 前端将 `token` 字段放入表单中，然后再将业务数据提交给后台
- 后端验证 `token` 有效期通过后则进行 delete，若操作成功则生成新的 `token` 返回给前端
- **提前申请**、**单次有效**

若先查询再 delete，存在并发的危险；当然可以采用分布式锁的方式进行 `token` 的 delete，但是这样做增加了复杂度。

### 乐观锁和悲观锁

从数据库层面实现数据加锁，业务场景允许的情况下采用乐观锁和悲观锁进行数据更新防重。

```sql
-- 乐观（id 最好是主键或唯一索引）
update tb set col='new val',ver='new version'
  where id='id' and col='old val' and ver='old version';
-- 悲观
select col from tb where id='id' for update;
update tb set col='new val' where id='id';
commit;
```

### 分布式锁

采用分布式锁对系统业务要求较高，一方面系统需要能处理分布式锁调用异常场景下的各种问题，一方面分布式锁也引入了性能消耗；

常见的分布式锁可以采用 redis、zookeeper 等。更多可看下之前的文章 [分布式锁设计](/distributed-lock)。

## 对外 API 接口幂等设计

### 必传字段

两个必传的字段 `source` 来源、`seq` 序列号；两个字段作为防重判断的依据条件。

### 内部业务

内部业务将这两个字段进行组合，作为请求判读的依据，采用上述的各种方案即可实现幂等接口设计。
