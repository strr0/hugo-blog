---
title: "Linux 环境下 Nginx 的安装与使用"
date: 2024-11-12T11:00:00+08:00
categories: ["operations"]
tags: ["linux"]
draft: false
---

## 介绍

Nginx (engine x) 是一个高性能的 HTTP 和反向代理 web 服务器，同时也提供了 IMAP/POP3/SMTP 服务。

## 1 安装

### 1.1 编译环境依赖

#### Debian 系

```
apt install gcc zlib1g zlib1g-dev libpcre3-dev libssl-dev
```

#### Redhat 系

```
yum install gcc zlib zlib-devel pcre pcre-devel openssl openssl-devel
```

### 1.2 获取源码

```
wget http://nginx.org/download/nginx-1.25.1.tar.gz
```

### 1.3 编译

```
tar -xzvf nginx-1.25.1.tar.gz
cd nginx-1.25.1
./configure
make
```

### 1.3 安装

安装到 /usr/local/bin
```
make install
```

## 2 使用

### 2.1 复制配置（可选）

```
cp /usr/local/nginx/conf/nginx.conf /usr/local/nginx/conf/nginx-web.conf
```

### 2.2 配置模板

```
http {
  upstream backend {
    ip_hash;
    server 127.0.0.1:8081;
  }

  server {
    listen 8080;
    server_name localhost;

    location / {
      root /home/xxxx;
      index index.html;
    }

    location /api/ {
      proxy_set_header Host $http_host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header REMOTE-HOST $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_pass http://backend/;
    }
  }
}
```

### 2.3 配置模板（多项目）

#### 2.3.1 修改前端配置

vite.config.js 配置添加前缀
```
export default defineConfig(({ mode, command }: ConfigEnv): UserConfig => {
  const env = loadEnv(mode, process.cwd());
  return {
    // 前缀配置，如 /system-a
    base: env.VITE_APP_CONTEXT_PATH
  }
}
```
router/index.js 配置添加前缀
```
const router = createRouter({
  // 前缀配置，如 /system-a
  history: createWebHistory(import.meta.env.VITE_APP_CONTEXT_PATH),
});
```
axios 请求添加前缀
```
const service = axios.create({
  // 前缀配置，如 /system-a/api
  baseURL: import.meta.env.VITE_APP_BASE_API,
  timeout: 50000
});
```

#### 2.3.2 Nginx 配置

```
http {
  upstream backend {
    ip_hash;
    server 127.0.0.1:8081;
  }

  server {
    listen 8080;
    server_name localhost;

    location / {
      root       /usr/share/nginx/html/dist;
      try_files  $uri $uri/ /index.html;
      index      index.html index.htm;
    }

    location /system-a {
      alias      /usr/share/nginx/html/system-a/dist;
      try_files  $uri $uri/ /system-a/index.html;
      index      index.html index.htm;
    }

    location /api/ {
      proxy_set_header Host $http_host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header REMOTE-HOST $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_pass http://backend/;
    }
  }
}
```

### 2.4 启动与停止

启动
```
/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx-web.conf
```
刷新配置
```
/usr/local/nginx/sbin/nginx -s reload -c /usr/local/nginx/conf/nginx-web.conf
```
停止
```
/usr/local/nginx/sbin/nginx -s stop
```

## 3 问题与修复

请求转发配置：proxy_pass 地址后缀的 / 会将拦截的 location 替换成转发后的地址，没有后缀则会将拦截的 location 补充到转发地址之后。