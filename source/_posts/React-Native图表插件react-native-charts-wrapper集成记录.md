---
title: React Native图表插件react-native-charts-wrapper集成记录
date: 2019-07-18 19:36:26
categories: 
- React-Native
tags:
- React-Native
---
React-Native 项目升级到 0.60.x 之后，iOS 端可以使用 CocoaPods，无需再使用`react-native link [packageName]`方式集成第三方库了。相反使用了 link 方式集成的需要 unlink。

引入 [react-native-charts-wrapper](https://github.com/wuxudong/react-native-charts-wrapper) 图表库，按照[官方文档](https://github.com/wuxudong/react-native-charts-wrapper/blob/master/installation_guide/README.md)操作发现总是会报
`double-conversion file not found`这个错。按照网上的方法比如添加 third-party，折腾了好久还是不行。那就麻烦一点，手动集成好了。
<!-- more -->

手动集成步骤：

- 在 package.json 的 scripts 内添加`"postinstall": "sed -i '' 's/#import <RCTAnimation\\/RCTValueAnimatedNode.h>/#import \"RCTValueAnimatedNode.h\"/' ./node_modules/react-native/Libraries/NativeAnimation/RCTNativeAnimatedNodesManager.h"`
- 把 /node_modules/react-native-charts-wrapper/ReactNativeCharts 的整个 ReactNativeCharts 文件夹，拖入到 XCode 打开的项目中，勾选“Copy items if needed”。
- 创建桥接文件：用 XCode（打开[你的项目名称].xcworkspace） 打开项目，新建一个空文件（command + N），选择 Swift 文件，XCode 会提示`Would you like to configure an Objective-C bridging header?`，选择`Create Bridging Header`选项。文件名为：[你的项目名称]-Bridging-Header.h，文件内容如下：

```h
#ifndef Jinfen_Bridging_Header_h
#define Jinfen_Bridging_Header_h

#import "React/RCTBridge.h"
#import "React/RCTViewManager.h"
#import "React/RCTUIManager.h"
#import "React/UIView+React.h"
#import "React/RCTBridgeModule.h"
#import "React/RCTEventDispatcher.h"
#import "React/RCTEventEmitter.h"
#import "React/RCTFont.h"

#endif /* Jinfen_Bridging_Header_h */
```

- 下载 [SwiftyJSON](https://github.com/SwiftyJSON/SwiftyJSON) 和 [iOS Charts](https://github.com/danielgindi/Charts) 后，把这两个文件夹放到项目的 ios 目录下。
- 把刚刚的两个文件夹中的 SwiftyJSON.xcodeproj 和 Charts.xcodeproj 拖到工程的 Libraries 目录下。
- 接下来 Build Phases -> Link Binary With Libraries 添加 SwiftyJSON.framework 和 Charts.framework。
- General -> Embedded Binaries -> 添加 SwiftyJSON.framework 和 Charts.framework。
- XCode Build 一下试试吧。

如果报错之类的可以参考一下[官方文档](https://github.com/wuxudong/react-native-charts-wrapper/blob/master/installation_guide/README.md)和[这篇](https://www.jianshu.com/p/432517c5b531)

如果报错`Value for SWIFT_VERSION cannot be empty`，可以再 Build Setting 搜索 swift，在 「Swift Compile - Language」下为「Swift Language Version」选择 Swift 版本。
