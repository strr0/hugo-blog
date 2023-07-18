---
title: "Nginx"
date: 2023-07-18T15:00:00+08:00
draft: false
---

### 介绍
Nginx

### Nginx安装
1. 下载[nginx](http://nginx.org/download/nginx-1.25.1.tar.gz)
2. 解压
```
tar -xzvf nginx-1.25.1.tar.gz -C /opt/software
```
3. 安装nginx环境依赖
```
### yum install gcc zlib zlib-devel pcre pcre-devel openssl openssl-devel
apt install gcc zlib1g zlib1g-dev libpcre3-dev libssl-dev
```
4. 编译及安装
```
/opt/software/nginx-1.25.1/configure
make
make install
```
5. 配置
```
cp /usr/local/nginx/conf/nginx.conf /usr/local/nginx/conf/nginx-web.conf
vim /usr/local/nginx/conf/nginx-web.conf
```
配置模板如下
```
nginx-web.conf
server {
  listen 8080;
  server_name localhost;
  location / {
    root /home/xin/web/dist;
    index index.html;
  }
}
```
6. 启动与停止
```
### 启动
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx-web.conf
### 重启
/usr/local/nginx/sbin/nginx -s reload -c /usr/local/nginx/conf/nginx-web.conf
### 停止
/usr/local/nginx/sbin/nginx -s stop
```