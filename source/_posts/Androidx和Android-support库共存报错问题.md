---
title: Androidx和Android support库共存报错问题
date: 2019-07-15 20:34:19
tags:
---
今天使用安卓真机安装项目，发现报错如下。因为之前也遇到这个错误，没有记录下来，这里记录一下解决步骤，方便以后查阅。
<!-- more -->

```bash
Task :app:processDebugManifest FAILED /Users/.../android/app/src/debug/AndroidManifest.xml:22:18-91 Error:Attribute application@appComponentFactory value=(android.support.v4.app.CoreComponentFactory) from [com.android.support:support-compat:28.0.0] AndroidManifest.xml:22:18-91 is also present at [androidx.core:core:1.0.0] AndroidManifest.xml:22:18-86 value=(androidx.core.app.CoreComponentFactory).
Suggestion: add 'tools:replace="android:appComponentFactory"' to <application> element at AndroidManifest.xml:7:5-117 to override.

See http://g.co/androidstudio/manifest-merger for more information about the manifest merger.
```

项目使用 react-native 开发的，本人对安卓开发不熟悉，所以使用我最容易理解和解决的方式。

经查阅资料知道是 Androidx 和 Android support 库不能共存导致的。（上面的提示中也显示 `support-compat:28.0.0` is also present at `androidx.core:core:1.0.0`）

解决方案：转换成 Android Support

1. 执行 `cd android && ./gradlew :app:dependencies` 查看那些库依赖了 Androidx
![/app.dependencies](/images/react-native/app.dependencies.png)

2. 解决依赖问题
找到依赖了 Androidx 的第三方库，查找官方 issue 是否有此类解决方案，或者是否是低版本的问题（可升级到新版本或特定版本）。
这里发现是 react-native-device-info 库的问题，升级到新版本就可以解决了。

其他解决方案：
转换成 Androidx。这里不做展示了，可以参考[这篇文章](https://www.jianshu.com/p/f7a7a8765294)。
