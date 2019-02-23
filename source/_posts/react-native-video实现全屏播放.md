---
title: react-native-video实现全屏播放
date: 2018-09-21 14:14:31
categories: 
- React-Native
tags:
- React-Native
---

[react-native-video](https://github.com/react-native-community/react-native-video) 有全屏播放的方法 [presentFullscreenPlayer](https://github.com/react-native-community/react-native-video#presentfullscreenplayer)，但是使用的时候发现在安卓上全屏方法不好使，虽然可以人为的控制播放窗口大小为全屏，视频是能播放了，但发现没有可以控制播放的组件（暂停/播放按钮，进度条等）。仔细看文档发现有这段说明：

> Put the player in fullscreen mode.
> On iOS, this displays the video in a fullscreen view controller with controls.
> On Android ExoPlayer & MediaPlayer, this puts the navigation controls in fullscreen mode. It is not a complete fullscreen implementation, so you will still need to apply a style that makes the width and height match your screen dimensions to get a fullscreen video.

<!-- more -->

尝试了好久还是搞不定，索性就自己定制播放的控制组件。

为了更好的体验，我决定把播放器做成一个单独全屏播放页面，当点击播放视频的时候，直接跳转到播放器页面，页面上有关闭按钮，而且安卓上播放页面点击回退可以返回刚刚的页面，比较符合逻辑。之前我把跳转播放前的页面和播放组件放同一个页面，点击播放时，设置播放组件全屏可见并把其他内容藏起来。但是这样做有些复杂，而且很难隐藏页面的状态栏和标题栏，点击返回时会回退到上一个页面，可能用户此时只是想退出播放，回到播放前的页面。

播放和暂停功能比较容易，图标可以使用 react-native-vector-icons 上的图标。因为项目使用的是 [antd-mobile-rn](https://github.com/ant-design/ant-design-mobile-rn) 的 UI 库，所以播放进度条可以直接使用现成的组件 Slider，拖动进度条时根据进度条位置重新设置播放进度可以简单的实现播放进度控制（this.player.seek(value)），这里用 this.state.paused 控制视频的播放和暂停。

关于横屏播放，项目暂时没有这个需求，为节省时间，这里也不深究，可以参考一下[这里](https://juejin.im/post/5a9f9fde518825557207e7b0)。最后展示一下效果

![视频正在加载](/images/common-articles/react-native-video1.png)
![视频播放中](/images/common-articles/react-native-video2.png)

详细代码如下：

```typescript
import * as React from "react";
import {
  View,
  Text,
  StatusBar,
  SafeAreaView,
  Platform,
  TouchableOpacity,
  ViewStyle,
  TextStyle,
  StyleSheet
} from "react-native";
import { Toast, ActivityIndicator, Slider } from "antd-mobile-rn";
import MaterialsIcon from "react-native-vector-icons/MaterialIcons";
import Video from "react-native-video";
import * as _ from "lodash";
import { size, color } from "../../config";

export default class VideoPlayer extends React.Component<any, any> {
  static navigationOptions = ({ navigation }: any) => ({
    header: null
  });

  player: any;
  constructor(props: IVideoPlayerProps) {
    super(props);
    this.state = {
      onLoading: false,
      videoHeight: size.height,
      playerIconVisible: true,
      paused: true,
      playableDuration: 0,
      currentTime: 0,
      progressPercent: 0
    };
  }

  render() {
    // videoSource: { uri: 'http://cdn.xxx.com/assets/videos/xxx.mp4' } OR require('../../local.mp4')
    const videoSource = _.get(this.props.navigation, "state.params.source");
    let sourceType = _.get(
      this.props.navigation,
      "state.params.sourceType",
      "url"
    );
    if (!videoSource || (sourceType === "url" && !videoSource.uri)) {
      Toast.fail("视频路径不正确！", 2);
      return <SafeAreaView style={indicatorStyles.container} />;
    }
    return (
      <SafeAreaView style={indicatorStyles.container}>
        <Video
          style={{ width: size.width, height: this.state.videoHeight }}
          source={videoSource}
          ref={(ref: any) => (this.player = ref)}
          paused={this.state.paused}
          onLoadStart={(e: any) => {
            console.log("onLoadStart: ", e);
            this.setState({ onLoading: true });
          }}
          onEnd={() =>
            this.setState({ playerIconVisible: true, onLoading: false })
          }
          onError={() => {
            Toast.fail("加载视频出错", 2);
            this.setState({ onLoading: false });
          }}
          onLoad={(e: any) => {
            console.log("onLoad && ready to play: ", e);
            this.setState({ paused: true, onLoading: false });
            const height = _.get(e, "naturalSize.height");
            const width = _.get(e, "naturalSize.width");
            console.log("video' width &&  height: ", width, height);
            let videoHeight = this.state.videoHeight;
            if (_.isNumber(height) && _.isNumber(width)) {
              videoHeight = (size.width * height) / width;
            }
            this.setState({
              playableDuration: _.get(e, "duration", 0),
              videoHeight: videoHeight,
              paused: false
            });
          }}
          onProgress={(e: any) => {
            // console.log('onProgress: ', e);
            let currentTime = _.get(e, "currentTime", 0);
            let playableDuration = _.get(e, "playableDuration", 1);
            this.setState({
              currentTime,
              progressPercent: (currentTime / playableDuration) * 100
            });
          }}
          onFullscreenPlayerDidPresent={() => {
            this.setState({ paused: false });
          }}
          onFullscreenPlayerWillDismiss={() => {
            this.setState({ paused: true });
          }}
        />

        <TouchableOpacity
          style={indicatorStyles.videoCover}
          onPress={async () => {
            await this.setState((prevState: any) => ({
              playerIconVisible: !prevState.playerIconVisible
            }));
            if (!this.state.paused && this.state.playerIconVisible) {
              setTimeout(() => {
                this.setState({ playerIconVisible: false });
              }, 3000);
            }
          }}
        >
          {this.state.onLoading && <ActivityIndicator color="#fff" />}

          <TouchableOpacity
            style={{
              position: "absolute",
              top: Platform.OS === "ios" ? 0 : StatusBar.currentHeight,
              right: 0,
              padding: 10
            }}
            onPress={() => this.props.navigation.goBack()}
          >
            <MaterialsIcon
              name="close"
              size={30}
              color={"rgba(255, 255, 255, 0.8)"}
            />
          </TouchableOpacity>

          {this.state.playerIconVisible && (
            <View style={indicatorStyles.playerController}>
              <TouchableOpacity
                style={indicatorStyles.playerIcon}
                onPress={() => {
                  this.setState((prevState: any) => ({
                    paused: !prevState.paused
                  }));
                }}
              >
                <MaterialsIcon
                  name={this.state.paused ? "play-arrow" : "pause"}
                  size={30}
                  color={"rgba(255, 255, 255, 0.8)"}
                />
              </TouchableOpacity>
              <Text style={indicatorStyles.playerTime}>
                {this.state.currentTime}
              </Text>
              <View style={indicatorStyles.playerProgress}>
                <Slider
                  value={this.state.currentTime}
                  min={0}
                  max={this.state.playableDuration}
                  onChange={(value: number) => {
                    this.player.seek(value);
                    this.setState({ paused: false });
                  }}
                  onAfterChange={(value: number) =>
                    console.log("afterChange: ", value)
                  }
                />
              </View>
              <Text style={indicatorStyles.playerTime}>
                {this.state.playableDuration}
              </Text>
            </View>
          )}
          <View />
        </TouchableOpacity>
      </SafeAreaView>
    );
  }
}

const PLAYER_ICON_WIDTH = 30;
const PLAYER_TIME = 50;
const PROGRESS_WIDTH = size.width - PLAYER_ICON_WIDTH - PLAYER_TIME * 2 - 10;
const style = {
  container: {
    flex: 1,
    backgroundColor: "#000",
    alignItems: "center",
    justifyContent: "center"
  },
  videoCover: {
    position: "absolute",
    width: "100%",
    height: "100%",
    backgroundColor: "transparent",
    alignItems: "center",
    justifyContent: "center"
  },
  playerController: {
    flexDirection: "row",
    position: "absolute",
    bottom: 5,
    left: 0,
    width: "100%",
    backgroundColor: "rgba(0,0,0,0.3)",
    alignItems: "center"
  },
  playerIcon: {
    width: PLAYER_ICON_WIDTH,
    height: 30,
    marginLeft: 10,
    justifyContent: "center",
    alignItems: "center"
  },
  playerProgress: {
    justifyContent: "center",
    height: 4,
    width: PROGRESS_WIDTH,
    flex: 1
  },
  playerTime: {
    width: PLAYER_TIME,
    height: 30,
    color: "#fff",
    padding: 5,
    textAlign: "center"
  }
};
```

参考文章

- [ReactNative——react-native-video 实现视频全屏播放](https://juejin.im/post/5a9f9fde518825557207e7b0)
