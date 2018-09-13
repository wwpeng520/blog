---
title: React-Native之FlatList(ListEmptyComponent属性)
date: 2018-06-02 11:40:11
categories: 
- React Native
tags:
- react-native
- 前端技术
---
题记：本文不是使用教程，只探讨记录 FlatList 的 ListEmptyComponent 属性问题。

> ListEmptyComponent?: ?ReactClass<any> | React.Element<any>
> 列表为空时渲染该组件。可以是React Component, 也可以是一个render函数， 或者渲染好的element。

以上是关于该属性的解释。

ListEmptyComponent 表示 FlatList 没有数据的时候填充的布局，一般情况我们会在列表没有数据时显示一个提示信息，比如提示错误、下拉刷新或者暂无数据等。
<!-- more -->

```typescript
ListEmptyComponent={() => {
  if (error) {
    return (
      <TouchableOpacity
        style={{ flex: 1 }}
        onPress={() => this._refresh()}
        activeOpacity={0.6}>
        <Text style={{ textAlign: 'center' }}>{error} 点击重试</Text>
      </TouchableOpacity>
    )
  } else {
    return <View style={{ height: '100%', alignItems: 'center', justifyContent: 'center' }}>
      <Text>暂无数据</Text>
    </View>
  }
}}
```

上面的示例代码看起来是发生错误时设置 ListEmptyComponent 组件 flex:1 或者给定固定高度，点击 ListEmptyComponent 区域重新获取数据，暂无数据居中显示，但是实际情况，你发现无论 flex:1，还是设定100%，都不起作用，它无法撑起空间，达不到我们等预期。经过测试，我们发现设定固定数组等高度是有效果的，但是当我们想撑开整个区域时，我们不知道具体高度是多少。

为了实现我们想要的效果，我们可以将 ListEmptyComponent 中组件的高度设置为 FlatList 的高度，要获取 FlatList 的高度，我们可以通过 onLayout 方法获取。
> onLayout：当组件挂载或者布局变化的时候调用，参数为：
> {nativeEvent: { layout: {x, y, width, height}}}
> 这个事件会在布局计算完成后立即调用一次，不过收到此事件时新的布局可能还没有在屏幕上呈现，尤其是一个布局动画正在进行中的时候。

通过以上的说明，我们知道 onLayout 是动态计算的，实际上也是如此：iOS ListEmptyComponent 渲染时 onLayout 还没有回调。所以我们需要使用 state 变量保存我们需要的高度值。

``onLayout={e => this.setState({ flatlistHeight: e.nativeEvent.layout.height })}``

这样设置后应该完美了吧，可是....在 Android 上能完美实现我们要的效果，在 iOS 上出现了来回闪屏的的问题。打印日志发现值一直是0和测量后的数值来回变化。所以修改 onLayout 如下：

```typescript
onLayout={e => {
  let height = e.nativeEvent.layout.height;
  if (this.state.flatlistHeight < height) {
    this.setState({ flatlistHeight: height })
  }
}}
```

有个地方需要注意的是，我们以上设置的高度是整个 FlatList 组件的高度，当我们如果设置了头部和尾部组件 ListHeaderComponent/ListFooterComponent 后，我们需要减去这两个部分对应的高度值。

<!-- 参考文章：http://www.sinmeng.net/info/31-1129-1.html -->
