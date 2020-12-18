---
title: Docker安装Nginx
date: 2020-05-14 11:38:51
categories: 笔记
tags: [Docker,Nginx]
type: posts
description: Docker安装Nginx
---
### docker-compose.yml
```yaml
version: '3.1'
services:
  nginx:
    restart: always
    image: nginx
    container_name: nginx
    ports:
      - 81:80
    volumes:
      - ./conf/nginx.conf:/etc/nginx/nginx.conf
      - ./wwwroot:/usr/share/nginx/wwwroot
```
#### `docker-compose.yml` 同级下新建 conf 、 wwwroot 文件夹 , wwwroot 里新建 index.html ，写个 hello 当测试页面
### conf 里创建 nginx.conf
```
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    
    keepalive_timeout  65;
    # 配置虚拟主机 192.168.80.144
    server {
	# 监听的端口
        listen       80;
	# 虚拟主机名称这里配置ip地址
        server_name  192.168.80.144;
	# 所有的请求都以 / 开始，所有的请求都可以匹配此 location
        location / {
	    # 使用 root 指令指定虚拟主机目录即网页存放目录
	    # 比如访问 http://ip/index.html 将找到 /usr/local/docker/nginx/wwwroot/html80/index.html
	    # 比如访问 http://ip/item/index.html 将找到 /usr/local/docker/nginx/wwwroot/html80/item/index.html

            root   /usr/share/nginx/wwwroot;
	    # 指定欢迎页面，按从左到右顺序查找
            index  index.html index.htm;
        }

    }
}
```
### 运行 `docker-compose up -d ` 浏览器输入 `http://192.168.80.144:81`