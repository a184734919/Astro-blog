---
title: Vite 代码分割
published: 2024-06-22
description: '动态导入和代码分割的详细介绍和学习笔记'
image: ''
tags: ["Vite","优化"]
category: 'Vite'
draft: false
lang: 'zh-CN'
---

## 概述

代码分割是现代前端性能优化的核心技术之一。它允许将应用代码分割成多个小块，按需加载，从而减少初始加载体积，提升应用性能。Vite 基于原生 ES 模块和 Rollup，提供了强大而灵活的代码分割能力。

## 代码分割原理

### 传统加载 vs 代码分割

```javascript
// 传统方式 - 所有代码一次性加载
import { heavyLibrary } from 'heavy-library'
import { anotherLib } from 'another-library'

// 代码分割 - 按需加载
async function loadFeature() {
  const { heavyLibrary } = await import('heavy-library')
  const { anotherLib } = await import('another-lib')
  // 使用这些库
}
```

### Vite 代码分割机制

```javascript
// 开发环境
// Vite 直接使用 ES 模块，浏览器原生按需加载
import('./module.js') // 浏览器直接请求

// 生产环境
// Rollup 将动态导入转换为代码分割
// chunk-1.js, chunk-2.js, chunk-3.js...
```

## 基本代码分割

### 1. 动态导入

#### 基础语法

```javascript
// 基础动态导入
const module = await import('./module.js')

// 带别名的动态导入
const { export1, export2 } = await import('./module.js')

// 命名动态导入
const { default: Module } = await import('./module.js')
```

#### 组件懒加载

```javascript
// Vue 3 组件懒加载
const AsyncComponent = defineAsyncComponent(() =>
  import('./components/AsyncComponent.vue')
)

// 在模板中使用
<template>
  <AsyncComponent />
</template>
```

```javascript
// React 组件懒加载
import { lazy, Suspense } from 'react'

const AsyncComponent = lazy(() => import('./components/AsyncComponent'))

// 在组件中使用
function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <AsyncComponent />
    </Suspense>
  )
}
```

#### 路由懒加载

```javascript
// Vue Router 懒加载
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/',
      name: 'Home',
      component: () => import('@/views/Home.vue')
    },
    {
      path: '/about',
      name: 'About',
      component: () => import('@/views/About.vue')
    },
    {
      path: '/dashboard',
      name: 'Dashboard',
      component: () => import('@/views/Dashboard.vue'),
      children: [
        {
          path: 'overview',
          component: () => import('@/views/dashboard/Overview.vue')
        },
        {
          path: 'settings',
          component: () => import('@/views/dashboard/Settings.vue')
        }
      ]
    }
  ]
})

export default router
```

```javascript
// React Router 懒加载
import { lazy, Suspense } from 'react'
import { BrowserRouter, Routes, Route } from 'react-router-dom'

const Home = lazy(() => import('./views/Home'))
const About = lazy(() => import('./views/About'))
const Dashboard = lazy(() => import('./views/Dashboard'))

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  )
}
```

### 2. 条件性加载

```javascript
// 基于条件的动态导入
async function initializeEditor() {
  if (user.preferences.enableRichText) {
    const { Editor } = await import('./editor/RichTextEditor')
    return Editor
  } else {
    const { Editor } = await import('./editor/PlainTextEditor')
    return Editor
  }
}

// 功能开关
async function loadFeature(featureName) {
  const features = {
    charts: () => import('./features/charts'),
    maps: () => import('./features/maps'),
    charts3d: () => import('./features/charts3d')
  }
  
  if (features[featureName]) {
    return await features[featureName]()
  }
  
  throw new Error(`Feature ${featureName} not found`)
}
```

### 3. 按需加载库

```javascript
// 按需加载大型库
async function generatePDF() {
  const { jsPDF } = await import('jspdf')
  const doc = new jsPDF()
  // 生成 PDF
}

async function processImage() {
  const { default: sharp } = await import('sharp')
  // 图片处理
}

async function formatDate() {
  const { format } = await import('date-fns')
  return format(new Date(), 'yyyy-MM-dd')
}
```

## 高级代码分割策略

### 1. 手动分包配置

```javascript
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        // 手动代码分割
        manualChunks: {
          // Vue 相关
          'vue-vendor': ['vue', 'vue-router', 'pinia', '@vueuse/core'],
          
          // UI 库
          'ui-vendor': ['element-plus', '@element-plus/icons-vue'],
          
          // 工具库
          'utils': ['axios', 'lodash-es', 'dayjs', 'js-cookie'],
          
          // 第三方库
          'third-party': ['echarts', 'html2canvas', 'jspdf'],
          
          // 图标库
          'icons': ['@ant-design/icons', '@element-plus/icons-vue']
        }
      }
    }
  }
})
```

### 2. 函数式分包

```javascript
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks(id) {
          // 将 node_modules 中的包分类
          if (id.includes('node_modules')) {
            // Vue 生态
            if (id.includes('vue') || id.includes('pinia')) {
              return 'vue-vendor'
            }
            
            // UI 库
            if (id.includes('element-plus') || id.includes('antd')) {
              return 'ui-vendor'
            }
            
            // 工具库
            if (id.includes('lodash') || id.includes('axios')) {
              return 'utils'
            }
            
            // 图表库
            if (id.includes('echarts') || id.includes('chart.js')) {
              return 'charts'
            }
            
            // 默认分包
            return 'vendor'
          }
        }
      }
    }
  }
})
```

### 3. 基于路由的分包

```javascript
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks(id) {
          // 基于路由分包
          if (id.includes('/views/home/')) {
            return 'home-page'
          }
          
          if (id.includes('/views/dashboard/')) {
            return 'dashboard-page'
          }
          
          if (id.includes('/views/admin/')) {
            return 'admin-page'
          }
          
          if (id.includes('/views/user/')) {
            return 'user-page'
          }
        }
      }
    }
  }
})
```

### 4. 按功能模块分包

```javascript
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks(id) {
          // 按功能模块分包
          const modules = {
            'auth': ['/api/auth', '/stores/auth', '/views/auth'],
            'user': ['/api/user', '/stores/user', '/views/user'],
            'product': ['/api/product', '/stores/product', '/views/product'],
            'order': ['/api/order', '/stores/order', '/views/order'],
            'dashboard': ['/api/dashboard', '/views/dashboard']
          }
          
          for (const [chunkName, paths] of Object.entries(modules)) {
            if (paths.some(path => id.includes(path))) {
              return chunkName
            }
          }
        }
      }
    }
  }
})
```

## 性能优化策略

### 1. 预加载优化

```javascript
// 预加载关键资源
function preloadCriticalResources() {
  // 预加载可能会用到的组件
  const preloadRoute = (route) => {
    import(/* webpackPrefetch: true */ `@/views/${route}.vue`)
  }
  
  // 预加载常见路由
  setTimeout(() => {
    preloadRoute('About')
    preloadRoute('Contact')
  }, 2000)
  
  // 预加载依赖
  const preloadLibrary = (lib) => {
    import(/* webpackPrefetch: true */ lib)
  }
  
  setTimeout(() => {
    preloadLibrary('echarts')
    preloadLibrary('html2canvas')
  }, 3000)
}
```

### 2. 智能预加载

```javascript
// 基于用户行为的预加载
class SmartPreloader {
  constructor() {
    this.loadedChunks = new Set()
    this.userBehavior = this.trackUserBehavior()
  }
  
  trackUserBehavior() {
    // 跟踪用户行为
    const behaviors = []
    
    document.addEventListener('mousemove', (e) => {
      if (e.clientX < 50 && e.clientY < 50) {
        behaviors.push('dashboard')
      }
    })
    
    return behaviors
  }
  
  predictNextPage() {
    // 基于行为预测下一页
    const frequency = {}
    this.userBehavior.forEach(page => {
      frequency[page] = (frequency[page] || 0) + 1
    })
    
    return Object.keys(frequency).sort((a, b) => 
      frequency[b] - frequency[a]
    )[0]
  }
  
  preloadPredictedPage() {
    const nextPage = this.predictNextPage()
    if (nextPage && !this.loadedChunks.has(nextPage)) {
      this.loadedChunks.add(nextPage)
      import(`@/views/${nextPage}.vue`)
    }
  }
}
```

### 3. 按需加载优化

```javascript
// 懒加载图片
const lazyLoadImage = (imgSrc) => {
  return new Promise((resolve) => {
    const img = new Image()
    img.onload = () => resolve(img.src)
    img.src = imgSrc
  })
}

// 交叉观察器懒加载
const lazyLoadElements = () => {
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const element = entry.target
        import(element.dataset.module).then((module) => {
          module.default(element)
        })
        observer.unobserve(element)
      }
    })
  })
  
  document.querySelectorAll('[data-lazy-load]').forEach(el => {
    observer.observe(el)
  })
}
```

## 实际应用案例

### 1. 电商应用代码分割

```javascript
// 路由配置
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    {
      path: '/',
      name: 'Home',
      component: () => import('@/views/home/Home.vue'),
      children: [
        {
          path: 'products',
          name: 'Products',
          component: () => import('@/views/home/Products.vue')
        },
        {
          path: 'product/:id',
          name: 'ProductDetail',
          component: () => import('@/views/home/ProductDetail.vue')
        }
      ]
    },
    {
      path: '/cart',
      name: 'Cart',
      component: () => import('@/views/cart/Cart.vue')
    },
    {
      path: '/checkout',
      name: 'Checkout',
      component: () => import('@/views/checkout/Checkout.vue')
    },
    {
      path: '/user',
      name: 'User',
      component: () => import('@/views/user/User.vue'),
      children: [
        {
          path: 'profile',
          component: () => import('@/views/user/Profile.vue')
        },
        {
          path: 'orders',
          component: () => import('@/views/user/Orders.vue')
        },
        {
          path: 'addresses',
          component: () => import('@/views/user/Addresses.vue')
        }
      ]
    },
    {
      path: '/admin',
      name: 'Admin',
      component: () => import('@/views/admin/Admin.vue'),
      children: [
        {
          path: 'products',
          component: () => import('@/views/admin/Products.vue')
        },
        {
          path: 'orders',
          component: () => import('@/views/admin/Orders.vue')
        },
        {
          path: 'users',
          component: () => import('@/views/admin/Users.vue')
        }
      ]
    }
  ]
})

export default router
```

```javascript
// vite.config.js
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks(id) {
          // 页面级分割
          if (id.includes('/views/home/')) return 'home-pages'
          if (id.includes('/views/cart/')) return 'cart-pages'
          if (id.includes('/views/checkout/')) return 'checkout-pages'
          if (id.includes('/views/user/')) return 'user-pages'
          if (id.includes('/views/admin/')) return 'admin-pages'
          
          // 功能分割
          if (id.includes('node_modules')) {
            if (id.includes('vue')) return 'vue-vendor'
            if (id.includes('element-plus')) return 'ui-vendor'
            if (id.includes('echarts')) return 'charts-vendor'
            if (id.includes('axios')) return 'http-vendor'
            return 'vendor'
          }
        }
      }
    }
  }
})
```

### 2. 管理后台代码分割

```javascript
// 基于权限的动态加载
const loadView = (view) => {
  return () => import(`@/views/${view}`).catch(() => {
    return import('@/views/Error404.vue')
  })
}

// 路由配置
const router = createRouter({
  routes: [
    {
      path: '/dashboard',
      name: 'Dashboard',
      component: loadView('dashboard/Index'),
      meta: { requiresAuth: true }
    },
    {
      path: '/users',
      name: 'Users',
      component: loadView('users/Index'),
      meta: { requiresAuth: true, permission: 'users:read' }
    },
    {
      path: '/products',
      name: 'Products',
      component: loadView('products/Index'),
      meta: { requiresAuth: true, permission: 'products:read' }
    }
  ]
})
```

### 3. 按需加载组件库

```javascript
// 组件按需加载配置
import { defineAsyncComponent } from 'vue'

// 基础组件映射
const componentMap = {
  // 表单组件
  Input: () => import('element-plus').then(m => m.ElInput),
  Select: () => import('element-plus').then(m => m.ElSelect),
  DatePicker: () => import('element-plus').then(m => m.ElDatePicker),
  
  // 数据展示
  Table: () => import('element-plus').then(m => m.ElTable),
  Tree: () => import('element-plus').then(m => m.ElTree),
  
  // 反馈组件
  Dialog: () => import('element-plus').then(m => m.ElDialog),
  Message: () => import('element-plus').then(m => m.ElMessage),
  Notification: () => import('element-plus').then(m => m.ElNotification)
}

// 动态注册组件
const registerComponent = async (name) => {
  if (componentMap[name]) {
    const component = await componentMap[name]()
    app.component(name, component)
  }
}

// 使用示例
async function loadFormComponents() {
  await Promise.all([
    registerComponent('Input'),
    registerComponent('Select'),
    registerComponent('DatePicker')
  ])
}
```

### 4. 第三方库按需加载

```javascript
// 图表库按需加载
const loadChart = async (type) => {
  const echarts = await import('echarts')
  
  const chartModules = {
    line: () => import('echarts/charts/line'),
    bar: () => import('echarts/charts/bar'),
    pie: () => import('echarts/charts/pie'),
    scatter: () => import('echarts/charts/scatter')
  }
  
  if (chartModules[type]) {
    const module = await chartModules[type]()
    echarts.use([module.default])
  }
  
  return echarts
}

// 使用示例
async function renderLineChart() {
  const echarts = await loadChart('line')
  const chart = echarts.init(document.getElementById('chart'))
  chart.setOption({
    xAxis: { data: ['Mon', 'Tue', 'Wed', 'Thu', 'Fri'] },
    yAxis: {},
    series: [{ type: 'line', data: [10, 20, 30, 40, 50] }]
  })
}
```

## 注意事项

### 1. 分包粒度

```javascript
// 过细的分包
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'chunk-1': ['vue'],
          'chunk-2': ['vue-router'],
          'chunk-3': ['pinia'],
          // ... 太多小块
        }
      }
    }
  }
})

// 合理的分包
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'vue-vendor': ['vue', 'vue-router', 'pinia'],
          'ui-vendor': ['element-plus'],
          'utils': ['axios', 'lodash-es']
        }
      }
    }
  }
})
```

### 2. 避免过度分割

```javascript
// 不推荐：每个组件单独打包
const Component1 = () => import('./components/Component1.vue')
const Component2 = () => import('./components/Component2.vue')
// ... 几十个组件

// 推荐：相关组件一起打包
const AdminComponents = () => import('./components/Admin/index.vue')
const UserComponents = () => import('./components/User/index.vue')
```

### 3. 首屏性能

```javascript
// 确保首屏关键代码不被分割
export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: (id) => {
          // 首页和关键路由不要分割
          if (id.includes('/views/home/') || id.includes('/views/common/')) {
            return 'critical'
          }
          // 其他按需分割
          return 'lazy'
        }
      }
    }
  }
})
```

## 最佳实践

### 1. 分包策略

- **路由级别**：每个路由一个 chunk
- **功能模块**：相关功能打包在一起
- **第三方库**：按使用频率和大小分组
- **公共代码**：提取共享代码

### 2. 加载优化

- **预加载**：对可能使用的代码进行预加载
- **懒加载**：非首屏代码延迟加载
- **按需加载**：基于用户行为智能加载
- **缓存策略**：合理利用浏览器缓存

### 3. 监控分析

```javascript
// 性能监控
const monitorChunkLoading = () => {
  const performanceObserver = new PerformanceObserver((list) => {
    list.getEntries().forEach((entry) => {
      if (entry.entryType === 'resource' && entry.name.includes('.js')) {
        console.log(`Chunk loaded: ${entry.name}`, {
          duration: entry.duration,
          size: entry.transferSize,
          timing: entry.timing
        })
      }
    })
  })
  
  performanceObserver.observe({ entryTypes: ['resource'] })
}

// 启动监控
monitorChunkLoading()
```

### 4. 持续优化

```javascript
// 分析打包结果
const analyzeBundle = () => {
  if (import.meta.env.DEV) {
    const stats = await import('./dist/stats.json')
    console.log('Bundle analysis:', stats)
    
    // 生成可视化报告
    const { BundleAnalyzerPlugin } = await import('webpack-bundle-analyzer')
    // ... 分析逻辑
  }
}
```

## 总结

Vite 代码分割通过以下方式优化应用性能：

- **减少初始加载**：按需加载减少首屏体积
- **提升用户体验**：页面切换更流畅
- **缓存优化**：独立 chunk 可以更好地利用缓存
- **维护便利性**：模块化代码更易于维护

合理的代码分割策略能够：

- **改善加载性能**：减少首次加载时间和网络传输
- **提升用户体验**：实现流畅的页面切换和功能加载
- **优化资源利用**：提高缓存命中率和带宽利用率
- **简化维护**：模块化结构便于代码组织和更新

掌握 Vite 代码分割技术，对于构建高性能的现代 Web 应用至关重要。建议根据应用特点和用户行为制定合适的分割策略，并持续监控和优化，以获得最佳的性能表现。