---
title: pnpm workspace
published: 2024-11-18
description: 'pnpm workspace 配置的详细介绍和学习笔记'
image: ''
tags: ["pnpm","Monorepo"]
category: '前端工具'
draft: false
lang: 'zh-CN'
---

## 概述

pnpm workspace 是 pnpm 提供的 Monorepo 解决方案，它允许在单个仓库中管理多个相关的包。与传统的 npm/yarn workspaces 相比，pnpm workspace 具有更好的性能、更小的磁盘占用和更严格的依赖管理。它特别适合大型前端项目、组件库、微前端架构等场景。

pnpm workspace 的核心优势包括：
- **高效的磁盘空间利用**：通过硬链接机制共享依赖
- **严格的依赖隔离**：避免幽灵依赖问题
- **快速安装速度**：比 npm/yarn 快 2-3 倍
- **原生支持 Monorepo**：无需额外工具即可管理多个包

## 核心概念

### Workspace 协议

pnpm 使用特殊的 workspace 协议来引用 workspace 内的其他包：

```json
{
  "name": "my-monorepo",
  "dependencies": {
    "packages-1": "workspace:*",
    "packages-2": "workspace:^1.0.0"
  }
}
```

- `workspace:*` - 使用 workspace 中的最新版本
- `workspace:^1.0.0` - 使用 workspace 中满足 1.0.0 版本的包
- `workspace:~` - 使用 workspace 中的兼容版本

### pnpm-workspace.yaml

workspace 配置文件，定义了包含哪些包：

```yaml
packages:
  - 'packages/*'
  - 'apps/*'
  - 'tools/*'
```

### 依赖提升

pnpm workspace 中的依赖提升策略：
- 根目录的 `node_modules` 包含所有 workspace 包
- 每个 workspace 包都有自己的 `node_modules`
- 依赖通过符号链接到全局存储

## 基本用法

### 初始化 Workspace

```bash
# 创建根目录
mkdir my-monorepo
cd my-monorepo

# 初始化 pnpm
pnpm init

# 创建 pnpm-workspace.yaml
cat > pnpm-workspace.yaml << 'EOF'
packages:
  - 'packages/*'
EOF

# 创建示例包
mkdir packages
cd packages
mkdir package-a package-b

# 初始化包
cd package-a
pnpm init

cd ../package-b
pnpm init
```

### 安装依赖

```bash
# 在根目录安装公共依赖
pnpm add -D -w typescript @types/node

# 在特定包中安装依赖
pnpm --filter package-a add lodash

# 在所有包中安装依赖
pnpm add -D jest --filter './packages/*'

# 安装 workspace 内部的依赖
pnpm --filter package-b add package-a
```

### 执行脚本

```bash
# 在所有包中执行脚本
pnpm -r run build

# 在特定包中执行脚本
pnpm --filter package-a run dev

# 并行执行脚本
pnpm -r --parallel run test

# 按拓扑顺序执行脚本（先执行依赖的包）
pnpm -r --workspace-concurrency=1 run build
```

### 过滤和筛选

```bash
# 按包名筛选
pnpm --filter package-a add react

# 按目录模式筛选
pnpm --filter './packages/*' run build

# 按依赖关系筛选
pnpm --filter '...package-a' run build  # 执行依赖 package-a 的所有包

# 排除某些包
pnpm --filter '!package-a' -r run build
```

## 实际应用

### 完整的 Monorepo 项目结构

```
my-monorepo/
├── pnpm-workspace.yaml
├── package.json
├── tsconfig.json
├── packages/
│   ├── ui-components/     # 组件库
│   ├── utils/            # 工具函数库
│   └── config/           # 共享配置
├── apps/
│   ├── web/              # Web 应用
│   └── admin/            # 管理后台
└── tools/
    └── scripts/          # 构建脚本
```

### 组件库示例

**packages/ui-components/package.json:**

```json
{
  "name": "@my-monorepo/ui-components",
  "version": "1.0.0",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.mjs",
      "require": "./dist/index.js"
    },
    "./components": "./dist/components/index.js"
  },
  "scripts": {
    "build": "vite build",
    "dev": "vite build --watch",
    "type-check": "tsc --noEmit"
  },
  "peerDependencies": {
    "react": ">=16.8.0",
    "react-dom": ">=16.8.0"
  }
}
```

**packages/utils/package.json:**

```json
{
  "name": "@my-monorepo/utils",
  "version": "1.0.0",
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "exports": {
    ".": "./src/index.ts",
    "./date": "./src/date.ts"
  }
}
```

**apps/web/package.json:**

```json
{
  "name": "@my-monorepo/web",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "@my-monorepo/ui-components": "workspace:*",
    "@my-monorepo/utils": "workspace:*"
  }
}
```

### 根目录 package.json

```json
{
  "name": "my-monorepo",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "pnpm --filter @my-monorepo/web dev",
    "build": "pnpm -r run build",
    "build:packages": "pnpm --filter './packages/*' run build",
    "build:apps": "pnpm --filter './apps/*' run build",
    "lint": "pnpm -r run lint",
    "test": "pnpm -r run test",
    "clean": "pnpm -r run clean && rm -rf node_modules"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "vite": "^5.0.0",
    "eslint": "^8.50.0",
    "prettier": "^3.0.0"
  },
  "engines": {
    "node": ">=18.0.0",
    "pnpm": ">=8.0.0"
  }
}
```

### 共享 TypeScript 配置

**tsconfig.base.json:**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "allowJs": true,
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  }
}
```

**packages/ui-components/tsconfig.json:**

```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "composite": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "references": [
    { "path": "../utils" }
  ]
}
```

## 高级用法

### 多级 Workspace

```yaml
packages:
  - 'apps/*'
  - 'packages/**'  # 匹配多级目录
```

### 工作区配置选项

**.npmrc:**

```ini
# 设置工作区内部包的链接方式
link-workspace-packages=true

# 设置工作区内部包的版本策略
auto-install-peers=true

# 禁用严格对等依赖
strict-peer-dependencies=false

# 设置存储目录
shamefully-hoist=false
```

### 版本管理

```bash
# 更新 workspace 内部依赖版本
pnpm update @my-monorepo/ui-components --filter @my-monorepo/web

# 发布到 npm
pnpm -r publish --access public

# 使用 changeset 管理版本
pnpm add -Dw @changesets/cli
pnpm changeset init
```

### 依赖分析

```bash
# 查看依赖关系
pnpm list --depth=0

# 查看特定包的依赖
pnpm list --filter @my-monorepo/web

# 检查过期的依赖
pnpm outdated

# 分析包的大小
pnpm why lodash
```

## 注意事项

### 1. 循环依赖处理

pnpm workspace 不允许循环依赖，需要重新设计包结构：

```bash
# 检测循环依赖
pnpm --filter package-a list --depth=Infinity

# 解决方案：提取公共依赖到独立包
```

### 2. 符号链接问题

某些工具可能不识别符号链接，需要特殊配置：

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  resolve: {
    preserveSymlinks: false  // 重要配置
  }
})
```

### 3. Node 版本一致性

确保所有开发者使用相同的 Node 版本：

```json
{
  "engines": {
    "node": ">=18.0.0",
    "pnpm": ">=8.0.0"
  }
}
```

### 4. 缓存清理

遇到奇怪问题时，清理缓存：

```bash
# 清理 pnpm 缓存
pnpm store prune

# 删除 node_modules
pnpm -r exec rm -rf node_modules
rm -rf node_modules

# 重新安装
pnpm install
```

### 5. CI/CD 配置

**.github/workflows/ci.yml:**

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: pnpm/action-setup@v2
        with:
          version: 8
      
      - uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'pnpm'
      
      - run: pnpm install --frozen-lockfile
      - run: pnpm -r run type-check
      - run: pnpm -r run lint
      - run: pnpm -r run test
      - run: pnpm run build
```

## 最佳实践

### 1. 包结构设计

```
packages/
├── shared/              # 共享工具和类型
├── config/             # 共享配置
├── ui/                 # UI 组件库
├── forms/              # 表单组件
└── utils/              # 工具函数
```

### 2. 依赖管理原则

- 最小化外部依赖
- 优先使用 workspace 内部包
- 明确区分 dependencies 和 devDependencies
- 使用 peerDependencies 定义 React 等运行时依赖

### 3. 发布策略

```bash
# 使用 changeset 管理版本
pnpm changeset

# 预览变更
pnpm changeset status

# 版本升级
pnpm changeset version

# 发布
pnpm changeset publish
```

### 4. 开发体验优化

**根目录 package.json:**

```json
{
  "scripts": {
    "dev:web": "pnpm --filter @my-monorepo/web dev",
    "dev:admin": "pnpm --filter @my-monorepo/admin dev",
    "new:package": "plop package",
    "changeset": "changeset",
    "version-packages": "changeset version",
    "release": "pnpm build && changeset publish"
  }
}
```

## 总结

pnpm workspace 是一个强大且高效的 Monorepo 解决方案，特别适合现代前端项目的开发。通过本文的学习，你应该能够：

1. 理解 pnpm workspace 的核心概念和工作原理
2. 熟练配置和使用 pnpm workspace
3. 设计合理的包结构和依赖关系
4. 处理常见的开发问题和场景
5. 遵循最佳实践进行 Monorepo 开发

pnpm workspace 的优势在于其性能、严格的依赖管理和原生支持，这些特性使其成为 Monorepo 开发的理想选择。在实际项目中，结合 TypeScript、Vite 等工具，可以构建出高效、可维护的大型前端项目。

## 参考资料

- [pnpm 官方文档](https://pnpm.io/workspaces)
- [pnpm workspace 配置](https://pnpm.io/pnpm-workspace_yaml)
- [Monorepo 工具对比](https://monorepo.tools/)
- [TurboRepo 文档](https://turbo.build/repo/docs)