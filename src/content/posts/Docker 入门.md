---
title: Docker 入门
published: 2025-02-02
description: 'Docker 基础命令的详细介绍和学习笔记'
image: ''
tags: ["Docker","容器"]
category: '开发工具'
draft: false
lang: 'zh-CN'
---

## 概述

Docker 是一个开源的容器化平台，可以将应用程序及其依赖项打包到一个轻量级、可移植的容器中。容器可以在任何安装了 Docker 的环境中运行，确保了开发、测试和生产环境的一致性。

## 核心概念

- **镜像（Image）**：只读的模板，包含运行应用所需的所有内容（代码、运行时、库、环境变量、配置文件等）
- **容器（Container）**：镜像的运行实例，可以被启动、停止、删除
- **仓库（Registry）**：存储和分发镜像的服务，如 Docker Hub
- **Dockerfile**：用于构建镜像的文本文件，包含一系列指令

## 基本用法

### 安装 Docker

```bash
# macOS 安装
brew install --cask docker

# Ubuntu 安装
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

### 镜像操作

```bash
# 搜索镜像
docker search nginx

# 拉取镜像
docker pull nginx:latest

# 查看本地镜像
docker images

# 删除镜像
docker rmi nginx:latest
```

### 容器操作

```bash
# 运行容器
docker run -d -p 80:80 --name my-nginx nginx

# 查看运行中的容器
docker ps

# 查看所有容器（包括停止的）
docker ps -a

# 停止容器
docker stop my-nginx

# 启动已停止的容器
docker start my-nginx

# 删除容器
docker rm my-nginx

# 查看容器日志
docker logs my-nginx

# 进入容器
docker exec -it my-nginx /bin/bash
```

### 常用参数

- `-d`：后台运行
- `-p`：端口映射（主机端口:容器端口）
- `--name`：指定容器名称
- `-v`：挂载卷（主机路径:容器路径）
- `-e`：设置环境变量
- `--rm`：容器停止后自动删除

## 实际应用

### 运行 Node.js 应用

```bash
# 创建简单的 Node.js 应用
mkdir myapp && cd myapp
echo "console.log('Hello Docker!')" > app.js

# 创建 Dockerfile
cat > Dockerfile << 'EOF'
FROM node:18-alpine
WORKDIR /app
COPY . .
CMD ["node", "app.js"]
EOF

# 构建镜像
docker build -t myapp:latest .

# 运行容器
docker run myapp
```

### 运行数据库

```bash
# 运行 MySQL
docker run -d \
  --name mysql \
  -p 3306:3306 \
  -e MYSQL_ROOT_PASSWORD=root123 \
  mysql:8.0

# 运行 MongoDB
docker run -d \
  --name mongodb \
  -p 27017:27017 \
  mongo:latest
```

### 数据持久化

```bash
# 使用数据卷
docker run -d \
  --name mysql \
  -v mysql-data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=root123 \
  mysql:8.0
```

## 注意事项

1. **镜像大小**：尽量使用 Alpine 等轻量级基础镜像来减小镜像体积
2. **安全**：不要在镜像中存储敏感信息，使用环境变量或配置文件
3. **端口冲突**：注意端口映射避免与主机端口冲突
4. **资源限制**：在生产环境中设置 CPU 和内存限制
5. **多阶段构建**：使用多阶段构建来减小最终镜像大小

## 总结

Docker 简化了应用程序的部署和管理，通过容器化技术实现了环境的一致性。掌握 Docker 的基本命令和概念是现代开发者的必备技能，它能够大幅提升开发效率和部署的可靠性。