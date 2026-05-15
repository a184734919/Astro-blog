---
title: Prettier 代码格式化
published: 2024-08-25
description: '代码格式化配置的详细介绍和学习笔记'
image: ''
tags: ["Prettier","工具"]
category: '前端工具'
draft: false
lang: 'zh-CN'
---

## 概述

Prettier 是一个零配置、多语言代码格式化工具，能够统一代码风格、消除格式争议。它通过强一致性策略输出可预测的格式，提升团队协作效率。

## 核心概念

- 零配置：开箱即用，少量配置即可满足大部分场景。
- 一致性：相同输入始终输出相同格式。
- 集成性：可与编辑器、Git Hooks、CI/CD 深度集成。
- 忽略文件：通过 .prettierignore 排除无需格式化的文件。
- 覆盖规则：支持注释覆盖与配置覆盖。
- 语言支持：JavaScript、TypeScript、JSX、TSX、CSS、Less、SCSS、JSON、Markdown 等。

## 基本用法

### 命令行用法

```bash
# 检查格式
npx prettier --check "**/*.{js,jsx,ts,tsx,json,css,md}"

# 格式化并写入
npx prettier --write "**/*.{js,jsx,ts,tsx,json,css,md}"

# 指定配置文件
npx prettier --write . --config .prettierrc

# 使用缓存加速
npx prettier --write . --cache
```

### 配置文件示例（.prettierrc 或 package.json）

```json
{
  "semi": false,
  "singleQuote": true,
  "tabWidth": 2,
  "useTabs": false,
  "trailingComma": "es5",
  "printWidth": 80,
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf",
  "htmlWhitespaceSensitivity": "css",
  "jsxSingleQuote": true,
  "quoteProps": "as-needed",
  "overrides": [
    {
      "files": "*.css",
      "options": {
        "tabWidth": 4
      }
    }
  ]
}
```

### .prettierignore 示例

```
dist
build
node_modules
public
package-lock.json
yarn.lock
pnpm-lock.yaml
CHANGELOG.md
```

### 注释覆盖（内联）

```js
// prettier-ignore
const messy =  {a:1,b:2};

const formatted = {
  a: 1,
  b: 2,
};
```

## 实际应用

### 与 ESLint 集成（使用 eslint-config-prettier）

```bash
npm install -D eslint-config-prettier
```

```js
// .eslintrc.yml
extends:
  - eslint:recommended
  - plugin:@typescript-eslint/recommended
  - prettier # 将 Prettier 规则置后，关闭冲突规则
```

```json
// package.json
{
  "scripts": {
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "lint": "eslint . --ext .js,.jsx,.ts,.tsx",
    "fix": "npm run format && npm run lint -- --fix"
  }
}
```

### 与 Git Hooks 集成（lint-staged + Husky）

```bash
npm install -D husky lint-staged
npx husky init
```

```json
// package.json
{
  "lint-staged": {
    "*.{js,jsx,ts,tsx,json,css,md}": [
      "prettier --write"
    ],
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix"
    ]
  }
}
```

```bash
# .husky/pre-commit
npx lint-staged
```

### 与 CI/CD 集成（GitHub Actions 示例）

```yaml
# .github/workflows/format.yml
name: Format Check
on: [push, pull_request]
jobs:
  prettier:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run format:check
```

### 编辑器集成

- VS Code：安装 Prettier 扩展，设置 formatOnSave 与 defaultFormatter。
- JetBrains WebStorm：启用 Prettier 插件并指定配置路径。

```json
// .vscode/settings.json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

## 注意事项

1. 与 ESLint 冲突：使用 eslint-config-prettier 并确保 extends 顺序正确。
2. 忽略文件：合理配置 .prettierignore，避免处理构建产物与依赖。
3. 缓存与性能：大型项目使用 --cache 加速检查。
4. 配置一致性：团队共享配置文件，避免差异。
5. 版本兼容：确保编辑器、命令行与 CI 使用相同版本。
6. 覆盖规则：少量使用注释覆盖，保持整体一致性。

## 总结

Prettier 通过零配置与强一致性策略，大幅降低代码格式维护成本。与 ESLint、Git Hooks、CI/CD 及编辑器的深度集成，可打造全自动的格式化工作流。在团队协作中，统一 Prettier 配置并结合自动化检查，是提升代码质量与开发效率的最佳实践。