---
title: Vite 配置详解
published: 2024-03-07
description: 'vite.config.js 配置项的详细介绍和学习笔记'
image: ''
tags: ["Vite","配置"]
category: 'Vite'
draft: false
lang: 'zh-CN'
---

## 概述

Vite 配置文件（vite.config.js）是 Vite 项目的核心配置文件，它使用 ES 模块语法导出配置对象。Vite 提供了丰富的配置选项，允许开发者自定义项目的构建、开发服务器、插件、别名等各个方面。

## 配置文件基础

### 配置文件位置

Vite 会从项目根目录自动加载配置文件，支持多种格式：

```bash
# 支持的配置文件名称（按优先级）
vite.config.js
vite.config.ts
vite.config.mjs
vite.config.cjs
```

### 配置文件结构

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path'

export default defineConfig({
  // 根配置选项
  root: process.cwd(),
  base: '/',
  mode: 'development',
  
  // 开发服务器配置
  server: {
    port: 3000,
    host: 'localhost'
  },
  
  // 构建配置
  build: {
    outDir: 'dist',
    assetsDir: 'assets'
  },
  
  // 插件配置
  plugins: [vue()],
  
  // 路径解析配置
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src')
    }
  },
  
  // CSS 配置
  css: {
    preprocessorOptions: {}
  },
  
  // 优化配置
  optimizeDeps: {
    include: []
  }
})
```

### defineConfig 辅助函数

使用 `defineConfig` 可以获得类型提示和自动补全：

```javascript
import { defineConfig } from 'vite'

export default defineConfig({
  plugins: [],
  server: {
    port: 3000
  }
})
```

## 核心配置选项

### 1. 根目录配置

#### root

指定项目根目录，默认为 `process.cwd()`：

```javascript
export default defineConfig({
  root: path.resolve(__dirname, 'my-project-root')
})
```

#### base

部署时的基础路径，默认为 `/`：

```javascript
// 部署到子目录
export default defineConfig({
  base: '/my-app/'
})

// 部署到 CDN
export default defineConfig({
  base: 'https://cdn.example.com/my-app/'
})

// 相对路径（适用于本地打开）
export default defineConfig({
  base: './'
})
```

#### mode

运行模式，默认为 `development` 或 `production`：

```javascript
export default defineConfig({
  mode: 'development'
})

// 或通过命令行参数
// vite --mode staging
```

### 2. 开发服务器配置

#### server.port

指定开发服务器端口：

```javascript
export default defineConfig({
  server: {
    port: 3000,
    strictPort: true, // 端口被占用时报错
    open: true        // 自动打开浏览器
  }
})
```

#### server.host

指定开发服务器主机：

```javascript
export default defineConfig({
  server: {
    host: '0.0.0.0', // 监听所有地址
    port: 3000
  }
})
```

#### server.proxy

代理配置，解决开发环境跨域问题：

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
      },
      
      // 正则表达式
      '^/fallback/.*': {
        target: 'http://jsonplaceholder.typicode.com',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/fallback/, '')
      },
      
      // 配合 WebSocket
      '/socket.io': {
        target: 'ws://localhost:3000',
        ws: true
      },
      
      // 多个代理
      '/api/v1': {
        target: 'http://localhost:8081',
        changeOrigin: true
      },
      '/api/v2': {
        target: 'http://localhost:8082',
        changeOrigin: true
      }
    }
  }
})
```

#### server.cors

CORS 配置：

```javascript
export default defineConfig({
  server: {
    cors: true,
    cors: {
      origin: 'http://localhost:8080',
      methods: ['GET', 'POST', 'PUT', 'DELETE'],
      allowedHeaders: ['Content-Type', 'Authorization']
    }
  }
})
```

#### server.headers

自定义响应头：

```javascript
export default defineConfig({
  server: {
    headers: {
      'X-Custom-Header': 'value',
      'Access-Control-Allow-Origin': '*'
    }
  }
})
```

#### server.fs

文件系统访问控制：

```javascript
export default defineConfig({
  server: {
    fs: {
      strict: true, // 限制服务文件访问范围
      allow: ['..'], // 允许访问的目录
      deny: ['.env', '.env.*'] // 拒绝访问的文件
    }
  }
})
```

#### server.hmr

热更新配置：

```javascript
export default defineConfig({
  server: {
    hmr: {
      overlay: true, // 错误覆盖层
      port: 24678,   // HMR 端口
      host: 'localhost'
    }
  }
})
```

#### server.watch

文件监听配置：

```javascript
export default defineConfig({
  server: {
    watch: {
      usePolling: true,     // 使用轮询
      interval: 100,        // 轮询间隔
      ignored: ['**/node_modules/**', '**/dist/**']
    }
  }
})
```

### 3. 构建配置

#### build.outDir

构建输出目录：

```javascript
export default defineConfig({
  build: {
    outDir: 'dist',
    emptyOutDir: true, // 构建前清空输出目录
    // 使用相对路径
    emptyOutDir: (outDir) => {
      return fs.existsSync(outDir)
    }
  }
})
```

#### build.assetsDir

静态资源输出目录：

```javascript
export default defineConfig({
  build: {
    assetsDir: 'assets',
    // 或者放在 CDN
    assetsDir: 'https://cdn.example.com/assets'
  }
})
```

#### build.sourcemap

生成源码映射：

```javascript
export default defineConfig({
  build: {
    sourcemap: false,    // 不生成
    sourcemap: true,     // 生成独立的 .map 文件
    sourcemap: 'inline', // 内联到 JS 文件中
    sourcemap: 'hidden'  // 生成但不引用
  }
})
```

#### build.minify

压缩配置：

```javascript
export default defineConfig({
  build: {
    minify: 'terser',  // 使用 terser
    minify: 'esbuild', // 使用 esbuild (默认)
    minify: false      // 不压缩
  }
})
```

#### terserOptions

Terser 压缩选项：

```javascript
export default defineConfig({
  build: {
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true,
        pure_funcs: ['console.log']
      },
      format: {
        comments: false
      }
    }
  }
})
```

#### build.rollupOptions

Rollup 配置选项：

```javascript
export default defineConfig({
  build: {
    rollupOptions: {
      // 输入配置
      input: {
        main: path.resolve(__dirname, 'index.html'),
        admin: path.resolve(__dirname, 'admin.html')
      },
      
      // 输出配置
      output: {
        // 文件命名
        entryFileNames: 'js/[name]-[hash].js',
        chunkFileNames: 'js/[name]-[hash].js',
        assetFileNames: '[ext]/[name]-[hash].[ext]',
        
        // 手动分包
        manualChunks: {
          'vue-vendor': ['vue', 'vue-router', 'pinia'],
          'ui-vendor': ['element-plus'],
          'utils': ['axios', 'lodash-es', 'dayjs']
        },
        
        // 全局变量
        globals: {
          vue: 'Vue',
          'vue-router': 'VueRouter'
        }
      },
      
      // 外部依赖
      external: ['vue', 'vue-router'],
      
      // 插件
      plugins: [
        // Rollup 插件
      ]
    }
  }
})
```

#### build.chunkSizeWarningLimit

块大小警告限制：

```javascript
export default defineConfig({
  build: {
    chunkSizeWarningLimit: 1000 // 1000KB
  }
})
```

#### build.cssCodeSplit

CSS 代码分割：

```javascript
export default defineConfig({
  build: {
    cssCodeSplit: true // 启用 CSS 代码分割
  }
})
```

#### build.lib

构建库模式：

```javascript
export default defineConfig({
  build: {
    lib: {
      entry: path.resolve(__dirname, 'src/index.js'),
      name: 'MyLibrary',
      fileName: (format) => `my-library.${format}.js`,
      formats: ['es', 'umd', 'cjs']
    },
    rollupOptions: {
      external: ['vue'],
      output: {
        globals: {
          vue: 'Vue'
        }
      }
    }
  }
})
```

#### build.reportCompressedSize

报告压缩大小：

```javascript
export default defineConfig({
  build: {
    reportCompressedSize: false, // 禁用压缩大小报告，加快构建
    chunkSizeWarningLimit: 1000
  }
})
```

### 4. 插件配置

#### plugins

配置 Vite 插件：

```javascript
import vue from '@vitejs/plugin-vue'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'
import AutoImport from 'unplugin-auto-import/vite'

export default defineConfig({
  plugins: [
    vue(),
    
    // 自动导入组件
    Components({
      resolvers: [
        ElementPlusResolver()
      ],
      dts: 'src/components.d.ts'
    }),
    
    // 自动导入 API
    AutoImport({
      imports: [
        'vue',
        'vue-router',
        'pinia'
      ],
      dts: 'src/auto-imports.d.ts',
      eslintrc: {
        enabled: true
      }
    }),
    
    // SVG 图标
    Components({
      resolvers: [
        (componentName) => {
          if (componentName.startsWith('Icon'))
            return {
              name: componentName.slice(4),
              from: '@element-plus/icons-vue'
            }
        }
      ]
    })
  ]
})
```

#### 插件顺序控制

```javascript
export default defineConfig({
  plugins: [
    {
      name: 'my-plugin-1',
      enforce: 'pre' // 在其他插件之前执行
    },
    vue(),
    {
      name: 'my-plugin-2',
      enforce: 'post' // 在其他插件之后执行
    }
  ]
})
```

### 5. 路径解析配置

#### resolve.alias

路径别名配置：

```javascript
export default defineConfig({
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@assets': path.resolve(__dirname, './src/assets'),
      '@utils': path.resolve(__dirname, './src/utils'),
      '@api': path.resolve(__dirname, './src/api'),
      '@styles': path.resolve(__dirname, './src/styles'),
      '@views': path.resolve(__dirname, './src/views'),
      '@router': path.resolve(__dirname, './src/router'),
      '@stores': path.resolve(__dirname, './src/stores')
    }
  }
})
```

#### resolve.extensions

扩展名解析：

```javascript
export default defineConfig({
  resolve: {
    extensions: ['.js', '.jsx', '.json', '.vue', '.ts', '.tsx']
  }
})
```

#### resolve.mainFields

入口字段解析：

```javascript
export default defineConfig({
  resolve: {
    mainFields: ['module', 'jsnext:main', 'jsnext']
  }
})
```

#### resolve.dedupe

去重依赖：

```javascript
export default defineConfig({
  resolve: {
    dedupe: ['vue', 'vue-router', 'pinia']
  }
})
```

### 6. CSS 配置

#### css.preprocessorOptions

预处理器选项：

```javascript
export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: `@import "@/styles/variables.scss";`,
        api: 'modern-compiler'
      },
      less: {
        math: 'always',
        globalVars: {
          primary: '#1890ff',
          success: '#52c41a',
          warning: '#faad14',
          error: '#f5222d'
        }
      },
      stylus: {
        additionalData: `@import "@/styles/variables.styl"`
      }
    }
  }
})
```

#### css.modules

CSS Modules 配置：

```javascript
export default defineConfig({
  css: {
    modules: {
      scopeBehaviour: 'local',
      generateScopedName: '[name]__[local]___[hash:base64:5]'
    }
  }
})
```

#### css.postcss

PostCSS 配置：

```javascript
export default defineConfig({
  css: {
    postcss: {
      plugins: [
        require('autoprefixer'),
        require('cssnano')
      ]
    }
  }
})
```

#### css.devSourcemap

开发环境 CSS 源码映射：

```javascript
export default defineConfig({
  css: {
    devSourcemap: true
  }
})
```

### 7. 优化配置

#### optimizeDeps

依赖优化配置：

```javascript
export default defineConfig({
  optimizeDeps: {
    // 强制预构建
    include: [
      'vue',
      'vue-router',
      'pinia',
      'axios',
      'element-plus/es/components/button/style/css'
    ],
    // 排除预构建
    exclude: [
      'your-local-package'
    ],
    // 强制重新优化
    force: false,
    // 自定义 esbuild 选项
    esbuildOptions: {
      target: 'es2020',
      define: {
        'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV)
      }
    }
  }
})
```

#### optimizeDeps.disabled

禁用依赖优化：

```javascript
export default defineConfig({
  optimizeDeps: {
    disabled: 'dev' // 在开发环境禁用
  }
})
```

### 8. 其他重要配置

#### define

全局变量定义：

```javascript
export default defineConfig({
  define: {
    __APP_VERSION__: JSON.stringify('1.0.0'),
    __BUILD_TIME__: JSON.stringify(new Date().toISOString())
  }
})
```

#### clearScreen

清屏控制：

```javascript
export default defineConfig({
  clearScreen: false // 禁用自动清屏
})
```

#### envPrefix

环境变量前缀：

```javascript
export default defineConfig({
  envPrefix: ['VITE_', 'APP_'] // 自定义前缀
})
```

#### logLevel

日志级别：

```javascript
export default defineConfig({
  logLevel: 'info', // 'info', 'warn', 'error', 'silent'
})
```

## 高级配置

### 1. 条件配置

基于环境的条件配置：

```javascript
export default defineConfig(({ command, mode }) => {
  const isDevelopment = command === 'serve'
  const isProduction = command === 'build'
  
  return {
    plugins: [
      vue(),
      isDevelopment ? someDevPlugin() : someProdPlugin()
    ],
    server: isDevelopment ? {
      port: 3000,
      proxy: {
        '/api': 'http://localhost:8080'
      }
    } : undefined,
    build: isProduction ? {
      minify: 'terser',
      sourcemap: false
    } : undefined
  }
})
```

### 2. 配置合并

使用函数返回配置对象：

```javascript
import { defineConfig, loadEnv } from 'vite'

export default defineConfig(({ mode }) => {
  // 加载环境变量
  const env = loadEnv(mode, process.cwd())
  
  return {
    define: {
      'import.meta.env.VITE_API_URL': JSON.stringify(env.VITE_API_URL)
    },
    server: {
      port: env.VITE_PORT || 3000
    }
  }
})
```

### 3. 多页面应用配置

```javascript
export default defineConfig({
  build: {
    rollupOptions: {
      input: {
        main: path.resolve(__dirname, 'index.html'),
        admin: path.resolve(__dirname, 'admin.html'),
        login: path.resolve(__dirname, 'login.html')
      }
    }
  }
})
```

### 4. 库模式完整配置

```javascript
export default defineConfig({
  build: {
    lib: {
      entry: path.resolve(__dirname, 'src/index.js'),
      name: 'MyLibrary',
      fileName: (format) => `my-library.${format}.js`,
      formats: ['es', 'umd', 'cjs']
    },
    rollupOptions: {
      external: ['vue'],
      output: {
        globals: {
          vue: 'Vue'
        },
        // 导出配置
        exports: 'named'
      }
    }
  },
  // 为库模式优化的配置
  css: {
    extract: true // 提取 CSS 到单独文件
  }
})
```

## 实际应用示例

### 1. 完整的生产级配置

```javascript
import { defineConfig, loadEnv } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path'
import Components from 'unplugin-vue-components/vite'
import { ElementPlusResolver } from 'unplugin-vue-components/resolvers'
import AutoImport from 'unplugin-auto-import/vite'
import viteCompression from 'vite-plugin-compression'

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd())
  const isProduction = mode === 'production'
  
  return {
    root: process.cwd(),
    base: env.VITE_BASE_URL || '/',
    mode,
    
    plugins: [
      vue(),
      
      // 自动导入组件
      Components({
        resolvers: [ElementPlusResolver()],
        dts: path.resolve(__dirname, 'src/components.d.ts')
      }),
      
      // 自动导入 API
      AutoImport({
        imports: ['vue', 'vue-router', 'pinia'],
        dts: path.resolve(__dirname, 'src/auto-imports.d.ts'),
        eslintrc: {
          enabled: true,
          filepath: path.resolve(__dirname, '.eslintrc-auto.json')
        }
      }),
      
      // 生产环境压缩
      isProduction ? viteCompression({
        verbose: true,
        disable: false,
        threshold: 10240, // 10KB
        algorithm: 'gzip',
        ext: '.gz'
      }) : null,
      
      isProduction ? viteCompression({
        verbose: true,
        disable: false,
        threshold: 10240,
        algorithm: 'brotliCompress',
        ext: '.br'
      }) : null
    ].filter(Boolean),
    
    resolve: {
      alias: {
        '@': path.resolve(__dirname, './src'),
        '@components': path.resolve(__dirname, './src/components'),
        '@assets': path.resolve(__dirname, './src/assets'),
        '@utils': path.resolve(__dirname, './src/utils'),
        '@api': path.resolve(__dirname, './src/api'),
        '@styles': path.resolve(__dirname, './src/styles')
      },
      extensions: ['.js', '.jsx', '.json', '.vue', '.ts', '.tsx']
    },
    
    css: {
      preprocessorOptions: {
        scss: {
          additionalData: `@import "@/styles/variables.scss";`
        }
      },
      devSourcemap: !isProduction
    },
    
    server: {
      port: env.VITE_PORT || 3000,
      host: '0.0.0.0',
      open: true,
      cors: true,
      proxy: {
        '/api': {
          target: env.VITE_API_BASE_URL || 'http://localhost:8080',
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/api/, '')
        }
      },
      hmr: {
        overlay: true
      },
      fs: {
        strict: true
      }
    },
    
    build: {
      outDir: 'dist',
      assetsDir: 'assets',
      sourcemap: isProduction ? false : true,
      minify: isProduction ? 'terser' : false,
      chunkSizeWarningLimit: 1000,
      cssCodeSplit: true,
      
      terserOptions: isProduction ? {
        compress: {
          drop_console: true,
          drop_debugger: true,
          pure_funcs: ['console.log', 'console.info', 'console.debug']
        },
        format: {
          comments: false
        }
      } : {},
      
      rollupOptions: {
        output: {
          chunkFileNames: 'js/[name]-[hash].js',
          entryFileNames: 'js/[name]-[hash].js',
          assetFileNames: '[ext]/[name]-[hash].[ext]',
          
          manualChunks: {
            'vue-vendor': ['vue', 'vue-router', 'pinia'],
            'ui-vendor': ['element-plus'],
            'utils': ['axios', 'lodash-es', 'dayjs']
          }
        }
      },
      
      reportCompressedSize: !isProduction
    },
    
    optimizeDeps: {
      include: [
        'vue',
        'vue-router',
        'pinia',
        'axios',
        'element-plus'
      ],
      esbuildOptions: {
        target: 'es2020'
      }
    },
    
    define: {
      __APP_VERSION__: JSON.stringify(process.env.npm_package_version || '1.0.0'),
      __BUILD_TIME__: JSON.stringify(new Date().toISOString())
    }
  }
})
```

### 2. 开发环境专用配置

```javascript
export default defineConfig({
  server: {
    port: 3000,
    host: '0.0.0.0',
    open: true,
    cors: true,
    proxy: {
      '/api': {
        target: 'http://localhost:8080',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
        configure: (proxy, options) => {
          proxy.on('error', (err, req, res) => {
            console.log('proxy error', err)
          })
          proxy.on('proxyReq', (proxyReq, req, res) => {
            console.log('Sending Request to the Target:', req.method, req.url)
          })
        }
      }
    },
    hmr: {
      overlay: true,
      port: 24678
    },
    watch: {
      usePolling: true,
      interval: 100
    }
  },
  
  css: {
    devSourcemap: true
  },
  
  clearScreen: false
})
```

## 注意事项

### 1. 配置优先级

- 命令行参数 > 环境变量 > 配置文件 > 默认值
- 不同模式的配置会覆盖基础配置

### 2. 路径处理

- 使用 `path.resolve()` 确保路径正确
- Windows 系统路径分隔符问题
- 相对路径 vs 绝对路径

### 3. 性能考虑

- 合理设置 `chunkSizeWarningLimit`
- 优化 `optimizeDeps` 配置
- 避免过度配置影响开发体验

### 4. 类型安全

使用 TypeScript 获得完整的类型提示：

```typescript
import { defineConfig } from 'vite'
import type { UserConfig, ConfigEnv } from 'vite'

export default defineConfig((config: ConfigEnv): UserConfig => {
  return {
    // 配置选项会有类型检查
    server: {
      port: 3000
    }
  }
})
```

## 最佳实践

### 1. 配置组织

- 按功能分组配置选项
- 添加注释说明配置用途
- 使用函数进行条件配置

### 2. 环境管理

- 合理使用环境变量
- 区分开发/测试/生产配置
- 敏感信息不要提交到版本控制

### 3. 性能优化

- 按需开启优化选项
- 合理配置代码分割
- 优化依赖预构建

### 4. 类型安全

- 优先使用 TypeScript 配置
- 使用 `defineConfig` 获得类型提示
- 定义配置接口增强可维护性

## 总结

Vite 配置系统提供了强大而灵活的配置能力，通过合理配置可以实现：

- **开发体验优化**：快速启动、热更新、代理配置
- **构建性能提升**：代码分割、依赖优化、压缩配置
- **项目结构定制**：路径别名、多页面、库模式
- **生态系统集成**：插件系统、预处理器、工具链

掌握 Vite 配置是使用 Vite 的关键，合理的配置能够显著提升开发效率和应用性能。建议根据项目需求逐步完善配置，避免过度配置影响开发体验。