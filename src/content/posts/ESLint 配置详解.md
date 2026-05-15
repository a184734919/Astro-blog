---
title: ESLint 配置详解
published: 2024-08-11
description: '规则配置和插件的详细介绍和学习笔记'
image: ''
tags: ["ESLint","工具"]
category: '前端工具'
draft: false
lang: 'zh-CN'
---

## 概述

ESLint 是一个可配置的 JavaScript/TypeScript/TSX/JSX 代码检查工具，能够发现代码中的潜在错误、不一致风格与安全问题。通过规则（rules）、插件（plugins）、解析器（parser）与共享配置（shareable config），可针对不同项目制定灵活的静态分析策略。

## 核心概念

- parser：将代码转换为 AST。常用：espree（默认）、@babel/eslint-parser、@typescript-eslint/parser。
- rules：单项检测规则，可配置为 'off'（0）、'warn'（1）、'error'（2）或数组形式参数。
- plugins：提供额外规则与处理逻辑，如 eslint-plugin-react、eslint-plugin-import。
- extends：继承预置规则集合，便于统一规范。
- env/globals：环境预设与全局变量声明，避免误报。
- settings：向插件提供项目级上下文，用于更精确分析。
- overrides：针对特定文件或路径进行差异化规则配置。
- ignorePatterns：排除无需检查的文件。

## 基本用法

### 命令行用法

```bash
npx eslint src/**/*.js
npx eslint --fix src/
npx eslint --init  # 交互式初始化
```

### 配置文件示例（ESLint 9+ flat config）

```js
// eslint.config.js
import js from '@eslint/js'
import tseslint from '@typescript-eslint/eslint-plugin'
import tsParser from '@typescript-eslint/parser'
import reactPlugin from 'eslint-plugin-react'
import reactHooks from 'eslint-plugin-react-hooks'
import importPlugin from 'eslint-plugin-import'

export default [
  js.configs.recommended,
  {
    files: ['**/*.{js,jsx,mjs,cjs,ts,tsx}'],
    languageOptions: {
      parser: tsParser,
      parserOptions: { ecmaVersion: 'latest', sourceType: 'module', ecmaFeatures: { jsx: true } },
      globals: { window: 'readonly', document: 'readonly' },
    },
    plugins: {
      '@typescript-eslint': tseslint,
      react: reactPlugin,
      'react-hooks': reactHooks,
      import: importPlugin,
    },
    rules: {
      '@typescript-eslint/no-explicit-any': 'warn',
      '@typescript-eslint/explicit-function-return-type': 'off',
      'react/prop-types': 'off',
      'react-hooks/rules-of-hooks': 'error',
      'react-hooks/exhaustive-deps': 'warn',
      'import/order': ['error', { groups: [['builtin', 'external'], 'internal', 'parent', 'sibling', 'index'] }],
      'no-console': 'warn',
    },
  },
  {
    ignores: ['dist', 'node_modules', 'public'],
  },
]
```

### 配置文件示例（ESLint 8 及以下 eslintrc）

```yaml
# .eslintrc.yml
parser: '@typescript-eslint/parser'
parserOptions:
  ecmaVersion: 2022
  sourceType: module
  ecmaFeatures:
    jsx: true
plugins:
  - '@typescript-eslint'
  - react
  - react-hooks
  - import
extends:
  - eslint:recommended
  - plugin:@typescript-eslint/recommended
  - plugin:react/recommended
  - plugin:react-hooks/recommended
rules:
  '@typescript-eslint/no-explicit-any': warn
  '@typescript-eslint/explicit-function-return-type': off
  react/react-in-jsx-scope: off
  react-hooks/rules-of-hooks: error
  import/order: [error, { groups: [ [builtin, external], internal, parent, sibling, index ] }]
  no-console: warn
ignorePatterns:
  - dist
  - node_modules
  - public
settings:
  react:
    version: detect
```

## 实际应用

### React + TypeScript 项目最佳实践

```js
// 忽略文件示例
export default [
  { ignores: ['dist', 'node_modules', 'build', '.next', 'out'] },
  {
    files: ['**/*.{ts,tsx}'],
    languageOptions: {
      parser: tsParser,
      parserOptions: {
        project: ['./tsconfig.json'],
        tsconfigRootDir: import.meta.dirname,
      },
    },
    plugins: { '@typescript-eslint': tseslint },
    rules: {
      '@typescript-eslint/strict-boolean-expressions': 'error',
      '@typescript-eslint/await-thenable': 'error',
      '@typescript-eslint/no-floating-promises': 'error',
      'no-unused-vars': 'off',
      '@typescript-eslint/no-unused-vars': ['error', { argsIgnorePattern: '^_', varsIgnorePattern: '^_' }],
    },
  },
  {
    files: ['**/*.{test,spec}.{ts,tsx}'],
    rules: {
      '@typescript-eslint/no-explicit-any': 'off',
      '@typescript-eslint/no-non-null-assertion': 'off',
      'no-console': 'off',
    },
  },
]
```

### 与 Git Hooks、Husky 集成（pre-commit 与 commit-msg 校验）

```json
// package.json
{
  "scripts": {
    "lint": "eslint . --ext .js,.jsx,.ts,.tsx --fix",
    "lint:check": "eslint . --ext .js,.jsx,.ts,.tsx"
  }
}
```

```bash
# .husky/pre-commit
npm run lint:check

# .husky/commit-msg
npx commitlint --edit $1
```

### 与 CI/CD 集成（GitHub Actions 示例）

```yaml
# .github/workflows/lint.yml
name: Lint
on: [push, pull_request]
jobs:
  eslint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 20
          cache: npm
      - run: npm ci
      - run: npm run lint:check
```

## 注意事项

1. 规则兼容性：注意插件与 ESLint 版本兼容，特别是 flat config 与旧版 extends 的转换。
2. 性能优化：
   - 使用 ignorePatterns 排除冗余目录。
   - 借助缓存（ESLint 9 可配置 cacheLocation 与 cacheStrategy）。
3. 与 Prettier 避免冲突：
   - 关闭格式化类规则，使用 eslint-config-prettier，确保 Prettier 负责格式、ESLint 负责质量与风格检查。
4. TypeScript 与 parserOptions：启用 parserOptions.project 以获取更精确的类型感知规则。
5. 共享配置与插件来源：采用主流社区插件与配置，保持更新并审核安全来源。
6. 团队协作：将配置纳入版本控制，通过 CI 强制检查与自动修复（--fix）降低沟通成本。

## 总结

掌握 ESLint 配置的关键在于理解 parser、rules、plugins、extends 与 overrides 的协同关系。合理使用类型感知规则、避免与 Prettier 冲突、与 Git Hooks 和 CI/CD 集成，可显著提升代码一致性与可维护性。针对不同项目（如 React、TypeScript、测试文件）使用 overrides 进行差异化配置，是团队协作与规模化项目的最佳实践。