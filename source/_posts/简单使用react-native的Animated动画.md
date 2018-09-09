---
title: 简单使用react-native的Animated动画
date: 2018-09-09 12:47:04
tags:
- react-native
---

在项目中简单使用了 react-native 的 Animated 动画，这里介绍项目中使用到的两种场景。
场景一：点击关闭一个弹窗时，弹窗会慢慢变小到消失，同时运动轨迹是慢慢从中心到标题栏的右侧。该场景是在用户关闭 APP 的功能介绍弹窗时，能够让用户下次需要了解该功能时知道从哪里获取信息。
场景二：点击切换标签页时，被激活的标签底部的横线滑动的动画效果。这个功能很多插件都自带这个功能，不需要额外定义，我也是在使用 react-native-scrollable-tab-view 库时添加了一些额外的标签栏样式时，需要自行添加改动画。
<!-- more -->

# Animated 动画

## Animated 动画组件

Animated 中默认导出了以下这些可以直接使用的动画组件：

- Animated.Image
- Animated.ScrollView
- Animated.Text
- Animated.View

我们还可以使用 createAnimatedComponent() 方法自定义动画组件，下面示例中演示了创建一个可点击的动画组件：

```bash
import { TouchableWithoutFeedback, Animated } from 'react-native';
const AnimatedTouchableWithoutFeedback = Animated.createAnimatedComponent(TouchableWithoutFeedback);

// ...
<AnimatedTouchableWithoutFeedback onPress={() => {...}}>
  <Animated.View style={{ opacity: this.state.fadeInOpacity }} />
</AnimatedTouchableWithoutFeedback>
```

## 两种类型的值

Animated 提供了两种类型的值：

- Animated.Value()：用于单个值
- Animated.ValueXY()：用于矢量值

## 配置动画

Animated 提供了三种动画类型。每种动画类型都提供了特定的函数曲线，用于控制动画值从初始值变化到最终值的变化过程：

- Animated.timing()：线性变化，使用 easing 函数让数值随时间动起来。
- Animated.decay()：衰变效果，以一个初始的速度和一个衰减系数逐渐减慢变为0。。
- Animated.spring()：弹簧效果，提供了一个简单的弹簧物理模型.

大多数情况下使用 timing()就可以了。默认情况下，它使用对称的 easeInOut 曲线，将对象逐渐加速到全速，然后通过逐渐减速停止结束。

## 组合动画

动画还可以使用组合函数以复杂的方式进行组合：

- Animated.delay()：动画延迟，在给定延迟后开始动画。
- Animated.parallel()：同时启动多个动画。默认情况下，如果有任何一个动画停止了，其余的也会被停止。可以通过stopTogether 选项设置为 false 来取消这种关联。
- Animated.sequence()：按顺序启动动画，等待每一个动画完成后再开始下一个动画。如果当前的动画被中止，后面的动画则不会继续执行。
- Animated.stagger()：按照给定的延时间隔，顺序并行的启动动画。即在前一个动画开始之后，隔一段指定时间开始执行下一个动画，并不关心前一个动画是否已经完成，所以有可能会出现多个动画同时执行的情况。

## 使用示例

创建动画最简单的工作流程是创建一个 Animated.Value ，将它连接到动画组件的一个或多个样式属性，然后使用 Animated.timing() 等动画效果展示数据的变化，tart/stop 方法来控制基于时间的动画执行。示例中透明度 new Animated.Value(0) ，首先设为 0，后面 timing 动画中 toValue: 1，即最终变为完全不透明。我们也可以定义一个渐隐的效果，从 1 变为 0，组件就会慢慢消失。

```bash
import React from 'react';
import { Animated, Text, View } from 'react-native';

class FadeInView extends React.Component {
  state = {
    fadeInOpacity: new Animated.Value(0),  // 透明度初始值设为0
  }
  componentDidMount() {
    Animated.timing(                       // 随时间变化而执行动画
      this.state.fadeInOpacity,            // 动画中的变量值
      {
        toValue: 1,                        // 透明度最终变为1，即完全不透明
        duration: 10000,                   // 让动画持续一段时间
      }
    ).start();                             // 开始执行动画
  }
  render() {
    const { fadeInOpacity } = this.state;

    return (
      <Animated.View                       // 使用专门的可动画化的View组件
        style={{
          ...this.props.style,
          opacity: fadeInOpacity,          // 将透明度指定为动画变量值
        }}
      >
        {this.props.children}
      </Animated.View>
    );
  }
}
```

### 场景一示例

```bash
class Comp1 extends React.Component<IProps, {}> {
  constructor(props: IProps) {
    super(props);
    this.state = {
      isModalVisible: false,                      // 弹窗是否可见
      modalWidth: new Animated.Value(1),          // 弹窗初始宽度
      modalHeight: new Animated.Value(1),         // 弹窗初始高度
    };
  }

  startAnimated = () => {
    // 同步执行的动画
    Animated.parallel([
      Animated.timing(this.state.modalHeight, {
        toValue: 0,
        duration: 500,
        easing: Easing.linear,
      }),
      Animated.timing(this.state.modalWidth, {
        toValue: 0,
        duration: 500,
        easing: Easing.linear,
      }),
      // 可以添加其他动画
    ]).start(() => {
      // 这里可以添加动画之后要执行的函数
      setTimeout(() => {
        this.setState({ isModalVisible: false });
      }, 100);
    });
  };
  
  render() {
    const modalWidth = this.state.modalWidth.interpolate({
      inputRange: [0, 1],
      outputRange: [1, 300],
    });
    const modalHeight = this.state.modalHeight.interpolate({
      inputRange: [0, 1],
      outputRange: [1, 200],
    });

    return (
      <View>
        {/* 其他组件... */}

        <Animated.View
          style={[
            styles.modalContent,
            {
              width: modalWidth,
              height: modalHeight,
            },
          ]}
        >
          <View style={{ // ... }}>
            <Animated.Text style={[styles.title, { fontSize: titleFontSize }]}>标题</Animated.Text>
            <ScrollView>
              <Animated.Text style={[styles.content, { fontSize: contentFontSize }]}>这是提示文本，这是提示文本，这是提示文本。</Animated.Text>
            </ScrollView>
            <Button onClick={() => {}}>
              <Animated.Text style={{ fontSize: titleFontSize }}>跳转</Animated.Text>
            </Button>
          </View>
          <TouchableOpacity
            style={{ marginTop: 20, padding: 10, alignItems: 'center' }}
            onPress={this.startAnimated}
          >
            {/* 关闭按钮 */}
          </TouchableOpacity>
        </Animated.View>
      </View>
    )
  }
}
```

### 场景二示例

```bash
class Comp2 extends React.Component<IProps, {}> {
  constructor(props: IProps) {
    super(props);
    this.state = {
      lineWidth: new Animated.Value(0),
      lineLeft: new Animated.Value(0),
      prevWidth: 0,
      prevLeft: 0,
      nextWidth: 0,
      nextLeft: 0,
    };
  }

  componentDidUpdate() {
    if (// ...) {
      this.startAnimated();
      this.setState((prevState: any, props: IProps) => ({
        prevLeft: prevState.nextLeft,
        prevWidth: prevState.nextWidth,
        nextLeft: this.tabbarInfos[activeTab].left,
        nextWidth: this.tabbarInfos[activeTab].width,
      }));
    }
  }

  startAnimated = () => {
    this.state.lineWidth.setValue(0);
    this.state.lineLeft.setValue(0);
    Animated.parallel([
      Animated.timing(this.state.lineWidth, {
        toValue: 1,
        duration: 200,
        easing: Easing.linear,
      }),
      Animated.timing(this.state.lineLeft, {
        toValue: 1,
        duration: 200,
        easing: Easing.linear,
      }),
    ]).start();
  };
  
  render() {
    const lineLeft = this.state.lineLeft.interpolate({
      inputRange: [0, 1],
      outputRange: [this.state.prevLeft, this.state.nextLeft],
    });
    const lineWidth = this.state.lineWidth.interpolate({
      inputRange: [0, 1],
      outputRange: [this.state.prevWidth, this.state.nextWidth],
    });

    const tabUnderlineStyle = {
      position: 'absolute',
      height: 1,
      backgroundColor: '#666',
      bottom: 0,
      width: lineWidth,
      left: lineLeft,
    };

    return (
      <View>
        {/* 其他组件... */}

        <Animated.View style={[tabUnderlineStyle, { //... }]} />
      </View>
    )
  }
}
```

参考文章

- [React Native 动画（Animated）](https://www.jianshu.com/p/7fd37d8ed138)
