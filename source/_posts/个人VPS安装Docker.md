---
title: 个人VPS安装Docker
date: 2019-01-02 10:39:32
categories: 
- Linux
tags:
- Linux
- VPS
---
题记：服务器部署很少涉及，虽然买了几个 VPS 玩玩，但作用一般也就部署个 Hexo 个人博客和配置个 ShadowSocks 翻墙用用。最近想做个小程序玩玩，前后端都自己来写，服务器端需要部署到 VPS，很多东西以前大致摸索过一些，只是用的少，很多步骤需要边 Google 边操作。所以一些过程打算自己记录下来，以供以后翻阅。
VPS 是 Ubuntu 系统。

0. 卸载老版本

docker 或 docker-engine

```bash
sudo apt-get remove docker docker-engine docker.io
```

<!-- more -->

1. 安装

```bash
# Update the apt package index
sudo apt-get update

# Install packages to allow apt to use a repository over HTTPS
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

# Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# 确认已获得指纹密钥
sudo apt-key fingerprint 0EBFCD88

pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22

# 设置 the stable repository
sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"

# 安装 docker-ce
sudo apt-get install docker-ce

# 或者选择安装特定版本
apt-cache madison docker-ce
sudo apt-get install docker-ce=<VERSION> # 示例：docker-ce | 18.09.0~ce-0~ubuntu
```

验证 docker-ce 已安装完成

```bash
sudo docker run hello-world
```

[参考链接](https://docs.docker.com/install/linux/docker-ce/ubuntu/)