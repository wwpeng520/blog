---
title: 使用URL Scheme方式打开APP
date: 2018-11-28 09:25:48
categories: 
- React-Native
tags:
- React-Native
---

平常我们做 APP 开发，会遇到打开其他应用的功能，比如在应用中跳转到微信和支付宝支付的页面，或者我们想要通过短信链接、H5 页面、超链接、其他 AP P跳转到你的 APP。

## iOS设置

一个完整的 URL Scheme 协议格式由scheme、host、port、path和query组成，其结构如下所示：
`<scheme>://<host>:<port>/<path>?<query>`
<!-- more -->

xcode打开工程 -> Info -> URL Types -> 点击“+” -> 在URL Schemes栏填上如 weixin 等名称
![查看微博版本](/images/react-native/Scheme打开APP-ios.png)

## Android设置

在AndroidManifest.xml中对`<activity>`标签增加`<intent-filter>`设置 Scheme
![查看微博版本](/images/react-native/Scheme打开APP-android.png)
根据 URL Scheme 协议格式设置 uri 即可正确唤起对应的 APP了。

## 唤起 APP 并跳转对应页面

- react-native 上使用`Linking.openURL('demo_scheme://pageA?a=aaa')`，如果你同时定义了 scheme 和 host，你则需要`Linking.openURL('demo_scheme://demo_host/pageA?a=aaa')`。这里演示打开 demo_scheme APP 并跳转 pageA 页面。
- iOS 上使用 Safari 浏览直接输入`demo_scheme://demo_host/pageA?a=aaa`即可。
- HTML 页面中只需使用`<a href="myapp://www.abc.com/person?id=123">跳转</a>`。