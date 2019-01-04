---
title: 阿里云ECS主机DataGrip连接pg数据库
date: 2019-01-04 11:15:54
tags:
---

环境

- 阿里云服务器 ECS + Ubuntu14.04
- Macbook pro
- Docker --version ：Docker version 18.06.1-ce, build e68fc7a
- Datagrip

<!-- more -->
前期准备

1. 购买阿里云服务器 ECS
2. 安装 Docker，创建或拉取镜像，示例如下：

```bash
# bash 下输入后回车
docker run --name database_name_pg -d \
--publish 5432:5432 \
--restart always \
--env 'DB_USER=database_user' --env 'DB_PASS=xxxxxxxxxxxxxxxxxxxxxxxxxx' \
--env 'DB_NAME=database_name' \
registry.docker-cn.com/sameersbn/postgresql:9.6-2
```

阿里云控制台 → 云服务器 ECS → 网络和安全 → 安全组 → 配置规则 → 添加安全组规则，按照以下信息填写提交
![云服务器 ECS](/images/aliyun/aliyun_ces_rule.png)
![配置规则](/images/aliyun/aliyun_ces_add_rule.png)

本地打开 Datagrip，如图添加数据库，测试连接即可
![Datagrip连接](/images/aliyun/datagrip_add_db.png)
