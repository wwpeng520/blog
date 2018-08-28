---
title: Egg.js中使用sequelize事务
date: 2018-08-12 19:16:42
tags:
- 数据库
---
对数据库的操作很多时候需要同时进行几个操作，比如需要同时改动几张表的数据，或者对同一张表中不同行（row）或列（column）做不同操作，比较典型的例子就是用户转账问题（A账户向B账号汇钱）：
1 从A账号中把余额读出来。
2 对A账号做减法操作。
3 把结果写回A账号中。
4 从B账号中把余额读出来。
5 对B账号做加法操作。
6 把结果写回B账号中。
为了数据的一致性，这6件事，要么操作全部成功，要么全部失败回滚。这就是事务的一个特性：原子性。关于事务的四大特性（ACID）这里不做深究。
项目使用的是 Egg+egg-sequelize 模式，查阅了一下 sequelize 的[官方文档](http://sequelize.readthedocs.io/en/v3/docs/transactions/)，使用方法如下：
<!-- more -->

```bash
// 受管理的事务（auto-callback）
return sequelize.transaction(function (t) {
  // 要确保所有的查询链都有return返回
  return User.create({
    firstName: 'Abraham',
    lastName: 'Lincoln'
  }, {transaction: t}).then(function (user) {
    return user.setShooter({
      firstName: 'John',
      lastName: 'Boothe'
    }, {transaction: t});
  });
}).then(function (result) {
  // Transaction 会自动提交
  // result 是事务回调中使用promise链中执行结果
}).catch(function (err) {
  // Transaction 会自动回滚
  // err 是事务回调中使用promise链中的异常结果
});

// 不受管理的事务（then-callback）
return sequelize.transaction().then(function (t) {
  return User.create({
    firstName: 'Homer',
    lastName: 'Simpson'
  }, {transaction: t}).then(function (user) {
    return user.addSibling({
      firstName: 'Lisa',
      lastName: 'Simpson'
    }, {transaction: t});
  }).then(function () {
    return t.commit();
  }).catch(function (err) {
    return t.rollback();
  });
});
````

使用 ES6 的语法如下：

```bash
let transaction;
try {
  transaction = await this.ctx.model.transaction();
  await this.service.xxx.xxx(parms, transaction);
  await this.service.xxx.xxx(parms1, parms2, transaction);
  await transaction.commit();

  return true
} catch (e) {
  await transaction.rollback();

  return false
}
```