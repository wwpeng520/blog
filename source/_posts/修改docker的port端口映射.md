---
title: 修改docker的port端口映射
date: 2019-01-08 16:54:20
tags:
- Docker
---

## 查看容器

```bash
docker ps -a
```

```
CONTAINER ID / IMAGE / COMMAND / CREATED / STATUS / PORTS / NAMES

e0167565bd4e / registry.docker-cn.com/sameersbn/postgresql:9.6-2 / "/sbin/entrypoint.sh" / 6 days ago / Up 18 minutes / 0.0.0.0:5432->5432/tcp / weapp_guitar_pg

b7e624cfc669 / hello-world / "/hello" / 9 days ago / Exited (0) 9 days ago / reverent_lumiere```

上面的命令可以查看到本地的所有 docker 容器：有两个 CONTAINER，ID 分别为 e0167565bd4e 和 b7e624cfc669。
<!-- more -->

## 停止容器

```bash
# 用 CONTAINER ID (如e0167565bd4e)代替下面的 xxx
docker stop xxx
```

## 修改容器的端口映射配置文件

```bash
# 用 CONTAINER ID (如e0167565bd4e)代替下面的 {container_id}
vim /var/lib/docker/containers/{container_id}/hostconfig.json
```

找到`"HostPort":"5432"`字段把端口 5432 改成新的端口

## 重启docker服务

```bash
service docker restart
docker start xxx
```
