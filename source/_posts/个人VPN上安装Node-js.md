---
title: 个人VPN上安装Node.js
date: 2018-12-29 16:10:45
categories: 
- Linux
tags:
- Linux
- VPN
- Node.js
---
题记：服务器部署很少涉及，虽然买了几个 VPN 玩玩，但作用一般也就部署个 Hexo 个人博客和配置个 ShadowSocks 翻墙用用。最近想做个小程序玩玩，前后端都自己来写，服务器端需要部署到 VPN，很多东西以前大致摸索过一些，只是用的少，很多步骤需要边 Google 边操作。所以一些过程打算自己记录下来，以供以后翻阅。

买的 VPN 是 [vultr](https://www.vultr.com) 的，Ubuntu 系统。

在Ubuntu系统上安装nodejs有很多种方法，分别为：apt-get在线安装，下载Node.js源码自己编译安装，下载编译好的文件，使用npm安装等方式。推荐通过源码编译安装。
<!-- more -->

1. 登录 VPN

使用终端命令

```bash
ssh root@45.xxx.xxx.xxx
```

根据终端提示输入 VPN 的密码即可进入 VPN Ubuntu 系统。如果需要经常登录，可以在 VPN 的管理网站添加 SSH Keys，然后登录 VPN 就不需要每次去复制粘贴密码了。

2. 可能需要的操作

用以下命令来升级系统，并且安装一些编译 Node.js 源码必要的包

```bash
apt-get update

apt-get install python gcc make g++
```

3. 下载 Node.js 源码

进入 Node.js 官网或者中文官网下载专区，找到源码下载地址，形如 node-v10.15.0.tar.gz 的地址

```bash
wget https://npm.taobao.org/mirrors/node/v10.15.0/node-v10.15.0.tar.gz
```

4. 解压、编译

依次执行

```bash
tar -zxvf node-v10.15.0.tar.gz

cd node-v10.15.0

./configure

make

sudo make install
```

最后一步可能需要等待较长时间。