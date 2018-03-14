---
title: react-navigation dva 路由懒加载探讨
date: 2018-03-09 15:37:06
categories: 
- React Native
tags:
- react-native
- react-navigation
- dva
- 前端技术
---

![React Native](/images/main/react-native-logo.png)

使用React Native开发APP小半年了，从开始的react-native+redux+react-redux杂糅在一起到现在尝试到react-native+dva+typescript+antd-mobile，感觉才有点算得上是易维护、易拓展到工程。由于typescript语言的优势，写出的代码也可避免很多低级且不易察觉的错误。

<!-- more -->

最初需要判断一个页面是否是APP显示的当前页面是因为react-navigation版本从v1.0.0-beta23（应该是从这个版本开始）到v1.1.2之间很长的时间内没有懒加载，这样导致项目中首次需要同时加载多个页面（使用react-navigation的TabNavigator）。现在react-navigation版本已经更新到v1.3.2版本了，已经原生默认是懒加载了，不需要懒加载还得需要设置lazy为false才行（但是因为项目中使用了dva，而dva是基于redux封装的，这里又有个坑，后面再细说吧）。

被懒加载搞得有点头大时，在GitHub上看到一个讨论懒加载的issue，下面有人提供了一个解决方案（判断当前页面是否处于激活状态）：点击底部菜单时调用tabBarOnPress方法跳转到该页面，并在该页面调用componentWillReceiveProps方法判断当前页面设置的tabname与nextProps中的tabname是否相同（当前页面处于激活状态），相同的话则加载该页面数据等操作，从而实现页面的懒加载。该[issue](https://github.com/react-navigation/react-navigation/issues/2961)地址<https://github.com/react-navigation/react-navigation/issues/2961>

```bash
import * as actions from "yourprojectname/src/redux/actions";
import { connect } from "react-redux";

const tabname = "TAB1";
var self; // cannot use this inside of static navigationOptions so need this "dirty hack"

class Tab1 extends React.PureComponent {

  static navigationOptions = ({ navigation }) => {
    return {
      tabBarIcon: ({ tintColor }) => < Icon name = "pagelines" size = {22} color = {tintColor} />,
      tabBarOnPress: (tab, jumpToIndex) => {
        navigation.navigate(tabname);
        self.props.action_changeTabName(tabname);
      }
    };
  };

  constructor(props) {
    super(props);
    self = this;
    this.state = {
      dataIsLoading: false,
      data: [],
    };
  }

  componentWillReceiveProps(nextProps) {
    if (tabname == nextProps.tabname) {
      this.setState({
        dataIsLoading: true
      });
      // ...load your data her
    }
  }
}

function mapToStateProps(data) {
  // extract the tabname from data returned by redux 
  return {
    tabname: data.tabname
  };
}
// inject action , will be available in component with this.props.actionname();
export default connect(mapToStateProps, actions)(Tab1);
```

姑且这种方案用着，到后来测试发现了一个问题，因为早起版本的react-navigation只有goBack()方法，只能返回前一页面，不能返回到前N个页面，而且当时制定的方案是验证码登录后会跳转到修改密码到新页面，如果使用navigate()方法直接跳转到下一个页面，那在安卓上使用触摸返回按钮，这样逻辑上是错误的，如果先用两次goBack()再navigate到下一页面，这样用户体验非常怪异。所以我在登录功能跳转的时候用到了重置路由reset()方法。但是按这样到逻辑在验证码登录后修改密码页面后，点击tabbar底部菜单就会提示tabBarOnPress中定义的方法未定义。不过后来应用中更改了默认验证码登录方式，这样我不需要用到重置路由方法，只需用NavigationActions.pop({ n: N }方法就可以了（N为需要返回的页面数量，新版本中提供此方法）。

在开发当前的一个项目后期时，发现react-navigation已支持自动懒加载，前文已提到又有坑。升级到新版本之后，发现TabNavigator4个页面，只有首页一个页面会渲染，其他3个页面点击进入页面无法渲染，甚至连componentDidMount等生命周期都无法触发。在官放issue上也看到有人出现一样都问题，最后都解决方案都是移除redux或者不保存其state（‘suggest not using it to hold your react-navigation state’），但是项目中使用都框架dva是基于redux的，所以本人也没找到一个合适的解决方案，如有高手有解决方案，烦请共享。该[issue](https://github.com/react-navigation/react-navigation/issues/3586)地址见：<https://github.com/react-navigation/react-navigation/issues/3586>

总结一下：

1. 使用reset重置路由在页面使用tabBarOnPress方法时会有方法未定义的问题。
2. 新版本react-navigation自动懒加载，但是如果使用redux会在使用TabNavigator时会有页面无法渲染问题。

第一次写博文，写的很浅，只为记录征途点滴，如果对读者有帮助本人幸之。