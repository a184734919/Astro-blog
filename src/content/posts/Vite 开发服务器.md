---
title: Vite 开发服务器
published: 2024-03-13
description: '开发服务器配置和使用的详细介绍和学习笔记'
image: ''
tags: ["Vite","开发"]
category: 'Vite'
draft: false
lang: 'zh-CN'
---

## 概述

Vite 开发服务器是其核心特性之一，它利用浏览器原生 ES 模块支持，提供了极速的开发服务器启动和热模块替换（HMR）体验。开发服务器不仅提供了基础的文件服务和代理功能，还集成了许多高级特性来优化开发体验。

## 开发服务器基础

### 启动开发服务器

```bash
# 基础启动
npm run dev
# 或
vite

# 指定端口
vite --port 3000

# 指定主机
vite --host

# 打开浏览器
vite --open

# HTTPS 模式
vite --https

# 自定义配置文件
vite --config vite.config.custom.js
```

### 核心特性

```javascript
// Vite 开发服务器核心特性
{
  // 1. 即时启动 - 无需打包
  startupTime: 'milliseconds',
  
  // 2. 原生 ES 模块支持
  moduleSystem: 'ES Modules',
  
  // 3. 极速热更新
  hmr: 'lightning-fast',
  
  // 4. 按需编译
  onDemandCompilation: true,
  
  // 5. 丰富的中间件支持
  middlewareSupport: ['proxy', 'cors', 'compression', 'hmr']
}
```

## 服务器配置

### 基础配置

```javascript
// vite.config.js
export default defineConfig({
  server: {
    // 服务器配置
    host: 'localhost',
    port: 3000,
    strictPort: false,
    open: true,
    cors: true,
    
    // HTTPS 配置
    https: false,
    
    // 代理配置
    proxy: {},
    
    // 文件系统配置
    fs: {
      strict: true,
      allow: [],
      deny: []
    },
    
    // HMR 配置
    hmr: {
      overlay: true,
      port: 24678
    },
    
    // 监听配置
    watch: {
      usePolling: false,
      interval: 100,
      ignored: []
    },
    
    // 中间件模式
    middlewareMode: false
  }
})
```

### 端口和主机配置

```javascript
export default defineConfig({
  server: {
    // 端口配置
    port: 3000,
    strictPort: true,  // 端口被占用时报错
    
    // 主机配置
    host: '0.0.0.0',  // 监听所有网络接口
    // 或使用函数动态配置
    host: () => 'localhost'
  }
})
```

### 自动打开浏览器

```javascript
export default defineConfig({
  server: {
    open: true,  // 自动打开默认浏览器
    
    // 指定浏览器
    open: 'chrome',
    // 或
    open: 'firefox',
    
    // 指定路径
    open: '/dashboard',
    
    // 高级配置
    open: {
      app: {
        name: 'chrome',
        arguments: ['--incognito', '--new-window']
      },
      url: '/dashboard'
    }
  }
})
```

### HTTPS 配置

```javascript
import fs from 'fs'
import path from 'path'

export default defineConfig({
  server: {
    https: {
      key: fs.readFileSync(path.resolve(__dirname, 'cert/key.pem')),
      cert: fs.readFileSync(path.resolve(__dirname, 'cert/cert.pem')),
      ca: fs.readFileSync(path.resolve(__dirname, 'cert/ca.pem'))
    },
    
    // 或使用自签名证书
    https: true  // Vite 会自动生成证书
  }
})
```

## 代理配置

### 基础代理

```javascript
export default defineConfig({
  server: {
    proxy: {
      // 字符串简写
      '/api': 'http://localhost:8080',
      
      // 对象配置
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, '')
      }
    }
  }
})
```

### 高级代理配置

```javascript
export default defineConfig({
  server: {
    proxy: {
      // 多个代理路径
      '/api/v1': {
        target: 'http://localhost:8081',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api\/v1/, '')
      },
      
      '/api/v2': {
        target: 'http://localhost:8082',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api\/v2/, '')
      },
      
      // 正则表达式代理
      '^/fallback/.*': {
        target: 'http://jsonplaceholder.typicode.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/fallback/, '')
      },
      
      // WebSocket 代理
      '/socket.io': {
        target: 'ws://localhost:3000',
        ws: true,
        changeOrigin: true
      },
      
      // 带认证的代理
      '/auth-api': {
        target: 'https://api.example.com',
        changeOrigin: true,
        headers: {
          'Authorization': 'Bearer your-token',
          'X-Custom-Header': 'value'
        }
      },
      
      // 重写路径
      '/old-api': {
        target: 'http://localhost:8080',
        rewrite: (path) => {
          return path.replace(/^\/old-api/, '/new-api')
        }
      },
      
      // 条件代理
      '/conditional-api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        bypass: (req, res, options) => {
          if (req.headers.accept?.includes('html')) {
            return '/index.html'
          }
        }
      }
    }
  }
})
```

### 代理错误处理

```javascript
export default defineConfig({
  server: {
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        
        // 代理错误处理
        configure: (proxy, options) => {
          proxy.on('error', (err, req, res) => {
            console.error('Proxy error:', err)
            res.end('Proxy error occurred')
          })
          
          proxy.on('proxyReq', (proxyReq, req, res) => {
            console.log('Proxying request:', req.method, req.url)
            // 添加请求头
            proxyReq.setHeader('X-Forwarded-For', req.ip)
          })
          
          proxy.on('proxyRes', (proxyRes, req, res) => {
            console.log('Proxy response:', proxyRes.statusCode)
            // 修改响应头
            proxyRes.headers['Access-Control-Allow-Origin'] = '*'
          })
        }
      }
    }
  }
})
```

## 中间件开发

### 基础中间件

```javascript
export default defineConfig({
  plugins: [
    {
      name: 'custom-middleware',
      configureServer(server) {
        server.middlewares.use((req, res, next) => {
          console.log(`${req.method} ${req.url}`)
          next()
        })
      }
    }
  ]
})
```

### 自定义 API 中间件

```javascript
export default defineConfig({
  plugins: [
    {
      name: 'api-middleware',
      configureServer(server) {
        server.middlewares.use('/api', (req, res, next) => {
          // 自定义 API 响应
          if (req.url === '/api/hello') {
            res.setHeader('Content-Type', 'application/json')
            res.end(JSON.stringify({ message: 'Hello from Vite!' }))
            return
          }
          
          // 模拟延迟
          if (req.url.startsWith('/api/delayed')) {
            setTimeout(() => {
              res.setHeader('Content-Type', 'application/json')
              res.end(JSON.stringify({ 
                message: 'Delayed response',
                delay: 2000 
              }))
            }, 2000)
            return
          }
          
          next()
        })
      }
    }
  ]
})
```

### 认证中间件

```javascript
export default defineConfig({
  plugins: [
    {
      name: 'auth-middleware',
      configureServer(server) {
        // 模拟用户认证
        const users = new Map([
          ['user1', { token: 'token1', role: 'admin' }],
          ['user2', { token: 'token2', role: 'user' }]
        ])
        
        server.middlewares.use((req, res, next) => {
          // 跳过静态资源和登录接口
          if (req.url.startsWith('/@fs') || 
              req.url === '/login' ||
              req.url.endsWith('.html')) {
            return next()
          }
          
          // 检查认证
          const token = req.headers['authorization']
          
          if (!token) {
            res.statusCode = 401
            res.end('Unauthorized')
            return
          }
          
          const user = Array.from(users.values()).find(
            u => u.token === token
          )
          
          if (!user) {
            res.statusCode = 403
            res.end('Forbidden')
            return
          }
          
          // 附加用户信息到请求
          req.user = user
          next()
        })
      }
    }
  ]
})
```

### 日志中间件

```javascript
export default defineConfig({
  plugins: [
    {
      name: 'logging-middleware',
      configureServer(server) {
        server.middlewares.use((req, res, next) => {
          const start = Date.now()
          
          res.on('finish', () => {
            const duration = Date.now() - start
            const status = res.statusCode
            const method = req.method
            const url = req.url
            
            const log = `${status} ${method} ${url} - ${duration}ms`
            
            if (status >= 500) {
              console.error(log)
            } else if (status >= 400) {
              console.warn(log)
            } else {
              console.log(log)
            }
          })
          
          next()
        })
      }
    }
  ]
})
```

## 文件系统配置

### 文件访问控制

```javascript
export default defineConfig({
  server: {
    fs: {
      // 严格模式
      strict: true,
      
      // 允许访问的目录
      allow: [
        path.resolve(__dirname, 'src'),
        path.resolve(__dirname, 'public'),
        '/shared'
      ],
      
      // 拒绝访问的文件
      deny: [
        '.env',
        '.env.*',
        'package.json',
        '*.config.js'
      ]
    }
  }
})
```

### 静态文件服务

```javascript
import express from 'express'

export default defineConfig({
  plugins: [
    {
      name: 'static-files',
      configureServer(server) {
        // 自定义静态文件服务
        server.middlewares.use(
          '/static',
          express.static(path.resolve(__dirname, 'public/static'))
        )
        
        // 支持虚拟目录
        server.middlewares.use(
          '/cdn',
          express.static(path.resolve(__dirname, '../cdn'))
        )
      }
    }
  ]
})
```

## 监听配置

### 文件监听优化

```javascript
export default defineConfig({
  server: {
    watch: {
      // 使用轮询（适用于某些网络文件系统）
      usePolling: false,
      
      // 轮询间隔
      interval: 100,
      
      // 忽略的文件和目录
      ignored: [
        '**/node_modules/**',
        '**/.git/**',
        '**/dist/**',
        '**/build/**',
        '**/.idea/**',
        '**/.vscode/**',
        '**/coverage/**',
        '**/*.test.js',
        '**/*.spec.js'
      ]
    }
  }
})
```

### 自定义监听逻辑

```javascript
import chokidar from 'chokidar'

export default defineConfig({
  plugins: [
    {
      name: 'custom-watcher',
      configureServer(server) {
        // 自定义文件监听器
        const watcher = chokidar.watch('src/content/**/*.md', {
          ignored: /node_modules/,
          persistent: true
        })
        
        watcher.on('change', (path) => {
          console.log('Markdown file changed:', path)
          // 自定义处理逻辑
          server.ws.send({
            type: 'custom',
            event: 'content-updated',
            data: { path }
          })
        })
        
        // 清理函数
        return () => {
          watcher.close()
        }
      }
    }
  ]
})
```

## HMR 配置

### HMR 基础配置

```javascript
export default defineConfig({
  server: {
    hmr: {
      // 启用 HMR
      enabled: true,
      
      // HMR 端口
      port: 24678,
      
      // HMR 主机
      host: 'localhost',
      
      // 协议
      protocol: 'ws',
      
      // 覆盖层配置
      overlay: true,
      overlay: {
        errors: true,    // 显示错误
        warnings: false  // 不显示警告
      }
    }
  }
})
```

### 自定义 HMR 处理

```javascript
export default defineConfig({
  plugins: [
    {
      name: 'custom-hmr',
      handleHotUpdate({ file, server, modules }) {
        console.log('File changed:', file)
        
        // 自定义 HMR 处理
        if (file.endsWith('.custom')) {
          // 发送自定义事件
          server.ws.send({
            type: 'custom-event',
            event: 'custom-update',
            data: { file }
          })
          
          // 阻止默认 HMR
          return []
        }
        
        // 过滤特定文件
        if (file.includes('no-hmr')) {
          // 强制刷新页面
          server.ws.send({
            type: 'full-reload',
            path: '*'
          })
          return []
        }
        
        return modules
      }
    }
  ]
})
```

## 实际应用场景

### 1. Mock API 服务器

```javascript
import express from 'express'

export default defineConfig({
  plugins: [
    {
      name: 'mock-api-server',
      configureServer(server) {
        const app = express()
        
        // 用户数据
        const users = [
          { id: 1, name: 'Alice', email: 'alice@example.com' },
          { id: 2, name: 'Bob', email: 'bob@example.com' }
        ]
        
        // Mock API 端点
        app.get('/api/users', (req, res) => {
          res.json({
            success: true,
            data: users
          })
        })
        
        app.get('/api/users/:id', (req, res) => {
          const user = users.find(u => u.id === parseInt(req.params.id))
          if (user) {
            res.json({ success: true, data: user })
          } else {
            res.status(404).json({ success: false, message: 'User not found' })
          }
        })
        
        app.post('/api/users', express.json(), (req, res) => {
          const newUser = {
            id: users.length + 1,
            ...req.body
          }
          users.push(newUser)
          res.status(201).json({ success: true, data: newUser })
        })
        
        // 挂载到 Vite 服务器
        server.middlewares.use('/api', app)
      }
    }
  ]
})
```

### 2. 开发环境工具

```javascript
export default defineConfig({
  plugins: [
    {
      name: 'dev-tools',
      configureServer(server) {
        server.middlewares.use('/dev-tools', (req, res) => {
          // 开发工具面板
          if (req.url === '/dev-tools') {
            res.setHeader('Content-Type', 'text/html')
            res.end(`
              <!DOCTYPE html>
              <html>
              <head>
                <title>开发工具</title>
                <script src="/socket.io/socket.io.js"></script>
                <style>
                  body { font-family: Arial, sans-serif; padding: 20px; }
                  .tool { margin: 10px 0; padding: 10px; border: 1px solid #ddd; }
                  button { padding: 8px 16px; margin-right: 10px; }
                </style>
              </head>
              <body>
                <h1>开发工具面板</h1>
                <div class="tool">
                  <h3>快捷操作</h3>
                  <button onclick="clearCache()">清除缓存</button>
                  <button onclick="reloadPage()">刷新页面</button>
                  <button onclick="showStats()">显示统计</button>
                </div>
                <div id="stats"></div>
                <script>
                  const socket = io();
                  
                  function clearCache() {
                    socket.emit('dev-tools', { action: 'clear-cache' });
                  }
                  
                  function reloadPage() {
                    location.reload();
                  }
                  
                  function showStats() {
                    socket.emit('dev-tools', { action: 'get-stats' });
                  }
                  
                  socket.on('dev-tools', (data) => {
                    document.getElementById('stats').innerHTML = 
                      '<pre>' + JSON.stringify(data, null, 2) + '</pre>';
                  });
                </script>
              </body>
              </html>
            `)
          }
        })
      }
    }
  ]
})
```

### 3. 多环境开发服务器

```javascript
export default defineConfig(({ mode }) => {
  const isDevelopment = mode === 'development'
  const isStaging = mode === 'staging'
  
  return {
    server: {
      port: isDevelopment ? 3000 : 3001,
      host: '0.0.0.0',
      open: true,
      
      proxy: {
        '/api': {
          target: isDevelopment 
            ? 'http://localhost:8080'
            : isStaging
            ? 'https://staging-api.example.com'
            : 'https://api.example.com',
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/api/, '')
        }
      },
      
      hmr: {
        overlay: isDevelopment,
        port: 24678
      }
    }
  }
})
```

### 4. 生产环境预览

```javascript
export default defineConfig({
  plugins: [
    {
      name: 'preview-server',
      configurePreviewServer(server) {
        return () => {
          // 配置预览服务器
          server.middlewares.use((req, res, next) => {
            // 添加自定义响应头
            res.setHeader('X-Preview-Mode', 'true')
            next()
          })
        }
      }
    }
  ]
})
```

## 注意事项

### 1. 性能优化

```javascript
export default defineConfig({
  server: {
    // 优化文件监听
    watch: {
      usePolling: false,
      interval: 100,
      ignored: ['**/node_modules/**']
    },
    
    // 优化代理配置
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        // 添加缓存
        buffer: true
      }
    }
  }
})
```

### 2. 安全考虑

```javascript
export default defineConfig({
  server: {
    // CORS 配置
    cors: {
      origin: ['http://localhost:3000', 'https://example.com'],
      credentials: true
    },
    
    // 文件访问限制
    fs: {
      strict: true,
      deny: ['.env', '.env.*', 'package.json']
    },
    
    // HTTPS
    https: true
  }
})
```

### 3. 错误处理

```javascript
export default defineConfig({
  server: {
    // 全局错误处理
    hmr: {
      overlay: true
    }
  },
  
  plugins: [
    {
      name: 'error-handler',
      configureServer(server) {
        server.middlewares.use((error, req, res, next) => {
          console.error('Server error:', error)
          
          if (!res.headersSent) {
            res.statusCode = 500
            res.setHeader('Content-Type', 'application/json')
            res.end(JSON.stringify({
              error: 'Internal Server Error',
              message: error.message
            }))
          }
        })
      }
    }
  ]
})
```

## 最佳实践

### 1. 配置组织

- 分离开发环境配置
- 使用环境变量管理敏感信息
- 模块化配置文件

### 2. 中间件开发

- 保持中间件简单专注
- 正确处理错误
- 提供清理函数

### 3. 性能优化

- 合理配置文件监听
- 优化代理配置
- 利用缓存机制

### 4. 开发体验

- 提供有用的错误信息
- 支持热更新
- 集成开发工具

## 总结

Vite 开发服务器提供了强大而灵活的开发环境配置能力：

- **极速启动**：利用 ES 模块实现即时启动
- **热更新**：快速响应代码变化
- **代理支持**：解决跨域问题
- **中间件系统**：可扩展的服务器功能
- **文件监听**：智能的文件变化检测

通过合理配置开发服务器，可以：

- **提升开发效率**：快速启动和热更新
- **改善开发体验**：丰富的开发工具集成
- **简化开发流程**：内置代理和 Mock 支持
- **增强可维护性**：模块化和可配置的架构

Vite 开发服务器是现代前端开发体验的重要组成部分，掌握其配置和使用方法对于提高开发效率至关重要。