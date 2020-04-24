---
layout: post
title: 在阿里云上搭建Nginx服务器
date: 2020-04-24
Author: Syaoran
tags: [aliyun, nginx, web, server]
comments: true
---

这篇文章用来记录如何在阿里云上搭建Nginx服务器

### 购买阿里云服务器

现在阿里云有[云翼计划](https://promotion.aliyun.com/ntms/act/campus2018.html?spm=5176.12901015.0.i12901015.2f17525cxY3hFS#stage)，学生优惠价一个月只要九块五
> 建议系统选centOS

购买完成后进入[ecs控制台](https://ecs.console.aliyun.com/)，有两件事情要做：  

- 初始化root密码  
阿里云的root初始密码不是通过手机短信发送的，要在控制台手动更改。选择实例->进入实例配置页->左侧实例详情->基本信息 更多->重置实例密码  

- 开放80端口  
这点很重要，我不明白那么多Nginx+阿里云的教程里为什么没有人提到这一点，阿里云服务器的80端口默认是屏蔽的，而Nginx默认监听的就是80端口，这会导致所有对Nginx的请求都会被丢掉。  
在[ecs控制台](https://ecs.console.aliyun.com/)左侧的网络与安全选项卡->安全组->为实例配置规则->新建规则：  
端口范围填80/80；授权对象填0.0.0.0/0

### 配置Nginx

ssh连接服务器
```
ssh root@ip
```
yum安装nginx
```
yum install nginx -y
```
启动nginx
```
nginx
```


>
```
// 其他常见命令
nginx -s reload  // 重启
nginx -s reload  // 停止
```

此时，访问 http://<您的域名或ip> 可以看到 Nginx 的测试页面。
> 不同环境下的测试页面并不相同，比如我的就是一个Welcome to centOS的页面，并没有网上教程里的Nginx的字样。

打开 Nginx 的默认配置文件 `/etc/nginx/nginx.conf` ，修改 Nginx 配置，将默认的 `root /usr/share/nginx/html;` 修改为: `root /data/www;`，如下
```
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /data/www;  # 修改这里

        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

}
```

配置文件将 `/data/www/` 作为所有静态资源请求的根路径，如访问: `http://<您的域名>/index.js` ，将会去 `/data/www/` 目录下去查找 `index.js`。

现在让我们新建一个静态文件，查看服务是否运行正常。首先让我们在 /data 目录 下创建 www 目录，如：
```
mkdir -p /data/www
vim /data/www/index.html
```
编写一个简单的index.html：
```
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <title>helloworld</title>
</head>
<body>
辣你禽蛋是真滴牛批
</body>
</html>
```

现在我们需要重启 Nginx 让新的配置生效：
```
nginx -s reload
```
现在访问 `http://<您的域名>/index.html`应该会看到刚刚编写的helloworld。 




















