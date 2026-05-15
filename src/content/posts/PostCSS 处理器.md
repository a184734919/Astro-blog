---
title: PostCSS 处理器
published: 2024-09-06
description: 'PostCSS 插件生态的详细介绍和学习笔记'
image: ''
tags: ["PostCSS","样式"]
category: '前端样式'
draft: false
lang: 'zh-CN'
---

## 概述

PostCSS 是一个 CSS 转换器，通过插件生态实现自动前缀、嵌套、变量、模块化、压缩、兼容性降级等能力。它是现代前端构建链（Vite、Webpack、Rspack）中的关键组件。

## 核心概念

- Parser：解析 CSS 为 AST。
- Transformer：插件转换 AST。
- Stringifier：将 AST 转回 CSS 字符串。
- 插件生态：Autoprefixer、PostCSS Nesting、PostCSS Preset Env、CSS Modules、PostCSS Custom Properties。
- Source Maps：生成源码映射。
- 语法支持：支持 CSS、SCSS/Less 混合写法（配合插件）。

## 基本用法

### 命令行用法

```bash
npm install -D postcss postcss-cli autoprefixer cssnano

# 配置 postcss.config.js（见下方）后
npx postcss src/styles.css -o dist/styles.css
```

### 配置文件示例（postcss.config.js）

```js
export default {
  plugins: {
    'autoprefixer': {},
    'postcss-preset-env': {
      stage: 3,
      features: {
        'nesting-rules': true,
        'custom-properties': true,
      },
    },
    'cssnano': {
      preset: 'default',
    },
    'postcss-import': {},
    'postcss-url': {
      url: 'inline',
    },
  },
}
```

### 使用方式：Vite

```js
// vite.config.ts
import { defineConfig } from 'vite'
import postcss from './postcss.config.js'

export default defineConfig({
  css: {
    postcss,
  },
})
```

### 使用方式：Webpack

```js
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'postcss-loader',
            options: {
              postcssOptions: {
                config: './postcss.config.js',
              },
            },
          },
        ],
      },
    ],
  },
}
```

## 实际应用

### 自动前缀（Autoprefixer）

```js
// postcss.config.js
export default {
  plugins: {
    autoprefixer: {
      overrideBrowserslist: ['> 1%', 'last 2 versions', 'not dead'],
    },
  },
}
```

```css
/* 输入 */
.box {
  display: flex;
  backdrop-filter: blur(10px);
}
```

```css
/* 输出 */
.box {
  display: flex;
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
}
```

### 嵌套与变量（PostCSS Preset Env）

```js
export default {
  plugins: {
    'postcss-preset-env': {
      stage: 3,
      features: {
        'nesting-rules': true,
        'custom-properties': true,
      },
    },
  },
}
```

```css
/* 输入 */
:root {
  --primary: #1890ff;
}

.container {
  color: var(--primary);
  .box {
    background: white;
  }
}
```

```css
/* 输出 */
:root {
  --primary: #1890ff;
}

.container {
  color: var(--primary);
}
.container .box {
  background: white;
}
```

### CSS Modules

```js
// postcss.config.js
export default {
  modules: true,
  plugins: {
    'postcss-modules': {
      generateScopedName: '[name]__[local]--[hash:base64:5]',
    },
  },
}
```

### CSS 自定义属性与降级（PostCSS Custom Properties）

```js
export default {
  plugins: {
    'postcss-custom-properties': {
      preserve: true,
      importFrom: './src/theme/variables.css',
    },
  },
}
```

### 压缩（CSSNano）

```js
export default {
  plugins: {
    cssnano: {
      preset: [
        'default',
        {
          discardComments: { removeAll: true },
          normalizeWhitespace: true,
        },
      ],
    },
  },
}
```

### 环境变量注入（PostCSS Simple Variables）

```bash
npm install -D postcss-simple-vars
```

```js
export default {
  plugins: {
    'postcss-simple-vars': {
      variables: {
        primaryColor: '#1890ff',
      },
    },
  },
}
```

```css
/* 输入 */
.box {
  color: $(primaryColor);
}
```

```css
/* 输出 */
.box {
  color: #1890ff;
}
```

## 注意事项

1. 插件顺序：注意插件执行顺序，如 autoprefixer 应在其他转换后执行。
2. 冲突与重复：避免同时使用多个相似功能的插件（如多个前缀插件）。
3. 配置共享：在 monorepo 中共享 postcss.config.js，确保一致性。
4. 性能优化：大型项目启用缓存与并行处理。
5. Source Maps：开发环境确保开启 sourceMaps，生产环境视需关闭以减小体积。
6. 兼容性：根据 target 浏览器调整 preset-env 与 autoprefixer 配置。

## 总结

PostCSS 以插件为核心，提供强大的 CSS 处理能力。与 Autoprefixer、PostCSS Preset Env、CSS Modules、CSSNano 等插件结合，可覆盖现代样式工程需求。在 Vite、Webpack 等构建工具中，合理配置 PostCSS 能显著提升样式开发效率与兼容性，是前端样式工程的基石之一。