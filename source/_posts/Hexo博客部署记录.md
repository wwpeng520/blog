---
title: Hexo博客部署记录
date: 2018-12-29 21:05:56
categories: 
- Linux
tags:
- Linux
- VPN
- Hexo
---
题记：服务器部署很少涉及，虽然买了几个 VPN 玩玩，但作用一般也就部署个 Hexo 个人博客和配置个 ShadowSocks 翻墙用用。最近想做个小程序玩玩，前后端都自己来写，服务器端需要部署到 VPN，很多东西以前大致摸索过一些，只是用的少，很多步骤需要边 Google 边操作。所以一些过程打算自己记录下来，以供以后翻阅。

VPN 是 [vultr](https://www.vultr.com) 的 Ubuntu 系统。

1. 安装 git 和 nginx

```bash
apt-get update
apt-get install git-core nginx
```

<!-- more -->

2. 配置 Nginx

目录说明：

- /var/www/blog 目录用于放置生成的静态文件
- /data/blog/git/blog.git 为 git 仓库目录
- /etc/nginx/conf.d/blog.conf 为配置文件

```bash
mkdir /var/www/blog

vim /etc/nginx/conf.d/blog.conf
```

输入配置信息：

```conf
server {
  listen 80;
  server_name abc.com blog.abc.com;
  location / {
    root /var/www/blog;
    index index.html index.htm;
  }
}
```

重启 nginx

```bash
systemctl restart nginx
```

3. 配置 Git Hooks

blog.git 作为远程 git 仓库，Hexo 在本地生成的博客静态文件可以通过 push 与其同步，post-receive 脚本将在 blog.git 仓库接收到 push 时执行。
在 /data/blog/git 目录下执行

```bash
mkdir blog.git && cd blog.git

git init --bare

vim hooks/post-receive
```

输入：

```sh
#!/bin/bash
git --work-tree=/var/www/blog --git-dir=/data/blog/git/blog.git checkout -f
```

保存后给 post-receive 脚本执行权限

```bash
chmod +x hooks/post-receive
```

4. 配置 git 用户

为了某些原因还是创建 git 用户部署，毕竟 root 还是让它尽量隐藏吧。

```bash
# 添加git用户
sudo adduser git

# 改变 blog.git 目录的拥有者为 git 用户
sudo chown -R git:git blog.git

sudo chown -R git:git /var/www/blog
```

出于安全考虑，我们要让 git 用户可以通过 ssh 正常使用 git，但是无法登录 shell。可以通过编辑 /etc/passwd 来实现，在 /etc/passwd 中找到类似下面的一行：

```
git:x:1000:1000:git,,,:/home/git:/bin/bash
```

修改为：
```
git:x:1000:1000:git,,,:/home/git:/usr/bin/git-shell
```

5. Hexo 项目配置

配置你的 Hexo 博客可以自动 deploy 到服务器上，不需要通过 ftp 上传。修改 Hexo 目录下的 _config.yml 文件，找到 [deploy] 条目，并修改为：

```yml
deploy:
  type: git
  repo: git@46.xxx.xxx.xxx:/data/blog/git/blog.git
  branch: master
```

可以看到上面 `git@46.xxx.xxx.xxx` 是服务器 46.xxx.xxx.xxx 上的 git 用户，git 仓库目录为 `/data/blog/git/blog.git`。
写完文字后发布只需要一行命令：

```bash
hexo generate && hexo deploy
```