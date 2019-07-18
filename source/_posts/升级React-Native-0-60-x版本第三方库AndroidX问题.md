---
title: '升级React-Native-0.60.x版本第三方库AndroidX问题 '
date: 2019-07-17 20:01:04
categories: 
- React-Native
tags:
- React-Native
---
最近 React-Native 升级到了 0.60.x 版本，完成了 Android 和 iOS 平台的一些重大迁移，比如支持 AndroidX 和 CocoaPods。刚好新项目开始没多久，打算新建一个模板，把项目迁移过来。

因为项目使用了一些第三方封装好的库，以往的版本中需要使用`react-native link [package]`的方式引入依赖，新版本中无需次操作，而且执行了 link 的所有依赖必须全部 unlink 才行。[这篇文章](https://github.com/react-native-community/cli/blob/master/docs/autolinking.md)介绍了。
<!-- more -->

因为引入的第三库有的已经支持了 AndroidX，这部分库安装后直接编译可以通过。但是一些库（这里列出我使用的一些库），如：react-native-vector-icons，react-native-share 都未支持。一开始我并不知道哪些库在新版本的 react-native 中使用会出现问题，因为开发一般使用 iOS 模拟器，所以刚开始未发现这个问题。直到安装了所有需要 link 的依赖才发现使用安卓模拟器各种错误。找了很多资料都未能解决，索性把所有的依赖都移除了，一项一项的加然后安装，折腾了很多遍，发现引入这几个库的时候都会报错，报错形式很相似，如下：

```bash
> Task :react-native-share:compileDebugJavaWithJavac FAILED

Deprecated Gradle features were used in this build, making it incompatible with Gradle 6.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/5.4.1/userguide/command_line_interface.html#sec:command_line_warnings
50 actionable tasks: 43 executed, 7 up-to-date
注: /Users/.../node_modules/react-native-device-info/android/src/main/java/com/learnium/RNDeviceInfo/RN
DeviceModule.java使用或覆盖了已过时的 API。
注: 有关详细信息, 请使用 -Xlint:deprecation 重新编译。
注: /Users/.../node_modules/react-native-localize/android/src/main/java/com/reactcommunity/rnlocalize/R
NLocalizeModule.java使用或覆盖了已过时的 API。
注: 有关详细信息, 请使用 -Xlint:deprecation 重新编译。
/Users/.../node_modules/react-native-share/android/src/main/java/cl/json/RNSharePathUtil.java:12: 错误:
 程序包android.support.annotation不存在
import android.support.annotation.NonNull;
                                 ^
/Users/.../node_modules/react-native-share/android/src/main/java/cl/json/RNSharePathUtil.java:13: 错误: 程序包android.support.v4.content不存在
import android.support.v4.content.CursorLoader;
                                 ^
/Users/.../node_modules/react-native-share/android/src/main/java/cl/json/RNSharePathUtil.java:14: 错误: 程序包android.support.v4.content不存在
import android.support.v4.content.FileProvider;
```

找了很多资料，有说更新本地 gradle 版本的，有说更改第三方库的 gradle 版本等信息的，有说把项目转化成支持 AndroidX 的，尝试了都解决不了。

机缘巧合意识到可能就是这个 AndroidX 惹的祸，然后发现 [jetifier](https://github.com/mikehardy/jetifier) 这玩意可以用于来修补 node_modules 库支持 AndroidX，这个工具提供了一个临时解决方案。

好吧，尝试了果然不报错了，又出了一个坑，如释重负。

使用如下：

```bash
yarn add -D jetifier
npx jetify
yarn react-native run-android
```

附A：react-native-vector-icons 图标不显示问题（报红）解决

iOS: ios/[projectname]/info.plist 文件加上

```plist
<key>UIAppFonts</key>
<array>
  <string>AntDesign.ttf</string>
  <string>Entypo.ttf</string>
  <string>EvilIcons.ttf</string>
  <string>Feather.ttf</string>
  <string>FontAwesome.ttf</string>
  <string>FontAwesome5_Brands.ttf</string>
  <string>FontAwesome5_Regular.ttf</string>
  <string>FontAwesome5_Solid.ttf</string>
  <string>Foundation.ttf</string>
  <string>Ionicons.ttf</string>
  <string>MaterialCommunityIcons.ttf</string>
  <string>MaterialIcons.ttf</string>
  <string>Octicons.ttf</string>
  <string>SimpleLineIcons.ttf</string>
  <string>Zocial.ttf</string>
</array>
```

Android: android/app/build.gradle 文件添加

```gradle
apply from: "../../node_modules/react-native-vector-icons/fonts.gradle"
```

附B：@ant-design/react-native 图标（引入 @ant-design/icons-react-native ）不显示问题（报红）解决
方法一、

a. 把 node_modules/@ant-design/icons-react-native/fonts 目录下的 antfill.ttf 和 antoutline.ttf 添加到 Xcode -> Resources folder 目录下
b. ios/[projectname]/info.plist 文件加上

```plist
<key>UIAppFonts</key>
<array>
  <string>antfill.ttf</string>
  <string>antoutline.ttf</string>
</array>
```

方法二、
运行 react-native link @ant-design/icons-react-native 如果不行可在 package.json dependencies 中添加 "@ant-design/icons-react-native": "^1.0.2"，然后执行 link。
