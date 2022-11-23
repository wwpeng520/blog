---
title: Vite搭建React项目
date: 2021-12-25 12:22:38
tags:
  - Vite
  - React
---

<!-- 也可以参考 https://cdmana.com/2021/07/20210724083644437n.html -->
<!-- 我们从 UmiJS 迁移到了 Vite https://juejin.cn/post/7033243995425734663 -->

先根据 [Vite](https://cn.vitejs.dev/) 的步骤创建 `react-ts` 为模板的项目，这里就不赘述了。

## 引入 Antd 组件库，主题定制

安装成功之后，我们先使用全局引入样式的方式，测试是否能正常。打开 main.jsx 添加样式：

```tsx
import 'antd/dist/antd.css';
```
<!-- more -->

测试一下打包
![全量引入 antd 样式打包](/images/vite/vite-antd-all.png)

测试一下未引入 antd 组件时打包
![全量引入 antd 样式打包](/images/vite/vite-init.png)

对比一下发现只是单单引入一个 Button 组件 js 文件大小未压缩时是 227.29kb（未引入时是 129.43kb），css 文件大小 529.29kb（未引入时是 0.75kb），css 文件足足大了 520kb...

作为一个有追求代码洁癖和质量的人怎么能容忍这样的状况呢所，以采取按需加载方式势在必行。

网上找了一下 Vite 的插件，主要是 `vite-plugin-imp` 和 `vite-plugin-style-import`，因为 Vite 组件库本身是按需引入的，只有样式不会按需导入，只需根据需要导入样式即可，所以这里使用 `vite-plugin-style-import`。

考虑到大部分项目都需要定制项目的主题色，所以需要更改 antd 的主题色。先在 `src/styles` 目录下创建文件 `variables.less`，添加内容 `@primary-color: '#1DA57A';`。安装 `less` 和 `less-vars-to-js` 依赖。`less-vars-to-js` 可以将 less 文件转化为 json 键值对的形式，当然你也可以直接在 modifyVars 属性后写 json 键值对。这样做的好处是可以把全局配置管理，方便维护。

再添加上打包分析插件，并添加 alias，最好 vite.config.ts 修改如下：

```ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import styleImport from 'vite-plugin-style-import';
import lessToJS from 'less-vars-to-js';
import { visualizer } from 'rollup-plugin-visualizer';
import path from 'path';
import fs from 'fs';

const themeVariables = lessToJS(
  fs.readFileSync(path.resolve(__dirname, './src/styles/variables.less'), 'utf8'),
);
const extraPlugins = [];
if (process.env.ANALYZE === 'true') {
  extraPlugins.push(
    // 打包分析 process.env.ANALYZE === 'true'
    visualizer({
      open: true,
      gzipSize: true,
      brotliSize: true,
    }),
  );
}

export default defineConfig({
  build: {
    // ... other options
    terserOptions: {
      compress: {
        drop_console: true, // discard calls to console.* functions
        drop_debugger: true, // remove debugger; statements
      },
    },
  },
  resolve: {
    alias: [
      {
        find: '@',
        replacement: path.resolve(process.cwd(), '/src'),
      },
    ],
  },
  css: {
    preprocessorOptions: {
      less: {
        javascriptEnabled: true,
        modifyVars: themeVariables,
      },
    },
  },
  plugins: [
    ...extraPlugins,
    react(),
    // antd 样式按需引入
    styleImport({
      libs: [
        {
          libraryName: 'antd',
          esModule: true,
          resolveStyle: (name) => {
            return `antd/es/${name}/style/index`;
          },
        },
      ],
    }),
  ],
});
```

再打包看一下打包大小，js 与未按需加载时未改变，css 文件变成 227.29kb，比全量引入少了 300kb。效果还是很明显的。

## 其他打包配置

上面还配置了别名 alias，只是还不完整，还需要配置 `tsconfig.json` 文件：在 compilerOptions 里面添加 `"baseUrl": "./"` 和 `"paths": { "@/*": ["src/*"] }`。

build 下的 terserOptions.compress `drop_console/drop_debugger` 属性是移除打包时的 `console.*` 和 `debugger` 的，最好加上。

## eslint, prettier 和 stylelint 配置

<!-- 也可以参考 https://cdmana.com/2021/07/20210724083644437n.html -->
<!-- 我们从 UmiJS 迁移到了 Vite https://juejin.cn/post/7033243995425734663 -->
代码规范很重要，尤其是多人协作开发的时候。网上有很多其他的推荐规范，这里偷个懒，采用和 umijs 集成的推荐方案 [@umijs/fabric](https://www.npmjs.com/package/@umijs/fabric)。按照这个规范要求，对代码质量和约束够用了，就像其说明的一样「💪严格但是不严苛的编码规范」。

## git commit 集成

多人协作开发时，代码质量保障和规范提交十分有必要。在项目目录下的 .git 目录下 hooks 文件夹下存在相关 git 生命周期的钩子函数配置，不过因为这是本地文件,对于团队协同开发同步配置不太友好，所以还是建议采用第三方库统一管理。

这里需要安装 husky、lint-staged、prettier 等工具。然后执行：

```bash
# 创建.husky， 会在根目录下创建.husky文件夹
npx husky install
```

执行完成后正在 package.json 中自动加入 `prepare`

```json
"scripts": {
  "prepare": "husky install"
}
```

手动添加如下内容

```json
"scripts": {
  "precommit": "lint-staged"
},
"lint-staged": {
  "**/*.less": "stylelint --syntax less",
  "**/*.{js,jsx,ts,tsx}": "npm run lint-staged:js",
  "**/*.{js,jsx,tsx,ts,less,md,json}": [
    "prettier --write"
  ]
},
```

执行下述命令会在 .husky 下创建一个 pre-commit 的文件（创建husky钩子）

```bash
npx husky add .husky/pre-commit "npm run precommit"
```

`precommit` 是刚才在 scripts 中添加的命令。提交代码后 commit 试一下成功与否，看到形如下图所示的就成功了。

![commit-husky](/images/vite/commit-husky.png)

## 其他

环境变量可以参考[官方文档](https://cn.vitejs.dev/guide/env-and-mode.html#env-files)。

新建 `.env` 文件定义变量（略...）。暴露给 Vite 源码的变量最终都将出现在客户端包中，所以 Vite 中需要使用的环境变量都需要以 `VITE_*` 的形式，并且应该不包含任何敏感信息。Vite 代码中以 `import.meta.env.VITE_*` 的方式使用。


