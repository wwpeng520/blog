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

附 pm2 命令
$ pm2 start app.js              # 启动app.js应用程序
$ pm2 start app.js -i 4         # cluster mode 模式启动4个app.js的应用实例
$ pm2 start app.js --name="api" # 启动应用程序并命名为 "api"
$ pm2 start app.js --watch      # 当文件变化时自动重启应用
$ pm2 start script.sh           # 启动 bash 脚本
$ pm2 list                      # 列表 PM2 启动的所有的应用程序
$ pm2 monit                     # 显示每个应用程序的CPU和内存占用情况
$ pm2 show [app-name]           # 显示应用程序的所有信息
$ pm2 logs                      # 显示所有应用程序的日志
$ pm2 logs [app-name]           # 显示指定应用程序的日志
pm2 flush
$ pm2 stop all                  # 停止所有的应用程序
$ pm2 stop 0                    # 停止 id为 0的指定应用程序
$ pm2 restart all               # 重启所有应用
$ pm2 reload all                # 重启 cluster mode下的所有应用
$ pm2 gracefulReload all        # Graceful reload all apps in cluster mode
$ pm2 delete all                # 关闭并删除所有应用
$ pm2 delete 0                  # 删除指定应用 id 0
$ pm2 scale api 10              # 把名字叫api的应用扩展到10个实例
$ pm2 reset [app-name]          # 重置重启数量
$ pm2 startup                   # 创建开机自启动命令
$ pm2 save                      # 保存当前应用列表
$ pm2 resurrect                 # 重新加载保存的应用列表
$ pm2 update                    # Save processes, kill PM2 and restore processes
$ pm2 generate                  # Generate a sample json configuration file
pm2 start app.js --node-args="--max-old-space-size=1024"
