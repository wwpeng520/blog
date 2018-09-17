---
title: 不会原生开发使用百度地图SDK开发APP
date: 2018-09-16 22:25:08
tags:
- react-native
---

众所周知，一般大的互联网平台都会提供官方 SDK 或者 API 提供给开发者使用。这里介绍自己在移动 APP 中使用百度地图 SDK 和 API 的方式查询信息的一次过程。

本人使用 react-native 开发安卓和 iOS 跨端应用，没有原生开发的经验，所以在开发中遇到使用原生功能的库时有时会难以解决，需要踩各种坑，幸好基本能够出坑。

<!-- more -->

## 集成第三方库

闲话少说，开始入坑。项目中采用的是 [react-native-baidu-map](https://github.com/lovebing/react-native-baidu-map) 的库，不出意外，在能够正常使用前需要费一下气力，官方文档描述的对不懂原生开发的人会有些懵。

安卓上碰到的坑：
1.js/MapView.js 中 react 已移除 PropTypes，需引入 prop-types
`import PropTypes from 'prop-types';`
2.android/.../BaiduMapPackage.java 文件移除（注释） createJSModules 方法

```java
// @Override
// public List<Class<? extends JavaScriptModule>> createJSModules() {
//     return Collections.emptyList();
// }
```

3.权限声明不对（官方文档未提供详细说明，需要结合百度地图文档说明等）

```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
<!-- react-native 获取网络状态 -->
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<!-- 这个权限用于进行网络定位-->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"></uses-permission>
<!-- 这个权限用于访问GPS定位-->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"></uses-permission>
<!-- 用于访问wifi网络信息，wifi信息会用于进行网络定位-->
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"></uses-permission>
<uses-permission android:name="com.android.launcher.permission.READ_SETTINGS" />
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.WRITE_SETTINGS" />
<!-- 读取设备硬件信息，统计数据 -->
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.WAKE_LOCK"/>
<uses-permission android:name="android.permission.GET_TASKS" />
```

4.打包后播放异常，确认是否在 android/app/proguard-rules.pro 下加入以下规则

`-keep class com.baidu.** { *; }
-keep class vi.com.** { *; }
-dontwarn com.baidu.**`

iOS 编译问题：
1.将RCTBaiduMap.xcodeproj/BaseModule.h里的 #import "RCTBridgeModule.h" 替换为 #import <React/RCTBridgeModule.h>
2.将 RCTBaiduMap.xcodeproj/RCTBaiduMapView.h 里的 #import "RCTViewManager.h"和#import "RCTConvert+CoreLocation.h" 替换为 #import <React/RCTViewManager.h>和 #import <React/RCTConvert+CoreLocation.h>

其他不做过多说明，[这篇文章](https://www.jianshu.com/p/fd8f24eb5b5c)介绍等比较详细，如果编译还有问题，不妨在 issue 和简书上搜索看看。

## 申请 KEY

在[百度地图开发平台](http://lbsyun.baidu.com/apiconsole/key)申请，因为我需要 iOS 和安卓上都使用，所以我需要创建浏览器端，Android SDK 和 iOS SDK 三种应用。把 iOS 和安卓的 KEY 按文档填在代码中，浏览器端申请的我需要在模拟请求 API 的时候使用（IP 白名单填写“\*”）。

## 简单使用

```typescript
// ...
import { MapView, MapTypes, MapModule, Geolocation } from "react-native-baidu-map";
import * as _ from 'lodash';

const MAP_HEIGHT = {
  Folded: 135,
  Nomal: size.width
};
class BaiduMap extends React.Component<IBaiduMapProps, any> {
  static navigationOptions = ({ navigation }: any) => ({
    title: "百度地图"
  });

  constructor(props: IBaiduMapProps) {
    super(props);
    this.state = {
      mapType: MapTypes.NORMAL,
      zoom: 10,
      center: {
        longitude: 116.46,
        latitude: 39.92
      },
      mapHeight: MAP_HEIGHT.Nomal, // 地图高度
      searchValue: ""
    };
  }

  async componentDidMount() {
    this.getLocation();
  }

  // 判断是否国内。我国纬度3.86~53.55，经度73.66~135.05；北京：纬度39.92，经度为116.46
  getLocation = async () => {
    try {
      const locationData = await Geolocation.getCurrentPosition();
      console.log("locationData: ", locationData);
      const longitude = _.get(locationData, "longitude");
      const latitude = _.get(locationData, "latitude");
      if (!_.isNumber(longitude) || !_.isNumber(latitude)) {
        const msg = _.get(locationData, "errmsg", "获取定位错误");
        return Toast.info(msg, 2);
      }

      if (latitude > 53.55 || latitude < 3.86 || longitude > 135.05 || longitude < 73.66){
        Toast.info('您的定位好像有问题', 2);
      } else {
        await this.setState({
          zoom: 15,
          center: {
            longitude: longitude || 116.46,
            latitude: latitude || 39.92
          }
        });
      }
    } catch (e) {
      console.log("get location error: ", e);
    }
  };

  render() {
    return (
      <View>
        {/** 其他代码 **/}
        <MapView
          style={{ width: "100%", height: "100%" }}
          zoom={this.state.zoom}
          mapType={mapType}
          center={this.state.center}
          onMarkerClick={(e: any) => {
            console.log("onMarkerClick: 自定义标记点点击时");
          }}
          onMapStatusChangeFinish={(e: any) => {
            // Android only
            console.log("onMapStatusChangeFinish: 地图变化完成时");
            const longitude = _.get(e, "target.longitude");
            const latitude = _.get(e, "target.latitude");
            this.setState({
              center: {
                longitude: _.isNumber(longitude) ? longitude : 116.46,
                latitude: _.isNumber(latitude) ? latitude : 39.92
              }
            });
          }}
          onMapLoaded={() => {
            console.log("onMapLoaded: 地图加载完成");
          }}
          onMapPoiClick={(e: any) => {
            console.log("onMapPoiClick: 地图上文字标记等点击时");
          }}
          onMapClick={(e: any) => {
            console.log("onMapClick: 地图上文字标记以外的点击时");
          }}
          onMapDoubleClick={(e: any) => {
            console.log("onMapDoubleClick: 双击");
          }}
        />
      </View>
    );
  }
}
```

## 搜索关键字

react-native-baidu-map 没有提供搜索的方法，而我不会原生开发，所以使用浏览器端的方式调用百度提供的 API，具体可查询[官方文档](http://lbsyun.baidu.com/index.php?title=webapi/guide/webservice-placeapi)。

```ts
// GET 请求的参数如下：
params = {
  query: '水果',
  output: 'json',
  page_size: 20,
  page_num: 1,
  scope: 2, // 取值为1 或空，则返回基本信息；取值为2，返回检索POI详细信息
  region: '上海市南京路',
  ak: '你申请的浏览器ak'
}

// 反向地理编码
reverseGeoCode = async (latitude: number, longitude: number) => {
  try {
    return await Geolocation.reverseGeoCode(latitude, longitude);
  } catch (e) {
    console.log('get reverseGeoCode error: ', e);
  }
};
onQuery = async (page_num = 1) => {
  // const addrResult = await this.reverseGeoCode(this.state.center.latitude, this.state.center.longitude);
  const result = await this.props.dispatch({
    type: 'baidu_map/query',
    payload: {
      query: '水果', // 要查询的关键字
      output: 'json',
      page_size: 20,
      page_num,
      scope: 2, // 取值为1 或空，则返回基本信息；取值为2，返回检索POI详细信息
      region: '上海市南京路', // 这里 region 可以用 reverseGeoCode 方法获得的地理信息
    },
  });
  // result 即为查询到的数据，可以显示给用户
};
```

最后请求的 URL 形如：`http://api.map.baidu.com/place/v2/search?ak=你申请的浏览器ak&query=%E6%B0%B4%E6%9E%9C&output=json&page_size=20&page_num=1&scope=2&region=%E5%8C%97%E4%BA%AC%E5%B8%82%E6%9C%9D%E9%98%B3%E5%8C%BA`

最终代码如下：

```typescript
// ...
import { MapView, MapTypes, MapModule, Geolocation } from "react-native-baidu-map";
import * as _ from 'lodash';

const MAP_HEIGHT = {
  Folded: 135,
  Nomal: size.width
};
class BaiduMap extends React.Component<IBaiduMapProps, any> {
  static navigationOptions = ({ navigation }: any) => ({
    title: "百度地图"
  });

  constructor(props: IBaiduMapProps) {
    super(props);
    this.state = {
      mapType: MapTypes.NORMAL,
      zoom: 10,
      center: {
        longitude: 116.46,
        latitude: 39.92
      },
      mapHeight: MAP_HEIGHT.Nomal, // 地图高度
      searchValue: ""
    };
  }

  async componentDidMount() {
    this.getLocation();
  }

  // 判断是否国内。我国纬度3.86~53.55，经度73.66~135.05；北京：纬度39.92，经度为116.46
  getLocation = async () => {
    try {
      const locationData = await Geolocation.getCurrentPosition();
      console.log("locationData: ", locationData);
      const longitude = _.get(locationData, "longitude");
      const latitude = _.get(locationData, "latitude");
      if (!_.isNumber(longitude) || !_.isNumber(latitude)) {
        const msg = _.get(locationData, "errmsg", "获取定位错误");
        return Toast.info(msg, 2);
      }

      if (latitude > 53.55 || latitude < 3.86 || longitude > 135.05 || longitude < 73.66){
        Toast.info('您的定位好像有问题', 2);
      } else {
        await this.setState({
          zoom: 15,
          center: {
            longitude: longitude || 116.46,
            latitude: latitude || 39.92
          }
        });
      }
    } catch (e) {
      console.log("get location error: ", e);
    }
  };

  // 反向地理编码
  reverseGeoCode = async (latitude: number, longitude: number) => {
    try {
      return await Geolocation.reverseGeoCode(latitude, longitude);
    } catch (e) {
      console.log("get reverseGeoCode error: ", e);
    }
  };

  onQuery = async (page_num = 1) => {
    if (!this.state.searchValue) {
      return Toast.info('请输入查询关键字', 1);
    }

    const addrResult = await this.reverseGeoCode(this.state.center.latitude, this.state.center.longitude);
    console.log('addrResult: ', addrResult);
    const fullSite = `${_.get(addrResult, 'city', '')}${_.get(addrResult, 'district', '')}`;
    const result = await this.props.dispatch({
      type: 'baidu_map/query',
      payload: {
        query: this.state.searchValue,
        output: 'json',
        page_size: 20,
        page_num,
        scope: 2, // 取值为1 或空，则返回基本信息；取值为2，返回检索POI详细信息
        region: fullSite,
      },
    });
    const searchData = _.get(result, 'results');
    // ...
  };

  render() {
    return (
      <View>
        {/** 其他代码 **/}
        <MapView
          style={{ width: "100%", height: "100%" }}
          zoom={this.state.zoom}
          mapType={mapType}
          center={this.state.center}
          onMarkerClick={(e: any) => {
            console.log("onMarkerClick: 自定义标记点点击时");
          }}
          onMapStatusChangeFinish={(e: any) => {
            // Android only
            console.log("onMapStatusChangeFinish: 地图变化完成时");
            const longitude = _.get(e, "target.longitude");
            const latitude = _.get(e, "target.latitude");
            this.setState({
              center: {
                longitude: _.isNumber(longitude) ? longitude : 116.46,
                latitude: _.isNumber(latitude) ? latitude : 39.92
              }
            });
          }}
          onMapLoaded={() => {
            console.log("onMapLoaded: 地图加载完成");
          }}
          onMapPoiClick={(e: any) => {
            console.log("onMapPoiClick: 地图上文字标记等点击时");
          }}
          onMapClick={(e: any) => {
            console.log("onMapClick: 地图上文字标记以外的点击时");
          }}
          onMapDoubleClick={(e: any) => {
            console.log("onMapDoubleClick: 双击");
          }}
        />
      </View>
    );
  }
}
```

最后上图
![百度地图使用](/images/common-articles/baidu_map1.jpeg)
![百度地图使用](/images/common-articles/baidu_map2.jpeg)
