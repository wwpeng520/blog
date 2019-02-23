---
title: react-native中使用realm数据库
date: 2018-09-22 15:23:51
categories: 
- React-Native
tags:
- React-Native
- 数据库
---

因为项目需要用到 APP 本地存取数据库的功能，所以采用了 [realm](https://github.com/realm/realm-js) 的方案，官方文档地址：[https://realm.io/docs/javascript/latest/](https://realm.io/docs/javascript/latest/)。
<!-- more -->

## react-native 存储方案

- AsyncStorage

react-native 官方存储方案是 [AsyncStorage](https://reactnative.cn/docs/asyncstorage/)。AsyncStorage 是一个简单的、异步的、持久化的 Key-Value 存储系统，它对于 App 来说是全局性的。可用来代替 LocalStorage。官方推荐使用者在 AsyncStorage 的基础上做一层抽象封装，而不是直接使用 AsyncStorage。

- AsyncStorage 封装库

[react-native-storage](https://github.com/sunnylqm/react-native-storage) 是一个比较好的框架，我们可以直接使用它，方法很简单，说明文档中说得很详细。其他库还有 [react-native-simple-store](https://github.com/jasonmerino/react-native-simple-store) 等。

- realm

realm 是一款专为移动 ​ 端开发的高性能数据库，其宣称自己是最快的 react-native 数据库。
realm 整体的优点有这么四点：1.简单易用，2.跨平台，3.快速性能优越，4.提供高级功能。realm 核心数据引擎用 C++ 打造，并不是建立在 SQLite 之上的 ORM。因此性能就是比普通的 ORM 要快很多，甚至比单独无封装的 SQLite 还要快。同时因为是 ORM，本身在设计时也针对移动设备（iOS、Android），所以简单易用，学习成本很低。

## realm 常用操作

1、 作为数据库，使用它无法就是「增删改查」等基本操作，使用之前，需要定义 model：

- name：表格名称。
- primaryKey：主键，这个属性的类型可以是 'int' 和 'string'，并且如果设置主键之后，在更新和设置值的时候这个值必须保持唯一性，并且无法修改。声明主键可以有效地查找和更新对象，并为每个值强制实现唯一性。
- properties：这个属性内放置我们需要的字段。

```typescript
// models/hello.ts
export const HelloSchema = {
  name: "Hello",
  primaryKey: "uid", // 定义主键后，无法创建同一主键的数据
  properties: {
    uid: "string",
    name: "string", // {type: 'string'} 的简写
    phone: { type: "string", default: "136xxxxxxxx" }
  }
};
```

2、 初始化

```ts
// 根据提供的表初始化 Realm，可同时往数组中放入多个表
let realm = new Realm({ schema: [HelloSchema] });
```

3、 增加数据
对域中对象的更改、创建、更新和删除，必须在 write（）事务块中进行。
需要注意：写入事务具有不可忽略的开销，应该尽量减少代码中写入块的数量。

```ts
try {
  realm.write(() => {
    realm.create("Hello", {
      uid: "a371d56d7b6f77ba31f71d22",
      name: "名字1",
      phone: "137xxxxxxxx"
    });
    realm.create("Hello", {
      uid: "a371d56d7b6f77ba31f71d22",
      name: "名字1",
      phone: "137xxxxxxxx"
    });
    // ...
  });
catch(e) {
  console.log("Error on creation");
}
```

定义 model 时我们使用了 primaryKey，如果我们写入 primaryKey 之前的数据中已存在时，我们写入会出错，并且如果此时我们写入的是多个数据时，会导致后面的写入失败，所以我们可以在单个 realm.create('Hello', item) 的外面使用一个 try...catch...。

```ts
const result = await new Promise((resolve, reject) => {
  try {
    realm.write(() => {
      for (let item of data) {
        try {
          realm.create("Hello", item);
        } catch (e) {
          console.log("write error: ", e);
        }
      }
      resolve("ok");
    });
  } catch (e) {
    console.log("write error: ", e);
    resolve("error");
  }
});
console.log("write result: ", result);
```

4、 查找和删除数据
查询数据允许从 realm 获取单个类型的对象，并可选择过滤和排序这些结果。最基本方法是使用 objects 方法获取给定类型的所有对象。
`let dogs = realm.objects('Dog'); // retrieves all Dogs from the Realm`

- Filtering 参数可过滤查找结果：`let dogs = realm.objects('Dog').filtered('color = "tan" AND name BEGINSWITH "B"');`
- Sorting 参数允许根据单个或多个属性指定排序条件和顺序：

> let hondas = realm.objects('Car').filtered('make = "Honda"');
// Sort Hondas by mileage
let sortedHondas = hondas.sorted('miles');
// Sort in descending order instead
sortedHondas = hondas.sorted('miles', true);
// Sort by price in descending order and then miles in ascending
sortedHondas = hondas.sorted([['price', true], ['miles', false]]);

- Limiting 参数提供了对查询结果进行“分页”的能力。这通常是为了避免从磁盘中读取太多内容，或者一次将太多结果拖入内存中。由于 realm 中的查询是惰性的，因此根本不需要执行这种分页行为，因为 realm 只会在显式访问后从查询结果中加载对象。`let cars = realm.objects('Car'); let firstCars = cars.slice(0, 5);`

```ts
// 删除单个数据
let result = await new Promise((resolve, reject) => {
  try {
    realm.write(() => {
      for (let uid of data) {
        try {
          let dataRource = realm.objects("Hello").filtered(`uid = "${uid}"`);
          realm.delete(dataRource);
          console.log("delete: ", uid);
        } catch (e) {
          console.log("delete error: ", uid);
        }
      }
      resolve("delete ok!");
    });
  } catch (e) {
    console.log("delete error: ", e);
    resolve("delete error!");
  }
});
console.log("delete done!: ", result);

// delete all Hello
let result = await new Promise((resolve, reject) => {
  try {
    let dataRource = realm.objects("Hello");
    realm.write(() => {
      realm.delete(dataRource);
      resolve("delete done!");
    });
  } catch (e) {
    resolve("delete error!");
  }
});
console.log("delete done!:", result);
```

5、 修改数据
如果定义 model 时包含主键，则可以让 realm 根据其主键值智能地更新或添加对象。更新时将 true 作为第三个参数传递给 create 方法。

```ts
realm.write(() => {
  realm.create("Hello", {
    uid: "a371d56d7b6f77ba31f71d22",
    name: "Recipes",
    phone: "135xxxxxxxx"
  });

  // Update book with new price keyed off the id
  realm.create(
    "Hello",
    { uid: "a371d56d7b6f77ba31f71d22", phone: "138xxxxxxxx" },
    true
  );
});
```

## 使用报错记录

1.安卓打包安装进入闪退
在 iOS 模拟器和安卓真机调试都是正常的，打包之后安装打开一进入就闪退，通过友盟的错误统计发现报了几个错误（现在找不到记录了 😓）。
GitHub 里面有对应的这些 issue，但是没有人能准确的定位错误，按照别人的建议都没能解决。后来找到一个：
[`java.lang.ClassNotFoundException: Didn't find class on path: DexPathList - io.realm.react.utils.SSLHelper`](https://github.com/realm/realm-js/issues/1896)
试着在 android/app/proguard-rules.pro 加上 `-keep class io.realm.react.**`，终于解决了，原来是打包的时候被混淆引起的。

2.删除操作记录时报错
`Accessing object of type X which has been invalidated or deleted`
删除部分数据后，必须更新 store ，如果需要显示删除后的数据也必须先把 store 中显示的 realm 数据置为空，然后再去重新查询。

3.安卓调试模式报错
`DOMException: Failed to execute 'open' on 'XMLHttpRequest': Invalid URL`
这个可以在[这个](https://github.com/realm/realm-js/issues/578)issue 上找到答案。
