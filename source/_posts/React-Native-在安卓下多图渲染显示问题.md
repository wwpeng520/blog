---
title: React-Native 在安卓下多图渲染显示问题
date: 2018-03-10 11:03:18
categories: 
- React Native
tags:
- react-native
- 前端技术
---
题记：这里描述个人在项目中遇到的此问题的的过程及解决方法，不一定适用所有此类现象。

一开始遇到这个问题是在之前的一个项目中，因为当时react native中建议FlatList/SectionList代替ListView没多久，我在一个很长的列表中（每个列表包含一张图片）使用的是FlatList列表，在安卓下测试的时候发现页面刷到下面（可能加载了几十页）是很多图片都不显示了（有点过时的安卓机），iPhone下正常，当时感觉和测试的机器性能有关。然后针对FlatList做了一些优化，当时的需求导致FlatList中每个单项高度固定，所以使用了getItemLayout属性优化。优化完之后发现还是有这个现象，然后我FlatList换成了VirtualizedList组件，貌似问题有很大改观。也因为当时那个项目暂停下拉了，所以这个问题就没往下继续了。

最近手头上的项目又碰到了这个问题，列表中每个单项的高度不一样，不能使用getItemLayout属性，且每项要展示的图片有很多，所以这个问题被凸显出来了。然后又是各种Google解决方案，不经意受到一篇文章的启发，才意识到可能不是FlatList组件的问题，而是Image组件的问题。然后给Image组件加上了resizeMethod="resize"（我测试了加载的图片尺寸都很大，显示的容器相对小很多），问题终于解决了。

可能对Image组件没深入研究，对resizeMethod属性（只针对android平台）没大在意，官方文档也没有强调此属性，所以踩进了这个坑。写下此文也是希望遇上相同情况的网友能够受到启发，能够帮助到读者也是作者之幸。

感谢网友[黄金夫](https://link.jianshu.com/?t=http%3A%2F%2Fblog.csdn.net%2FHJF_HUANGJINFU%2Farticle%2Fdetails%2F79281829)的博文，文章地址：<http://blog.csdn.net/HJF_HUANGJINFU/article/details/79281829>。截图如下：
![React Native](/images/react-native/React-Native在安卓下多图渲染显示问题-1.png)