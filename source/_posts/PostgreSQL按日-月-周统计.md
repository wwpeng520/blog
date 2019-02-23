---
title: PostgreSQL按日/月/周统计
date: 2019-01-14 23:00:53
tags:
- SQL
- 数据库
---

先看一个 SQL 语句

```sql
SELECT TO_CHAR("createdAt", 'YYYY-MM-DD') AS date, count(*)
FROM "Users"
WHERE
    "Users"."createdAt">'2018-01-13 00:00:00.000+08' 
    AND "Users"."createdAt"<'2019-01-12 00:00:00.000+08'
GROUP BY date
```

<!-- more-->
上面的 SQL 语句`TO_CHAR("createdAt", 'YYYY-MM-DD')`把 createdAt 字段转化成 YYYY-MM-DD 格式的字符串，即按日期统计数量。同理：`TO_CHAR("createdAt", 'YYYY-MM')`为按月份统计数量；`TO_CHAR("createdAt", 'YYYY')`为按年份统计数量。
输出结果如下：

    // YYYY-MM-DD
    [ { date: '2018-09-13', count: '4' },
    { date: '2018-10-14', count: '1' },
    { date: '2018-06-01', count: '2' },
    { date: '2018-06-05', count: '1' },
    { date: '2018-08-10', count: '2' } ]

    // YYYY-MM
    [ { date: '2018-06', count: '3' },
    { date: '2018-10', count: '1' },
    { date: '2018-08', count: '2' },
    { date: '2018-09', count: '4' } ]

    // YYYY
    [ { date: '2018', count: '10' } ]

说明：
TO_CHAR 是把日期或数字转换为字符串
TO_DATE 是把字符串转换为数据库中得日期类型转换函数
TO_NUMBER 将字符转化为数字

```sql
SELECT EXTRACT (WEEK from "Users"."createdAt") AS week, count(*)
FROM "Users"
WHERE
    "Users"."createdAt">'2018-01-13 00:00:00.000+08'
    AND "Users"."createdAt"<'2019-01-12 00:00:00.000+08'
GROUP BY "Users"."createdAt"
```

结果如下：

    [ { week: 37, count: '1' },
    { week: 22, count: '1' },
    { week: 32, count: '1' },
    { week: 22, count: '1' },
    { week: 37, count: '1' },
    { week: 37, count: '1' },
    { week: 23, count: '1' },
    { week: 37, count: '1' },
    { week: 32, count: '1' },
    { week: 41, count: '1' } ]

可以看到结果中列出了 createdAt 的 week （当年的第几个周），但是并没有达到预期，即把相同 week 的数量统计在一起，改进如下：

```sql
SELECT week, count(count)
FROM (
    SELECT EXTRACT (WEEK FROM "Users"."createdAt") AS week, count(*)
    FROM "Users"
    WHERE
        "Users"."createdAt">'2018-01-13 00:00:00.000+08'
        AND "Users"."createdAt"<'2019-01-12 00:00:00.000+08'
    GROUP BY "Users"."createdAt"
) AS UserCount
GROUP BY week
```

结果如下：

    [ { week: 22, count: '2' },
    { week: 23, count: '1' },
    { week: 32, count: '2' },
    { week: 41, count: '1' },
    { week: 37, count: '4' } ]

extract 函数可以方便的获取日期中的某一部分值，如：日期、月份、年、年中的第几天等。函数格式如下：
source必须是timestamp、time、interval类型的值表达式。field是一个标识符或字符串，是从源数据中的抽取的域

```sql
extract （field from source）
```

| field 值 | 描述 |
| ------ | ------ |
| DAY | 日期（本月的第几天） |
| MONTH | 月份 |
| YEAR | 年份 |
| DOY（day of year） | 年中的第几天 |
| DOW（day of week） | 星期几，周日：0，周一：1，周二：2，... |
| QUARTER | 季度 |
| WEEK | 当前是当年的第几个周 |
| HOUR | 小时 |
| MIN | 分钟 |
| SEC | 秒 |
| CENTURY | 世纪 |
| DECADE | 年份除10的值 |
| MILLENNIUM | 第几个千年，0-1000第一个，1001-2000第二个，2001-3000第三个 |
