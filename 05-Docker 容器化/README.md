# 🐳 Docker 容器化

> 10 分钟上手，掌握大厂容器化技术

---

## 1. 为什么用 Docker？

**解决了什么问题：**
- 环境不一致？
- 部署太复杂？
- 依赖冲突？

**大厂为什么用：**
- ✅ 环境一致：一次构建，到处运行
- ✅ 快速部署：秒级启动
- ✅ 资源隔离：容器间互不影响
- ✅ 易于扩展：轻松水平扩展

**应用场景：**
- 应用容器化
- 微服务部署
- CI/CD
- 开发环境

---

## 2. 快速开始（10 分钟上手）

### 环境要求

- Docker Desktop（Windows/Mac）
- 或 Docker CE（Linux）

### 第一个 Docker 示例

**步骤 1：安装 Docker**

```bash
# Windows/Mac：下载 Docker Desktop
# https://www.docker.com/products/docker-desktop

# Linux（Ubuntu）
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

**步骤 2：验证安装**

```bash
docker --version
docker run hello-world
```

**步骤 3：运行第一个容器**

```bash
# 运行 Nginx
docker run -d -p 8080:80 --name my-nginx nginx

# 访问：http://localhost:8080
```

**步骤 4：查看容器**

```bash
# 查看运行中的容器
docker ps

# 查看所有容器
docker ps -a

# 查看容器日志
docker logs my-nginx
```

**步骤 5：停止和删除**

```bash
# 停止容器
docker stop my-nginx

# 删除容器
docker rm my-nginx

# 删除镜像
docker rmi nginx
```

---

## 3. 核心概念

### Docker 架构图

```
┌──────────────┐
│   Docker     │
│   Client    │
└──────┬───────┘
       │
┌──────┴───────┐
│   Docker     │
│   Daemon    │
└──────┬───────┘
       │
┌──────┴───────┐     ┌──────────┐
│   Images    │     │ Registry │
│   镜像      │     │  仓库    │
└──────┬───────┘     └──────────┘
       │
┌──────┴───────┐
│  Containers  │
│   容器       │
└──────────────┘
```

### 核心组件

| 组件 | 说明 |
|------|------|
| **Image（镜像）** | 只读模板，包含应用和依赖 |
| **Container（容器）** | 镜像的运行实例 |
| **Dockerfile** | 构建镜像的脚本 |
| **Registry（仓库）** | 存放镜像的地方 |

### 镜像 vs 容器

| 镜像 | 容器 |
|------|------|
| 只读模板 | 运行实例 |
| 可以创建多个容器 | 可以启动、停止、删除 |
| 类似类 | 类似对象 |

---

## 4. 常用命令

### 镜像命令

```bash
# 搜索镜像
docker search nginx

# 拉取镜像
docker pull nginx:latest

# 查看镜像
docker images

# 删除镜像
docker rmi nginx

# 构建镜像
docker build -t my-app .
```

### 容器命令

```bash
# 运行容器
docker run -d -p 8080:80 --name my-app nginx

# 查看容器
docker ps

# 停止容器
docker stop my-app

# 启动容器
docker start my-app

# 重启容器
docker restart my-app

# 删除容器
docker rm my-app

# 查看日志
docker logs my-app

# 进入容器
docker exec -it my-app bash

# 查看容器信息
docker inspect my-app
```

### 常用参数

| 参数 | 说明 |
|------|------|
| **-d** | 后台运行 |
| **-p** | 端口映射 |
| **--name** | 容器名称 |
| **-v** | 挂载卷 |
| **-e** | 环境变量 |
| **--network** | 网络 |
| **--restart** | 重启策略 |

---

## 5. 实战场景

### 场景 1：Dockerfile 构建镜像

```dockerfile
# 基础镜像
FROM openjdk:8-jdk-alpine

# 维护者信息
LABEL maintainer="your@email.com"

# 设置工作目录
WORKDIR /app

# 复制 jar 包
COPY target/my-app.jar app.jar

# 暴露端口
EXPOSE 8080

# 启动命令
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**构建和运行：**

```bash
# 构建镜像
docker build -t my-app:1.0 .

# 运行容器
docker run -d -p 8080:8080 --name my-app my-app:1.0
```

### 场景 2：数据持久化

```bash
# 挂载卷（数据持久化）
docker run -d \
  -p 3306:3306 \
  -v /my/data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=123456 \
  --name mysql \
  mysql:5.7
```

### 场景 3：多容器部署

```bash
# 创建网络
docker network create my-network

# 启动数据库
docker run -d \
  --name mysql \
  --network my-network \
  -e MYSQL_ROOT_PASSWORD=123456 \
  mysql:5.7

# 启动应用
docker run -d \
  --name my-app \
  --network my-network \
  -p 8080:8080 \
  -e DB_HOST=mysql \
  my-app:1.0
```

### 场景 4：Docker Compose

```yaml
# docker-compose.yml
version: '3'
services:
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: 123456
    volumes:
      - ./data:/var/lib/mysql
    networks:
      - my-network
  
  my-app:
    build: .
    ports:
      - "8080:8080"
    environment:
      DB_HOST: mysql
    depends_on:
      - mysql
    networks:
      - my-network

networks:
  my-network:
    driver: bridge
```

**使用：**

```bash
# 启动所有服务
docker-compose up -d

# 查看状态
docker-compose ps

# 查看日志
docker-compose logs

# 停止所有服务
docker-compose down
```

---

## 6. 常见问题

### 问题 1：容器无法启动

**现象：** 容器启动后立即退出

**解决方案：**
1. 查看日志：`docker logs container_name`
2. 检查端口是否被占用
3. 检查配置文件

### 问题 2：容器间无法通信

**现象：** 容器 A 无法访问容器 B

**解决方案：**
1. 使用自定义网络
2. 使用容器名访问
3. 检查防火墙

### 问题 3：数据丢失

**现象：** 容器删除后数据没了

**解决方案：**
1. 使用挂载卷
2. 使用 Docker 卷
3. 定期备份

### 问题 4：镜像太大

**现象：** 镜像文件过大

**解决方案：**
1. 使用精简基础镜像（alpine）
2. 多阶段构建
3. 清理缓存

---

## 7. 面试题

### Q1：Docker 和虚拟机的区别？

**参考答案：**
- **架构：** Docker 是容器化，虚拟机是虚拟化
- **启动速度：** Docker 秒级，虚拟机分钟级
- **资源占用：** Docker 轻量，虚拟机重量
- **隔离性：** Docker 进程级，虚拟机系统级

### Q2：Dockerfile 常用指令有哪些？

**参考答案：**
- FROM：基础镜像
- LABEL：元信息
- WORKDIR：工作目录
- COPY：复制文件
- RUN：执行命令
- EXPOSE：暴露端口
- ENTRYPOINT：启动命令
- CMD：默认命令

### Q3：Docker 数据持久化怎么做？

**参考答案：**
- **挂载卷：** -v /host/path:/container/path
- **Docker 卷：** docker volume create
- **绑定挂载：** 挂载主机目录

### Q4：Docker 网络模式有哪些？

**参考答案：**
- **bridge：** 桥接模式（默认）
- **host：** 主机模式
- **none：** 无网络
- **container：** 共享网络命名空间

### Q5：Docker Compose 的作用？

**参考答案：**
- 多容器编排
- 一键启动/停止
- 服务依赖管理
- 配置文件管理

---

**下一篇：** [Docker 快速入门](./Docker 快速入门.md)

---

**最后更新：** 2026-03-25  
**状态：** ✅ 已完成
