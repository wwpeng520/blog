---
title: react-navigation 安卓标题居中问题
date: 2018-05-18 14:57:44
categories: 
- React-Native
tags:
- React-Native
- 前端技术
---

在做 React Native 应用的时候，我们常常使用 react-navigation 做导航栏，发现 Android 上的标题不居中，IOS 上没问题。
<!-- more -->

1 如果只有标题，那就在 headerTitleStyle 设置 alignSelf:'center' 就可以。
2 如果标题栏左侧还有返回按钮，发现标题偏右依然不居中，则简单的处理方式是:
在右边再添加一个等宽高的空 View，如下:

``headerRight: <View />``

升级新版本之后发现这招不灵了，可以在 headerTitleStyle 里设置 flex:1, textAlign: 'center' 来实现。
