---
title: react-native下使用原生SDK-Android
date: 2019-04-21 15:32:01
categories: 
- React-Native
tags:
- React-Native
---
项目中需要接入国外的一款客服系统（Zendesk Chat），使用 Web 版打开在线客服体验不好，等待时间需要10秒左右，难以接受，所以周末试着接入原生 SDK。先从安卓版 SDK 开始，SDK 文档在[这里](https://developer.zendesk.com/embeddables/docs/android-chat-sdk/introduction)。

## 安卓 SDK

文档中介绍可以使用 Gradle 或 Maven 系管理系统安装 SDK。

1、在 android/build.gradle 文件中 repositories 下面加上 `maven { url 'https://zendesk.jfrog.io/zendesk/repo' }`。

2、在 android/app/build.gradle 文件中 dependencies 下加上 `compile group: 'com.zopim.android', name: 'sdk', version: '1.4.4'`。

3、在 android/app/src/main/java/com/xxx 目录下创建文件夹 zendeskchat 用来存放 Zendesk Chat 相关的文件。

4、在 3 中目录下新建 ZendeskChatModule.java 文件内容如下：(以下所有的 com.xxx 需要用实际项目名替换)

```java
package com.xxx.zendeskchat;

import android.content.Intent;
import android.app.Activity;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactContext;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReadableArray;
import com.facebook.react.bridge.ReadableMap;

import com.zopim.android.sdk.api.ZopimChat;
import com.zopim.android.sdk.prechat.ZopimChatActivity;
import com.zopim.android.sdk.model.VisitorInfo;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;

public class ZendeskChatModule extends ReactContextBaseJavaModule {

  public ZendeskChatModule(ReactApplicationContext reactContext) {
      super(reactContext);
  }

  @Override
  public String getName() {
      return "zendeskchat";
  }

  @ReactMethod
  public void initZopimChat(String key) {
    ZopimChat.init(key);
  }

  @ReactMethod
  public void startZopimChat(ReadableMap customFields) {
    VisitorInfo.Builder builder = new VisitorInfo.Builder();

    if (customFields.hasKey("name")) {
        builder.name(customFields.getString("name"));
    }
    if (customFields.hasKey("email")) {
        builder.email(customFields.getString("email"));
    }
    if (customFields.hasKey("phone")) {
        builder.phoneNumber(customFields.getString("phone"));
    }

    VisitorInfo visitorData = builder.build();

    ZopimChat.setVisitorInfo(visitorData);

    Activity activity = getCurrentActivity();

    if(activity != null){
        Intent callZopimChat = new Intent(getReactApplicationContext(), ZopimChatActivity.class);
        callZopimChat.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        getReactApplicationContext().startActivity(callZopimChat);
    }
  }

}
```

5、在 3 中目录下新建 ZendeskChatPackage.java 文件内容如下：

```java
package com.xxx.zendeskchat;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.JavaScriptModule;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class ZendeskChatPackage implements ReactPackage {

  @Override
  public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
    return Collections.emptyList();
  }

  // Deprecated in RN 0.47.0
  public List<Class<? extends JavaScriptModule>> createJSModules() {
    return Collections.emptyList();
  }

  @Override
  public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
    List<NativeModule> modules = new ArrayList<>();

    modules.add(new ZendeskChatModule(reactContext));

    return modules;
  }
}
```

6、在 android/app/src/main/java/com/xxx/MainApplication.java 中添加

```java
// ...
// 添加下面
import com.zopim.android.sdk.api.ZopimChat;
import com.zopim.android.sdk.widget.ChatWidgetService;
import com.zopim.android.sdk.prechat.EmailTranscript;
import com.xxx.zendeskchat.ZendeskChatPackage;

public class MainApplication extends Application implements ReactApplication {

  private final ReactNativeHost mReactNativeHost = new ReactNativeHost(this) {
    // ...

    @Override
    protected List<ReactPackage> getPackages() {
      return Arrays.<ReactPackage>asList(
          new MainReactPackage(),
          // ...
          // 添加下面
          new ZendeskChatPackage()
      );
    }
  };


  @Override
  public void onCreate() {
    super.onCreate();

    // ...
    // 添加下面
    ZopimChat
      .init("your accoutKey")
      .emailTranscript(EmailTranscript.DISABLED);  // 关闭聊天窗口不提示发送邮件窗口
    // 隐藏聊天窗口不可用：即聊天后返回页面不显示 widget 组件
    ChatWidgetService.disable();
  }
}
```

7、在 react-native 代码中使用

```js
import { NativeModules } from "react-native";

const ZendeskChat = NativeModules.ZendeskChat;
// visitorInfo 可以填入需要用户填写的信息，也可以为空
const visitorInfo = {
  // name: 'name',
  // email: 'email',
  // phone: 'phone',
}
ZendeskChat && ZendeskChat.startZopimChat(visitorInfo);
```

8、定制 Zendesk Chat 聊天窗口等

示例如下。其他定制详见[文档](https://developer.zendesk.com/embeddables/docs/android-chat-sdk/customization)

在 android/app/src/main/AndroidManifest.xml 中 application 下添加：

```xml
<activity
    android:name="com.zendesk.sdk.feedback.ui.ContactZendeskActivity"
    android:configChanges="orientation|keyboardHidden|screenSize"
    android:theme="@style/RequestsTheme"
    android:windowSoftInputMode="stateVisible|adjustResize" />
<activity
    android:name="com.zendesk.sdk.requests.ViewRequestActivity"
    android:configChanges="orientation|keyboardHidden|screenSize"
    android:theme="@style/RequestsTheme"
    android:windowSoftInputMode="stateHidden|adjustResize" />
<!-- 其他配置选项... -->
```

在 android/app/src/main/res/styles.xml 中添加：

```xml
<resources>

    <style name="ZopimChatTheme" parent="AppTheme">
        <!--Chat UI uses the toolbar so no need to show ActionBar-->
        <item name="windowActionBar">false</item>
        <!-- 聊天窗口顶部标题栏 #0f2947 #4285F4 -->
        <item name="colorPrimary">#0f2947</item>
        <!-- 聊天窗口顶部状态栏 #031121 #3376D6 -->
        <item name="colorPrimaryDark">#031121</item>
        <item name="colorAccent">#DB4537</item>
    </style>

    <style name="HelpCenterTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <item name="colorPrimary">#4285F4</item>
        <item name="colorPrimaryDark">#3376D6</item>
        <item name="colorAccent">#DB4537</item>
    </style>

    <style name="RequestsTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <item name="colorPrimary">#44BB44</item>
        <item name="colorPrimaryDark">#448844</item>
        <item name="colorAccent">#DDDD00</item>
    </style>

    <!--Customize not agents button-->
    <style name="no_agents_button">
        <item name="android:background">#0ab0d9</item>
        <item name="android:padding">8dp</item>
        <item name="android:textColor">@android:color/white</item>
    </style>

    <color name="chat_bubble_visitor">#025E73</color>

</resources>
```