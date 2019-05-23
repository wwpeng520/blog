---
title: react-native下集成友盟推送
date: 2019-05-22 20:15:12
categories: 
- React-Native
tags:
- React-Native
---
上一篇{% post_link react-native下集成友盟打点 %}介绍了集成友盟打点的 SDK，这里继续集成友盟推送。下载 SDK 部分和上篇一样，这里略过了。
<!-- more -->
## Android 集成

maven 依赖配置和上篇一致。

在 android/app/build.gradle 配置脚本 dependencies 段中添加基础组件库和统计 SDK 库依赖：

```gradle
dependencies {
  compile 'com.umeng.umsdk:analytics:8.0.0'
  compile 'com.umeng.umsdk:common:2.0.1'
  compile 'com.umeng.umsdk:push:5.0.2'
}
```

### 桥接文件

将下载的安卓部分桥接文件放入工程中。为便于管理在 android/app/src/main/java/com/[你的项目名] 目录下创建 umeng 文件夹。
分别将下列文件放入文件夹中：

- DplusReactPackage.java
- RNUMConfigure.java
- PushModule.java

打开这几个放入的文件，首行的 `package com.xxx.umeng;` 都需要改成你的项目名称（即 ‘xxx’ 改成你的 android/app/src/main/java/com/[你的项目名] 中中括号内的名称）。

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

    // ...

    RNUMConfigure.init( // <-- ADD
        this,
        "5cda22f1570df36e253333",
        "Umeng",
        UMConfigure.DEVICE_TYPE_PHONE,
        "de440d7baefc5047458bb08b4c1fffff"
    );
    mPushAgent = PushAgent.getInstance(this);
    // 如果包名和 applicationId 不一致需要使两者一致，或者调用以下方法，下面的 com.xxxxx 是当前包名
    mPushAgent.setResourcePackageName("com.xxxxx");
    mPushAgent.register(new IUmengRegisterCallback() {
      @Override
      public void onSuccess(String deviceToken) {
        // 注册成功会返回deviceToken，deviceToken是推送消息的唯一标志
        Log.i("TAG", "mPushAgent.register0 注册成功：deviceToken：--------> " + deviceToken);
      }

      @Override
      public void onFailure(String s, String s1) {
        Log.e("TAG", "mPushAgent.register0 注册失败：--------> " + "s:" + s + ",s1:" + s1);
      }
    });

    PushAgent.getInstance(this).onAppStart();
}
```

其他配置可参考[官方文档](https://developer.umeng.com/docs/66632/detail/67587)。

### 消息处理

从官方下载的桥接文件只有六个方法：addTag、deleteTag、listTag、addAlias、addExclusiveAlias、deleteAlias、appInfo，并没有监听消息内容的方法。

发送消息有「打开指定页面」的方法，输入指定页面可以打开指定页面。我不懂安卓开发，经我连接手机使用命令 `adb shell dumpsys window | grep mCurrentFocus` 测试当前应用只有 MainActivity 页面。所以这个方法应该是针对原生开发的安卓应用的。

那如果我需要点击状态栏消息跳转进入 App 指定页面该怎么实现呢？最后决定在原生里面自己添加对应方法。

这里我参考了人家封装的直接可以集成在 react-native 项目的库的方法。参考代码在[这里](https://github.com/yolinsoft/react-native-dk-umeng/blob/master/android/src/main/java/com/dk/umeng/PushApplication.java)。主要代码在下面

```java
// android/app/src/main/java/com/[你的项目名]/umeng/PushApplication.java

UmengNotificationClickHandler notificationClickHandler = new UmengNotificationClickHandler() {
  @Override
  public void launchApp(Context context, UMessage msg) {
      super.launchApp(context, msg);
      clikHandlerSendEvent(PushModule.DidOpenMessage, msg);
  }

  @Override
  public void openUrl(Context context, UMessage msg) {
      super.openUrl(context, msg);
      clikHandlerSendEvent(PushModule.DidOpenMessage, msg);
  }

  @Override
  public void openActivity(Context context, UMessage msg) {
      super.openActivity(context, msg);
      clikHandlerSendEvent(PushModule.DidOpenMessage, msg);
  }

  @Override
  public void dealWithCustomAction(Context context, UMessage msg) {
      super.dealWithCustomAction(context, msg);
      clikHandlerSendEvent(PushModule.DidOpenMessage, msg);
  }
};
```

```java
// android/app/src/main/java/com/[你的项目名]/umeng/PushModule.java

public class PushModule extends ReactContextBaseJavaModule implements LifecycleEventListener {

  // 定义监听的事件
  protected static final String DidReceiveMessage = "didReceiveMessage";
  protected static final String DidOpenMessage = "didOpenMessage";

}
```

android/app/src/main/java/com/[你的项目名]/MainApplication.java 添加以下代码

```java
import com.xxx.umeng.PushApplication;

public class MainApplication extends PushApplication implements ShareApplication, ReactApplication {
  // ...

  @Override
  public void onCreate() {
    super.onCreate();

    // ...

    RNUMConfigure.init( // <-- ADD
        this,
        "5cda22f1570df36e253333",
        "Umeng",
        UMConfigure.DEVICE_TYPE_PHONE,
        "de440d7baefc5047458bb08b4c1fffff"
    );
    mPushAgent = PushAgent.getInstance(this);
    // 如果包名和 applicationId 不一致需要使两者一致，或者调用以下方法，下面的 com.xxxxx 是当前包名
    mPushAgent.setResourcePackageName("com.xxxxx");
    mPushAgent.register(new IUmengRegisterCallback() {
      @Override
      public void onSuccess(String deviceToken) {
        // 注册成功会返回deviceToken，deviceToken是推送消息的唯一标志
        Log.i("TAG", "mPushAgent.register0 注册成功：deviceToken：--------> " + deviceToken);
      }

      @Override
      public void onFailure(String s, String s1) {
        Log.e("TAG", "mPushAgent.register0 注册失败：--------> " + "s:" + s + ",s1:" + s1);
      }
    });

    PushAgent.getInstance(this).onAppStart();
  }
}
```

## iOS 集成

因为 iOS 部分是搞 iOS 的同学给我写的方法，所以没有去研究了，不懂的同学可以参考[这里](https://github.com/yolinsoft/react-native-dk-umeng/blob/master/ios/RNUMCommon/UMPushModule.m)。

关于 iOS 推送比较麻烦一些，需要配置证书，可以参考一下[这篇文章](https://www.jianshu.com/p/a582c4a423d9)。

### RN 中调用

在页面中使用

```js
import { NativeModules, NativeEventEmitter, DeviceEventEmitter } from "react-native";

// 上面定义了 didOpenMessage 和 didReceiveMessage 事件
DeviceEventEmitter.addListener("didOpenMessage", (result) => {
  // 点击状态栏消息时监听消息内容
});

DeviceEventEmitter.addListener("didReceiveMessage", (result) => {
  // 收到消息时操作
});
```