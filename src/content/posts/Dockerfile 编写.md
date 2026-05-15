---
title: Dockerfile 编写
published: 2025-02-08
description: '镜像构建文件的详细介绍和学习笔记'
image: ''
tags: ["Docker","容器"]
category: '开发工具'
draft: false
lang: 'zh-CN'
---

## 概述

Dockerfile 是一个文本文件，包含了构建 Docker 镜像所需的所有指令。通过 Dockerfile，我们可以自动化镜像构建过程，确保构建的可重复性和一致性。

## 核心概念

- **基础镜像**：使用 `FROM` 指令指定，是构建的起点
- **构建上下文**：构建时的工作目录，包含所有需要复制到镜像中的文件
- **层（Layer）**：每条指令都会创建一个新的镜像层，合理的分层可以优化构建效率

## 基本用法

### 常用指令

```dockerfile
# FROM - 指定基础镜像
FROM node:18-alpine

# WORKDIR - 设置工作目录
WORKDIR /app

# COPY - 复制文件到镜像
COPY package*.json ./

# RUN - 执行命令
RUN npm install

# COPY - 复制应用代码
COPY . .

# ENV - 设置环境变量
ENV NODE_ENV=production
ENV PORT=3000

# EXPOSE - 声明端口
EXPOSE 3000

# CMD - 容器启动时执行的命令
CMD ["node", "index.js"]

# ENTRYPOINT - 容器入口点
ENTRYPOINT ["npm"]
```

### 完整示例

```dockerfile
# 多阶段构建示例
# 构建阶段
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# 生产阶段
FROM node:18-alpine AS production
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

## 实际应用

### Node.js 应用 Dockerfile

```dockerfile
FROM node:18-alpine

# 安装 dumb-init 用于信号转发
RUN apk add --no-cache dumb-init

# 创建非 root 用户
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# 复制依赖文件
COPY package*.json ./

# 安装生产依赖
RUN npm ci --only=production

# 复制应用代码
COPY --chown=nodejs:nodejs . .

# 切换到非 root 用户
USER nodejs

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

EXPOSE 3000

# 使用 dumb-init 启动
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "index.js"]
```

### Python 应用 Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# 安装系统依赖
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    gcc && \
    rm -rf /var/lib/apt/lists/*

# 复制依赖文件
COPY requirements.txt .

# 安装 Python 依赖
RUN pip install --no-cache-dir -r requirements.txt

# 复制应用代码
COPY . .

EXPOSE 8000

CMD ["python", "app.py"]
```

### 静态站点 Dockerfile

```dockerfile
FROM nginx:alpine

# 复制构建产物到 nginx 目录
COPY dist /usr/share/nginx/html

# 复制自定义 nginx 配置
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

## 注意事项

1. **使用 .dockerignore**：排除不需要的文件，减小构建上下文大小
   ```
   node_modules
   .git
   .env
   *.log
   dist
   ```

2. **缓存优化**：先复制依赖文件，再复制代码，利用 Docker 缓存

3. **多阶段构建**：分离构建和运行环境，减小最终镜像大小

4. **安全性**：
   - 使用具体版本标签而非 `latest`
   - 及时更新基础镜像
   - 避免在镜像中存储敏感信息
   - 使用非 root 用户运行应用

5. **镜像大小**：
   - 选择合适的基础镜像（Alpine、Distroless）
   - 清理不必要的文件和缓存
   - 合并 RUN 指令减少层数

6. **健康检查**：添加 HEALTHCHECK 确保容器健康状态可监控

## 总结

编写高效的 Dockerfile 需要理解各个指令的作用和最佳实践。通过合理使用多阶段构建、优化缓存、减小镜像大小等技巧，可以构建出高性能、安全且易于维护的 Docker 镜像。