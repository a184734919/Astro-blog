---
title: Vite 组件库开发
published: 2024-07-24
description: '组件库打包和发布的详细介绍和学习笔记'
image: ''
tags: ["Vite","开发"]
category: 'Vite'
draft: false
lang: 'zh-CN'
---

## Vite 简介

Vite 是新一代前端构建工具，它利用浏览器原生 ES 模块支持，实现了极速的开发服务器启动。

## 快速开始

```bash
# 创建项目
npm create vite@latest my-app -- --template vue

# 安装依赖
cd my-app
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build
```

## 配置文件

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import path from 'path';

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  },
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true
      }
    }
  },
  build: {
    outDir: 'dist',
    sourcemap: true
  }
});
```

## 环境变量

```bash
# .env.development
VITE_API_URL=http://localhost:8080

# .env.production
VITE_API_URL=https://api.example.com
```

```javascript
// 使用环境变量
const apiUrl = import.meta.env.VITE_API_URL;
```

## 总结

Vite 提供了极快的开发体验，是现代前端项目的最佳选择。