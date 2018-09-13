---
title: SQL语句中HAVING与WHERE的区别
date: 2018-07-23 17:22:57
tags:
- SQL
- 数据库
---
在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与聚合函数一起使用，HAVING 子句可以让我们筛选分组后的各组数据。

SQL HAVING 语法：

```sql
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
HAVING aggregate_function(column_name) operator value;
```

<!-- more -->

从上面的语法中可以看出 HAVING 和 WHERE 是不冲突，作用不一样的。
WHERE 是一个约束声明，使用 WHERE 约束来自数据库的数据，WHERE 是在结果返回之前起作用的，WHERE 中不能使用聚合函数。
HAVING 是一个过滤声明，是在查询返回结果集以后对查询结果进行的过滤操作，在 HAVING 中可以使用聚合函数。
在查询过程中聚合语句(sum,min,max,avg,count)要比 HAVING 子句优先执行。而 WHERE 子句在查询过程中执行优先级高于聚合语句。

即当同时含有 WHERE 子句、GROUP BY 子句 、HAVING 子句及聚集函数时，执行顺序如下：

- 执行 WHERE 子句查找符合条件的数据；
- 使用 GROUP BY 子句对数据进行分组；对 GROUP BY 子句形成的组运行聚集函数计算每一组的值；
- 最后用 HAVING 子句去掉不符合条件的组。

## 栗子🌰

拥有下面这个 "Orders" 表：
![Orders](/images/sql/Orders表.png)
现在，我们希望查找订单总金额少于 2000 的客户。

```sql
SELECT Customer,SUM(OrderPrice) FROM Orders
GROUP BY Customer
HAVING SUM(OrderPrice)<2000
```

结果如下：
![结果](/images/sql/Orders_result1.png)

现在我们希望查找客户 "Bush" 或 "Adams" 拥有超过 1500 的订单总金额。

```sql
SELECT Customer,SUM(OrderPrice) FROM Orders
WHERE Customer='Bush' OR Customer='Adams'
GROUP BY Customer
HAVING SUM(OrderPrice)>1500
```

结果如下：
![结果](/images/sql/Orders_result2.png)
