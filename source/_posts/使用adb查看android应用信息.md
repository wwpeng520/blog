---
title: 使用adb查看android应用信息
date: 2018-07-15 10:38:16
tags:
- 前端技术
- adb
---
# adb-查看手机安装应用信息

```bash
adb shell dumpsys package <package_name>  //获取全部信息
adb shell dumpsys package <package_name> | grep XXX //获取XXX信息
```

![查看微博相关信息](/images/adb/weibo.png)

![查看微博版本](/images/adb/weibo_version.png)

以下命令可以获取当前应用的包名，以及当前页面所在的 Activity

```bash
adb shell dumpsys window | grep mCurrentFocus
// 或者
adb shell dumpsys window | grep mFocusedWindow
```

![查看微博版本](/images/adb/mCurrentFocus.png)
