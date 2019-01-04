---
title: 个人VPS部署Node项目
date: 2019-01-03 11:21:35
categories: 
- Linux
tags:
- Linux
- VPS
- Node
---
题记：服务器部署很少涉及，虽然买了几个 VPS 玩玩，但作用一般也就部署个 Hexo 个人博客和配置个 ShadowSocks 翻墙用用。最近想做个小程序玩玩，前后端都自己来写，服务器端需要部署到 VPS，很多东西以前大致摸索过一些，只是用的少，很多步骤需要边 Google 边操作。所以一些过程打算自己记录下来，以供以后翻阅。
VPS 是 Ubuntu 系统。

查看 ~/.ssh 目录下是否已创建密钥

```bash
cd ~/.ssh && ll -ls
```

如果存在 id_rsa.pub 文件则本机已创建过 SSH 密钥。否则创建：

```bash
ssh-keygen -t rsa -C "your_email@example.com"
```

<!-- more -->
![ssh key 创建](/images/vps/ssh_key.png)

```bash
cat ~/.sshid_rsa.pub
```

在项目的 SSH 管理页面添加刚刚创建的 SSH key。如托管在阿里云的新增 SSH 页面如下：
![ssh key 创建](/images/vps/add_ssh_key.png)

```bash
# 通过 git 的方式拉取到服务器
git clone git@code.aliyun.com:xxx/xxxx
```

启动 Node 项目，可以使用 pm2 或者其他方式

在 /etc/nginx/conf.d 目录下创建 Nginx 配置文件 api.conf

```conf
# 60000 为 Node 应用运行的端口号
server {
  listen 80;
  server_name api.abc.com;
  location / {
      proxy_pass http://127.0.0.1:60000;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection 'upgrade';
      proxy_set_header Host $host;
      proxy_cache_bypass $http_upgrade;
  }
}
```

```bash
# 测试配置文件是否正确
nginx -t

# 修改了配置文件，重启服务
nginx -s reload
```

输入`api.abc.com`测试一下是否运行成功。