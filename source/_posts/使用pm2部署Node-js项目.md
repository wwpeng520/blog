---
title: 使用pm2部署Node.js项目
date: 2019-02-23 09:48:29
categories: 
- Node.js
tags:
- Node.js
---
如果直接通过 node app.js 等方式来启动应用，如果报错了可能直接停止整个应用运行，而且经常会需要启动多个应用的情况。
pm2 可以帮我们很方便的启动多个进程，进程崩溃自动重启，也可以帮我们创建系统启动脚本，这样，机器重启了，我们的项目也可以自动重启。
<!-- more -->
1 安装 PM2 包：

```bash
npm install -g pm2
```

2 利用 pm2 启动

```bash
# 在项目文件夹下
pm2 start app.js
# pm2 start ./bin/www
```

也可以配置 JSON 文件

```json
{
  "apps": [{
    "name": "api",
    "script": "server/index.js"
  },{
    "name": "front-end",
    "script": "client/index.js"
  }]
}
```

然后启动如下命令，这样相当于启动了前端和后端两个应用。

```bash
pm2 start app.json
```

3 配置 Nginx 方向代理

```conf
# /etc/nginx/conf.d/xxx.conf 文件
server {
  listen 80;
  server_name app.xxx.com;
  rewrite ^(.*) https://$host$1 permanent;
}

server {
  listen 443 ssl;
  server_name app.xxx.com;

  location / {
      proxy_pass http://127.0.0.1:3000;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection 'upgrade';
      # proxy_set_header Host $host;
      proxy_cache_bypass $http_upgrade;

      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;
      proxy_set_header X-NginX-Proxy true;
  }
}
```

重启 Nginx：依次执行下面命令

```bash
nginx -t
service nginx restart
service nginx reload
```
