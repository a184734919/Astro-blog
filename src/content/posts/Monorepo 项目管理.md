---
title: Monorepo 项目管理
published: 2024-11-04
description: 'Monorepo 架构设计的详细介绍和学习笔记'
image: ''
tags: ["Monorepo","架构"]
category: '前端工具'
draft: false
lang: 'zh-CN'
---

## 概述

Monorepo（Monolithic Repository）是一种将多个相关项目存储在同一个代码仓库中的开发策略。与传统的 Polyrepo（每个项目独立仓库）相比，Monorepo 提供了代码共享、统一管理、原子提交等优势，特别适合大型企业、微前端架构、组件库开发等场景。

Monorepo 的核心优势：
- **代码共享**：轻松共享通用代码、组件和工具
- **原子提交**：跨项目改动可以在一个 PR 中完成
- **统一管理**：统一的版本控制、CI/CD 流程和工具链
- **简化依赖**：内部包之间不需要发布到 npm
- **团队协作**：提高团队协作效率和代码可见性

常见的 Monorepo 工具包括 pnpm workspace、Turborepo、Nx、Lerna、Rush 等。

## 核心概念

### Monorepo vs Polyrepo

| 特性 | Monorepo | Polyrepo |
|------|----------|----------|
| 代码仓库 | 单一仓库 | 多个仓库 |
| 代码共享 | 简单直接 | 需要发布到 npm |
| 跨项目改动 | 原子提交 | 需要 PR 跨仓库 |
| 构建速度 | 需要优化 | 相对较快 |
| 权限控制 | 较粗粒度 | 精细控制 |
| 学习曲线 | 较陡 | 较平缓 |

### Workspace 概念

Workspace 是 Monorepo 中的基本单元，每个 Workspace 都是一个独立的包：

```
my-monorepo/
├── packages/
│   ├── shared/          # 共享代码
│   ├── ui-components/   # UI 组件
│   ├── utils/           # 工具函数
│   └── config/          # 共享配置
├── apps/
│   ├── web/            # Web 应用
│   ├── admin/          # 管理后台
│   └── mobile/         # 移动应用
└── tools/
    └── scripts/        # 构建脚本
```

### 依赖图管理

Monorepo 需要维护包之间的依赖关系：

```
web-app -> ui-components -> shared
         -> utils -> shared
admin-app -> ui-components -> shared
         -> utils -> shared
```

## 基本用法

### 使用 pnpm workspace 创建 Monorepo

```bash
# 1. 创建项目根目录
mkdir my-monorepo && cd my-monorepo

# 2. 初始化 package.json
pnpm init

# 3. 创建 pnpm-workspace.yaml
cat > pnpm-workspace.yaml << 'EOF'
packages:
  - 'packages/*'
  - 'apps/*'
  - 'tools/*'
EOF

# 4. 创建目录结构
mkdir -p packages/{shared,ui-components,utils} \
         apps/{web,admin} \
         tools/scripts

# 5. 初始化各个包
cd packages/shared
pnpm init
echo "# Shared Package" > README.md
echo "export const hello = () => 'Hello World';" > src/index.ts

cd ../ui-components
pnpm init
echo "# UI Components" > README.md

cd ../utils
pnpm init
echo "# Utils Package" > README.md

cd ../../apps/web
pnpm init
echo "# Web App" > README.md

cd ../admin
pnpm init
echo "# Admin App" > README.md

# 6. 在根目录安装公共依赖
cd ../..
pnpm add -D -w typescript vite eslint prettier
```

### 基本目录结构

```
my-monorepo/
├── pnpm-workspace.yaml
├── package.json
├── tsconfig.json
├── .eslintrc.js
├── .prettierrc
├── packages/
│   ├── shared/
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── src/
│   │       └── index.ts
│   ├── ui-components/
│   │   ├── package.json
│   │   └── src/
│   └── utils/
│       ├── package.json
│       └── src/
├── apps/
│   ├── web/
│   │   ├── package.json
│   │   ├── vite.config.ts
│   │   └── src/
│   └── admin/
│       ├── package.json
│       └── src/
└── tools/
    └── scripts/
        └── build.ts
```

### 根目录 package.json

```json
{
  "name": "my-monorepo",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "pnpm --filter @my-monorepo/web dev",
    "dev:admin": "pnpm --filter @my-monorepo/admin dev",
    "build": "pnpm -r run build",
    "build:packages": "pnpm --filter './packages/*' run build",
    "build:apps": "pnpm --filter './apps/*' run build",
    "test": "pnpm -r run test",
    "lint": "pnpm -r run lint",
    "format": "pnpm -r run format",
    "type-check": "pnpm -r run type-check",
    "clean": "pnpm -r run clean && rm -rf node_modules",
    "changeset": "changeset",
    "version-packages": "changeset version",
    "release": "pnpm build && changeset publish"
  },
  "devDependencies": {
    "@changesets/cli": "^2.26.0",
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "eslint": "^8.45.0",
    "prettier": "^3.0.0",
    "typescript": "^5.0.0",
    "vite": "^5.0.0"
  },
  "engines": {
    "node": ">=18.0.0",
    "pnpm": ">=8.0.0"
  }
}
```

## 实际应用

### 完整的项目配置示例

**packages/shared/package.json:**

```json
{
  "name": "@my-monorepo/shared",
  "version": "1.0.0",
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    },
    "./types": "./dist/types/index.d.ts"
  },
  "scripts": {
    "build": "tsup src/index.ts --format cjs,esm --dts",
    "dev": "tsup src/index.ts --format cjs,esm --dts --watch",
    "type-check": "tsc --noEmit",
    "test": "vitest",
    "lint": "eslint src --ext .ts",
    "format": "prettier --write src"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "tsup": "^8.0.0",
    "typescript": "^5.0.0",
    "vitest": "^1.0.0"
  }
}
```

**packages/shared/src/index.ts:**

```typescript
// 类型定义
export interface User {
  id: string;
  name: string;
  email: string;
}

// 常量
export const API_BASE_URL = 'https://api.example.com';

// 工具函数
export const formatDate = (date: Date): string => {
  return date.toISOString().split('T')[0];
};

export const validateEmail = (email: string): boolean => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

// 配置
export const config = {
  appName: 'My App',
  version: '1.0.0',
  environment: process.env.NODE_ENV || 'development'
};
```

**packages/ui-components/package.json:**

```json
{
  "name": "@my-monorepo/ui-components",
  "version": "1.0.0",
  "type": "module",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "types": "./dist/index.d.ts",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs"
    },
    "./button": "./dist/components/Button/index.js",
    "./input": "./dist/components/Input/index.js"
  },
  "scripts": {
    "build": "tsup src/index.ts --format cjs,esm --dts --external react",
    "dev": "tsup src/index.ts --format cjs,esm --dts --external react --watch",
    "type-check": "tsc --noEmit",
    "test": "vitest",
    "lint": "eslint src --ext .ts,.tsx"
  },
  "dependencies": {
    "@my-monorepo/shared": "workspace:*",
    "clsx": "^2.0.0"
  },
  "peerDependencies": {
    "react": ">=18.0.0",
    "react-dom": ">=18.0.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "tsup": "^8.0.0",
    "typescript": "^5.0.0",
    "vitest": "^1.0.0"
  }
}
```

**packages/ui-components/src/components/Button/index.tsx:**

```typescript
import React from 'react';
import { clsx } from 'clsx';

export interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
}

const variantStyles = {
  primary: 'bg-blue-500 hover:bg-blue-600 text-white',
  secondary: 'bg-gray-500 hover:bg-gray-600 text-white',
  danger: 'bg-red-500 hover:bg-red-600 text-white'
};

const sizeStyles = {
  sm: 'px-3 py-1 text-sm',
  md: 'px-4 py-2 text-base',
  lg: 'px-6 py-3 text-lg'
};

export const Button: React.FC<ButtonProps> = ({
  variant = 'primary',
  size = 'md',
  loading = false,
  children,
  disabled,
  className,
  ...props
}) => {
  return (
    <button
      className={clsx(
        'rounded font-medium transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed',
        variantStyles[variant],
        sizeStyles[size],
        className
      )}
      disabled={disabled || loading}
      {...props}
    >
      {loading ? 'Loading...' : children}
    </button>
  );
};
```

**packages/utils/package.json:**

```json
{
  "name": "@my-monorepo/utils",
  "version": "1.0.0",
  "type": "module",
  "main": "./src/index.ts",
  "types": "./src/index.d.ts",
  "exports": {
    ".": "./src/index.ts",
    "./array": "./src/array.ts",
    "./string": "./src/string.ts",
    "./object": "./src/object.ts"
  },
  "dependencies": {
    "@my-monorepo/shared": "workspace:*"
  },
  "scripts": {
    "test": "vitest",
    "lint": "eslint src --ext .ts"
  },
  "devDependencies": {
    "vitest": "^1.0.0"
  }
}
```

**apps/web/package.json:**

```json
{
  "name": "@my-monorepo/web",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "type-check": "tsc --noEmit",
    "lint": "eslint src --ext .ts,.tsx",
    "format": "prettier --write src"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "@my-monorepo/ui-components": "workspace:*",
    "@my-monorepo/utils": "workspace:*",
    "@my-monorepo/shared": "workspace:*"
  },
  "devDependencies": {
    "@types/react": "^18.2.0",
    "@types/react-dom": "^18.2.0",
    "@vitejs/plugin-react": "^4.0.0",
    "typescript": "^5.0.0",
    "vite": "^5.0.0"
  }
}
```

**apps/web/src/App.tsx:**

```typescript
import React from 'react';
import { Button } from '@my-monorepo/ui-components';
import { formatDate, validateEmail } from '@my-monorepo/utils';
import { User, API_BASE_URL } from '@my-monorepo/shared';

function App() {
  const user: User = {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com'
  };

  const handleClick = () => {
    console.log('API:', API_BASE_URL);
    console.log('Email valid:', validateEmail(user.email));
    console.log('Date:', formatDate(new Date()));
  };

  return (
    <div className="min-h-screen bg-gray-100 flex items-center justify-center">
      <div className="bg-white p-8 rounded-lg shadow-lg">
        <h1 className="text-2xl font-bold mb-4">Hello, {user.name}!</h1>
        <p className="text-gray-600 mb-4">{user.email}</p>
        <Button onClick={handleClick} variant="primary">
          Test Functions
        </Button>
      </div>
    </div>
  );
}

export default App;
```

### TypeScript 配置

**tsconfig.base.json:**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "allowJs": true,
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "composite": true,
    "incremental": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true
  }
}
```

**packages/ui-components/tsconfig.json:**

```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src",
    "jsx": "react-jsx"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"],
  "references": [
    { "path": "../shared" }
  ]
}
```

**apps/web/tsconfig.json:**

```json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist",
    "rootDir": "./src",
    "jsx": "react-jsx"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"],
  "references": [
    { "path": "../../packages/ui-components" },
    { "path": "../../packages/utils" },
    { "path": "../../packages/shared" }
  ]
}
```

### ESLint 配置

**.eslintrc.js:**

```javascript
module.exports = {
  root: true,
  parser: '@typescript-eslint/parser',
  parserOptions: {
    ecmaVersion: 2022,
    sourceType: 'module',
    project: './tsconfig.json'
  },
  plugins: ['@typescript-eslint'],
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:@typescript-eslint/recommended-requiring-type-checking'
  ],
  rules: {
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_' }]
  },
  ignorePatterns: ['dist', 'node_modules', '*.config.js']
};
```

## 高级用法

### 使用 Turborepo 优化构建

```bash
# 安装 Turborepo
pnpm add -D -w turbo

# 创建 turbo.json
cat > turbo.json << 'EOF'
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": ["dist/**", ".next/**", "!.next/cache/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "lint": {
      "outputs": []
    },
    "test": {
      "dependsOn": ["build"],
      "outputs": [],
      "inputs": ["src/**/*.tsx", "src/**/*.ts", "test/**/*.ts", "test/**/*.tsx"]
    }
  }
}
EOF

# 使用 turbo 运行命令
pnpm turbo run build
pnpm turbo run lint
pnpm turbo run test
```

### Changesets 版本管理

```bash
# 安装 changesets
pnpm add -D -w @changesets/cli

# 初始化
pnpm changeset init

# 创建 changeset
pnpm changeset

# 预览变更
pnpm changeset status

# 版本升级
pnpm changeset version

# 发布
pnpm changeset publish
```

**.changeset/config.json:**

```json
{
  "$schema": "https://unpkg.com/@changesets/config@3.0.0/schema.json",
  "changelog": "@changesets/cli/changelog",
  "commit": false,
  "fixed": [],
  "linked": [],
  "access": "public",
  "baseBranch": "main",
  "updateInternalDependencies": "patch",
  "ignore": []
}
```

### Husky 和 Git Hooks

```bash
# 安装 husky
pnpm add -D -w husky

# 初始化
pnpm exec husky install

# 添加 pre-commit hook
pnpm exec husky add .husky/pre-commit "pnpm lint-staged"

# 添加 commit-msg hook
pnpm exec husky add .husky/commit-msg 'pnpm exec commitlint --edit $1'
```

**package.json scripts:**

```json
{
  "scripts": {
    "prepare": "husky install",
    "lint-staged": "lint-staged",
    "commitlint": "commitlint"
  },
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

## CI/CD 配置

### GitHub Actions 配置

**.github/workflows/ci.yml:**

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Type check
        run: pnpm turbo run type-check

      - name: Lint
        run: pnpm turbo run lint

      - name: Test
        run: pnpm turbo run test

      - name: Build
        run: pnpm turbo run build

  release:
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'pnpm'
          registry-url: 'https://registry.npmjs.org'

      - name: Install dependencies
        run: pnpm install --frozen-lockfile

      - name: Build
        run: pnpm turbo run build

      - name: Create Release Pull Request or Publish
        id: changesets
        uses: changesets/action@v1
        with:
          publish: pnpm changeset publish
          version: pnpm changeset version
          commit: 'chore: version packages'
          title: 'chore: version packages'
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## 注意事项

### 1. 避免循环依赖

```typescript
// ❌ 错误：循环依赖
// package-a depends on package-b
// package-b depends on package-a

// ✅ 正确：提取公共依赖
// package-a depends on shared
// package-b depends on shared
// shared contains common functionality
```

### 2. 管理包的边界

明确定义每个包的职责：

```
packages/
├── shared/          # 纯类型、常量
├── utils/           # 纯函数、工具类
├── ui-components/   # UI 组件
├── forms/           # 表单组件
└── api/             # API 客户端
```

### 3. 性能优化

```bash
# 使用 Turborepo 缓存
pnpm turbo run build --force

# 并行构建
pnpm -r --parallel run build

# 使用 Docker 多阶段构建
# 优化构建时间
```

### 4. 文档管理

```markdown
# 每个 package 的 README.md
## @my-monorepo/ui-components

UI 组件库，提供可复用的 React 组件。

### 安装
```bash
pnpm add @my-monorepo/ui-components
```

### 使用
```tsx
import { Button } from '@my-monorepo/ui-components';
```

### 组件列表
- Button
- Input
- Modal
```

## 最佳实践

### 1. 包命名规范

```json
{
  "name": "@company-name/package-name",
  "description": "Package description",
  "version": "1.0.0"
}
```

### 2. 版本管理策略

- 使用语义化版本（SemVer）
- 使用 Changesets 管理版本变更
- 保持包之间的版本兼容性

### 3. 测试策略

```typescript
// packages/shared/src/__tests__/utils.test.ts
import { describe, it, expect } from 'vitest';
import { formatDate, validateEmail } from '../index';

describe('utils', () => {
  describe('formatDate', () => {
    it('should format date correctly', () => {
      const date = new Date('2024-01-15');
      expect(formatDate(date)).toBe('2024-01-15');
    });
  });

  describe('validateEmail', () => {
    it('should validate valid email', () => {
      expect(validateEmail('test@example.com')).toBe(true);
    });

    it('should reject invalid email', () => {
      expect(validateEmail('invalid-email')).toBe(false);
    });
  });
});
```

### 4. 构建优化

```typescript
// tsup.config.ts
import { defineConfig } from 'tsup';

export default defineConfig({
  entry: ['src/index.ts'],
  format: ['cjs', 'esm'],
  dts: true,
  splitting: false,
  sourcemap: true,
  clean: true,
  external: ['react', 'react-dom'],
  treeshake: true,
  minify: process.env.NODE_ENV === 'production'
});
```

## 常见问题解决

### 1. 工作区依赖问题

```bash
# 检查依赖关系
pnpm why @my-monorepo/ui-components

# 重新安装
pnpm install --force

# 清理缓存
pnpm store prune
```

### 2. 构建顺序问题

```bash
# 使用 turborepo 管理依赖
pnpm turbo run build

# 手动指定顺序
pnpm --filter @my-monorepo/shared build
pnpm --filter @my-monorepo/ui-components build
pnpm --filter @my-monorepo/web build
```

### 3. 类型检查问题

```json
// tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "incremental": true
  }
}
```

## 工具推荐

### 核心工具
- **pnpm workspace** - 包管理和工作区支持
- **Turborepo** - 构建优化和任务编排
- **Changesets** - 版本管理和发布
- **TypeScript** - 类型安全
- **Vite** - 快速构建工具

### 开发工具
- **ESLint** - 代码检查
- **Prettier** - 代码格式化
- **Husky** - Git hooks
- **lint-staged** - 暂存文件检查
- **Vitest** - 单元测试

### 发布工具
- **semantic-release** - 自动化发布
- **np** - 发布流程优化
- **Changesets** - 变更日志管理

## 总结

Monorepo 架构通过统一管理多个相关项目，提供了代码共享、原子提交、统一管理等优势。通过本文的学习，你应该能够：

1. 理解 Monorepo 的核心概念和优势
2. 使用 pnpm workspace 创建和管理 Monorepo
3. 配置和优化 Monorepo 项目
4. 解决常见的 Monorepo 开发问题
5. 集成 CI/CD 和自动化工具

Monorepo 特别适合大型企业、组件库、微前端等场景。选择合适的工具和架构设计，可以显著提高开发效率和代码质量。

## 参考资料

- [pnpm Workspace 文档](https://pnpm.io/workspaces)
- [Turborepo 文档](https://turbo.build/repo/docs)
- [Changesets 文档](https://github.com/changesets/changesets)
- [Monorepo 最佳实践](https://monorepo.tools/)
- [Nx 文档](https://nx.dev/)