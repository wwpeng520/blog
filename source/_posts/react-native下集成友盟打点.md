---
title: react-native下集成友盟打点
date: 2019-05-21 20:53:21
categories: 
- React-Native
tags:
- React-Native
---
之前的项目中使用的是一个封装好的库 [rn-umeng](https://www.npmjs.com/package/@pangu/react-native-umeng)，虽然热度不高，但也没什么坑，所以就一直用着（极光推送+友盟打点）。
<!-- more -->
这次项目中选用的是友盟的推送和打点，开始我打点依旧使用这个封装好的库，推送试了几个别人封装好的库发现没有作用，折腾几次之后索性放弃这个方案，采用官方的 SDK。
引入官方 SDK 后发现一只报错，貌似是重复引用的意思，因为不熟悉 iOS 和 Android 原生开发，所以索性觉得打点也使用官方提供的 SDK。
这里先集成打点 SDK。

## 下载 SDK

下载地址在[这里](https://developer.umeng.com/sdk/reactnative)。
下载 React-native 版本的会有三个文件夹：

1. Android：公共部分（umeng-common-2.0.1.jar）和 umeng-analytics-8.0.0.jar 等 .jar 文件
2. iOS：*.framework 文件
3. ReactNative：iOS 和 Android 部分的桥接文件等

## Android 集成

[官方文档](https://developer.umeng.com/docs/66632/detail/67587)让我们将 *.jar 文件放在 android/app/libs 目录下。这里我们可以不采用这个方式，直接使用 Android 版自动集成的方式（参考[此处](https://developer.umeng.com/docs/66632/detail/101848)）。

### maven依赖配置

在 android/build.gradle 配置脚本中 buildscript 和 allprojects 段中添加【友盟+】sdk 新 maven 仓库地址：

```gradle
maven { url 'https://dl.bintray.com/umsdk/release' }
```

在 android/app/build.gradle 配置脚本 dependencies 段中添加基础组件库和统计 SDK 库依赖：

```gradle
compile 'com.umeng.umsdk:analytics:8.0.0'
compile 'com.umeng.umsdk:common:2.0.1'
compile 'com.umeng.umsdk:utdid:1.1.5.3'
```

### 桥接文件

将下载的安卓部分桥接文件放入工程中。为便于管理在 android/app/src/main/java/com/[你的项目名] 目录下创建 umeng 文件夹。
分别将下列文件放入文件夹中：

- DplusReactPackage.java
- RNUMConfigure.java
- AnalyticsModule.java
- PushModule.java（集成推送时放入）

打开这几个放入的文件，首行的 `package com.xxx.umeng;` 都需要改成你的项目名称（即 ‘xxx’ 改成你的 android/app/src/main/java/com/[你的项目名] 中中括号内的名称）

### 配置

android/app/src/main/java/com/[你的项目名]/MainApplication.java 添加以下代码

```java
import com.cdforex.umeng.DplusReactPackage;
import com.cdforex.umeng.RNUMConfigure;

@Override
protected List<ReactPackage> getPackages() {
    return Arrays.<ReactPackage>asList(
        new MainReactPackage(),
        new DplusReactPackage(), // <-- ADD
        // ...
    );
}

@Override
public void onCreate() {
    super.onCreate();
    // 在此处调用基础组件包提供的初始化函数 相应信息可在应用管理 -> 应用信息 中找到 http://message.umeng.com/list/apps
    // 参数一：当前上下文context；
    // 参数二：应用申请的Appkey（需替换）；
    // 参数三：渠道名称；
    // 参数四：设备类型，必须参数，传参数为UMConfigure.DEVICE_TYPE_PHONE则表示手机；传参数为UMConfigure.DEVICE_TYPE_BOX则表示盒子；默认为手机；
    // 参数五：Push推送业务的secret 填充Umeng Message Secret对应信息（需替换）
    RNUMConfigure.init( // <-- ADD
        this,
        "5cda22f1570df36e253333",
        "Umeng",
        UMConfigure.DEVICE_TYPE_PHONE,
        "de440d7baefc5047458bb08b4c1fffff"
    );
}
```

android/app/src/main/java/com/[你的项目名]/MainActivity.java 添加以下代码

```java
import com.umeng.analytics.MobclickAgent;

public class MainActivity extends ReactActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        MobclickAgent.setSessionContinueMillis(1000 * 4);
        MobclickAgent.setScenarioType(this, MobclickAgent.EScenarioType.E_UM_NORMAL);
    }

    @Override
    protected void onResume() {
        super.onResume();
        MobclickAgent.onResume(this);
    }

    @Override
    protected void onPause() {
        super.onPause();
        MobclickAgent.onPause(this);
    }
}
```

### 权限授予

在 android/aap/src/main/AndroidManifest.xml 里面配置所需权限：

```xml
<manifest ……>
<uses-sdk android:minSdkVersion="8"></uses-sdk>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
<uses-permission android:name="android.permission.INTERNET"/>
<application ……>
```

### 混淆设置

如果设置了混淆，则需在 android/app/proguard-rules.pro 中配置

```pro
-ignorewarnings
-keep class com.umeng.** {*;}
-keepclassmembers class * {
  public <init> (org.json.JSONObject);
}
-keepclassmembers enum * {
  public static **[] values();
  public static ** valueOf(java.lang.String);
}
-keep public class com.cdforex.R$*{
  public static final int *;
}
```

## iOS 集成

### SDK 文件和桥接文件导入

将上面下载的 SDK 文件中 iOS 目录下的 *.framework 文件集中放在一个目录下，如 UMComponent。集中后把 UMComponent 文件夹拖入 XCode 打开的项目中，选中「Copy items if needed」，导入 XCode 中。
将上面下载的 SDK 文件中 ReactNative 目录下的关于 iOS 的桥接文件（*.h, *.m）文件也拖入项目中。

### 配置Appkey

在 Appdelegate.m 中设置初始化代码

```m
#import "RNUMConfigure.h"
#import <UMCommon/UMCommon.h>
#import <UMPush/UMessage.h>
#import <UserNotifications/UserNotifications.h>
#import <UMCommonLog/UMCommonLogHeaders.h>
#import <UMAnalytics/MobClick.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  [UMCommonLogManager setUpUMCommonLogManager];
  [UMConfigure setLogEnabled:YES];
  [RNUMConfigure initWithAppkey:@"599d6d81c62dca07c5001db6" channel:@"App Store"];
  [MobClick setScenarioType:E_UM_NORMAL];
  ...
}
```

### RN 中调用

在页面中使用

```js
import { NativeModules } from "react-native";
const UMAnalyticsModule = NativeModules.UMAnalyticsModule;

UMAnalyticsModule.onEvent(eventName);

// 带参数的事件
UMAnalyticsModule.onEventObject(eventName, params);
```