---
title: 个人VPS上安装Node.js
date: 2018-12-29 16:10:45
categories: 
- Linux
tags:
- Linux
- VPS
- Node.js
---
题记：服务器部署很少涉及，虽然买了几个 VPS 玩玩，但作用一般也就部署个 Hexo 个人博客和配置个 ShadowSocks 翻墙用用。最近想做个小程序玩玩，前后端都自己来写，服务器端需要部署到 VPS，很多东西以前大致摸索过一些，只是用的少，很多步骤需要边 Google 边操作。所以一些过程打算自己记录下来，以供以后翻阅。

买的 VPS 是 [vultr](https://www.vultr.com) 的，Ubuntu 系统。

在Ubuntu系统上安装nodejs有很多种方法，分别为：apt-get在线安装，下载Node.js源码自己编译安装，下载编译好的文件，使用npm安装等方式。推荐通过源码编译安装。
<!-- more -->

## 登录 VPS

使用终端命令

```bash
ssh root@45.xxx.xxx.xxx
```

根据终端提示输入 VPS 的密码即可进入 VPS Ubuntu 系统。如果需要经常登录，可以在 VPS 的管理网站添加 SSH Keys，然后登录 VPS 就不需要每次去复制粘贴密码了。

## 可能需要的操作

用以下命令来升级系统，并且安装一些编译 Node.js 源码必要的包

```bash
apt-get update

apt-get install python gcc make g++
```

## 下载 Node.js 源码

进入 Node.js 官网或者中文官网下载专区，找到源码下载地址，形如 node-v10.15.0.tar.gz 的地址

```bash
wget https://npm.taobao.org/mirrors/node/v10.15.0/node-v10.15.0.tar.gz
```

## 解压、编译

依次执行

```bash
tar -zxvf node-v10.15.0.tar.gz

cd node-v10.15.0

./configure

make

sudo make install
```

最后一步可能需要等待较长时间。

## 卸载

这里只介绍使用源码编译安装的卸载方式。编译用的源码文件夹的话 cd 进去之后执行

```bash
sudo make uninstall
```

不在的话可以再下一个同版本的 Node 源码，解压后依次执行：

```bash
make
sudo make uninstall
```

## 使用 nvm 管理 Node 版本

nvm 是管理 Node.js 版本的工具，它支持在多个 Node.js 版本间切换。

```bash
# 安装
git clone https://github.com/creationix/nvm.git ~/.nvm
cd ~/.nvm
git checkout `git describe --abbrev=0 --tags`

# 激活
. ~/.nvm/nvm.sh
```

为了每次登录后自动激活 nvm，需要将 NMV_DIR、nvm.sh 补齐加入 bash 的 ~/.bashrc（或 zsh 的 ~/.zshrc）

```bash
vim ~/.bashrc
```

```bashrc
# .bashrc 或 .zshrc 加上
export NVM_DIR=~/.nvm
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
[ -r $NVM_DIR/bash_completion ] && . $NVM_DIR/bash_completion
```

```bash
# 配置生效
source ~/.bashrc
```

### nvm 常用命令

```bash
# 查看本地已安装版本
nvm ls

# 查看远程可供安装版本
nvm ls-remote

# 设置本地默认版本
nvm alias default v11

# 安装
nvm install 11

# 卸载
nvm uninstall 11

# 卸载提示`incorrect permissions on installation folder`时执行下面两命令先
sudo chown -R 用户名 "$NVM_DIR/versions/node/版本"
sudo chmod -R u+w "$NVM_DIR/versions/node/版本"
```