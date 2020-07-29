---
title: '解决:adb server version (xx) doesn''t match this client (xx)'
date: 2020-05-11 09:05:11
tags:
---
如果你在用Android studio，那么很好解决，方法如下：

关闭所有adb有关的进程

```bash
adb kill-server
```

使用Android studio，进入到Android SDK选项界面，如下图，把勾去掉然后Apply，完成后再把勾点上然后Apply，让AS自动帮你下载最新的adb

![Android studio 设置](https://upload-images.jianshu.io/upload_images/3442898-a18b5eae418ceaaf.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

mac 重新安装 adb(brew cask install android-sdk) 报错
`It seems there is already a Binary at '/usr/local/bin/adb'; not linking`

可删除之前的版本重新安装，依次执行下面命令

```bash
rm -rf /usr/local/Caskroom/android-sdk
rm -rf /usr/local/bin/adb
rm -rf /usr/local/bin/fastboot
brew cask install android-sdk
```
