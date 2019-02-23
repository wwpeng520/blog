---
title: 用typescript写styled-components报错一例
date: 2018-04-25 17:08:37
tags:
- 前端技术
---
# 小试 styled-components

styled-components 是一个常用的很适合 React 技术栈的项目开发的 css-in-js 类库。附一张 [CSS Evolution: From CSS, SASS, BEM, CSS Modules to Styled Components](https://www.tuicool.com/articles/uuMBjub) 文中的图：
![styled-components-improve](/images/plugins/styled-components-improve.jpg)

如何使用可以参考[官网文档](https://www.styled-components.com/)，这里有一篇简明的[视频教程](https://zhuanlan.zhihu.com/p/30824893)。
<!-- more -->

## typescript 下使用 styled-components

安装并引入 styled-components 后，提取公共组件，提高组件复用，也可减少很多代码。可以把常用的组件提取到一个公共组件上，然后同时到处，按需引入到各个页面，也可以把一个组件报存为一个组件，视情况而定。
参考官方文档和各个教程示例编写公共组件 base.tsx 如下

```javascript
import * as React from "react";
import styled from 'styled-components';

export const BContainer = styled.div`
  align-items: flex-start;
  justify-content: center;
  background-color: ${props => props.backgroundColor ? props.backgroundColor : '#fff'};
`;

export const BTitle = styled.p`
  font-size: 13px;
  line-height: 18px;
  color: ${props => props.color ? props.color : '#555'};
`;

export const BSmallText = styled.span`
  font-size: ${props => props.size ? props.size : '8px'};
  line-height: 12px;
  color: ${props => props.color ? props.color : '#999'};
`;
```

编辑器下可以看到提示 props 不存在属性的报错提示。

>[ts] 类型“ThemedStyledProps<DetailedHTMLProps<HTMLAttributes<HTMLDivElement>, >HTMLDivElement>, any>”上不存在属性“backgroundColor”。
>
>[ts] 类型“ThemedStyledProps<DetailedHTMLProps<HTMLAttributes<HTMLSpanElement>, >HTMLSpanElement>, any>”上不存在属性“size”。

编译报错如下：
![styled-components 组件](/images/plugins/typescript下写styled-components报错1.png)

根据报错提示，可以在 props 处增加定义，如

```javascript
export const BTitle = styled.p`
  font-size: 13px;
  line-height: 18px;
  color: ${(props: { color: string; }) => props.color ? props.color : '#555'};
`;
```

最后代码如下：

```javascript
import * as React from "react";
import styled from 'styled-components';

interface ITextProps {
  size?: string;
  color?: string;
}

interface IViewProps {
  backgroundColor?: string;
}

export const BContainer = styled.div`
  align-items: flex-start; 
  justify-content: center;
  background-color: ${(props: IViewProps) => props.backgroundColor ? props.backgroundColor : '#fff'};
`;

export const BTitle = styled.p`
  font-size: 13px;
  line-height: 18px;
  color: ${(props: ITextProps) => props.color ? props.color : '#555'};
`;

export const BSmallText = styled.span`
  font-size: ${(props: ITextProps) => props.size ? props.size : '8px'};
  line-height: 12px;
  color: ${(props: ITextProps) => props.color ? props.color : '#999'};
`;
```
