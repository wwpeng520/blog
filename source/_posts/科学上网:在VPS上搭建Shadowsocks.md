---
title: 科学上网：在 Vultr VPS 上搭建 Shadowsocks
date: 2018-03-14 15:11:05
categories: 
- 通用技巧
tags:
- common
---
题记：网上此类详细教程有很多，我只根据个人需要捡几个重点记录下来。

相信很多人都知道，中国有两个长城：

1. 世界七大奇迹之一的万里长城The Great Wall;
2. 长城防火墙Great Firewall (of China)，简称GFW.

## 1. 购买 VPS

这里说的是第二个。很多特殊群体有访问国外网站的需求，比如 Google, Fackbook, Youtube ...尤其是广大的程序猿同志，但是这些网站被我们伟大的墙挡在了墙外，通过正常方法是访问不了的。网上也有很多网站提供 VPN 服务，只是稳定性等不是很理想。而利用自己的服务器搭建中转服务间接访问此类墙外的网站成了很多人的首选。

<!-- more -->

购买服务器的步骤这里就掠过了，其实也很简单，我用的是 [Vultr](https://www.vultr.com/)，主流的 VPS 服务商还有很多，比如搬瓦工等等。购买国外的 VPS 可能需要您的信用卡是双币卡，支持 VISA/MASTERCARD 等，不过现在很多服务商也支持支付宝的方式付款，具体可以自行搜索，[这里](https://www.zhujiceping.com/17104.html)是随便找到的一个介绍支持支付宝的服务商。详细购买过程可以参考[这里](https://medium.com/@zoomyale/%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91%E7%9A%84%E7%BB%88%E6%9E%81%E5%A7%BF%E5%8A%BF-%E5%9C%A8-vultr-vps-%E4%B8%8A%E6%90%AD%E5%BB%BA-shadowsocks-fd57c807d97e)。

购买 VPS 后安装虚拟系统的时候最好添加 SSH Key，把你在用的笔记本或者台式机的 ～/.ssh/id_rsa 目录下的内容添加到设置选项中。当然这步也可以放在后面做，你可以随时用 ssh 方式登录 VPS，然后在服务器上添加你的 SSH Key。

## 2. 连接 VPS

如果你使用的是 Windows 操作系统，连接 VPS 需要借助一些工具，比如 Putty。你可以参考这篇[百度经验](http://jingyan.baidu.com/article/8ebacdf0d9e86549f75cd57b.html)完成安装，安装完成后步骤与下文 Mac 一致。

Mac 只要打开「终端」（或者 iTerm 等）应用即可开始使用 SSH 连接 VPS。终端输入命令

```bash
ssh root@45.xxx.xxx.xxx
```

root@后面的是你购买的 VPS 服务器 IP 地址（IP 地址和密码一般可在购买 VPS 的网站上获取）。下图是我的 Vultr VPS 信息：
![Vultr VPS 信息](/images/common-articles/my-vultr.png)
接下来确认一系列信息后输入密码，可以直接复制你的密码粘贴上去，这里不管是输入还是粘贴，屏幕上都不会显示字符，所以贴完后也是看不到字符的，回车就行。当出现上图那串 root@vultr:~# 时，说明已成功登录。

每次都需要输入一串奇形怪状的密码是不是很麻烦，在上一步提到了添加 SSH Key 的方式，如果添加成功了，以后每次登录只需输入
> ssh root@45.xxx.xxx.xxx

直接登录而不需要密码了。方法自行谷歌吧。

## 3. VPS 上部署 Shadowsocks

这里使用 [teddysun](https://teddysun.com/342.html) 的一键安装脚本。

以下是3条命令，每次输入一行、回车，等待屏幕上的操作完成后再输入下一条。

```bash
1 wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
2 chmod +x shadowsocks.sh
3 ./shadowsocks.sh 2>&1 | tee shadowsocks.log
```

最后一步输完，终端会显示信息要你为 Shadowsocks 服务设置一个个人密码和端口号（1-65535间都行），这里不知道或者无所谓的话可以直接回车，结束后就会看到你的 Shadowsocks 的配置信息，最好截屏下来保存以防以后需要。
![Shadowsocks VPS 配置](/images/common-articles/vps-shadowsocks.png)

## 4. 安装 Shadowsocks 客户端

根据操作系统下载相应的客户端

- [Mac 版客户端下载](https://sourceforge.net/projects/shadowsocksgui)
- [Win 版客户端下载](https://github.com/shadowsocks/shadowsocks-windows/releases)

推荐使用 brew 安装，可以参考[这篇文章](https://shino.space/2017/%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E6%90%AD%E5%BB%BAshadowsocks%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91%E7%B3%BB%E7%BB%9F-%E5%AE%A2%E6%88%B7%E7%AB%AF%E7%AF%87/)

打开客户端，在「服务器设定」里新增服务器。然后依次填入服务器 IP、服务器端口、你设的密码和加密方式。
![Shadowsocks设置]](/images/common-articles/shadowsocks-setting.png)
设置完服务器后选择刚刚添加的服务器设置为代理服务器就可以访问限制网站了，Shadowsocks 最好选择 PAC 模式上网。

参考文章

- [科学上网的终极姿势-在-vultr-vps-上搭建-shadowsocks](https://medium.com/@zoomyale/%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91%E7%9A%84%E7%BB%88%E6%9E%81%E5%A7%BF%E5%8A%BF-%E5%9C%A8-vultr-vps-%E4%B8%8A%E6%90%AD%E5%BB%BA-shadowsocks-fd57c807d97e)
