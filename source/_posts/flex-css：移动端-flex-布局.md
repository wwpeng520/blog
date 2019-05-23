---
title: flex.css：移动端 flex 布局
date: 2018-04-18 11:00:23
tags:
- 前端技术
---
## 什么是 flex.css

css3 flex 布局相信很多人已经听说过甚至已经在开发中使用过它，flex.css 就是对 flex 布局的一种封装，通过简洁的属性设置就能使得它完美的运行在移动端的各种浏览器，甚至能运行在 ie10+ 的各种PC端浏览器中。它天然的能够很好的将页面布局和css进行分离，让 css 专注于元素的显示效果，我称之为声明式布局。
<!-- more -->

## 为什么需要 flex.css

引用 flex.css [github地址](https://github.com/lzxb/flex.css)的说明：
“在移动端开发中，并不是所有的浏览器，webview，微信等各种版本都支持标准的flex，
但是基本上都会支持-webkit-box，所以flex.css的主要作用是保证每一个属性都能支持标准flex或旧版本的-webkit-box。
由于flex.css采用了autoprefixer编译，所以能够保证在浏览器不支持标准flex布局的情况下，
回滚到旧版本的-webkit-box，保证移动设备中能呈现出一样的布局效果。
于是，一款移动端快速布局的神器诞生了...”

## npm 安装

```bash
npm install flex.css --save
or
yarn add flex.css
```

## flex 和 data-flex

flex.css 有两个版本，一个是 flex.css，一个是 data-flex.css，这两个版本其实是一样的，唯一的区别是，一个是使用 flex 属性设置，一个是使用 data-flex 属性设置。react 不支持 flex 属性直接布局，所以 data-flex.css 实际上是为了 react 而诞生的。如果使用了 webpack 打包，npm安装后，并且配置了 ES6 编译器的话，flex 属性匹配可以直接使用。

## 使用

### 引入

```css
import 'flex.css';
or
import 'flex.css/dist/data-flex.css';
```

*也可在 html 文件中引入 `<link rel="stylesheet" href="./flex.css">`

### 直接在元素上添加属性，示例如下：

```html
<!-- flex属性匹配，简单的子元素居中例子： -->
<div flex="main:center cross:center" style="width:500px; height: 500px; background: #108423">
  <div style="background: #fff">看看我是不是居中的</div>
</div>

<!-- data-flex属性匹配，简单的子元素居中例子： -->
<div data-flex="main:center cross:center" style="width:500px; height: 500px; background: #f1d722">
  <div style="background: #fff">看看我是不是居中的</div>
</div>
```

### flex 属性大全

```bash
dir：主轴方向
  top：从上到下
  right：从右到左
  bottom：从下到上
  left：从左到右（默认）

main：主轴对齐方式
  right：从右到左
  left：从左到右（默认）
  justify：两端对齐
  center：居中对齐

cross：交叉轴对齐方式
  top：从上到下（默认）
  bottom：从上到下
  baseline：基线对齐
  center：居中对齐
  stretch：高度并排铺满

box：子元素设置
  mean：子元素平分空间
  first：第一个子元素不要多余空间，其他子元素平分多余空间
  last：最后一个子元素不要多余空间，其他子元素平分多余空间
  justify：两端第一个元素不要多余空间，其他子元素平分多余空间
```

flex-box属性说明:
取值范围(0-10)，单独设置子元素多余空间的如何分配，设置为0，则子元素不占用多余的多余空间
多余空间分配 = 当前 flex-box 值/子元素的 flex-box 值相加之和

```html
<h2>两端不需要多余空间，中间占满剩余空间</h2>
<div class="box" flex>
  <div flex-box="0" style="background: gold; width: 50px; height: 50px">1</div>
  <div flex-box="1" style="background: blue; height: 50px">2</div>
  <div flex-box="0" style="background: blue; width: 50px; height: 50px">3</div>
</div>
```

参考文章

- [flex.css：移动端 flex 布局神器](https://blog.csdn.net/zhanglongdream/article/details/53957876)