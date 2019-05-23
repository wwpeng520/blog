---
title: SQL排序并更新(UPDATE TOP...ORDER BY)
date: 2018-07-12 20:11:38
tags:
- SQL
- 数据库
---
我期望的目标是：查找 Proxies 表中 count 小于10的一条数据，并把 count 加上1，代码如下：

```sql
UPDATE Proxies SET count=count+1 WHERE count<10 ORDER BY createdAt ASC LIMIT 1;
```
<!-- more -->

现实是骨感的，运行发现报错了，提示更新不能与 ORDER BY 一起使用。
解决方案：

```sql
UPDATE "Proxies"
SET "count"="count"+1
WHERE "id" IN (
    SELECT "Proxies"."id"
    FROM "Proxies"
    WHERE "count"<10
    ORDER BY "Proxies"."createdAt"
    LIMIT 1
)
RETURNING "count",...

// 或者
UPDATE "Proxies"
SET "count"="count"+1
FROM (
    SELECT "Proxies"."id"
    FROM "Proxies"
    WHERE "count"<10
    ORDER BY "Proxies"."createdAt"
    LIMIT 1
) AS tab1
WHERE "Proxies"."id"=tab1."id"
```