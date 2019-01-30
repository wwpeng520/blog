---
title: 使用Taro+dva+ts开发小程序(一)
date: 2019-01-13 14:54:07
categories: 
- 小程序
tags:
- 小程序
- Taro
- dva
---

## 初始化

安装 Taro 开发工具 @tarojs/cli

```bash
npm install -g @tarojs/cli
yarn global add @tarojs/cli
```

<!-- more-->
使用命令创建模板项目：TypeScript + Sass + 默认模板

```bash
taro init weapp-voice
```

![初始化项目](/images/taro/init.png)

## 集成 dva

```bash
npm i dva-core dva-loading dva-immer --save
```

新建文件 `src/utils/dva.ts`

```typescript
import { create } from 'dva-core'
import createLoading from 'dva-loading'
import dvaImmer from 'dva-immer'

let app
let store
let dispatch

function createApp(opt) {
  app = create(opt)
  app.use(createLoading({}))
  app.use(dvaImmer())

  if (!global['registered']) {
    opt.models.forEach(model => app.model(model))
  }
  global['registered'] = true
  app.start()

  store = app._store
  dispatch = store.dispatch
  app.getStore = () => store
  app.dispatch = dispatch
  return app
}

export default {
  createApp,
  getDispatch() {
    return app.dispatch
  }
}
```

并修改入口文件 src/app.tsx

```typescript
import { Provider } from '@tarojs/redux'
import dva from './utils/dva'
import models from './models'

const dvaApp = dva.createApp({
  initialState: {},
  models: models,
});
const store = dvaApp.getStore();

class App extends Component {
  // ...
  render() {
    return (
      <Provider store={store}>
        <Index />
      </Provider >
    )
  }
}
```

models 目录是存放 global model 的目录，models/index.ts 内容如下：

```typescript
import common from './common'
import user from './user'

export default [common, user]
```

执行以下命令开始体验：

```bash
# 调试
npm run dev:weapp

# 打包
npm run build:weapp
```

<!-- https://github.com/EasyTuan/taro-msparis -->