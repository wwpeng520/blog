---
title: React Native Text 组件显示不全解决方案
date: 2019-07-28 17:48:10
categories:
  - React-Native
tags:
  - React-Native
---

使用 React Native 开发经常会遇到 iOS 和 Android 上表现不一致的问题，比如 Text 文字在 Android 上偶尔会出现显示不完全的现象，但是在 iOS 却不会出现，之前没有深究这个问题，只是发现后才会去处理，之前处理的办法一般是给外部容器设置一个 minWidth。但是如果深究的话，这个 bug 其实是没有解决的，因为我们难以做到所有页面所有情况文字展示都能够测试到位。

所以这里稍微研究一下，暂时能全局解决这个 bug，如有不对之处请见谅。

<!-- more -->

这个 bug 据说是 Android 上在设置 fontWeight 字体粗细后可能产生这个问题，比如我可能为突出展示一个数字（如$100.00），然后给这个文字设置粗体加大号字体，然后在安卓上却显示成了$100.0，比我们期望的少了一位。

解决方案：

- 给外层容器设置 minWidth
  此方案需要知道内部文字长度，当然有的时候我们可以给多一点余量。不过有时我们不能确定内部字体的长度，或者我们没意识到会出现显示不全的问题，希望可以做的自适应，此方案就不好使了。

- 单独设置 fontFamily
  给出现此类问题的 Text 组件设置一下 fontFamily。此方案的问题也是需要知道哪些文字会出现这个问题，也不能完美解决，最好是能够设置全局的 fontFamily。

- 全局设置 fontFamily
  通过改写覆盖 Text 组件 的 render，实现修改字体全局配置，代码放到入口文件，比如 app.js 里面就可以了。
  不过，这种方案会全局改掉字体，覆盖系统默认字体，可以试试改成 fontFamily: ''，这样不会覆盖默认字体，出问题的 Text 组件也可以显示正常。
  然而，现实并没有那么丰满，因为项目中引入了字体库 [react-native-vector-icons](https://github.com/oblador/react-native-vector-icons)，这就导致了所有使用字体图标的地方都不能显示正常了。
  通过调试发现 react-native-vector-icons 其实也是用的 Text 组件包装的。如下面的代码使用 Ionicons 字体其 fontFamily 就是 Ionicons。所以针对 Android 把 fontFamily 值为 ''，就会导致字体图标没法显示了。

```js
import Ionicons from "react-native-vector-icons/Ionicons";
<Ionicons name="ios-alert" size={18} color="red" />;
```

  这里就做个简单处理如下。当然可以针对 react-native-vector-icons 的字体类型做一个筛选，即 fontFamily 只在 ['Ionicons', 'Entypo', 'Feather', ...]（react-native-vector-icons 的所有字体类型） 之外的设置 fontFamily 为空。

```js
import React from "react";
import { Text, Platform } from "react-native";

const TextRender = Text.render;
Text.render = function(...args) {
  const originText = TextRender.apply(this, args);

  const originStyle = originText.props.style;
  const style = _.isArray(originStyle) ? originStyle : [originStyle];
  let flag = false;
  let tempObj = {};
  for (let val of style) {
    if (_.isObject(val) && !_.isArray(val)) {
      tempObj = {
        ...tempObj,
        ...val
      };
    }
  }
  if (tempObj.hasOwnProperty("fontFamily")) {
    flag = true;
  }
  return React.cloneElement(originText, {
    allowFontScaling: false,
    style: [
      ...style,
      !flag
        ? {
            ...Platform.select({
              android: {
                fontFamily: ""
              }
            })
          }
        : {}
    ]
  });
};
```
