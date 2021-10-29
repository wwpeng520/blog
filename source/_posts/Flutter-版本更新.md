---
title: Flutter 版本更新
date: 2020-08-21 09:40:40
tags:
  - Flutter
---

Flutter 更新迭代很快，老项目贸然升级新版本的可能会有些坑，而新项目可能希望使用新版本的 Flutter 开发。

使用 fvm 管理 Flutter 版本还比较方便，只是 fvm 安装 Flutter 只能 `fvm install [master/stable/dev/beta]`，而不能直接指定版本号。不过可以直接去 Flutter 官网下载对应的 SDK，文件夹名称改成对应的版本（如1.20.2），然后拷贝到 fvm 安装目录的 version 下，如`/Users/[你的用户]/Library/Application Support/fvm/versions/1.20.2`。然后执行`fvm list`就可以看到本地安装的版本，使用`fvm use 1.20.2`就可以设置当前的 Flutter 版本。（安装 fvm 的时候需要配置 .zshrc 或 .bashrc/.bash_profile`export PATH=/Users/[你的用户]/Library/Application\ Support/fvm/current/bin:$PATH`）

Android Studio 配置：Preferences - Languages & Frameworks - Dart 的 Dart SDK path配置`/Users/[你的用户]/Library/Application Support/fvm/versions/1.20.2/bin/cache/dart-sdk`。

然后项目文件夹下的 .packages 需要删除，然后重新 pub 安装依赖
