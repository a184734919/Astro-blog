---
title: Vite 热更新 HMR
published: 2024-05-03
description: '模块热更新原理的详细介绍和学习笔记'
image: ''
tags: ["Vite","开发"]
category: 'Vite'
draft: false
lang: 'zh-CN'
---

## 概述

热模块替换（Hot Module Replacement，HMR）是现代前端开发体验的核心功能之一。Vite 利用浏览器原生 ES 模块支持，实现了极速的 HMR 体验，使得代码修改能够即时反映到浏览器中，无需刷新页面，从而保持应用状态。

## HMR 工作原理

### 传统构建工具 vs Vite

```javascript
// Webpack HMR 流程
1. 监听文件变化
2. 重新编译整个模块图
3. 生成新的 hash
4. 通知浏览器更新
5. 浏览器请求新的模块代码
6. 执行 HMR runtime 替换模块

// Vite HMR 流程
1. 监听文件变化
2. 只编译变化的模块
3. 通过 WebSocket 通知浏览器
4. 浏览器直接请求变化的模块
5. 利用 ES 模块天然支持更新
```

### Vite HMR 核心机制

```javascript
// 浏览器端 HMR 客户端
if (import.meta.hot) {
  import.meta.hot.accept('./module.js', (newModule) => {
    // 当 module.js 变化时执行
    console.log('Module updated:', newModule)
    // 可以在这里更新应用状态
  })
}

// Vite 服务器端
// 监听文件变化
chokidar.watch('src/**/*').on('change', (file) => {
  // 编译变化的文件
  const compiled = compileFile(file)
  
  // 通过 WebSocket 通知客户端
  ws.send({
    type: 'update',
    updates: [
      {
        type: 'js-update',
        path: '/src/module.js',
        acceptedPath: '/src/module.js',
        timestamp: Date.now()
      }
    ]
  })
})
```

## HMR API 详解

### 基础 HMR API

#### import.meta.hot

检查 HMR 是否可用：

```javascript
// 在代码中检查 HMR 支持
if (import.meta.hot) {
  console.log('HMR is enabled')
  // HMR 相关代码
}
```

#### accept

接受模块更新：

```javascript
// 1. 接受自身更新
import.meta.hot.accept()

// 2. 接受依赖更新
import.meta.hot.accept('./dependency.js', (newDep) => {
  console.log('Dependency updated:', newDep)
  // 更新逻辑
})

// 3. 接受多个依赖更新
import.meta.hot.accept(
  ['./dep1.js', './dep2.js'],
  ([newDep1, newDep2]) => {
    console.log('Dependencies updated')
    // 更新逻辑
  }
)

// 4. 使用相对路径
import.meta.hot.accept('../components/MyComponent.js', (newComponent) => {
  // 更新组件
})
```

#### dispose

清理旧模块副作用：

```javascript
let timer = null
let eventListener = null

// 设置定时器
timer = setInterval(() => {
  console.log('Timer running')
}, 1000)

// 添加事件监听
window.addEventListener('resize', eventListener)

// HMR 清理逻辑
if (import.meta.hot) {
  import.meta.hot.dispose((data) => {
    // 清理定时器
    if (timer) {
      clearInterval(timer)
      timer = null
    }
    
    // 移除事件监听
    if (eventListener) {
      window.removeEventListener('resize', eventListener)
      eventListener = null
    }
    
    // 传递数据给新模块
    data.timerInterval = 1000
  })
}
```

#### prune

模块被移除时清理：

```javascript
if (import.meta.hot) {
  import.meta.hot.prune(() => {
    // 当模块从页面移除时执行
    console.log('Module pruned')
    cleanupResources()
  })
}
```

#### decline

拒绝 HMR 更新：

```javascript
// 拒绝自身更新
import.meta.hot.decline()

// 拒绝依赖更新
import.meta.hot.decline('./critical-module.js')

// 示例：关键模块拒绝热更新
if (import.meta.hot) {
  import.meta.hot.decline()
  console.log('This module requires full page reload')
}
```

### 高级 HMR API

#### invalidate

使模块失效：

```javascript
// 使当前模块失效，触发重新加载
if (import.meta.hot) {
  import.meta.hot.invalidate()
}

// 条件性失效
function checkValidation() {
  if (!isValidState()) {
    if (import.meta.hot) {
      console.log('Invalid state, reloading...')
      import.meta.hot.invalidate()
    }
  }
}
```

#### on

监听 HMR 事件：

```javascript
if (import.meta.hot) {
  // 监听模块更新前事件
  import.meta.hot.on('vite:beforeUpdate', (context) => {
    console.log('Modules about to update:', context.updates)
    // 可以在这里准备更新逻辑
  })
  
  // 监听模块更新后事件
  import.meta.hot.on('vite:afterUpdate', (context) => {
    console.log('Modules updated:', context.updates)
    // 更新后的处理逻辑
  })
  
  // 监听自定义错误
  import.meta.hot.on('vite:error', (error) => {
    console.error('HMR error:', error)
    // 错误处理逻辑
  })
  
  // 监听连接事件
  import.meta.hot.on('vite:connected', () => {
    console.log('HMR connected')
  })
  
  // 监听断开连接事件
  import.meta.hot.on('vite:disconnected', () => {
    console.log('HMR disconnected')
  })
}
```

#### send

发送自定义消息：

```javascript
// 发送自定义消息到服务器
if (import.meta.hot) {
  import.meta.hot.send('custom-event', {
    type: 'log',
    message: 'Custom message from client'
  })
}
```

## 框架级 HMR 集成

### Vue 3 HMR

#### 基础组件 HMR

```vue
<script setup>
import { ref, onMounted } from 'vue'

const count = ref(0)
let intervalId = null

onMounted(() => {
  intervalId = setInterval(() => {
    count.value++
  }, 1000)
})

// HMR 支持
if (import.meta.hot) {
  import.meta.hot.accept((newComponent) => {
    // Vue 3 会自动处理组件更新
    console.log('Component updated')
  })
  
  import.meta.hot.dispose(() => {
    if (intervalId) {
      clearInterval(intervalId)
    }
  })
}
</script>

<template>
  <div>
    <h1>Count: {{ count }}</h1>
  </div>
</template>
```

#### 复杂组件状态保持

```vue
<script setup>
import { ref, computed, watch } from 'vue'
import { storeToRefs } from 'pinia'
import useUserStore from '@/stores/user'

const userStore = useUserStore()
const { name, email } = storeToRefs(userStore)
const localState = ref({})

// HMR 状态保持
if (import.meta.hot) {
  import.meta.hot.dispose((data) => {
    // 保存状态
    data.localState = localState.value
    data.storeState = userStore.$state
  })
  
  import.meta.hot.accept((newComponent) => {
    // 恢复状态
    if (data.localState) {
      localState.value = data.localState
    }
    if (data.storeState) {
      userStore.$patch(data.storeState)
    }
  })
}
</script>

<template>
  <div>
    <h2>User Profile</h2>
    <p>Name: {{ name }}</p>
    <p>Email: {{ email }}</p>
  </div>
</template>
```

#### 组合式函数 HMR

```javascript
// composables/useCounter.js
import { ref, onUnmounted } from 'vue'

export function useCounter(initialValue = 0) {
  const count = ref(initialValue)
  let intervalId = null
  
  const start = () => {
    intervalId = setInterval(() => {
      count.value++
    }, 1000)
  }
  
  const stop = () => {
    if (intervalId) {
      clearInterval(intervalId)
      intervalId = null
    }
  }
  
  // HMR 清理
  if (import.meta.hot) {
    import.meta.hot.dispose(() => {
      stop()
    })
  }
  
  return { count, start, stop }
}
```

### React HMR

#### 函数组件 HMR

```jsx
import { useState, useEffect } from 'react'

function Counter() {
  const [count, setCount] = useState(0)
  
  useEffect(() => {
    const timer = setInterval(() => {
      setCount(prev => prev + 1)
    }, 1000)
    
    return () => clearInterval(timer)
  }, [])
  
  // React Fast Refresh 自动处理 HMR
  return (
    <div>
      <h1>Count: {count}</h1>
    </div>
  )
}

export default Counter

// HMR 支持（React Fast Refresh 自动处理）
if (import.meta.hot) {
  import.meta.hot.accept()
}
```

#### 自定义 HMR 处理

```jsx
import { useState, useEffect, useRef } from 'react'

function ComplexComponent({ config }) {
  const stateRef = useRef({})
  const [data, setData] = useState(null)
  
  useEffect(() => {
    // 初始化逻辑
    initializeComponent()
    
    return () => {
      // 清理逻辑
      cleanupComponent()
    }
  }, [])
  
  // HMR 状态保持
  if (import.meta.hot) {
    import.meta.hot.dispose((data) => {
      // 保存状态
      data.savedState = stateRef.current
      data.savedData = data
    })
    
    import.meta.hot.accept((newModule) => {
      // 恢复状态
      if (data.savedState) {
        stateRef.current = data.savedState
      }
      if (data.savedData) {
        setData(data.savedData)
      }
    })
  }
  
  return <div>{/* 组件内容 */}</div>
}

export default ComplexComponent
```

#### 自定义 Hook HMR

```javascript
// hooks/useWebSocket.js
import { useState, useEffect, useCallback } from 'react'

export function useWebSocket(url) {
  const [socket, setSocket] = useState(null)
  const [connected, setConnected] = useState(false)
  
  const connect = useCallback(() => {
    const ws = new WebSocket(url)
    
    ws.onopen = () => {
      setConnected(true)
      setSocket(ws)
    }
    
    ws.onclose = () => {
      setConnected(false)
      setSocket(null)
    }
    
    ws.onerror = (error) => {
      console.error('WebSocket error:', error)
    }
  }, [url])
  
  const disconnect = useCallback(() => {
    if (socket) {
      socket.close()
    }
  }, [socket])
  
  useEffect(() => {
    connect()
    
    return () => {
      disconnect()
    }
  }, [connect, disconnect])
  
  // HMR 清理
  if (import.meta.hot) {
    import.meta.hot.dispose(() => {
      disconnect()
    })
  }
  
  return { socket, connected, connect, disconnect }
}
```

## HMR 配置选项

### 服务器配置

```javascript
// vite.config.js
export default defineConfig({
  server: {
    // HMR 配置
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
        errors: true,
        warnings: false
      },
      
      // 自定义 HMR 监听器
      clientPort: 24678
    },
    
    // 文件监听配置
    watch: {
      // 使用轮询
      usePolling: false,
      
      // 轮询间隔
      interval: 100,
      
      // 忽略的文件
      ignored: [
        '**/node_modules/**',
        '**/.git/**',
        '**/dist/**'
      ]
    }
  }
})
```

### 禁用 HMR

```javascript
// 完全禁用 HMR
export default defineConfig({
  server: {
    hmr: false
  }
})

// 生产环境自动禁用
export default defineConfig(({ mode }) => ({
  server: {
    hmr: mode !== 'production'
  }
}))
```

### 自定义 HMR 处理器

```javascript
// vite.config.js
export default defineConfig({
  plugins: [
    {
      name: 'custom-hmr-plugin',
      
      handleHotUpdate({ file, server, modules }) {
        console.log('File changed:', file)
        
        // 自定义 HMR 处理
        if (file.endsWith('.custom')) {
          server.ws.send({
            type: 'custom-event',
            event: 'custom-update',
            data: { file, content: readFileSync(file, 'utf-8') }
          })
          
          // 不触发默认 HMR
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
        
        // 添加自定义模块
        modules.push({
          id: file,
          type: 'js-update'
        })
        
        return modules
      }
    }
  ]
})
```

## HMR 故障排查

### 常见问题

#### 1. HMR 不生效

**问题表现**：修改代码后没有自动更新

**解决方案**：

```javascript
// 检查 HMR 是否启用
if (import.meta.hot) {
  console.log('HMR is available')
} else {
  console.log('HMR is not available')
}

// 确保正确使用 HMR API
import.meta.hot.accept(() => {
  console.log('Module accepted')
})
```

#### 2. 状态丢失

**问题表现**：HMR 后应用状态重置

**解决方案**：

```javascript
// 使用 dispose 和 accept 保持状态
if (import.meta.hot) {
  import.meta.hot.dispose((data) => {
    // 保存状态
    data.appState = getAppState()
  })
  
  import.meta.hot.accept((newModule) => {
    // 恢复状态
    if (data.appState) {
      setAppState(data.appState)
    }
  })
}
```

#### 3. 样式不更新

**问题表现**：修改 CSS 后样式没有变化

**解决方案**：

```javascript
// 确保 CSS 文件被正确导入
import './styles.css'

// 检查 CSS 配置
export default defineConfig({
  css: {
    devSourcemap: true // 开启 CSS 源码映射
  }
})
```

#### 4. 组件更新失败

**问题表现**：React/Vue 组件更新后报错

**解决方案**：

```javascript
// React - 确保 Fast Refresh 正确配置
export default defineConfig({
  plugins: [
    react({
      fastRefresh: true,
      jsxImportSource: 'react'
    })
  ]
})

// Vue - 确保组件正确导出
export default {
  name: 'MyComponent',
  // 组件内容
}
```

### 调试技巧

#### 1. 启用详细日志

```javascript
// vite.config.js
export default defineConfig({
  server: {
    hmr: {
      overlay: true,
      protocol: 'ws',
      host: 'localhost',
      port: 24678
    }
  },
  clearScreen: false
})
```

#### 2. 监听 HMR 事件

```javascript
if (import.meta.hot) {
  import.meta.hot.on('vite:beforeUpdate', (context) => {
    console.log('Before update:', context.updates)
  })
  
  import.meta.hot.on('vite:afterUpdate', (context) => {
    console.log('After update:', context.updates)
  })
  
  import.meta.hot.on('vite:error', (error) => {
    console.error('HMR Error:', error)
  })
}
```

#### 3. 网络检查

```javascript
// 检查 WebSocket 连接
const ws = new WebSocket('ws://localhost:24678')
ws.onopen = () => console.log('WebSocket connected')
ws.onerror = (error) => console.error('WebSocket error:', error)
ws.onclose = () => console.log('WebSocket disconnected')
```

## 性能优化

### 减少不必要的 HMR

```javascript
// 排除不需要热更新的文件
export default defineConfig({
  server: {
    watch: {
      ignored: [
        '**/node_modules/**',
        '**/dist/**',
        '**/*.test.js',
        '**/*.spec.js',
        '**/__tests__/**'
      ]
    }
  }
})
```

### 优化 HMR 性能

```javascript
// 使用更高效的文件监听
export default defineConfig({
  server: {
    watch: {
      usePolling: false, // 优先使用原生文件监听
      interval: 100      // 轮询间隔
    }
  },
  // 优化依赖预构建
  optimizeDeps: {
    include: ['vue', 'react', 'react-dom']
  }
})
```

### 按需 HMR

```javascript
// 只在特定条件下启用 HMR
export default defineConfig(({ mode }) => ({
  server: {
    hmr: mode === 'development',
    watch: {
      ignored: mode === 'production' 
        ? ['**/*'] 
        : ['**/node_modules/**']
    }
  }
}))
```

## 注意事项

### 1. 生命周期管理

```javascript
// 正确清理副作用
if (import.meta.hot) {
  import.meta.hot.dispose(() => {
    // 清理定时器
    clearInterval(timerId)
    
    // 移除事件监听
    window.removeEventListener('resize', handleResize)
    
    // 断开连接
    socket.close()
    
    // 清理 DOM
    cleanupDOM()
  })
}
```

### 2. 状态同步

```javascript
// 确保 HMR 后状态一致
if (import.meta.hot) {
  import.meta.hot.dispose((data) => {
    data.timestamp = Date.now()
    data.state = currentState
  })
  
  import.meta.hot.accept((newModule) => {
    if (data.state) {
      // 恢复状态
      restoreState(data.state)
    }
  })
}
```

### 3. 错误处理

```javascript
// 优雅处理 HMR 错误
if (import.meta.hot) {
  import.meta.hot.on('vite:error', (error) => {
    console.error('HMR Error:', error)
    
    // 显示用户友好的错误信息
    showErrorToast('模块更新失败，请刷新页面')
    
    // 可选：触发页面刷新
    // window.location.reload()
  })
}
```

## 最佳实践

### 1. 模块设计

- 保持模块职责单一
- 避免副作用
- 提供 clear/cleanup 方法
- 支持 HMR API

### 2. 状态管理

- 使用外部状态管理库
- 设计可序列化的状态
- 实现状态恢复逻辑
- 避免全局状态污染

### 3. 错误处理

- 捕获 HMR 错误
- 提供降级方案
- 记录错误日志
- 通知开发者

### 4. 性能考虑

- 减少不必要的模块更新
- 优化文件监听策略
- 使用缓存机制
- 避免频繁的全量刷新

## 总结

Vite HMR 通过以下机制提供极速的开发体验：

- **原生 ES 模块**：利用浏览器原生模块系统，无需重新打包
- **即时编译**：只编译变化的模块，大幅减少更新时间
- **智能更新**：保持应用状态，避免页面刷新
- **丰富 API**：提供完善的 HMR API 支持各种场景
- **框架集成**：与 Vue、React 等框架深度集成

HMR 是现代前端开发体验的重要组成部分，掌握其原理和最佳实践能够：

- 提升开发效率
- 改善开发体验
- 减少调试时间
- 保持应用状态

通过合理使用 HMR API 和配置，可以构建出具有优秀开发体验的前端应用。Vite 的 HMR 实现为开发者提供了快速、可靠的代码更新机制，是现代前端开发的重要工具。