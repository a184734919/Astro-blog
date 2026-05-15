---
title: Docker Compose
published: 2025-02-14
description: '多容器编排的详细介绍和学习笔记'
image: ''
tags: ["Docker","容器"]
category: '开发工具'
draft: false
lang: 'zh-CN'
---

## 概述

Docker Compose 是一个用于定义和运行多容器 Docker 应用程序的工具。通过 YAML 配置文件，我们可以一次性启动、停止和管理多个相互关联的容器。

## 核心概念

- **服务（Service）**：定义容器如何运行，包括镜像、端口、环境变量等
- **网络（Network）**：定义服务之间的网络连接
- **卷（Volume）**：定义数据持久化存储
- **项目（Project）**：一组相互关联的服务，通常是一个应用

## 基本用法

### docker-compose.yml 基本结构

```yaml
version: '3.8'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    networks:
      - frontend

  db:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: example
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend

networks:
  frontend:
  backend:

volumes:
  db-data:
```

### 常用命令

```bash
# 启动服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f

# 停止服务
docker-compose down

# 重启服务
docker-compose restart

# 构建镜像
docker-compose build

# 重新构建并启动
docker-compose up -d --build

# 进入容器
docker-compose exec web bash

# 执行命令
docker-compose run web ls -la
```

## 实际应用

### Node.js + MySQL 完整配置

```yaml
version: '3.8'

services:
  # 应用服务
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=mysql
      - DB_USER=root
      - DB_PASSWORD=root123
      - DB_NAME=myapp
    depends_on:
      mysql:
        condition: service_healthy
    networks:
      - app-network
    restart: unless-stopped

  # MySQL 数据库
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: myapp
    volumes:
      - mysql-data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3306:3306"
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-proot123"]
      interval: 5s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  # Redis 缓存
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - app-network
    command: redis-server --appendonly yes
    restart: unless-stopped

  # Nginx 反向代理
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - app
    networks:
      - app-network
    restart: unless-stopped

networks:
  app-network:
    driver: bridge

volumes:
  mysql-data:
  redis-data:
```

### 开发环境配置

```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
      - "9229:9229"  # 调试端口
    environment:
      - NODE_ENV=development
    command: npm run dev

  mongodb:
    image: mongo:latest
    ports:
      - "27017:27017"
    volumes:
      - mongodb-data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    command: redis-server --save 900 1 --save 300 10

volumes:
  mongodb-data:
```

### 微服务架构

```yaml
version: '3.8'

services:
  # API Gateway
  gateway:
    build: ./gateway
    ports:
      - "8080:8080"
    depends_on:
      - auth
      - user
      - order
    networks:
      - microservices

  # 认证服务
  auth:
    build: ./auth-service
    environment:
      - DB_URL=postgres://user:pass@auth-db:5432/auth
    depends_on:
      - auth-db
    networks:
      - microservices

  auth-db:
    image: postgres:14
    environment:
      POSTGRES_DB: auth
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - auth-data:/var/lib/postgresql/data
    networks:
      - microservices

  # 用户服务
  user:
    build: ./user-service
    environment:
      - DB_URL=mongodb://user:pass@user-db:27017/user
    depends_on:
      - user-db
    networks:
      - microservices

  user-db:
    image: mongo:latest
    environment:
      MONGO_INITDB_ROOT_USERNAME: user
      MONGO_INITDB_ROOT_PASSWORD: pass
    volumes:
      - user-data:/data/db
    networks:
      - microservices

  # 订单服务
  order:
    build: ./order-service
    environment:
      - REDIS_URL=redis://redis:6379
    depends_on:
      - redis
    networks:
      - microservices

  redis:
    image: redis:alpine
    networks:
      - microservices

networks:
  microservices:
    driver: bridge

volumes:
  auth-data:
  user-data:
```

## 注意事项

1. **环境变量**：使用 `.env` 文件管理敏感信息
   ```yaml
   services:
     db:
       image: postgres:14
       environment:
         - POSTGRES_PASSWORD=${DB_PASSWORD}
   ```

2. **依赖关系**：使用 `depends_on` 确保服务启动顺序

3. **健康检查**：添加健康检查确保服务可用
   ```yaml
   healthcheck:
     test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
     interval: 30s
     timeout: 10s
     retries: 3
   ```

4. **资源限制**：限制容器资源使用
   ```yaml
   deploy:
     resources:
       limits:
         cpus: '0.50'
         memory: 512M
       reservations:
         cpus: '0.25'
         memory: 256M
   ```

5. **日志管理**：配置日志驱动和限制
   ```yaml
   logging:
     driver: "json-file"
     options:
       max-size: "10m"
       max-file: "3"
   ```

6. **网络隔离**：使用自定义网络隔离不同环境

## 总结

Docker Compose 简化了多容器应用的部署和管理，特别适合开发环境和小型生产环境。通过合理的配置和服务编排，可以快速搭建复杂的应用架构，提高开发效率和部署可靠性。