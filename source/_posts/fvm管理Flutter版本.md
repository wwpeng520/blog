---
title: fvm管理Flutter版本(mac)
date: 2020-04-28 07:29:48
tags:
  - Flutter
---

本地开发 Flutter App 可能需要使用不同版本的 Flutter SDK，fvm 是管理 Flutter SDK 版本的工具。

安装 fvm

```bash
brew tap befovy/taps
brew install fvm
```

安装 Flutter

```bash
fvm install <version>
# fvm install 1.12.13+hotfix.9
# fvm install dev
```

如果本地之前已经安装过 Flutter SDK 了，可以使用以下导入命令，这个命令会把之前 Flutter SDK 安装目录剪切到 fvm 安装目录。移动后的 Flutter SDK 目录是：`/Users/[用户名]/Library/Application Support/fvm/versions/1.12.13+hotfix.9`。

```bash
fvm import <name>
# fvm import 1.12.13+hotfix.9
```

因为之前安装的 Flutter SDK 配置过环境变量，现在 Flutter SDK 文件目录移动了，配置的环境变量就失效了，在终端输入 Flutter 命令也无效了，所以需要配置一下 .zshrc/.bash_profile 的环境变量。把之前的 `export PATH=/[flutter安装目录]/bin:$PATH` 修改成 `export PATH=/Users/[用户名]/Library/Application\ Support/fvm/current/bin:$PATH`。这里使用的是 fvm 下的 current 下的软链接，而不是 `.../fvm/versions/1.12.13+hotfix.9/bin:$PATH`，是因为这里的 Flutter 可以根据我们使用 fvm 指定的版本绑定。

其他常用命令如下

```bash
# 查看帮助
fvm --help

# 列表已安装的版本
fvm list

# 设置当前 Flutter SDK 版本
fvm use <version>

# 显示当前版本信息
fvm current

# 移除指定的版本的 SDK
fvm remove <version>
```

fvm 会在项目中创建一个名为 fvm 的符号链接，该链接链接到已安装的SDK版本，这将使用本地项目 SDK 运行 flutter run 命令。

VSCode 配置
将以下内容添加到settings.json：

```json
"dart.flutterSdkPaths": [
    "fvm"
]
```
