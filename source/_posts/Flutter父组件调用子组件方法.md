---
title: Flutter父组件调用子组件方法
date: 2020-07-17 21:00:53
tags:
  - Flutter
---

Flutter 中很多灵感来自 React，我们知道 React 中经常会用到 key 的概念，尤其是遍历数组生成组件是。和 React/Vue 中 key 值的含义类似，Flutter 中如果列表 key 不更改，则即便数据又修改视图也没有更改。

Flutter 中 key 有多种：

- ValueKey：以一个值为 key。
- ObjectKey：以一个对象为 key。
- UniqueKey：生成唯一的随机数作为 key。
- PageStorageKey：专用于存储页面滚动位置的 key。
- GlobalKey：每个 GlobalKey 都是一个在整个应用内唯一的 GlobalKey 相对而言是比较昂贵的，如果你并不需要 GlobalKey 的某些特性，那么可以考虑使用 Key、ValueKey、ObjectKey 或 UniqueKey。
<!-- more -->

GlobalKey 的用途： 1.允许 widget 在应用程序中的任何位置更改其 parent 而不丢失其状态。
应用场景：在两个不同的屏幕上显示相同的 widget，并保持状态相同。

2.GlobalKey 唯一定义了某个 element，它使你能够访问与 element 相关联的其他对象，例如 buildContext、state 等。
应用场景：跨 widget 访问状态。

直接上代码：

```dart
/// 子组件
import 'package:flutter/material.dart';

/// 很重要，父组件通过这个 globalKey 调用组件的方法
GlobalKey<_ChildState> globalKey = GlobalKey();

class Child extends StatefulWidget {
  Child({Key key}) : super(key: key);
  @override
  _ChildState createState() => _ChildState();
}
class _ChildState extends State<Child> {
  //子组件方法，父组件中调用
  childMethod(){}
  // ....
}
```

```dart
/// 父组件
import 'package:flutter/material.dart';

class Parent extends StatefulWidget {
  @override
  _ParentState createState() => _ParentState();
}

class _ParentState extends State<Parent> {
  parentMethod(){
    // 父组件中调用子组件方法
    globalKey.currentState.childMethod();
  }

  @override
  Widget build(BuildContext context) {
    return Child(key: globalKey)
  }
}
```
