---
title: Vite 入门介绍
published: 2024-03-01
description: 'Vite 的特点和快速上手的详细介绍和学习笔记'
image: ''
tags: ["Vite","基础"]
category: 'Vite'
draft: false
lang: 'zh-CN'
---

## 概述

Vite（法语意为"快速的"）是由 Vue.js 作者尤雨溪开发的下一代前端构建工具。它利用浏览器原生 ES 模块（ESM）支持，提供了极快的开发服务器启动时间和热更新（HMR）体验。Vite 在开发环境中无需打包，在生产环境使用 Rollup 进行构建，兼顾了开发体验和生产优化。

## 核心特性

### 1. 极速的服务器启动
Vite 利用浏览器原生的 ES 模块加载机制，在开发环境中按需编译源代码。这意味着：

```bash
# 传统构建工具需要先打包整个应用
webpack: 几十秒到几分钟的启动时间

# Vite 只在请求时编译
Vite: 几百毫秒到几秒的启动时间
```

### 2. 即时的模块热更新
Vite 基于 ESM 的 HMR 实现使得无论应用大小如何，HMR 都能始终保持快速更新：

```javascript
// Vite 的 HMR API
if (import.meta.hot) {
  import.meta.hot.accept('./foo.js', (newFoo) => {
    // 当 foo.js 变化时执行的回调
    console.log('foo.js updated:', newFoo);
  });
}
```

### 3. 丰富的功能支持
- TypeScript、JSX、CSS 预处理器开箱即用
- 支持异步模块加载
- 优化的构建，使用 Rollup 进行代码分割
- CSS 代码分割
- 模块热替换（HMR）
- 通过 Rollup 插件机制扩展

### 4. 完整的生态系统
Vite 提供了官方插件和各种框架集成：
- `@vitejs/plugin-vue` - Vue 3 支持
- `@vitejs/plugin-react` - React 支持
- `@vitejs/plugin-react-swc` - React + SWC 支持
- `@vitejs/plugin-legacy` - 浏览器兼容性支持

## 核心概念

### 开发环境 vs 生产环境

Vite 在开发和生产环境采用不同的策略：

**开发环境：**
```javascript
// 浏览器直接请求源码
<script type="module" src="/src/main.js"></script>

// Vite 开发服务器拦截请求并转换
/src/main.js → 编译后的 ES Module
```

**生产环境：**
```javascript
// 使用 Rollup 打包优化
npm run build
// 生成优化后的静态资源
dist/
├── assets/
│   ├── index-[hash].js
│   └── index-[hash].css
└── index.html
```

### ES 模块机制

Vite 的开发模式完全基于浏览器的 ES 模块支持：

```javascript
// main.js
import { createApp } from 'vue'
import App from './App.vue'

createApp(App).mount('#app')
```

浏览器会：
1. 解析 ES 模块语法
2. 通过 HTTP 请求加载依赖
3. Vite 服务器拦截并转换请求

## 基本用法

### 1. 快速开始项目

#### 使用官方脚手架

```bash
# 使用 npm
npm create vite@latest my-vite-app

# 使用 yarn
yarn create vite my-vite-app

# 使用 pnpm
pnpm create vite my-vite-app
```

#### 指定模板

```bash
# 创建 Vue 项目
npm create vite@latest my-vue-app -- --template vue

# 创建 React 项目
npm create vite@latest my-react-app -- --template react

# 创建 Vanilla JS 项目
npm create vite@latest my-js-app -- --template vanilla

# 创建 TypeScript 项目
npm create vite@latest my-ts-app -- --template react-ts
```

#### 项目目录结构

```bash
my-vite-app/
├── public/           # 静态资源
├── src/              # 源代码
│   ├── assets/       # 资源文件
│   ├── components/   # 组件
│   ├── App.vue       # 根组件
│   ├── main.js       # 入口文件
│   └── style.css     # 样式文件
├── index.html        # HTML 模板
├── package.json      # 项目配置
└── vite.config.js    # Vite 配置文件
```

### 2. 项目配置

#### 基础配置文件

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@assets': path.resolve(__dirname, './src/assets')
    }
  },
  server: {
    port: 3000,
    open: true,
    cors: true
  },
  build: {
    outDir: 'dist',
    assetsDir: 'assets',
    sourcemap: false,
    minify: 'terser',
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor': ['vue', 'vue-router'],
          'ui': ['element-plus']
        }
      }
    }
  }
})
```

#### TypeScript 支持

```javascript
// vite.config.ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src')
    }
  }
})
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ESNext",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "jsx": "preserve",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "esModuleInterop": true,
    "lib": ["ESNext", "DOM"],
    "skipLibCheck": true,
    "noEmit": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### 3. 开发脚本配置

```json
{
  "name": "my-vite-app",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext .vue,.js,.jsx,.cjs,.mjs --fix --ignore-path .gitignore"
  },
  "dependencies": {
    "vue": "^3.3.4"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^4.3.4",
    "vite": "^4.4.9"
  }
}
```

### 4. 常用命令

```bash
# 启动开发服务器
npm run dev
# 或
vite

# 构建生产版本
npm run build
# 或
vite build

# 预览生产构建
npm run preview
# 或
vite preview

# 直接运行配置文件
vite --config my-config.js

# 指定端口和主机
vite --port 3000 --host

# 开启调试模式
vite --debug
```

## 实际应用

### 1. Vue 3 项目示例

#### 项目初始化

```bash
npm create vite@latest my-vue3-app -- --template vue
cd my-vue3-app
npm install
npm run dev
```

#### 安装常用依赖

```bash
# 路由
npm install vue-router

# 状态管理
npm install pinia

# UI 组件库
npm install element-plus

# 工具库
npm install axios dayjs lodash
```

#### 项目结构

```bash
src/
├── api/              # API 接口
├── assets/           # 静态资源
├── components/       # 通用组件
├── composables/      # 组合式函数
├── layouts/          # 布局组件
├── router/           # 路由配置
├── stores/           # Pinia 状态管理
├── styles/           # 全局样式
├── utils/            # 工具函数
├── views/            # 页面组件
├── App.vue           # 根组件
└── main.js           # 入口文件
```

#### 主入口文件

```javascript
// src/main.js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import ElementPlus from 'element-plus'
import 'element-plus/dist/index.css'
import App from './App.vue'
import router from './router'
import './styles/main.css'

const app = createApp(App)

app.use(createPinia())
app.use(router)
app.use(ElementPlus)

app.mount('#app')
```

### 2. React 项目示例

#### 项目初始化

```bash
npm create vite@latest my-react-app -- --template react
cd my-react-app
npm install
npm run dev
```

#### 安装依赖

```bash
# 路由
npm install react-router-dom

# 状态管理
npm install zustand

# UI 组件库
npm install antd

# 工具库
npm install axios dayjs lodash
```

#### 项目结构

```bash
src/
├── api/              # API 接口
├── assets/           # 静态资源
├── components/       # 通用组件
├── hooks/            # 自定义 Hooks
├── layouts/          # 布局组件
├── pages/            # 页面组件
├── router/           # 路由配置
├── stores/           # 状态管理
├── styles/           # 全局样式
├── utils/            # 工具函数
├── App.jsx           # 根组件
└── main.jsx          # 入口文件
```

#### 主入口文件

```javascript
// src/main.jsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { BrowserRouter } from 'react-router-dom'
import { ConfigProvider } from 'antd'
import zhCN from 'antd/locale/zh_CN'
import App from './App.jsx'
import './styles/main.css'

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <BrowserRouter>
      <ConfigProvider locale={zhCN}>
        <App />
      </ConfigProvider>
    </BrowserRouter>
  </React.StrictMode>
)
```

### 3. 环境变量配置

#### 环境变量文件

```bash
# .env.development
VITE_APP_TITLE=开发环境
VITE_API_BASE_URL=http://localhost:3000/api
VITE_APP_MODE=development
```

```bash
# .env.production
VITE_APP_TITLE=生产环境
VITE_API_BASE_URL=https://api.example.com
VITE_APP_MODE=production
```

```bash
# .env.staging
VITE_APP_TITLE=测试环境
VITE_API_BASE_URL=https://staging-api.example.com
VITE_APP_MODE=staging
```

#### 使用环境变量

```javascript
// src/utils/request.js
const baseURL = import.meta.env.VITE_API_BASE_URL
const mode = import.meta.env.VITE_APP_MODE

export const apiClient = axios.create({
  baseURL,
  timeout: 10000,
  headers: {
    'Content-Type': 'application/json'
  }
})

// 环境判断
const isDevelopment = import.meta.env.DEV
const isProduction = import.meta.env.PROD
const isStaging = import.meta.env.MODE === 'staging'
```

## 高级特性

### 1. CSS 预处理器支持

```bash
# 安装预处理器
npm install -D sass
npm install -D less
npm install -D stylus
```

```javascript
// vite.config.js
export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss";`
      },
      less: {
        math: 'always',
        globalVars: {
          primary: '#1890ff'
        }
      }
    }
  }
})
```

### 2. 静态资源处理

```javascript
// 导入图片
import logo from './assets/logo.png'

// 在模板中使用
<img :src="logo" alt="Logo" />

// URL 导入
import logoUrl from './assets/logo.png?url'
// 获取公共资源
import favicon from '/favicon.ico'

// 导入为字符串
import codeString from './code.txt?raw'

// 导入为 Worker
import worker from './worker.js?worker'
const workerInstance = new worker()
```

### 3. 构建优化

```javascript
// vite.config.js
export default defineConfig({
  build: {
    // 输出目录
    outDir: 'dist',
    // 静态资源目录
    assetsDir: 'assets',
    // 生成源码映射
    sourcemap: false,
    // 压缩配置
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true
      }
    },
    // 块大小警告限制
    chunkSizeWarningLimit: 1000,
    // CSS 代码分割
    cssCodeSplit: true,
    // Rollup 配置
    rollupOptions: {
      output: {
        // 手动分包
        manualChunks: {
          'vue-vendor': ['vue', 'vue-router', 'pinia'],
          'ui-vendor': ['element-plus'],
          'utils': ['axios', 'lodash-es', 'dayjs']
        },
        // 文件命名
        chunkFileNames: 'js/[name]-[hash].js',
        entryFileNames: 'js/[name]-[hash].js',
        assetFileNames: '[ext]/[name]-[hash].[ext]'
      }
    }
  }
})
```

## 注意事项

### 1. 浏览器兼容性

Vite 默认使用 ES2020+，如果需要支持旧浏览器：

```javascript
// vite.config.js
import legacy from '@vitejs/plugin-legacy'

export default defineConfig({
  plugins: [
    vue(),
    legacy({
      targets: ['defaults', 'not IE 11'],
      additionalLegacyPolyfills: ['regenerator-runtime/runtime']
    })
  ]
})
```

### 2. 性能优化

#### 预构建优化

```javascript
// vite.config.js
export default defineConfig({
  optimizeDeps: {
    include: ['vue', 'vue-router', 'axios'],
    exclude: ['your-local-package'],
    force: false
  }
})
```

#### 开发服务器优化

```javascript
// vite.config.js
export default defineConfig({
  server: {
    fs: {
      strict: false
    },
    hmr: {
      overlay: true
    }
  }
})
```

### 3. 常见问题解决

#### 路径别名不生效

```javascript
// vite.config.js
export default defineConfig({
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src')
    }
  }
})
```

#### TypeScript 识别路径别名

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    }
  }
}
```

#### 构建后页面空白

```javascript
// vite.config.js
export default defineConfig({
  base: './', // 相对路径
  // 或
  base: 'https://cdn.example.com/' // CDN 路径
})
```

## 最佳实践

### 1. 项目组织

- 按功能模块组织代码结构
- 使用统一的命名规范
- 合理使用路径别名简化导入

### 2. 开发配置

- 使用 `.env` 文件管理环境变量
- 配置合适的代理解决跨域问题
- 启用代码检查和格式化工具

### 3. 构建优化

- 合理进行代码分割
- 压缩和混淆生产代码
- 使用 CDN 加速静态资源

### 4. 开发工具

- 集成 ESLint 和 Prettier
- 配置 Git 钩子自动检查
- 使用 TypeScript 增强类型安全

## 总结

Vite 是一个革命性的前端构建工具，它通过：

- **极速启动**：利用 ES 模块实现即时开发服务器启动
- **热更新**：基于 ESM 的快速 HMR 体验
- **现代构建**：使用 Rollup 进行生产环境优化
- **丰富生态**：完善的插件系统和框架集成
- **开发体验**：开箱即用的现代开发工具链

Vite 不仅提供了开发效率的提升，还通过其现代化的设计理念推动了前端构建工具的发展。无论是小型项目还是大型应用，Vite 都能提供优秀的开发体验和生产性能。

随着 Vite 生态的不断完善，它已经成为现代前端项目的首选构建工具之一。掌握 Vite 的使用和配置，对于提高开发效率和应用性能都具有重要意义。