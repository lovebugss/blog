---
title: Nginx配置虚拟主机
tags: Nginx
categories: Nginx
---
>背景：cdn项目上，由于客户的IDC源站需要传入Host请求头，然后在预热的时候，也需要传入Host请求头进行预热，这就导致了和预热接口的冲突。所有需要自己搭建一套环境来模拟客户的IDC源。这里主要使用Nginx虚拟主机来完成。

虚拟主机（Virtual Host）可以在一台服务器上绑定多个域名，架设多个不同的网站，一般在开发机或者要部署多个小网站的服务器上需要配置虚拟主机。Nginx配置虚拟主机主要有三种方式：

    基于域名的虚拟主机
    基于端口的虚拟主机
    基于IP的虚拟主机 
这里我们主要使用基于域名的虚拟主机，主要配置了 cdn.goclouds.cn，test1.goclouds.cn 和 hello.com 三个host。具体配置如下：
```
user root; 
worker_processes auto; 
#error_log /var/log/nginx/error.log; # 制定日志路径，级别。这个设置可以放入全局块，http块，server块，级别以此为：debug|info|notice|warn|error|crit|alert|emerg 
#pid /run/nginx.pid; 
events { 
    worker_connections  1024; 
} 
http { 
    include       mime.types; # 文件扩展名与文件类型映射表 
    default_type  application/octet-stream; # 默认文件类型(默认为text/plain） 
    sendfile        on; # 允许sendfile方式传输文件，默认为off，可以在http块，server块，location块。 
    keepalive_timeout  65; # 连接超时时间，默认为75s，可以在http，server，location块。 
    server { # 配置默认server，拒绝所有请求 
        listen 80 ; 
        server_name _; # _ 并不是重点 __ 也可以 ___也可以 
        return 403; # 403 forbidden 
    } 
    server { # 配置host1 
        listen       80; 
        server_name  cdn.goclouds.cn; 
        location / { 
            root   /home/ec2-user/web1/; # 根路径 
            index  index.html index.htm; # 设置欢迎页 
        } 
    } 
    server { # 配置host2 
        listen 80; 
        server_name  test1.goclouds.cn; 
        location / { 
            root   /home/ec2-user/web2/; 
            index  index.html index.htm; 
        } 
    } 
    server { # 配置host3 
        listen 80; 
        server_name  hello.com; 
        location / { 
            root   /home/ec2-user/web3/; 
            index  index.html index.htm; 
        } 
    } 
}
```