---
title: TypeScript 配置详解
published: 2022-11-29
description: 'tsconfig.json 配置项的详细介绍和学习笔记'
image: ''
tags: ["TypeScript"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

`tsconfig.json` 是 TypeScript 项目的核心配置文件,用于指定编译选项、文件包含/排除规则、类型定义等。合理配置能够提高开发效率、代码质量和编译性能。

## 核心概念

TypeScript 编译器通过 `tsc` 命令或 IDE 插件读取配置,根据配置进行类型检查和代码转换。配置分为以下几个类别:

- **编译选项**: 控制编译行为和输出
- **文件包含**: 指定需要编译的文件
- **文件排除**: 指定不需要编译的文件
- **引用项目**: 支持项目引用和 monorepo
- **类型声明**: 配置类型定义文件的查找

## 基本用法

### 基础配置结构

```json
{
  "compilerOptions": {
    // 编译选项
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"],
  "extends": "@tsconfig/recommended"
}
```

### 常用编译选项

#### 1. 基础选项

```json
{
  "compilerOptions": {
    "target": "ES2020",              // 编译目标 ECMAScript 版本
    "module": "ESNext",              // 模块系统
    "lib": ["ES2020", "DOM"],        // 包含的类型定义库
    "jsx": "react",                  // JSX 处理方式
    "outDir": "./dist",              // 输出目录
    "rootDir": "./src",              // 源代码根目录
    "removeComments": true,          // 删除注释
    "sourceMap": true,               // 生成 source map
    "declaration": true,             // 生成 .d.ts 声明文件
    "declarationMap": true           // 生成声明文件的 source map
  }
}
```

#### 2. 类型检查选项

```json
{
  "compilerOptions": {
    "strict": true,                  // 启用所有严格类型检查选项
    "noImplicitAny": true,           // 禁止隐式 any 类型
    "strictNullChecks": true,        // 严格空值检查
    "strictFunctionTypes": true,     // 严格函数类型检查
    "strictPropertyInitialization": true,  // 严格属性初始化检查
    "noImplicitThis": true,          // 禁止 this 隐式 any 类型
    "noUnusedLocals": true,          // 检查未使用的局部变量
    "noUnusedParameters": true,      // 检查未使用的参数
    "noImplicitReturns": true,       // 检查函数是否有返回值
    "noFallthroughCasesInSwitch": true  // 检查 switch 是否有 break
  }
}
```

#### 3. 模块解析选项

```json
{
  "compilerOptions": {
    "moduleResolution": "node",      // 模块解析策略
    "baseUrl": "./src",              // 非相对模块的基础路径
    "paths": {                       // 路径映射
      "@/*": ["./*"],
      "@components/*": ["components/*"],
      "@utils/*": ["utils/*"]
    },
    "typeRoots": ["./node_modules/@types", "./src/types"],  // 类型声明文件目录
    "types": ["node", "jest"],       // 包含的类型声明
    "allowSyntheticDefaultImports": true,  // 允许合成默认导入
    "esModuleInterop": true          // 启用 ES 模块互操作
  }
}
```

#### 4. 其他常用选项

```json
{
  "compilerOptions": {
    "skipLibCheck": true,            // 跳过库文件的类型检查
    "forceConsistentCasingInFileNames": true,  // 强制文件名大小写一致
    "resolveJsonModule": true,       // 允许导入 JSON 文件
    "allowJs": true,                 // 允许编译 JavaScript 文件
    "checkJs": true,                 // 检查 JavaScript 文件
    "isolatedModules": true,         // 确保每个文件可以独立编译
    "preserveConstEnums": true,      // 保留 const enum
    "experimentalDecorators": true,  // 启用装饰器
    "emitDecoratorMetadata": true    // 启用装饰器元数据
  }
}
```

### include 和 exclude

```json
{
  "include": [
    "src/**/*",           // src 目录下所有文件
    "types/**/*.d.ts",    // types 目录下的声明文件
    "custom.d.ts"         // 自定义声明文件
  ],
  "exclude": [
    "node_modules",       // 排除 node_modules
    "dist",               // 排除输出目录
    "**/*.spec.ts",       // 排除测试文件
    "**/*.test.ts"
  ]
}
```

### extends 配置

使用基础配置:

```json
{
  "extends": "@tsconfig/recommended/tsconfig.json"
}
```

组合多个配置:

```json
{
  "extends": [
    "@tsconfig/strictest/tsconfig.json",
    "@tsconfig/node18/tsconfig.json"
  ]
}
```

自定义基础配置:

```json
{
  "extends": "./tsconfig.base.json",
  "compilerOptions": {
    "outDir": "./dist"
  }
}
```

## 实际应用

### 1. React 项目配置

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "jsx": "react-jsx",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "allowJs": true,
    "checkJs": false,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"],
      "@/hooks/*": ["./src/hooks/*"],
      "@/utils/*": ["./src/utils/*"]
    }
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist", "build"]
}
```

### 2. Node.js 后端项目配置

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022"],
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "./dist",
    "rootDir": "./src",
    "sourceMap": true,
    "declaration": true,
    "removeComments": true,
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declarationMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.spec.ts"]
}
```

### 3. Next.js 项目配置

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### 4. Monorepo 项目配置

使用项目引用:

```json
// packages/shared/tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "references": []
}
```

```json
// packages/app/tsconfig.json
{
  "extends": "../../tsconfig.base.json",
  "compilerOptions": {
    "composite": true,
    "outDir": "./dist"
  },
  "include": ["src/**/*"],
  "references": [
    { "path": "../shared" }
  ]
}
```

```json
// tsconfig.base.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "bundler"
  }
}
```

### 5. 不同环境的配置

开发环境:

```json
// tsconfig.dev.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "sourceMap": true,
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "noImplicitReturns": false
  }
}
```

生产环境:

```json
// tsconfig.prod.json
{
  "extends": "./tsconfig.json",
  "compilerOptions": {
    "sourceMap": false,
    "removeComments": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true
  }
}
```

## 注意事项

1. **配置继承顺序**: `extends` 的配置会被本文件的配置覆盖,按顺序应用

2. **路径映射与 baseUrl**: `paths` 必须配合 `baseUrl` 使用

3. **模块解析策略**:
   - `node`: 传统的 Node.js 模块解析
   - `node16`/`nodenext`: Node.js 16+ 的 ESM 解析
   - `bundler`: 用于打包工具(如 Vite、Webpack)

4. **严格模式**: 建议新项目启用 `strict: true`,老项目可以逐步启用

5. **性能优化**:
   - 使用 `skipLibCheck` 加速编译
   - 合理使用 `include`/`exclude` 减少编译文件
   - 启用 `incremental` 或 `tsbuildinfofile` 增量编译

6. **与打包工具配合**:
   - Vite/Webpack 会处理大多数编译选项
   - `target` 和 `module` 通常由打包工具决定
   - 重点配置类型检查和路径映射

7. **版本兼容性**: 确保使用的 TypeScript 版本与项目依赖兼容

8. **vscode 配置**: 在 `.vscode/settings.json` 中指定配置文件:
   ```json
   {
     "typescript.tsdk": "node_modules/typescript/lib",
     "typescript.enablePromptUseWorkspaceTsdk": true
   }
   ```

## 总结

`tsconfig.json` 是 TypeScript 项目的核心,合理配置能够:

- **提高代码质量**: 通过严格类型检查发现问题
- **提升开发体验**: 路径别名、智能提示等
- **优化构建性能**: 增量编译、跳过库检查等
- **适配不同环境**: 开发/生产环境配置分离

建议:
- 新项目使用 `@tsconfig/recommended` 作为基础
- 根据项目需求调整配置选项
- 定期更新 TypeScript 版本以获得最新特性
- 在团队中保持配置一致性