---
title: JavaScript 中交换两个数组元素的几种方法
date: 2022-11-23 20:55:35
categories: 
- JavaScript
tags:
- JavaScript
---

交换数组元素位置是开发项目中经常用到的场景，比如需要对列表中一项数据上移或者下移排序。

下面总结一下常用的几种方法。

## 借助临时变量

就像普通的两个变量交换元素一样，需要使用一个临时变量保存其中一个元素（A），然后通过把另一个元素（B）赋值给 A，最后把临时变量保存的赋值给 B，即可实现。代码如下：

```js
function swapElements(array, index1, index2) => {
  const temp = array[index1];
  array[index1] = array[index2];
  array[index2] = temp;
};
```

<!-- more -->

## 用 splice 方法

数组的方法 splice 语法 `splice(start, deleteCount, item1, item2, itemN, ...)`。

start 是指定开始的位置，deleteCount （可选）是要移除的数组元素的个数，后面的（可选）表示要添加进数组的元素。返回值是由被删除的元素组成的一个数组。如果只删除了一个元素，则返回只包含一个元素的数组。如果没有删除元素，则返回空数组。

先上代码：

```js
function swapElements(array, index1, index2) => {
  array.splice(
    index2,
    1,
    ...array.splice(index1, 1, array[index2])
  );
};
```

`array.splice(index1, 1, array[index2])`会将 index1 位置上的元素替换为 index2 位置的元素，同时返回原数组 [array[index1]] （返回的是被删除元素组成的数组，所以在代码中加入了扩展运算符 `...` 将数组转为参数序列）。再利用同样的方式将 index2 位置上的元素替换为被删除的原数组的 array[index1] 的值，完成交换。

还有一种写法如下：

```js
function swapElements(array, index1, index2) => {
  array[index2] = array.splice(index1, 1, array[index2])[0];
};

```

原理同上面的一样，只是直接把`array.splice(index1, 1, array[index2])`返回的原数组的 [array[index1]] 的第 0 项 array[index1]，把它赋值给
array[index2]。

## ES6 的解构赋值交换两个数组元素

我们知道 ES6 提供了一个很方便的方式交换两个变量: `[a, b] = [b, a]`。

同理，可以创建一个可重复使用的函数来处理这个问题，你可以指定数组和你希望交换的两个索引。

```js
function swapElements(array, index1, index2) => {
  [array[index1], array[index2]] = [array[index2], array[index1]];
};
```

用这个方法最简便，推荐使用。
