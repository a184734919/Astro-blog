---
title: Vite 插件系统
published: 2024-03-26
description: '插件开发和使用的详细介绍和学习笔记'
image: ''
tags: ["Vite","插件"]
category: 'Vite'
draft: false
lang: 'zh-CN'
---

## 概述

Vite 插件系统基于 Rollup 插件机制设计，提供了强大的扩展能力。Vite 插件可以处理从代码转换、资源优化到开发体验增强等各个方面。插件机制是 Vite 生态系统的核心，使其能够适应各种前端框架和开发需求。

## 插件系统架构

### 插件工作原理

Vite 插件系统分为两个主要阶段：

```javascript
// 开发环境
请求 → Vite 开发服务器 → 插件转换 → 返回给浏览器

// 生产环境
构建过程 → 插件转换 → Rollup 打包 → 输出文件
```

### 插件生命周期

```javascript
// 插件示例
export default function myPlugin() {
  return {
    name: 'my-plugin',
    
    // 配置阶段钩子
    config(config) {
      return config
    },
    
    configResolved(resolvedConfig) {
      console.log('配置已解析')
    },
    
    // 构建阶段钩子
    options(options) {
      return options
    },
    
    // 模块生命周期钩子
    resolveId(source) {
      return source
    },
    
    load(id) {
      return null
    },
    
    transform(code, id) {
      return code
    },
    
    // 构建结束钩子
    buildEnd() {
      console.log('构建结束')
    },
    
    closeBundle() {
      console.log('打包完成')
    }
  }
}
```

## 插件类型

### 1. 通用插件

适用于开发和生产环境的插件：

```javascript
export default function universalPlugin() {
  return {
    name: 'universal-plugin',
    
    transform(code, id) {
      // 在两个环境中都会执行
      if (id.endsWith('.custom')) {
        return {
          code: `console.log('processed');\n${code}`,
          map: null
        }
      }
    }
  }
}
```

### 2. 开发环境专用插件

只在开发环境运行：

```javascript
export default function devPlugin() {
  return {
    name: 'dev-plugin',
    
    // 配置阶段
    config(config, { command }) {
      if (command === 'serve') {
        return {
          server: {
            middlewareMode: true
          }
        }
      }
    },
    
    // 中间件模式
    configureServer(server) {
      server.middlewares.use((req, res, next) => {
        if (req.url === '/api/test') {
          res.end('Hello from dev server!')
        } else {
          next()
        }
      })
    }
  }
}
```

### 3. 生产环境专用插件

只在生产环境运行：

```javascript
export default function prodPlugin() {
  return {
    name: 'prod-plugin',
    
    buildEnd() {
      console.log('生产构建完成')
    },
    
    generateBundle(options, bundle) {
      // 处理生成的文件
      Object.keys(bundle).forEach(fileName => {
        if (fileName.endsWith('.js')) {
          bundle[fileName].code += '\n// Production build'
        }
      })
    }
  }
}
```

## 核心钩子详解

### 1. 配置阶段钩子

#### config

修改 Vite 配置：

```javascript
export default function configPlugin() {
  return {
    name: 'config-plugin',
    
    config(config, { command }) {
      return {
        server: {
          port: 3000,
          proxy: {
            '/api': 'http://localhost:8080'
          }
        },
        resolve: {
          alias: {
            '@test': '/src/test'
          }
        }
      }
    }
  }
}
```

#### configResolved

在配置解析后执行：

```javascript
export default function configResolvedPlugin() {
  return {
    name: 'config-resolved-plugin',
    
    configResolved(config) {
      console.log('服务器端口:', config.server.port)
      console.log('输出目录:', config.build.outDir)
      
      // 保存配置引用
      this.viteConfig = config
    }
  }
}
```

### 2. 构建阶段钩子

#### options

修改 Rollup 选项：

```javascript
export default function optionsPlugin() {
  return {
    name: 'options-plugin',
    
    options(options) {
      return {
        ...options,
        input: {
          main: options.input
        }
      }
    }
  }
}
```

### 3. 模块生命周期钩子

#### resolveId

解析模块 ID：

```javascript
export default function resolveIdPlugin() {
  return {
    name: 'resolve-id-plugin',
    
    resolveId(source, importer) {
      // 虚拟模块
      if (source.startsWith('virtual:')) {
        return {
          id: source,
          resolvedBy: 'my-plugin'
        }
      }
      
      // 路径别名
      if (source.startsWith('@')) {
        const path = source.replace('@', '/src')
        return path
      }
      
      return null
    }
  }
}
```

#### load

加载模块内容：

```javascript
export default function loadPlugin() {
  return {
    name: 'load-plugin',
    
    load(id) {
      // 虚拟模块
      if (id === 'virtual:module') {
        return `
          export const value = 'from virtual module';
          export function hello() {
            console.log('Hello from virtual module!');
          }
        `
      }
      
      // JSON 文件
      if (id.endsWith('.json')) {
        const json = JSON.stringify({ message: 'Hello' })
        return `export default ${json}`
      }
      
      return null
    }
  }
}
```

#### transform

转换代码：

```javascript
export default function transformPlugin() {
  return {
    name: 'transform-plugin',
    
    transform(code, id) {
      // 只处理特定文件
      if (id.endsWith('.vue')) {
        return {
          code: code.replace(/__VERSION__/g, '1.0.0'),
          map: null
        }
      }
      
      // 自动导入
      if (id.endsWith('.js')) {
        const imports = []
        if (code.includes('useState')) {
          imports.push("import { useState } from 'react'")
        }
        
        if (imports.length > 0) {
          return {
            code: imports.join('\n') + '\n\n' + code,
            map: null
          }
        }
      }
      
      return null
    }
  }
}
```

### 4. 构建结束钩子

#### buildEnd

构建结束时调用：

```javascript
export default function buildEndPlugin() {
  return {
    name: 'build-end-plugin',
    
    buildEnd(error) {
      if (error) {
        console.error('构建失败:', error)
      } else {
        console.log('构建成功完成')
        
        // 执行构建后任务
        this.copyAssets()
        this.generateDocs()
      }
    },
    
    copyAssets() {
      // 复制静态资源
      console.log('复制静态资源...')
    },
    
    generateDocs() {
      // 生成文档
      console.log('生成文档...')
    }
  }
}
```

#### generateBundle

在生成打包文件之前：

```javascript
export default function generateBundlePlugin() {
  return {
    name: 'generate-bundle-plugin',
    
    generateBundle(options, bundle) {
      // 遍历生成的文件
      Object.keys(bundle).forEach(fileName => {
        const file = bundle[fileName]
        
        if (file.type === 'asset') {
          console.log('资源文件:', fileName, file.fileName)
        } else if (file.type === 'chunk') {
          console.log('代码块:', fileName, file.code.length)
          
          // 分析依赖
          file.imports.forEach(importId => {
            console.log('依赖:', importId)
          })
        }
      })
      
      // 添加额外文件
      bundle['version.txt'] = {
        type: 'asset',
        source: 'Version: 1.0.0\nBuild Time: ' + new Date().toISOString()
      }
    }
  }
}
```

#### closeBundle

打包文件写入完成后：

```javascript
import fs from 'fs'
import path from 'path'

export default function closeBundlePlugin() {
  return {
    name: 'close-bundle-plugin',
    
    closeBundle() {
      console.log('打包完成')
      
      // 上传到 CDN
      this.uploadToCDN()
      
      // 通知团队
      this.notifyTeam()
    },
    
    uploadToCDN() {
      console.log('上传到 CDN...')
    },
    
    notifyTeam() {
      console.log('通知团队...')
    }
  }
}
```

## 开发环境专用钩子

### configureServer

配置开发服务器：

```javascript
export default function serverPlugin() {
  return {
    name: 'server-plugin',
    
    configureServer(server) {
      // 添加中间件
      server.middlewares.use((req, res, next) => {
        if (req.url.startsWith('/api/')) {
          res.setHeader('Content-Type', 'application/json')
          res.end(JSON.stringify({ message: 'API response' }))
        } else {
          next()
        }
      })
      
      // WebSocket 处理
      server.ws.on('connection', (socket) => {
        socket.send('Welcome to Vite dev server')
      })
      
      // 返回清理函数
      return () => {
        server.middlewares = server.middlewares.filter(
          middleware => middleware.name !== 'api-middleware'
        )
      }
    }
  }
}
```

### handleHotUpdate

自定义热更新处理：

```javascript
export default function hmrPlugin() {
  return {
    name: 'hmr-plugin',
    
    handleHotUpdate({ file, server, modules }) {
      console.log('文件变更:', file)
      
      // 自定义 HMR 处理
      if (file.endsWith('.custom')) {
        server.ws.send({
          type: 'custom-event',
          event: 'custom-update',
          data: { file }
        })
        
        return [] // 不触发默认 HMR
      }
      
      // 过滤不需要热更新的模块
      return modules.filter(module => 
        !module.url.includes('exclude-path')
      )
    }
  }
}
```

### transformIndexHtml

转换 HTML：

```javascript
export default function htmlTransformPlugin() {
  return {
    name: 'html-transform-plugin',
    
    transformIndexHtml(html, { path, filename }) {
      // 添加环境变量
      html = html.replace(
        '__APP_VERSION__',
        process.env.npm_package_version || '1.0.0'
      )
      
      // 添加 meta 标签
      const metaTags = `
        <meta name="generator" content="Vite">
        <meta name="build-time" content="${new Date().toISOString()}">
      `
      html = html.replace('<head>', '<head>' + metaTags)
      
      // 添加 CDN 链接
      if (process.env.NODE_ENV === 'production') {
        const cdnLink = `
          <script src="https://cdn.example.com/vite-cdn.js"></script>
        `
        html = html.replace('</head>', cdnLink + '</head>')
      }
      
      return {
        html,
        tags: [
          {
            tag: 'script',
            attrs: { src: '/custom-script.js' },
            injectTo: 'head'
          }
        ]
      }
    }
  }
}
```

## 插件开发实践

### 1. 简单转换插件

```javascript
export default function simpleTransformPlugin() {
  return {
    name: 'simple-transform',
    
    transform(code, id) {
      // 只处理 .txt 文件
      if (!id.endsWith('.txt')) return null
      
      // 将文本转换为 JavaScript 模块
      const jsCode = `
        export default ${JSON.stringify(code)}
      `
      
      return {
        code: jsCode,
        map: null
      }
    }
  }
}
```

### 2. 虚拟模块插件

```javascript
export default function virtualModulePlugin(options = {}) {
  const { prefix = 'virtual:' } = options
  const virtualModules = {}
  
  return {
    name: 'virtual-module-plugin',
    
    resolveId(id) {
      if (id.startsWith(prefix)) {
        return {
          id,
          resolvedBy: 'virtual-module-plugin'
        }
      }
      return null
    },
    
    load(id) {
      if (virtualModules[id]) {
        return virtualModules[id]
      }
      return null
    },
    
    // 提供注册接口
    registerModule(id, content) {
      virtualModules[`${prefix}${id}`] = content
    }
  }
}

// 使用示例
const plugin = virtualModulePlugin()
plugin.registerModule('config', `
  export const config = {
    api: 'https://api.example.com'
  }
`)
```

### 3. 自动导入插件

```javascript
export default function autoImportPlugin(options = {}) {
  const { imports = [], dts = 'auto-imports.d.ts' } = options
  
  return {
    name: 'auto-import-plugin',
    
    transform(code, id) {
      if (!id.endsWith('.js') && !id.endsWith('.jsx')) return null
      
      const addedImports = []
      
      imports.forEach(item => {
        const [from, names] = typeof item === 'string' 
          ? [item, ['default']]
          : [item[0], item[1]]
        
        names.forEach(name => {
          const pattern = name === 'default' 
            ? new RegExp(`\\b${from}\\b`)
            : new RegExp(`\\b${from}\\.${name}\\b`)
          
          if (pattern.test(code)) {
            const importName = name === 'default' ? from : name
            addedImports.push(
              `import { ${importName} } from '${from}';`
            )
            code = code.replace(pattern, importName)
          }
        })
      })
      
      if (addedImports.length > 0) {
        return {
          code: addedImports.join('\n') + '\n\n' + code,
          map: null
        }
      }
    }
  }
}
```

### 4. 组件自动注册插件

```javascript
import fs from 'fs'
import path from 'path'

export default function componentAutoRegisterPlugin(options = {}) {
  const { 
    componentsDir = 'src/components',
    outputDir = 'src/components.d.ts'
  } = options
  
  return {
    name: 'component-auto-register',
    
    buildEnd() {
      const components = this.findComponents(componentsDir)
      this.generateDeclarations(components, outputDir)
    },
    
    findComponents(dir) {
      const components = []
      const files = fs.readdirSync(dir, { recursive: true })
      
      files.forEach(file => {
        if (file.endsWith('.vue')) {
          const componentName = path.basename(file, '.vue')
          const componentPath = path.join(dir, file)
          components.push({
            name: componentName,
            path: componentPath
          })
        }
      })
      
      return components
    },
    
    generateDeclarations(components, outputFile) {
      const declarations = components.map(comp => `
  '${comp.name}': () => import('${comp.path}')`).join(',')
      
      const content = `
/* eslint-disable */
/* prettier-ignore */
// @ts-nocheck
// Generated by vite-plugin-component-auto-register
export default {
${declarations}
}
`
      
      fs.writeFileSync(outputFile, content)
      console.log(`Generated ${outputFile}`)
    }
  }
}
```

## 常用生态插件

### 1. Vue 相关插件

```javascript
import vue from '@vitejs/plugin-vue'

export default {
  plugins: [
    vue({
      // 自定义模板编译器选项
      template: {
        compilerOptions: {
          isCustomElement: tag => tag.startsWith('my-')
        }
      },
      // 默认脚本语言
      script: {
        defineModel: true,
        propsDestructure: true
      }
    })
  ]
}
```

### 2. React 相关插件

```javascript
import react from '@vitejs/plugin-react'

export default {
  plugins: [
    react({
      // 使用 SWC 替换 Babel
      jsxImportSource: 'react',
      jsxRuntime: 'automatic',
      babel: {
        plugins: ['@babel/plugin-proposal-decorators']
      }
    })
  ]
}
```

### 3. 自动导入插件

```javascript
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'
import AutoImport from 'unplugin-auto-import/vite'

export default {
  plugins: [
    // 自动导入组件
    Components({
      resolvers: [
        ElementPlusResolver(),
        (componentName) => {
          if (componentName.startsWith('Icon'))
            return {
              name: componentName.slice(4),
              from: '@element-plus/icons-vue'
            }
        }
      ],
      dts: 'src/components.d.ts'
    }),
    
    // 自动导入 API
    AutoImport({
      imports: [
        'vue',
        'vue-router',
        'pinia',
        {
          'axios': [
            ['default', 'axios'],
            ['AxiosInstance', 'AxiosInstance']
          ]
        }
      ],
      dts: 'src/auto-imports.d.ts',
      eslintrc: {
        enabled: true,
        filepath: './.eslintrc-auto.json'
      }
    })
  ]
}
```

### 4. UI 框架插件

```javascript
import { VantResolver } from 'unplugin-vue-components/resolvers'

export default {
  plugins: [
    Components({
      resolvers: [
        VantResolver(),
        AntDesignVueResolver(),
        ElementPlusResolver()
      ]
    })
  ]
}
```

### 5. 压缩插件

```javascript
import viteCompression from 'vite-plugin-compression'

export default {
  plugins: [
    // Gzip 压缩
    viteCompression({
      verbose: true,
      disable: false,
      threshold: 10240,
      algorithm: 'gzip',
      ext: '.gz'
    }),
    
    // Brotli 压缩
    viteCompression({
      verbose: true,
      disable: false,
      threshold: 10240,
      algorithm: 'brotliCompress',
      ext: '.br'
    })
  ]
}
```

### 6. PWA 插件

```javascript
import { VitePWA } from 'vite-plugin-pwa'

export default {
  plugins: [
    VitePWA({
      registerType: 'autoUpdate',
      includeAssets: ['favicon.ico', 'robots.txt', 'apple-touch-icon.png'],
      manifest: {
        name: 'My PWA App',
        short_name: 'MyApp',
        description: 'My awesome PWA app',
        theme_color: '#ffffff',
        icons: [
          {
            src: '/pwa-192x192.png',
            sizes: '192x192',
            type: 'image/png'
          },
          {
            src: '/pwa-512x512.png',
            sizes: '512x512',
            type: 'image/png'
          }
        ]
      }
    })
  ]
}
```

### 7. 图片优化插件

```javascript
import viteImagemin from 'vite-plugin-imagemin'

export default {
  plugins: [
    viteImagemin({
      gifsicle: {
        optimizationLevel: 7,
        interlaced: false
      },
      optipng: {
        optimizationLevel: 7
      },
      mozjpeg: {
        quality: 80
      },
      pngquant: {
        quality: [0.8, 0.9],
        speed: 4
      },
      svgo: {
        plugins: [
          {
            name: 'removeViewBox'
          },
          {
            name: 'removeEmptyAttrs',
            active: false
          }
        ]
      }
    })
  ]
}
```

## 插件最佳实践

### 1. 插件命名规范

```javascript
export default function myPlugin(options) {
  return {
    name: 'my-plugin', // 使用 kebab-case
    // 插件逻辑
  }
}
```

### 2. 错误处理

```javascript
export default function robustPlugin() {
  return {
    name: 'robust-plugin',
    
    transform(code, id) {
      try {
        // 插件逻辑
        return { code, map: null }
      } catch (error) {
        this.error(`Plugin error processing ${id}: ${error.message}`)
        return null
      }
    }
  }
}
```

### 3. 性能优化

```javascript
export default function optimizedPlugin() {
  const cache = new Map()
  
  return {
    name: 'optimized-plugin',
    
    transform(code, id) {
      // 检查缓存
      if (cache.has(id)) {
        return cache.get(id)
      }
      
      // 只处理需要的文件
      if (!id.endsWith('.js')) return null
      
      const result = this.processCode(code, id)
      
      // 缓存结果
      cache.set(id, result)
      
      return result
    },
    
    buildEnd() {
      cache.clear()
    }
  }
}
```

### 4. 插件顺序控制

```javascript
export default defineConfig({
  plugins: [
    {
      name: 'pre-plugin',
      enforce: 'pre' // 在其他插件之前
    },
    vue(),
    {
      name: 'post-plugin',
      enforce: 'post' // 在其他插件之后
    },
    {
      name: 'default-plugin' // 默认顺序
    }
  ]
})
```

### 5. 插件配置

```javascript
export default function configurablePlugin(options = {}) {
  const {
    include = [],
    exclude = [],
    transform = (code) => code
  } = options
  
  return {
    name: 'configurable-plugin',
    
    transform(code, id) {
      // 检查是否应该处理
      if (!this.shouldTransform(id, include, exclude)) {
        return null
      }
      
      return transform(code, id)
    },
    
    shouldTransform(id, include, exclude) {
      const fileName = path.basename(id)
      
      // 检查排除列表
      if (exclude.some(pattern => fileName.match(pattern))) {
        return false
      }
      
      // 检查包含列表
      if (include.length === 0) {
        return true
      }
      
      return include.some(pattern => fileName.match(pattern))
    }
  }
}
```

## 注意事项

### 1. 插件兼容性

- 确保 Vite 版本兼容性
- 检查依赖的 Rollup 版本
- 测试跨平台兼容性

### 2. 性能考虑

- 避免在转换中进行文件 I/O
- 合理使用缓存机制
- 减少正则表达式复杂度

### 3. 错误处理

- 提供清晰的错误信息
- 避免阻塞构建流程
- 实现优雅降级

### 4. 类型安全

```typescript
import type { Plugin } from 'vite'

export default function typedPlugin(options?: PluginOptions): Plugin {
  return {
    name: 'typed-plugin',
    // 实现类型安全的钩子
  }
}

interface PluginOptions {
  enabled?: boolean
  config?: Record<string, any>
}
```

## 总结

Vite 插件系统提供了强大而灵活的扩展能力：

- **统一的 API**：开发和生产环境使用相同的插件 API
- **丰富的钩子**：支持从配置到构建结束的完整生命周期
- **生态完善**：拥有丰富的社区插件
- **易于开发**：简单的插件 API 便于自定义扩展

通过掌握 Vite 插件开发，可以：
- 定制化开发体验
- 优化构建性能
- 集成第三方工具
- 实现项目特定需求

插件系统是 Vite 生态的核心，理解其工作原理和开发方法对于深度使用 Vite 至关重要。无论是使用现有插件还是开发自定义插件，都能显著提升开发效率和应用性能。