---
title: react-native报错'config.h' file not found
date: 2018-10-18 17:15:54
categories: 
- React-Native
tags:
- React-Native
---
react-native 有时安装新的库或者删除 node_modules/* 重新安装依赖后执行 react-native run-ios 命令时报错 `'config.h' file not found`。使用 XCode 执行如下：

![config.h not found](/images/react-native/config.h_not_found.png)

解决方法：

1. 使用终端进入项目目录 `cd node_modules/react-native/third-party/glog-0.3.4`
2. 执行 ../../scripts/ios-configure-glog.sh
3. 重新 rebuild
