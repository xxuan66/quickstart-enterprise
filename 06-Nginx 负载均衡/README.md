# 🌐 Nginx 负载均衡

> 10 分钟上手，掌握大厂负载均衡技术

---

## 1. 为什么用 Nginx？

**解决了什么问题：**
- 单点故障？
- 并发扛不住？
- 静态资源慢？

**大厂为什么用：**
- ✅ 高性能：10 万 + 并发
- ✅ 稳定性好：7x24 小时运行
- ✅ 功能丰富：负载均衡、反向代理、静态资源
- ✅ 配置简单：易学易用

**应用场景：**
- 负载均衡
- 反向代理
- 静态资源服务
- API 网关

---

## 2. 快速开始（10 分钟上手）

### 环境要求

- Docker（推荐）或本地安装

### 第一个 Nginx 示例

**步骤 1：启动 Nginx**

```bash
# 使用 Docker 启动
docker run -d -p 8080:80 --name nginx nginx

# 访问：http://localhost:8080
```

**步骤 2：查看配置**

```bash
# 进入容器
docker exec -it nginx bash

# 查看配置文件
cat /etc/nginx/nginx.conf
```

**步骤 3：简单配置**

```nginx
# /etc/nginx/nginx.conf
events {
    worker_connections 1024;
}

http {
    server {
        listen 80;
        server_name localhost;
        
        location / {
            root /usr/share/nginx/html;
            index index.html;
        }
    }
}
```

**步骤 4：重启 Nginx**

```bash
docker restart nginx
```

---

## 3. 核心概念

### Nginx 架构图

```
┌──────────┐
│  Client  │
│  客户端  │
└────┬─────┘
     │
┌────┴─────┐
│  Nginx   │
│  负载均衡 │
└────┬─────┘
     │
┌────┴─────┐     ┌──────────┐
│ Server 1 │     │ Server 2 │
│  服务器 1 │     │  服务器 2 │
└──────────┘     └──────────┘
```

### 核心功能

| 功能 | 说明 |
|------|------|
| **反向代理** | 代理后端服务 |
| **负载均衡** | 分发请求到多个服务器 |
| **静态资源** | 提供静态文件服务 |
| **SSL 终结** | HTTPS 加密解密 |

### 负载均衡策略

| 策略 | 说明 |
|------|------|
| **轮询** | 默认，轮流分配 |
| **weight** | 权重，按权重分配 |
| **ip_hash** | 同一 IP 到同一服务器 |
| **least_conn** | 最少连接优先 |

---

## 4. 常用配置

### 反向代理

```nginx
server {
    listen 80;
    server_name api.example.com;
    
    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 负载均衡

```nginx
upstream backend {
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}

server {
    listen 80;
    server_name api.example.com;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
    }
}
```

### 权重配置

```nginx
upstream backend {
    server 192.168.1.10:8080 weight=3;  # 权重 3
    server 192.168.1.11:8080 weight=2;  # 权重 2
    server 192.168.1.12:8080 weight=1;  # 权重 1
}
```

### IP 哈希

```nginx
upstream backend {
    ip_hash;
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
}
```

### 静态资源

```nginx
server {
    listen 80;
    server_name static.example.com;
    
    location / {
        root /var/www/static;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
```

---

## 5. 实战场景

### 场景 1：微服务负载均衡

```nginx
upstream user-service {
    server 192.168.1.10:8081;
    server 192.168.1.11:8081;
}

upstream order-service {
    server 192.168.1.10:8082;
    server 192.168.1.11:8082;
}

server {
    listen 80;
    server_name api.example.com;
    
    location /user/ {
        proxy_pass http://user-service;
    }
    
    location /order/ {
        proxy_pass http://order-service;
    }
}
```

### 场景 2：动静分离

```nginx
server {
    listen 80;
    server_name www.example.com;
    
    # 静态资源
    location /static/ {
        root /var/www;
        expires 30d;
    }
    
    # 动态请求
    location / {
        proxy_pass http://backend;
    }
}
```

### 场景 3：HTTPS 配置

```nginx
server {
    listen 443 ssl;
    server_name www.example.com;
    
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    
    location / {
        proxy_pass http://backend;
    }
}

# HTTP 重定向到 HTTPS
server {
    listen 80;
    server_name www.example.com;
    return 301 https://$server_name$request_uri;
}
```

### 场景 4：限流配置

```nginx
# 限制请求速率
limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;

server {
    location / {
        limit_req zone=one burst=5;
        proxy_pass http://backend;
    }
}
```

---

## 6. 常见问题

### 问题 1：502 Bad Gateway

**现象：** 访问返回 502

**解决方案：**
1. 检查后端服务是否启动
2. 检查 proxy_pass 地址是否正确
3. 检查网络连接

### 问题 2：404 Not Found

**现象：** 静态资源 404

**解决方案：**
1. 检查文件路径是否正确
2. 检查 root 配置
3. 检查文件权限

### 问题 3：跨域问题

**现象：** 前端请求被拦截

**解决方案：**
```nginx
location / {
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
    add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
    
    if ($request_method = 'OPTIONS') {
        return 204;
    }
    
    proxy_pass http://backend;
}
```

### 问题 4：配置不生效

**现象：** 修改配置后不生效

**解决方案：**
1. 检查配置语法：`nginx -t`
2. 重新加载配置：`nginx -s reload`
3. 重启 Nginx

---

## 7. 面试题

### Q1：Nginx 的工作原理？

**参考答案：**
- 基于事件驱动的异步架构
- 单进程多工作进程模式
- 使用 epoll/kqueue 高效处理连接

### Q2：Nginx 负载均衡策略有哪些？

**参考答案：**
- 轮询（默认）
- weight（权重）
- ip_hash（IP 哈希）
- least_conn（最少连接）

### Q3：Nginx 和 Apache 有什么区别？

**参考答案：**
- **架构：** Nginx 异步，Apache 同步
- **性能：** Nginx 高并发更好
- **配置：** Nginx 更简洁
- **功能：** Apache 模块更丰富

### Q4：Nginx 如何处理静态资源？

**参考答案：**
- 直接返回文件
- 配置 expires 缓存
- 开启 gzip 压缩
- 使用 sendfile

### Q5：Nginx 如何实现 HTTPS？

**参考答案：**
- 配置 SSL 证书
- 监听 443 端口
- 配置 ssl_certificate
- HTTP 重定向到 HTTPS

---

**下一篇：** [Nginx 快速入门](./Nginx 快速入门.md)

---

**最后更新：** 2026-03-25  
**状态：** ✅ 已完成
