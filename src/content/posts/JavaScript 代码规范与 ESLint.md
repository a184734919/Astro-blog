---
title: JavaScript 代码规范与 ESLint
published: 2022-10-28
description: '代码规范和静态检查的详细介绍和学习笔记'
image: ''
tags: ["工具"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

代码规范是团队协作开发的基础，它能提高代码可读性、可维护性，减少潜在 bug。ESLint 是目前最流行的 JavaScript 代码检查工具，本文将详细介绍 ESLint 的配置使用以及 JavaScript 代码规范的最佳实践。

## 核心概念

### 代码规范的重要性

1. **提高可读性**：统一的代码风格让代码更容易理解
2. **减少错误**：静态检查能发现潜在问题
3. **提升效率**：减少代码审查时的风格争议
4. **降低维护成本**：规范的代码更容易维护和扩展
5. **促进团队协作**：统一的标准便于团队成员协作

### ESLint 的核心特性

- **完全可配置**：可以自定义规则和配置
- **插件生态**：丰富的第三方插件支持
- **自动修复**：很多规则支持自动修复
- **框架支持**：支持 React、Vue、TypeScript 等
- **集成方便**：可与各种编辑器和 CI/CD 工具集成

## 基本用法

### ESLint 基础配置

```bash
# 安装 ESLint
npm install eslint --save-dev

# 初始化配置
npx eslint --init

# 或全局安装
npm install -g eslint
```

```javascript
// .eslintrc.js - 基础配置
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true,
    jest: true
  },
  extends: [
    'eslint:recommended'
  ],
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module'
  },
  rules: {
    'indent': ['error', 2],
    'linebreak-style': ['error', 'unix'],
    'quotes': ['error', 'single'],
    'semi': ['error', 'always'],
    'no-console': 'warn',
    'no-unused-vars': 'error',
    'no-undef': 'error'
  }
};
```

### 常用规则配置

```javascript
// .eslintrc.js - 详细规则配置
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true
  },
  extends: [
    'eslint:recommended'
  ],
  parserOptions: {
    ecmaVersion: 'latest',
    sourceType: 'module'
  },
  rules: {
    // 缩进
    'indent': ['error', 2, { SwitchCase: 1 }],

    // 行尾换行符
    'linebreak-style': ['error', 'unix'],

    // 引号风格
    'quotes': ['error', 'single', { avoidEscape: true }],

    // 分号
    'semi': ['error', 'always'],

    // 变量声明
    'no-var': 'error',
    'prefer-const': 'error',
    'no-unused-vars': ['error', { argsIgnorePattern: '^_' }],

    // 函数相关
    'arrow-parens': ['error', 'as-needed'],
    'arrow-spacing': ['error', { before: true, after: true }],
    'no-duplicate-imports': 'error',

    // 对象和数组
    'object-curly-spacing': ['error', 'always'],
    'array-bracket-spacing': ['error', 'never'],
    'comma-dangle': ['error', 'always-multiline'],

    // 空格和换行
    'space-before-blocks': ['error', 'always'],
    'space-before-function-paren': ['error', {
      anonymous: 'always',
      named: 'never',
      asyncArrow: 'always'
    }],
    'space-infix-ops': 'error',
    'keyword-spacing': ['error', { before: true, after: true }],

    // 命名规范
    'camelcase': ['error', { properties: 'never' }],
    'new-cap': ['error', { newIsCap: true }],
    'no-underscore-dangle': ['error', { allowAfterThis: true }],

    // 代码质量
    'no-console': process.env.NODE_ENV === 'production' ? 'error' : 'warn',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'error' : 'warn',
    'no-alert': 'warn',
    'no-eval': 'error',
    'no-implied-eval': 'error',
    'no-multi-spaces': 'error',
    'no-trailing-spaces': 'error',

    // 最佳实践
    'eqeqeq': ['error', 'always'],
    'curly': ['error', 'all'],
    'default-case': 'error',
    'no-else-return': ['error', { allowElseIf: false }],
    'no-empty-function': 'warn',
    'no-loop-func': 'error',
    'no-multiple-empty-lines': ['error', { max: 1, maxEOF: 1 }],
    'no-return-await': 'error'
  }
};
```

### React 项目配置

```javascript
// .eslintrc.js - React 项目配置
module.exports = {
  env: {
    browser: true,
    es2021: true,
    node: true
  },
  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'plugin:jsx-a11y/recommended',
    'plugin:prettier/recommended'
  ],
  parserOptions: {
    ecmaFeatures: {
      jsx: true
    },
    ecmaVersion: 'latest',
    sourceType: 'module'
  },
  plugins: [
    'react',
    'react-hooks',
    'jsx-a11y',
    'prettier'
  ],
  settings: {
    react: {
      version: 'detect'
    }
  },
  rules: {
    // React 相关规则
    'react/prop-types': 'off',
    'react/react-in-jsx-scope': 'off',
    'react/jsx-uses-react': 'off',
    'react/jsx-props-no-spreading': 'off',
    'react/jsx-filename-extension': ['error', { extensions: ['.js', '.jsx'] }],
    'react-hooks/rules-of-hooks': 'error',
    'react-hooks/exhaustive-deps': 'warn',

    // JSX 相关规则
    'react/jsx-tag-spacing': ['error', {
      closingSlash: 'never',
      beforeSelfClosing: 'always',
      afterOpening: 'never'
    }]
  }
};
```

## 实际应用

### 项目级配置文件

```javascript
// .eslintrc.js - 完整项目配置
module.exports = {
  root: true,
  env: {
    browser: true,
    es2021: true,
    node: true,
    jest: true
  },
  extends: [
    'eslint:recommended',
    'plugin:react/recommended',
    'plugin:react-hooks/recommended',
    'plugin:import/errors',
    'plugin:import/warnings',
    'plugin:prettier/recommended'
  ],
  parserOptions: {
    ecmaFeatures: {
      jsx: true
    },
    ecmaVersion: 'latest',
    sourceType: 'module'
  },
  plugins: [
    'react',
    'react-hooks',
    'import',
    'prettier'
  ],
  settings: {
    react: {
      version: 'detect'
    },
    'import/resolver': {
      node: {
        extensions: ['.js', '.jsx', '.json']
      },
      alias: {
        map: [
          ['@', './src'],
          ['@components', './src/components'],
          ['@utils', './src/utils'],
          ['@hooks', './src/hooks'],
          ['@services', './src/services']
        ],
        extensions: ['.js', '.jsx']
      }
    }
  },
  rules: {
    // Prettier 冲突规则关闭
    'prettier/prettier': 'error',

    // Import 相关
    'import/order': ['error', {
      groups: [
        'builtin',
        'external',
        'internal',
        'parent',
        'sibling',
        'index'
      ],
      'newlines-between': 'always',
      alphabetize: {
        order: 'asc',
        caseInsensitive: true
      }
    }],
    'import/no-unresolved': ['error', { commonjs: true }],
    'import/no-duplicates': 'error',

    // 通用规则
    'no-console': process.env.NODE_ENV === 'production' ? 'error' : 'warn',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'error' : 'warn',
    'no-unused-vars': ['error', { argsIgnorePattern: '^_' }],
    'prefer-const': 'error',
    'no-var': 'error'
  },
  overrides: [
    {
      files: ['**/*.test.js', '**/*.test.jsx', '**/*.spec.js', '**/*.spec.jsx'],
      env: {
        jest: true
      },
      rules: {
        'no-unused-vars': 'off',
        'no-console': 'off'
      }
    }
  ]
};
```

### Prettier 集成配置

```javascript
// .prettierrc.js
module.exports = {
  // 单行长度
  printWidth: 100,

  // 缩进空格数
  tabWidth: 2,

  // 使用空格缩进
  useTabs: false,

  // 语句末尾添加分号
  semi: true,

  // 使用单引号
  singleQuote: true,

  // 对象属性引号
  quoteProps: 'as-needed',

  // JSX 使用单引号
  jsxSingleQuote: false,

  // 尾随逗号
  trailingComma: 'es5',

  // 对象花括号内空格
  bracketSpacing: true,

  // JSX 标签右括号换行
  bracketSameLine: false,

  // 箭头函数参数括号
  arrowParens: 'always',

  // 换行符
  endOfLine: 'lf',

  // HTML 空白敏感度
  htmlWhitespaceSensitivity: 'css',

  // Vue 文件缩进
  vueIndentScriptAndStyle: false
};
```

```javascript
// .prettierignore
# 依赖文件
node_modules
dist
build

# 配置文件
package-lock.json
yarn.lock

# 测试覆盖率
coverage

# 其他
*.min.js
*.min.css
.lock
```

### Husky 和 lint-staged 集成

```bash
# 安装 Husky 和 lint-staged
npm install --save-dev husky lint-staged

# 初始化 Husky
npx husky install
npx husky add .husky/pre-commit "npx lint-staged"
```

```javascript
// package.json
{
  "scripts": {
    "lint": "eslint . --ext .js,.jsx",
    "lint:fix": "eslint . --ext .js,.jsx --fix",
    "format": "prettier --write \"**/*.{js,jsx,json,md}\"",
    "format:check": "prettier --check \"**/*.{js,jsx,json,md}\""
  },
  "lint-staged": {
    "*.{js,jsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

### VS Code 集成配置

```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.fixAll.prettier": true
  },
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact"
  ],
  "eslint.options": {
    "extensions": [".js", ".jsx", ".ts", ".tsx"]
  }
}
```

### 自定义规则和插件

```javascript
// 自定义规则示例
module.exports = {
  rules: {
    // 自定义规则：强制函数参数不超过 3 个
    'max-params': ['warn', 3],

    // 自定义规则：强制函数行数不超过 50 行
    'max-lines-per-function': ['warn', {
      max: 50,
      skipBlankLines: true,
      skipComments: true
    }],

    // 自定义规则：强制圈复杂度
    'complexity': ['warn', 10],

    // 自定义规则：禁止特定的全局变量
    'no-restricted-globals': ['error', {
      name: 'event',
      message: 'Use event argument instead of global event variable'
    }]
  }
};
```

```javascript
// 自定义插件示例（custom-plugin.js）
module.exports = {
  rules: {
    'require-author-comment': {
      meta: {
        type: 'suggestion',
        docs: {
          description: 'Require author comment in file header',
          category: 'Best Practices',
          recommended: false
        },
        schema: []
      },
      create(context) {
        const sourceCode = context.getSourceCode();
        const comments = sourceCode.getAllComments();

        return {
          Program() {
            const hasAuthorComment = comments.some(comment =>
              comment.value.toLowerCase().includes('@author')
            );

            if (!hasAuthorComment) {
              context.report({
                node: context.ast.body[0],
                message: 'File should include an author comment'
              });
            }
          }
        };
      }
    }
  }
};
```

### 忽略文件配置

```javascript
// .eslintignore
# 依赖
node_modules
dist
build

# 配置文件
.eslintrc.js
.prettierrc.js

# 测试覆盖率
coverage

# 其他
*.min.js
*.min.css
webpack.config.js
babel.config.js
```

### CI/CD 集成

```yaml
# .github/workflows/lint.yml
name: Lint

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint

      - name: Run Prettier check
        run: npm run format:check
```

## 注意事项

1. **团队一致性**：确保团队成员使用相同的配置和工具
2. **渐进式采用**：不要一次性开启所有规则，逐步提高标准
3. **自动修复**：充分利用 ESLint 的自动修复功能
4. **定期更新**：保持 ESLint 和插件的最新版本
5. **合理配置**：根据项目特点调整规则，不要盲目照搬
6. **文档说明**：为自定义规则编写清晰的文档说明

## 总结

代码规范和 ESLint 的合理使用能够：

- **提高代码质量**：通过静态检查发现潜在问题
- **统一代码风格**：保持代码的一致性和可读性
- **提升团队效率**：减少代码审查时的风格争议
- **自动化检查**：在开发过程中自动检查和修复问题
- **最佳实践**：推广行业最佳实践和编码习惯

良好的代码规范是专业开发的基础，投入时间建立和执行规范，将为团队带来长期的收益。记住，规范的目的不是限制创造力，而是让代码更加清晰、可维护。