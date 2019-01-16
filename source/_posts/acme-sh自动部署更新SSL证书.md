---
title: acme.sh自动部署更新SSL证书
date: 2019-01-05 15:28:54
categories: 
- Linux
tags:
- Linux
- SSL
---

我在另一篇{% post_link Node项目部署并实现Nginx域名绑定 %}文章里介绍了 FreeSSL 免费证书的部署过程，这里介绍使用[acme.sh](https://github.com/Neilpang/acme.sh)的方式，可以快速配置全站的https。
<!-- more -->
acme.sh 是一个自动申请 https 证书的脚本，实现了 acme 协议, 可以从 [letsencrypt](https://letsencrypt.org/) 生成免费的证书。

## 安装

安装过程不会污染已有的系统任何功能和文件, 所有的修改都限制在安装目录中: ~/.acme.sh/

```bash
curl  https://get.acme.sh | sh
```

## 生成证书

一般有两种方式验证: http 和 dns 验证

1. http 方式需要在你的网站根目录下放置一个文件, 来验证你的域名所有权，完成验证，然后就可以生成证书了。
2. dns 方式, 在域名上添加一条 txt 解析记录, 验证域名所有权。

方式2无法自动更新证书，因为每次都需要手动重新解析验证域名所有权，如图
![dns解析](/images/ssl/freessl4.png)

所以推荐使用方式1，实现 SSL 证书的自动更新。不过使用这种方式需要域名服务商的支持，我使用的是阿里云的，其他的可以查看 [How to use DNS API](https://github.com/Neilpang/acme.sh/blob/master/dnsapi/README.md) 找到自己的 DNS 服务商对应的命令。
a. 在 bash 中执行下面两句命令，阿里云的如下：

```bash
# 换成自己的 API Key ID 和 API Key Secret
export Ali_Key="xxxxx4x7md0xKyUxxfWP"
export Ali_Secret="xxxxxyUGgd4LkKRHWKgxxx0j8x6"
```

Ali_Key 和 Ali_Secret 可以在[这里](https://ak-console.aliyun.com/#/accesskey)找到。

b. 生成证书：

```bash
acme.sh --issue --dns dns_ali -d example.com -d '*.example.com'

# 如果提示 acme.sh 命令找不到，可以试试先执行
alias acme.sh=~/.acme.sh/acme.sh
```

等待执行完成，期间会有 120 秒的倒计时，结束后如果显示成功，则证书申请成功。
![dns解析](/images/ssl/acmesh_success.png)

c. 更新 Nginx 配置信息

如果原来是有配置 SSL 证书的，所以只需要把对应的证书路径进行替换即可。

```conf
# /etc/nginx/conf.d/xxx.conf
ssl_certificate     /root/.acme.sh/example.com/example.com.cer;
ssl_certificate_key /root/.acme.sh/example.com/example.com.key;
```

如果原本没有配置ssl证书的，可以在nginx.conf里面加上一段配置信息：

```conf
server {
  listen 443;

  ssl on;
  ssl_certificate /root/.acme.sh/example.com/example.com.cer;
  ssl_certificate_key /root/.acme.sh/example.com/example.com.key;
}
```

使用全站加密，http 自动跳转 https（可选）

```conf
server {
  listen         80;
  server_name    my.domain.com;
  return         rewrite ^(.*) https://$host$1 permanent;
}

server {
  listen         443 ssl;
  server_name    my.domain.com;
  [....]
}
```

也可以在 /etc/nginx/conf.d/ 目录下创建 SSL 配置文件，里面填写如下配置信息：

```conf
ssl_certificate /root/.acme.sh/example.com/example.com.cer;
ssl_certificate_key /root/.acme.sh/example.com/example.com.key;
```

重启 Nginx。
需要注意的是：如果使用阿里云的云主机需要添加安全组规则，如下图：
入口：阿里云控制台 → 云服务器 ECS → 网络和安全 → 安全组 → 配置规则 → 添加安全组规则
![云服务器 ECS](/images/aliyun/443.png)

## 更新证书

目前证书在 60 天以后会自动更新，你无需任何操作。今后有可能会缩短这个时间，不过都是自动的。

## 更新 acme.sh

目前由于 acme 协议和 letsencrypt CA 都在频繁的更新，因此 acme.sh 也经常更新以保持同步。

```bash
# 升级 acme.sh 到最新版
acme.sh --upgrade

# 如果你不想手动升级, 可以开启自动升级
acme.sh  --upgrade  --auto-upgrade

# 可以随时关闭自动更新
acme.sh --upgrade  --auto-upgrade  0
```

参考文章

- [轻松全站 HTTPS，还没用上 https (可申请泛域名证书)的朋友可以操练起来了](https://cnodejs.org/topic/5be29f7c21d75b74609f4fbf)
- [CentOS 7 使用 acme.sh 自动申请免费 SSL 证书](https://blog.imzhengfei.com/centos-7-shi-yong-acme-sh-zi-dong-shen-qing-mian-fei-ssl-zheng-shu/)
- [acme.sh 中文说明](https://github.com/Neilpang/acme.sh/wiki/%E8%AF%B4%E6%98%8E)