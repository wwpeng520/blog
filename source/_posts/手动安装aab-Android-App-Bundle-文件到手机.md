---
title: 手动安装aab(Android App Bundle)文件到手机
date: 2020-07-29 23:02:08
tags:
---
Android App Bundle 是 Google Play 中的推荐发布格式。通过使用 App Bundle 来发布应用，可以缩减应用大小、简化发布流程，同时还可启用各种高级分发功能。

打包出的 aab 格式的安装包不能直接安装在安卓手机上（需要转换成 apk 格式才可以安装）。
<!-- more -->

想要手动安装已经打包好的 aab 文件需要借助 [bundletool](https://developer.android.com/studio/command-line/bundletool) 工具才行。mac 上可以通过`brew install bundletool`命令安装。

安装好之后可以使用其生成 apk 文件

```bash
bundletool build-apks --bundle=/MyApp/my_app.aab --output=/MyApp/my_app.apks
```

如果要将这些 APK 部署到设备，您还需要添加应用的签名信息，如下面的命令所示。如果您未指定签名信息，bundletool 会尝试使用调试密钥为 APK 签名。

```bash
bundletool build-apks --bundle=/MyApp/my_app.aab --output=/MyApp/my_app.apks
--ks=/MyApp/keystore.jks
--ks-pass=file:/MyApp/keystore.pwd
--ks-key-alias=MyKeyAlias
--key-pass=file:/MyApp/key.pwd
```

通过以上操作，可以取得一个 apks 文件，然后执行以下命令可以直接把应用安装到手机上（需要电脑连接上手机）

```bash
bundletool install-apks --apks=my_app.apks
```
