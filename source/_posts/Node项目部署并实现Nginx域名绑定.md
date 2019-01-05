---
title: Node项目部署并实现Nginx域名绑定
date: 2019-01-04 20:44:10
categories: 
- Linux
tags:
- Linux
- Node
- Nginx
---

0. 准备工作

- 购买 VPS 或阿里云服务器 ECS
- VPS 安装 Docker, Node, Nginx
- 购买一个域名并设置域名解析

因为我的 VPS 使用的是 Ubuntu 的系统，所以这里只以 Ubuntu 为例演示。
<!-- more -->

进入 /etc/nginx/ 目录（使用 apt-get 安装的 Nginx的路径），查看 nginx.conf 文件（ nginx.conf 是nginx的主配置文件,里面包含了当前目录的所有配置文件）是否有如下两行，并且未注释：

```conf
include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;
```

目录说明：

- conf.d/ 目录里面可以写我们自己自定义的配置文件
- sites-enabled/ 这里面的配置文件其实就是 sites-available/ 里面的配置文件的软连接，但是由于nginx.conf 默认包含的是这个文件夹,所以我们在 sites-available/ 里面建立了新的站点之后，还要建立个软连接到 sites-enabled/ 里面才行
- sites-available/ 这里是我们的虚拟主机的目录，我们在在这里面可以创建多个虚拟主机

我们把配置文件放在 /etc/nginx/conf.d 目录下，不同的站点使用不同的 *.conf 配置文件。如果需要禁用某个站点，只需将文件名重新命名为不再具有 .conf 后缀，这非常简单，直接且防错。[这篇文章](http://yo.zgserver.com/nginxsite-availableconf-d.html)介绍了 site-available 与 conf.d 目录的异同，写的挺细的。

1. 项目部署

我在另外一篇{% post_link 个人VPS部署Node项目 %}文章里介绍过了，这里就不再赘述了

2. 创建 Nginx 配置文件

```bash
vim /etc/nginx/conf.d/api.example.com.conf
```

输入如下内容：

```conf
server {
  listen 80;
  server_name api.example.com;
  location / {
      proxy_pass http://127.0.0.1:node运行的端口;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection 'upgrade';
      proxy_set_header Host $host;
      proxy_cache_bypass $http_upgrade;
  }
}
```

然后依次执行

```bash
# 检查配置是否有误，并按照报错提示修复错误
nginx -t

# 重启 Nginx 服务
service nginx restart

# 重新载入 Nginx 服务(或者 nginx -s reload)
service nginx reload
```

3. 安装 SSL 证书

可以考虑使用阿里云和又拍云的 SSL 证书，因为我的域名暂未注册，这里演示从 [FreeSSL.org](https://freessl.cn/) 申请的证书。

输入需要申请证书的域名，点击「创建免费的SSL证书」
![图1](/images/ssl/freessl1.png)

输入邮箱地址后，点击「点击创建」
![图2](/images/ssl/freessl2.png)

上一步中选择了 DNS 验证方式，完成上一步显示如下
![图3](/images/ssl/freessl3.png)

这时需要去你的域名服务商那里添加生成的 TXT 记录名与记录值添加到该域名下，等待大约 1 分钟即可验证成功。
![图4](/images/ssl/freessl4.png)

点击图3中的「点击验证」即可看到成功了
![图5](/images/ssl/freessl5.png)

下载证书，上传到服务器上保存起来。
然后登陆服务器修改 Nginx 配置信息，然后重启 Nginx 服务。配置信息如下：

```conf
server {
  # 如果要保留 http 方式，可同时保留下两行
  # listen 80;
  listen 443 ssl;
  server_name api.example.com;

  ssl on;
  ssl_certificate /etc/nginx/conf.d/ssl_files/full_chain.pem;
  ssl_certificate_key /etc/nginx/conf.d/ssl_files/private.key;

  location / {
      proxy_pass http://127.0.0.1:node运行的端口;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection 'upgrade';
      proxy_set_header Host $host;
      proxy_cache_bypass $http_upgrade;
  }
}
```

参考文章

- [使用 Nginx 为 Linux 实例绑定多个域名](https://help.aliyun.com/knowledge_detail/41467.html?spm=5176.11065259.1996646101.searchclickresult.55b56d8br6gbEz)
- [对于nginx而言，site-available与conf.d目录有什么不同的用法](http://yo.zgserver.com/nginxsite-availableconf-d.html)
- [关于FreeSSL 申请证书的相关说明](https://blog.freessl.cn/about-freessl-org-apply-cert-introduce/)