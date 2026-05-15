---
title: Vite 环境变量
published: 2024-04-14
description: '环境变量和模式配置的详细介绍和学习笔记'
image: ''
tags: ["Vite","配置"]
category: 'Vite'
draft: false
lang: 'zh-CN'
---

## 概述

环境变量是现代前端开发中管理不同环境配置的重要机制。Vite 提供了完善的环境变量支持，允许开发者在不同环境（开发、测试、生产）中使用不同的配置值，同时保持代码的整洁和可维护性。

## 环境变量基础

### 内置环境变量

Vite 在所有文件中提供了一些内置环境变量：

```javascript
// 开发环境
import.meta.env.MODE          // 运行模式
import.meta.env.BASE_URL      // 部署的基础 URL
import.meta.env.PROD          // 是否为生产环境
import.meta.env.DEV           // 是否为开发环境
import.meta.env.SSR           // 是否为服务端渲染
```

### 环境变量访问

```javascript
// 在 JavaScript 中使用
const apiUrl = import.meta.env.VITE_API_URL
const isProduction = import.meta.env.PROD
const mode = import.meta.env.MODE

// 在模板中使用
console.log(`API URL: ${import.meta.env.VITE_API_URL}`)
console.log(`Environment: ${import.meta.env.MODE}`)

// 条件判断
if (import.meta.env.DEV) {
  console.log('Development mode')
  // 开发环境特定逻辑
}

if (import.meta.env.PROD) {
  console.log('Production mode')
  // 生产环境特定逻辑
}
```

## 环境变量文件

### 文件命名规范

Vite 支持以下环境变量文件：

```bash
# 通用环境变量（所有环境）
.env

# 开发环境
.env.development

# 生产环境
.env.production

# 测试环境
.env.test

# 自定义模式
.env.staging
.env.pre-production
```

### 文件优先级

环境变量的加载优先级从高到低：

```javascript
// 优先级示例
// 1. 命令行环境变量
VITE_API_URL=https://api.production.com vite build

// 2. .env.production (生产环境)
VITE_API_URL=https://api.example.com

// 3. .env (通用)
VITE_API_URL=http://localhost:3000

// 4. 系统环境变量
```

## 基本用法

### 创建环境变量文件

#### 开发环境配置

```bash
# .env.development
# 应用配置
VITE_APP_TITLE=开发环境 - 我的应用
VITE_APP_VERSION=1.0.0-dev

# API 配置
VITE_API_BASE_URL=http://localhost:3000/api
VITE_API_TIMEOUT=10000

# 功能开关
VITE_ENABLE_MOCK=true
VITE_ENABLE_DEBUG=true

# 第三方服务
VITE_SENTRY_DSN=https://sentry.io/12345
VITE_GOOGLE_ANALYTICS_ID=UA-123456789-1

# 其他配置
VITE_UPLOAD_MAX_SIZE=10485760
VITE_PAGE_SIZE=20
```

#### 生产环境配置

```bash
# .env.production
# 应用配置
VITE_APP_TITLE=生产环境 - 我的应用
VITE_APP_VERSION=1.0.0

# API 配置
VITE_API_BASE_URL=https://api.example.com/api
VITE_API_TIMEOUT=5000

# 功能开关
VITE_ENABLE_MOCK=false
VITE_ENABLE_DEBUG=false

# 第三方服务
VITE_SENTRY_DSN=https://sentry.io/production-12345
VITE_GOOGLE_ANALYTICS_ID=UA-123456789-2

# CDN 配置
VITE_CDN_URL=https://cdn.example.com
VITE_STATIC_URL=https://static.example.com
```

#### 测试环境配置

```bash
# .env.test
# 应用配置
VITE_APP_TITLE=测试环境 - 我的应用
VITE_APP_VERSION=1.0.0-test

# API 配置
VITE_API_BASE_URL=https://test-api.example.com/api
VITE_API_TIMEOUT=8000

# 功能开关
VITE_ENABLE_MOCK=true
VITE_ENABLE_DEBUG=true

# 测试专用配置
VITE_TEST_MODE=true
VITE_MOCK_DELAY=500
```

#### 通用配置

```bash
# .env
# 默认配置（被其他环境覆盖）
VITE_APP_TITLE=我的应用
VITE_APP_VERSION=1.0.0

# API 配置默认值
VITE_API_BASE_URL=/api
VITE_API_TIMEOUT=5000

# 默认功能开关
VITE_ENABLE_MOCK=false
VITE_ENABLE_DEBUG=false
```

### 在代码中使用

#### 基础使用

```javascript
// config/env.js
export const env = {
  app: {
    title: import.meta.env.VITE_APP_TITLE || '默认应用',
    version: import.meta.env.VITE_APP_VERSION || '1.0.0',
    mode: import.meta.env.MODE,
    isDev: import.meta.env.DEV,
    isProd: import.meta.env.PROD
  },
  
  api: {
    baseUrl: import.meta.env.VITE_API_BASE_URL || '/api',
    timeout: Number(import.meta.env.VITE_API_TIMEOUT) || 5000
  },
  
  features: {
    enableMock: import.meta.env.VITE_ENABLE_MOCK === 'true',
    enableDebug: import.meta.env.VITE_ENABLE_DEBUG === 'true',
    testMode: import.meta.env.VITE_TEST_MODE === 'true'
  },
  
  thirdParty: {
    sentryDsn: import.meta.env.VITE_SENTRY_DSN,
    googleAnalyticsId: import.meta.env.VITE_GOOGLE_ANALYTICS_ID
  }
}

// 导出类型安全的访问器
export const getAppTitle = () => env.app.title
export const getApiUrl = () => env.api.baseUrl
export const isDevelopment = () => env.app.isDev
export const isProduction = () => env.app.isProd
```

#### Axios 配置

```javascript
// utils/request.js
import axios from 'axios'
import { env } from '@/config/env'

const apiClient = axios.create({
  baseURL: env.api.baseUrl,
  timeout: env.api.timeout,
  headers: {
    'Content-Type': 'application/json',
    'X-App-Version': env.app.version
  }
})

// 请求拦截器
apiClient.interceptors.request.use(
  config => {
    if (env.features.enableDebug) {
      console.log('API Request:', config)
    }
    return config
  },
  error => Promise.reject(error)
)

// 响应拦截器
apiClient.interceptors.response.use(
  response => {
    if (env.features.enableDebug) {
      console.log('API Response:', response)
    }
    return response.data
  },
  error => {
    console.error('API Error:', error)
    return Promise.reject(error)
  }
)

export default apiClient
```

#### Mock 数据集成

```javascript
// utils/mock.js
import { env } from '@/config/env'

export function setupMock() {
  if (!env.features.enableMock) {
    return
  }
  
  // 在开发环境启用 Mock
  if (env.app.isDev) {
    console.log('Mock enabled')
    
    // 注册 Mock 接口
    mockApi.get('/api/users', (req, res) => {
      res.json({
        code: 200,
        data: [
          { id: 1, name: 'User 1' },
          { id: 2, name: 'User 2' }
        ]
      })
    })
  }
}

// 在 main.js 中调用
import { setupMock } from '@/utils/mock'
setupMock()
```

## 高级配置

### 自定义环境变量前缀

```javascript
// vite.config.js
export default defineConfig({
  envPrefix: ['VITE_', 'APP_', 'CUSTOM_']
})
```

```bash
# .env
VITE_API_URL=https://api.example.com
APP_NAME=MyApp
CUSTOM_TOKEN=secret-token
```

```javascript
// 都可以访问
const apiUrl = import.meta.env.VITE_API_URL
const appName = import.meta.env.APP_NAME
const token = import.meta.env.CUSTOM_TOKEN
```

### 加载自定义环境

```javascript
// package.json
{
  "scripts": {
    "dev": "vite --mode development",
    "build": "vite build --mode production",
    "build:staging": "vite build --mode staging",
    "build:test": "vite build --mode test",
    "preview": "vite preview"
  }
}
```

```bash
# .env.staging
VITE_APP_TITLE=预发布环境
VITE_API_BASE_URL=https://staging-api.example.com
VITE_ENABLE_MOCK=false
```

### 环境变量验证

```javascript
// utils/env-validator.js
export function validateEnv(requiredVars = []) {
  const missingVars = []
  
  requiredVars.forEach(varName => {
    if (!import.meta.env[varName]) {
      missingVars.push(varName)
    }
  })
  
  if (missingVars.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missingVars.join(', ')}`
    )
  }
  
  console.log('Environment validation passed')
}

// 在应用启动时验证
import { validateEnv } from '@/utils/env-validator'

const requiredVars = [
  'VITE_API_BASE_URL',
  'VITE_APP_VERSION'
]

validateEnv(requiredVars)
```

### 类型安全的环境变量

```typescript
// src/env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  // 应用配置
  readonly VITE_APP_TITLE: string
  readonly VITE_APP_VERSION: string
  readonly VITE_API_BASE_URL: string
  readonly VITE_API_TIMEOUT: string
  
  // 功能开关
  readonly VITE_ENABLE_MOCK: string
  readonly VITE_ENABLE_DEBUG: string
  
  // 第三方服务
  readonly VITE_SENTRY_DSN?: string
  readonly VITE_GOOGLE_ANALYTICS_ID?: string
  
  // 自定义变量
  readonly CUSTOM_CONFIG?: string
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

## 实际应用场景

### 1. API 配置管理

```javascript
// api/index.js
import { env } from '@/config/env'

const api = {
  // 用户相关
  user: {
    login: `${env.api.baseUrl}/user/login`,
    logout: `${env.api.baseUrl}/user/logout`,
    profile: `${env.api.baseUrl}/user/profile`,
    update: `${env.api.baseUrl}/user/update`
  },
  
  // 产品相关
  product: {
    list: `${env.api.baseUrl}/products`,
    detail: (id) => `${env.api.baseUrl}/products/${id}`,
    create: `${env.api.baseUrl}/products`,
    update: (id) => `${env.api.baseUrl}/products/${id}`,
    delete: (id) => `${env.api.baseUrl}/products/${id}`
  },
  
  // 订单相关
  order: {
    list: `${env.api.baseUrl}/orders`,
    detail: (id) => `${env.api.baseUrl}/orders/${id}`,
    create: `${env.api.baseUrl}/orders`
  }
}

export default api
```

### 2. 功能开关控制

```javascript
// features/index.js
import { env } from '@/config/env'

export const features = {
  // 新功能开关
  newDashboard: env.features.enableDebug,
  darkMode: true,
  multiLanguage: true,
  
  // 功能判断
  isEnabled(featureName) {
    return this[featureName] === true
  },
  
  // 条件渲染
  whenEnabled(featureName, component) {
    return this.isEnabled(featureName) ? component : null
  }
}

// 在组件中使用
export default {
  computed: {
    showNewFeatures() {
      return features.isEnabled('newDashboard')
    }
  }
}
```

### 3. 日志配置

```javascript
// utils/logger.js
import { env } from '@/config/env'

class Logger {
  constructor() {
    this.isDebugEnabled = env.features.enableDebug
    this.isProduction = env.app.isProd
  }
  
  log(...args) {
    if (this.isDebugEnabled) {
      console.log('[LOG]', ...args)
    }
  }
  
  info(...args) {
    console.info('[INFO]', ...args)
  }
  
  warn(...args) {
    console.warn('[WARN]', ...args)
  }
  
  error(...args) {
    console.error('[ERROR]', ...args)
    
    // 生产环境上报错误
    if (this.isProduction && env.thirdParty.sentryDsn) {
      this.reportToSentry(args)
    }
  }
  
  debug(...args) {
    if (this.isDebugEnabled) {
      console.debug('[DEBUG]', ...args)
    }
  }
  
  reportToSentry(error) {
    // 实现错误上报
    console.log('Reporting to Sentry:', error)
  }
}

export default new Logger()
```

### 4. 第三方服务集成

```javascript
// services/sentry.js
import * as Sentry from '@sentry/browser'
import { env } from '@/config/env'

export function initSentry() {
  if (env.thirdParty.sentryDsn) {
    Sentry.init({
      dsn: env.thirdParty.sentryDsn,
      environment: env.app.mode,
      release: env.app.version,
      integrations: [
        new Sentry.BrowserTracing(),
        new Sentry.Replay()
      ],
      tracesSampleRate: env.app.isDev ? 1.0 : 0.1,
      replaysSessionSampleRate: 0.1,
      replaysOnErrorSampleRate: 1.0
    })
    
    console.log('Sentry initialized')
  }
}

// services/analytics.js
import { env } from '@/config/env'

export function initAnalytics() {
  if (env.thirdParty.googleAnalyticsId) {
    // 初始化 Google Analytics
    window.gtag('config', env.thirdParty.googleAnalyticsId, {
      send_page_view: false
    })
    
    console.log('Analytics initialized')
  }
}

// 在 main.js 中初始化
import { initSentry } from '@/services/sentry'
import { initAnalytics } from '@/services/analytics'

initSentry()
initAnalytics()
```

### 5. 条件性配置

```javascript
// config/index.js
import { env } from '@/config/env'

const config = {
  // 基础配置
  title: env.app.title,
  version: env.app.version,
  
  // 路由配置
  router: {
    base: env.app.mode === 'production' ? '/app/' : '/',
    mode: env.app.mode === 'production' ? 'history' : 'hash'
  },
  
  // 存储配置
  storage: {
    prefix: `${env.app.title}_`,
    driver: env.app.isDev ? 'localStorage' : 'indexedDB'
  },
  
  // 缓存配置
  cache: {
    enabled: !env.app.isDev,
    ttl: 3600000 // 1 hour
  },
  
  // 日志配置
  logging: {
    level: env.features.enableDebug ? 'debug' : 'warn',
    console: env.features.enableDebug
  }
}

// 条件性导出
if (env.app.isProd) {
  config.prodOnly = {
    cdn: env.thirdParty.cdnUrl,
    compression: true,
    minification: true
  }
}

if (env.app.isDev) {
  config.devOnly = {
    hotReload: true,
    sourceMaps: true,
    detailedErrors: true
  }
}

export default config
```

## 配置管理最佳实践

### 1. 环境变量组织

```bash
# 推荐的文件结构
.env                    # 通用默认配置
.env.development        # 开发环境
.env.production         # 生产环境
.env.test              # 测试环境
.env.staging           # 预发布环境
.env.local             # 本地覆盖（不提交）
.env.*.local           # 本地覆盖（不提交）
```

### 2. 敏感信息处理

```bash
# .env.example (提交到版本控制)
VITE_API_BASE_URL=https://api.example.com
VITE_APP_VERSION=1.0.0
VITE_ENABLE_MOCK=false

# .env (不提交敏感信息)
VITE_API_SECRET_KEY=your-secret-key
VITE_DB_PASSWORD=your-password
```

```bash
# .gitignore
.env
.env.local
.env.*.local
```

### 3. 配置验证

```javascript
// config/validator.js
export const envSchema = {
  VITE_API_BASE_URL: {
    type: 'string',
    required: true,
    validator: (value) => value.startsWith('http')
  },
  
  VITE_APP_VERSION: {
    type: 'string',
    required: true,
    validator: (value) => /^\d+\.\d+\.\d+/.test(value)
  },
  
  VITE_API_TIMEOUT: {
    type: 'number',
    required: false,
    default: 5000,
    validator: (value) => value > 0
  }
}

export function validateEnvConfig() {
  const errors = []
  
  Object.entries(envSchema).forEach(([key, schema]) => {
    const value = import.meta.env[key]
    
    // 检查必需项
    if (schema.required && !value) {
      errors.push(`${key} is required`)
      return
    }
    
    // 使用默认值
    const finalValue = value || schema.default
    
    // 类型验证
    if (schema.type === 'number') {
      if (isNaN(Number(finalValue))) {
        errors.push(`${key} must be a number`)
      }
    }
    
    // 自定义验证
    if (schema.validator && !schema.validator(finalValue)) {
      errors.push(`${key} validation failed`)
    }
  })
  
  if (errors.length > 0) {
    throw new Error(`Environment validation failed:\n${errors.join('\n')}`)
  }
  
  console.log('Environment configuration is valid')
}
```

### 4. 配置模块化

```javascript
// config/app.js
export default {
  title: import.meta.env.VITE_APP_TITLE,
  version: import.meta.env.VITE_APP_VERSION,
  mode: import.meta.env.MODE
}

// config/api.js
export default {
  baseUrl: import.meta.env.VITE_API_BASE_URL,
  timeout: Number(import.meta.env.VITE_API_TIMEOUT),
  version: 'v1'
}

// config/features.js
export default {
  enableMock: import.meta.env.VITE_ENABLE_MOCK === 'true',
  enableDebug: import.meta.env.VITE_ENABLE_DEBUG === 'true',
  testMode: import.meta.env.VITE_TEST_MODE === 'true'
}

// config/index.js
import app from './app'
import api from './api'
import features from './features'

export default {
  app,
  api,
  features
}
```

## 注意事项

### 1. 安全性考虑

- **不要在前端代码中使用敏感信息**：所有以 `VITE_` 开头的变量都会暴露给浏览器
- **使用服务端代理**：敏感操作应该通过后端 API 进行
- **环境变量文件不要提交**：`.env` 和 `.env.local` 应该加入 `.gitignore`

### 2. 类型安全

```typescript
// 创建类型定义文件
// src/env.d.ts
interface ImportMetaEnv {
  readonly VITE_API_BASE_URL: string
  readonly VITE_APP_VERSION: string
  readonly VITE_ENABLE_DEBUG: string
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

### 3. 构建优化

```javascript
// vite.config.js
export default defineConfig({
  define: {
    // 构建时替换，运行时不存在
    __APP_VERSION__: JSON.stringify(import.meta.env.VITE_APP_VERSION),
    __BUILD_TIME__: JSON.stringify(new Date().toISOString())
  }
})
```

### 4. 调试支持

```javascript
// 开发环境显示所有环境变量
if (import.meta.env.DEV) {
  console.group('Environment Variables')
  Object.keys(import.meta.env).forEach(key => {
    if (key.startsWith('VITE_')) {
      console.log(`${key}:`, import.meta.env[key])
    }
  })
  console.groupEnd()
}
```

## 总结

Vite 环境变量系统提供了灵活而强大的配置管理能力：

- **简单易用**：通过 `.env` 文件管理不同环境配置
- **类型安全**：支持 TypeScript 类型定义
- **优先级控制**：明确的环境变量加载优先级
- **开发体验**：内置的环境变量和便利的访问方式

合理使用环境变量可以：

- **简化配置管理**：不同环境使用不同配置文件
- **提高安全性**：避免硬编码敏感信息
- **增强灵活性**：便于切换部署环境
- **改善维护性**：集中管理配置信息

掌握 Vite 环境变量的使用，对于构建可维护、可扩展的前端应用至关重要。建议在实际项目中建立完善的环境变量管理体系，遵循最佳实践，确保配置的安全性和可维护性。