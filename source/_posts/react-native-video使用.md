---
title: react-native-video使用
date: 2018-09-17 12:50:52
categories: 
- React-Native
tags:
- React-Native
---
react-native APP 需要播放视频功能，这里选用 [react-native-video](https://github.com/react-native-community/react-native-video) 库，这个库在 [GitHub](https://github.com) 上有 3k+ 的星。
<!-- more -->

## 简单使用

```javascript
import Video from 'react-native-video';
import * as _ from 'lodash';

class Introduction extends React.Component<IIntroductionProps, any> {
  static navigationOptions = ({ navigation }: any) => ({
    title: "title",
  });

  player: any;
  constructor(props: IIntroductionProps) {
    super(props);
    this.state = {
      videoWidth: 0,
      videoHeight: 0,
      playerIconVisible: true,
      paused: true,
      playableDuration: 0,
      currentTime: 0
    };
  }

  render() {
    return (
      <View style={{ flex: 1, backgroundColor: "#fff" }}>
        <Video
          style={{
            flex: 1,
            width: this.state.videoWidth,
            height: this.state.videoHeight
          }}
          source={{ uri: "http://cdn.xxx.com/videos/xxx.mp4" }} // Can be a URL or a local file.
          ref={(ref: any) => (this.player = ref)}
          paused={this.state.paused}
          onEnd={() => {}}
          onError={() => {}}
          onLoad={(e: any) => {
            // console.log('onLoad: ', e);
            this.setState({ playableDuration: _.get(e, "duration", 0) });
          }}
          onProgress={(e: any) => {
            this.setState({ currentTime: _.get(e, "currentTime", 0) });
          }}
          // onFullscreenPlayerDidPresent={() => {}}
        />
      </View>
    );
  }
}
```

## 编译问题

安卓编译报错 1：

> \* What went wrong:
> A problem occurred configuring project ':app'.
> \> Could not resolve all dependencies for configuration ':app:\_debugApk'.
> \> A problem occurred configuring project ':react-native-video'.
> \> Could not resolve all dependencies for configuration ':react-native-video:\_debugPublishCopy'.
> \> Could not find com.android.support:support-annotations:27.0.0.
> Searched in the following locations:
> file:/Users/wwpeng520/Library/Android/sdk/extras/android/m2repository/com/android/support/support-annotations/27.0.0/support-annotations-27.0.0.pom

解决方案：android/build.gradle 中加上 `maven { url 'https://maven.google.com' }`，如下所示：

```gradle
allprojects {
    repositories {
        mavenLocal()
        jcenter()
        maven {
            url 'https://maven.google.com'
        }
        maven {
            url "$rootDir/../node_modules/react-native/android"
        }
    }
}
```

安卓编译报错 2：

> Dex: The number of method references in a .dex file cannot exceed 64K.
> Learn how to resolve this issue at https://developer.android.com/tools/building/multidex.html
> UNEXPECTED TOP-LEVEL EXCEPTION:
> com.android.dex.DexIndexOverflowException: method ID not in [0, 0xffff]: 65536
> at com.android.dx.merge.DexMerger$6.updateIndex(DexMerger.java:502)
> // ...其他信息
> :app:transformClassesWithDexForDebug FAILED
> FAILURE: Build failed with an exception. \* What went wrong:
> Execution failed for task ':app:transformClassesWithDexForDebug'.
> \> com.android.build.api.transform.TransformException: com.android.ide.common.process.ProcessException: java.util.concurrent.ExecutionException: java.lang.UnsupportedOperationException

解决方案：android/app/build.gradle 中 defaultConfig 加上 `multiDexEnabled = true`。

安卓编译报错 3：

> Error:Execution failed for task ':app:transformClassesWithDexForDebug'.
> \> com.android.build.api.transform.TransformException: com.android.ide.common.process.ProcessException: java.util.concurrent.ExecutionException: java.lang.UnsupportedOperationException

这个编译错误是因为引入过多的第三方 jar 包导致的。在编译文件 build.gradle 中 android 下加入如下代码，即可解决。

```gradle
dexOptions{
    javaMaxHeapSize "4g"
}
```
