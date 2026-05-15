---
title: pnpm 包管理器
published: 2024-10-06
description: 'pnpm 的优势和使用的详细介绍和学习笔记'
image: ''
tags: ["pnpm","工具"]
category: '前端工具'
draft: false
lang: 'zh-CN'
---

## 概述

pnpm（Performant npm）是一个快速、节省磁盘空间的包管理器，由 Zoltan Kochan 开发。它采用独特的符号链接机制来管理依赖，相比传统的 npm 和 yarn，能够显著减少磁盘占用、提高安装速度，并解决幽灵依赖问题。

pnpm 的核心优势：
- **高效利用磁盘空间**：通过硬链接机制共享依赖，节省大量磁盘空间
- **快速安装速度**：比 npm 快 2-3 倍，比 yarn 快 2 倍
- **严格的依赖管理**：每个包都有独立的 node_modules，避免幽灵依赖
- **兼容性好**：完全兼容 npm 的包管理和命令
- **原生支持 Monorepo**：无需额外工具即可支持多包管理

## 核心概念

### 内容寻址存储

pnpm 使用内容寻址存储（Content-Addressable Storage）来管理包：

```
~/.pnpm-store/
├── v3/
│   ├── files/00/1a2b3c4d5e6f...
│   ├── files/00/2b3c4d5e6f7a...
│   └── index/             # 包索引
```

每个包按内容哈希存储，相同的包只存储一份，通过硬链接引用。

### 符号链接机制

pnpm 的依赖结构：

```
node_modules/
├── .pnpm/
│   ├── react@18.2.0/
│   │   └── node_modules/react/
│   ├── react-dom@18.2.0/
│   │   └── node_modules/react-dom/
│   └── lodash@4.17.21/
│       └── node_modules/lodash/
├── react -> .pnpm/react@18.2.0/node_modules/react
├── react-dom -> .pnpm/react-dom@18.2.0/node_modules/react-dom
└── lodash -> .pnpm/lodash@4.17.21/node_modules/lodash
```

### 幽灵依赖问题

**npm/yarn 的幽灵依赖问题：**

```javascript
// package.json
{
  "dependencies": {
    "package-a": "1.0.0"
  }
}

// package-a 的依赖
{
  "dependencies": {
    "lodash": "4.17.21"
  }
}

// 在 npm/yarn 中可以直接使用 lodash（幽灵依赖）
import _ from 'lodash'  // ⚠️ 可能导致问题
```

**pnpm 的严格依赖管理：**

```javascript
// pnpm 中必须显式声明依赖
import _ from 'lodash'  // ✅ 必须在 package.json 中声明
```

## 基本用法

### 安装 pnpm

```bash
# 使用 npm 安装
npm install -g pnpm

# 使用 Homebrew 安装（macOS）
brew install pnpm

# 使用 curl 安装
curl -fsSL https://get.pnpm.io/install.sh | sh -

# 验证安装
pnpm --version
```

### 项目初始化

```bash
# 创建新项目
pnpm create vite my-app

# 进入项目目录
cd my-app

# 安装依赖
pnpm install

# 或者简写
pnpm i
```

### 安装依赖

```bash
# 安装生产依赖
pnpm add react react-dom

# 安装开发依赖
pnpm add -D typescript @types/react

# 安装精确版本
pnpm add lodash@4.17.21

# 安装多个依赖
pnpm add react react-dom @types/react

# 从 GitHub 安装
pnpm add username/repo

# 从 Git URL 安装
pnpm add git+https://github.com/username/repo.git

# 安装本地包
pnpm add ../local-package

# 安装同版本范围
pnpm add -F "./packages/*" react  # 在所有子包中安装相同版本
```

### 卸载依赖

```bash
# 卸载生产依赖
pnpm remove lodash

# 卸载开发依赖
pnpm remove -D typescript

# 卸载多个依赖
pnpm remove react react-dom
```

### 更新依赖

```bash
# 更新所有依赖到最新版本
pnpm update

# 更新特定依赖
pnpm update react

# 交互式更新
pnpm update -i

# 更新到最新主要版本
pnpm update react@latest

# 检查过期的依赖
pnpm outdated

# 更新到符合 package.json 版本范围
pnpm update --latest
```

### 查看依赖

```bash
# 查看所有依赖
pnpm list

# 查看依赖树
pnpm list --depth=0

# 查看特定包的依赖
pnpm list react

# 查看全局包
pnpm list -g

# 分析依赖大小
pnpm why lodash
```

## 实际应用

### 项目配置

**.npmrc 文件配置：**

```ini
# 设置 pnpm 版本
package-manager-strict=true

# 自动安装对等依赖
auto-install-peers=true

# 严格对等依赖模式
strict-peer-dependencies=true

# 是否提升所有依赖到根目录
shamefully-hoist=false

# 公共 hoist 模式
public-hoist-pattern[]=*eslint*
public-hoist-pattern[]=*prettier*

# 忽略开发脚本
ignore-scripts=false

# 设置 registry
registry=https://registry.npmmirror.com

# 设置存储位置
store-dir=~/.pnpm-store

# 禁用更新脚本
engine-strict=true
```

### package.json 最佳实践

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext .ts,.tsx",
    "format": "prettier --write .",
    "type-check": "tsc --noEmit"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "typescript": "^5.0.0",
    "vite": "^5.0.0",
    "eslint": "^8.50.0",
    "prettier": "^3.0.0"
  },
  "engines": {
    "node": ">=18.0.0",
    "pnpm": ">=8.0.0"
  },
  "packageManager": "pnpm@8.15.0"
}
```

### 锁文件管理

```bash
# 生成锁文件
pnpm install

# 更新锁文件（不安装）
pnpm install --lockfile-only

# 使用锁文件安装
pnpm install --frozen-lockfile

# 验证锁文件
pnpm install --prefer-frozen-lockfile
```

### 环境变量

```bash
# 设置 pnpm 存储位置
export PNPM_STORE_DIR=~/.pnpm-store

# 设置缓存目录
export PNPM_CACHE_DIR=~/.pnpm-cache

# 设置配置文件位置
export PNPM_CONFIG_FILE=~/.npmrc

# 设置注册表
export PNPM_REGISTRY=https://registry.npmmirror.com
```

## 高级用法

### 离线模式

```bash
# 使用缓存的包进行离线安装
pnpm install --offline

# 检查离线可用性
pnpm install --prefer-offline
```

### 脚本管理

```bash
# 运行脚本
pnpm run dev
pnpm dev  # 简写

# 传递参数
pnpm run build -- --mode production

# 并行运行多个脚本
pnpm run -r --parallel test

# 按拓扑顺序运行（先执行依赖）
pnpm run -r --workspace-concurrency=1 build
```

### 过滤和选择

```bash
# 在特定包中运行命令
pnpm --filter package-name add react

# 在匹配模式的包中运行命令
pnpm --filter "./packages/*" run build

# 排除特定包
pnpm --filter "!package-a" run test

# 在依赖当前包的所有包中运行
pnpm --filter "...package-name" run build
```

### 覆盖依赖

**package.json:**

```json
{
  "pnpm": {
    "overrides": {
      "react": "18.2.0",
      "react-dom": "18.2.0"
    }
  }
}
```

### 快速启动本地包

```bash
# 在开发模式链接本地包
pnpm link --global

# 在其他项目中使用
pnpm link --global package-name

# 取消链接
pnpm unlink --global package-name
```

### 脚本钩子

```json
{
  "scripts": {
    "preinstall": "node ./scripts/check-node-version.js",
    "postinstall": "husky install",
    "prebuild": "pnpm run type-check",
    "build": "vite build",
    "postbuild": "node ./scripts/copy-assets.js"
  }
}
```

## 性能优化

### 并发安装

```bash
# 设置并发数
pnpm install --force --network-concurrency=16

# 设置工作区并发
pnpm run -r --workspace-concurrency=4 build
```

### 缓存管理

```bash
# 查看缓存内容
pnpm store status

# 清理旧缓存
pnpm store prune

# 完全清理存储
pnpm store prune --force

# 添加到 .gitignore
node_modules
.pnpm-store
```

### 安装选项

```bash
# 仅安装生产依赖
pnpm install --prod

# 仅安装开发依赖
pnpm install --dev

# 强制重新下载
pnpm install --force

# 忽略脚本
pnpm install --ignore-scripts

# 严格模式（严格对等依赖）
pnpm install --strict-peer-dependencies
```

## 常见问题解决

### 1. 符号链接问题

某些工具不支持符号链接，需要特殊配置：

```javascript
// webpack.config.js
module.exports = {
  resolve: {
    symlinks: false
  }
}

// vite.config.js
export default {
  resolve: {
    preserveSymlinks: false
  }
}

// jest.config.js
module.exports = {
  resolver: require.resolve('jest-pnpm-resolver')
}
```

### 2. 依赖冲突

```bash
# 查看冲突的依赖
pnpm why package-name

# 使用覆盖解决冲突
pnpm add package-name@version --force
```

### 3. 缓存问题

```bash
# 清理缓存并重新安装
pnpm store prune
rm -rf node_modules
pnpm install
```

### 4. 权限问题

```bash
# 设置正确的权限
chmod +x ~/.pnpm-store
chmod +x node_modules
```

## 注意事项

### 1. 避免 Ghost Dependencies

```javascript
// ❌ 错误：未声明依赖但直接使用
import { debounce } from 'lodash';

// ✅ 正确：显式声明依赖
import { debounce } from 'lodash';
// package.json 中必须有 "lodash": "^4.17.21"
```

### 2. 版本一致性

```bash
# 在所有子包中安装相同版本
pnpm add -F "./packages/*" react@18.2.0

# 使用 overrides 确保版本一致
{
  "pnpm": {
    "overrides": {
      "react": "18.2.0"
    }
  }
}
```

### 3. 存储空间管理

```bash
# 定期清理存储
pnpm store prune

# 检查存储使用情况
du -sh ~/.pnpm-store
```

### 4. CI/CD 配置

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
      - run: pnpm run build
      - run: pnpm test
```

## 最佳实践

### 1. 使用 .npmrc 配置

```ini
# .npmrc
auto-install-peers=true
strict-peer-dependencies=true
shamefully-hoist=false
registry=https://registry.npmmirror.com
```

### 2. 锁文件管理

```bash
# 提交 pnpm-lock.yaml 到版本控制
git add pnpm-lock.yaml

# 在 CI 中使用 --frozen-lockfile
pnpm install --frozen-lockfile
```

### 3. 依赖分类

```json
{
  "dependencies": {
    // 运行时依赖
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    // 开发工具
    "typescript": "^5.0.0",
    "vite": "^5.0.0",
    "eslint": "^8.50.0"
  },
  "peerDependencies": {
    // 对等依赖
    "react": ">=16.8.0"
  },
  "optionalDependencies": {
    // 可选依赖
    "sharp": "^0.32.0"
  }
}
```

### 4. 监控依赖

```bash
# 定期检查过期依赖
pnpm outdated

# 使用 npm-check-updates
npx npm-check-updates -u
pnpm install
```

## 工具集成

### 与 TypeScript 集成

```json
{
  "compilerOptions": {
    "moduleResolution": "node",
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

### 与 ESLint 集成

```json
{
  "extends": [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended"
  ],
  "settings": {
    "import/resolver": "eslint-import-resolver-typescript"
  }
}
```

### 与 Docker 集成

**Dockerfile:**

```dockerfile
FROM node:18-alpine
WORKDIR /app

# 复制锁文件
COPY pnpm-lock.yaml package.json ./

# 安装 pnpm
RUN npm install -g pnpm

# 安装依赖
RUN pnpm install --frozen-lockfile

# 复制源代码
COPY . .

# 构建应用
RUN pnpm run build

# 运行应用
CMD ["pnpm", "run", "start"]
```

## 总结

pnpm 是一个现代化的包管理器，通过其独特的内容寻址存储和符号链接机制，在性能和磁盘空间使用方面都优于传统的 npm 和 yarn。通过本文的学习，你应该能够：

1. 理解 pnpm 的核心概念和优势
2. 熟练使用 pnpm 进行依赖管理
3. 解决常见的依赖问题
4. 优化 pnpm 的性能
5. 在各种场景下正确使用 pnpm

pnpm 特别适合大型项目、Monorepo 架构和需要严格控制依赖的场景。随着前端生态的发展，pnpm 正在成为越来越多项目的首选包管理器。

## 参考资料

- [pnpm 官方文档](https://pnpm.io/)
- [pnpm 为什么比 npm 快](https://pnpm.io/motivation)
- [pnpm vs npm vs yarn](https://pnpm.io/benchmarks)
- [pnpm FAQ](https://pnpm.io/faq)