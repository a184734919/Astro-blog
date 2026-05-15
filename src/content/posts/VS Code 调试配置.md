---
title: VS Code 调试配置
published: 2024-12-26
description: 'launch.json 配置的详细介绍和学习笔记'
image: ''
tags: ["VSCode","工具"]
category: '开发工具'
draft: false
lang: 'zh-CN'
---

## 概述

VS Code 调试功能是前端开发中不可或缺的重要工具，通过 `launch.json` 配置文件可以实现断点调试、变量监视、调用堆栈跟踪等强大的调试功能。本文将详细介绍 VS Code 调试配置的核心概念、配置方法和实际应用场景，帮助开发者建立高效的调试工作流。

## 核心概念

### 调试系统架构

VS Code 调试系统基于以下核心组件：

- **Debug Adapter Protocol (DAP)**: 标准化的调试协议
- **Debug Adapters**: 调试适配器，实现调试协议的适配层
- **Launch Configuration**: 启动配置，定义调试会话的启动方式
- **Breakpoints**: 断点，控制程序执行的暂停点
- **Variable Inspection**: 变量检查，实时查看变量值

### 配置文件结构

调试配置主要通过以下文件管理：

```javascript
// 项目结构
.vscode/
├── launch.json      // 调试启动配置
├── tasks.json       // 任务配置（可选）
└── settings.json    // 工作区设置（可选）
```

### 调试类型分类

VS Code 支持多种调试类型：

1. **前端调试**: Chrome、Edge、Firefox 浏览器调试
2. **Node.js 调试**: 服务端 JavaScript 调试
3. **框架调试**: React、Vue、Angular 等框架特定调试
4. **混合调试**: 同时调试前端和后端
5. **远程调试**: 调试远程服务器上的代码

## 基本用法

### 1. 创建调试配置

#### 初始化调试配置

```javascript
// 方法1：通过调试面板创建
// 1. 点击左侧调试图标 (Ctrl+Shift+D / Cmd+Shift+D)
// 2. 点击 "create a launch.json file"
// 3. 选择调试环境
// 4. 自动生成 launch.json 文件

// 方法2：通过命令面板创建
// 1. 打开命令面板: Ctrl+Shift+P (Cmd+Shift+P)
// 2. 输入 "Debug: Open launch.json"
// 3. 如果文件不存在，会提示创建

// 方法3：手动创建文件
// 创建 .vscode/launch.json 文件
```

### 2. 基础配置格式

#### launch.json 基本结构

```javascript
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Program",
      "program": "${workspaceFolder}/app.js"
    }
  ]
}
```

#### 配置属性说明

```javascript
{
  "version": "0.2.0",  // 配置文件版本
  "configurations": [  // 配置数组
    {
      "type": "node",              // 调试器类型
      "request": "launch",         // 请求类型: launch/attach
      "name": "调试配置名称",       // 配置显示名称
      "program": "启动程序路径",    // 要调试的程序路径
      "args": [],                  // 命令行参数
      "cwd": "${workspaceFolder}", // 工作目录
      "runtimeExecutable": "node", // 运行时可执行文件
      "runtimeArgs": [],           // 运行时参数
      "env": {},                   // 环境变量
      "sourceMaps": true,          // 启用 source maps
      "outFiles": [],              // 输出文件路径
      "preLaunchTask": "任务名称"  // 启动前的任务
    }
  ]
}
```

### 3. 变量和表达式

#### 内置变量

```javascript
// VS Code 提供的内置变量
{
  "${workspaceFolder}",      // 工作区根目录
  "${workspaceFolderBasename}", // 工作区文件夹名
  "${file}",                 // 当前文件路径
  "${fileBasename}",         // 当前文件名
  "${fileBasenameNoExtension}", // 当前文件名(无扩展名)
  "${fileDirname}",          // 当前文件所在目录
  "${fileExtname}",          // 当前文件扩展名
  "${cwd}",                  // 当前工作目录
  "${pathSeparator}",        // 路径分隔符
  "${lineNumber}",           // 当前行号
  "${selectedText}",         // 选中文本
  "${env:ENV_VAR}",          // 环境变量
  "${config:settingName}"    // VS Code 设置值
}

// 使用示例
{
  "program": "${workspaceFolder}/src/index.js",
  "cwd": "${workspaceFolder}",
  "args": [
    "--config",
    "${workspaceFolder}/config.json"
  ]
}
```

#### 环境变量配置

```javascript
{
  "name": "带环境变量的调试",
  "type": "node",
  "request": "launch",
  "program": "${workspaceFolder}/index.js",
  "env": {
    "NODE_ENV": "development",
    "DEBUG": "true",
    "PORT": "3000",
    "API_BASE": "http://localhost:8000"
  },
  "envFile": "${workspaceFolder}/.env"  // 从 .env 文件加载
}
```

### 4. 常见调试配置

#### Node.js 调试配置

```javascript
// 基础 Node.js 调试
{
  "name": "Launch Node.js Program",
  "type": "node",
  "request": "launch",
  "program": "${workspaceFolder}/server.js",
  "console": "integratedTerminal",
  "restart": true,
  "protocol": "auto",
  "runtimeExecutable": "node"
}

// 使用 npm script 调试
{
  "name": "Launch via NPM",
  "type": "node",
  "request": "launch",
  "runtimeExecutable": "npm",
  "runtimeArgs": ["start"],
  "console": "integratedTerminal",
  "restart": true
}

// 调试 TypeScript
{
  "name": "Debug TypeScript",
  "type": "node",
  "request": "launch",
  "program": "${workspaceFolder}/src/index.ts",
  "preLaunchTask": "tsc: build - tsconfig.json",
  "outFiles": [
    "${workspaceFolder}/dist/**/*.js"
  ],
  "sourceMaps": true
}
```

#### 前端调试配置

```javascript
// Chrome 调试配置
{
  "name": "Launch Chrome",
  "type": "chrome",
  "request": "launch",
  "url": "http://localhost:3000",
  "webRoot": "${workspaceFolder}",
  "sourceMaps": true,
  "sourceMapPathOverrides": {
    "webpack:///*": "${webRoot}/*"
  }
}

// 调试 React 应用
{
  "name": "Debug React App",
  "type": "chrome",
  "request": "launch",
  "url": "http://localhost:3000",
  "webRoot": "${workspaceFolder}",
  "sourceMaps": true,
  "sourceMapPathOverrides": {
    "webpack://*": "${webRoot}/*"
  },
  "timeout": 30000,
  "breakOnLoad": true,
  "breakOnLoadStrategy": "instrument"
}

// 调试 Vue 应用
{
  "name": "Debug Vue App",
  "type": "chrome",
  "request": "launch",
  "url": "http://localhost:8080",
  "webRoot": "${workspaceFolder}/src",
  "breakOnLoad": true,
  "sourceMapPathOverrides": {
    "webpack:///src/*": "${webRoot}/*",
    "webpack:///./src/*": "${webRoot}/*",
    "webpack:///*": "${webRoot}/*"
  }
}
```

#### 测试文件调试

```javascript
// Jest 测试调试
{
  "name": "Debug Jest Tests",
  "type": "node",
  "request": "launch",
  "runtimeExecutable": "npm",
  "runtimeArgs": [
    "test",
    "--",
    "--runInBand",
    "--no-cache",
    "--watchAll=false"
  ],
  "console": "integratedTerminal",
  "internalConsoleOptions": "neverOpen"
}

// 调试单个测试文件
{
  "name": "Debug Current Jest Test",
  "type": "node",
  "request": "launch",
  "runtimeExecutable": "npm",
  "runtimeArgs": [
    "test",
    "--",
    "${relativeFile}"
  ],
  "console": "integratedTerminal",
  "internalConsoleOptions": "neverOpen"
}
```

## 实际应用

### 1. 全栈项目调试配置

```javascript
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Frontend - Chrome",
      "type": "chrome",
      "request": "launch",
      "url": "http://localhost:3000",
      "webRoot": "${workspaceFolder}/frontend",
      "sourceMaps": true
    },
    {
      "name": "Backend - Node.js",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/backend/index.js",
      "console": "integratedTerminal",
      "restart": true
    },
    {
      "name": "Full Stack - All",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/backend/index.js",
      "console": "integratedTerminal",
      "preLaunchTask": "dev:frontend",
      "restart": true
    }
  ]
}

// .vscode/tasks.json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "dev:frontend",
      "type": "shell",
      "command": "npm",
      "args": ["run", "dev:frontend"],
      "options": {
        "cwd": "${workspaceFolder}/frontend"
      },
      "isBackground": true,
      "problemMatcher": []
    }
  ]
}
```

### 2. 微服务调试配置

```javascript
// 多个微服务调试配置
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "User Service",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/services/user/index.js",
      "env": {
        "SERVICE_NAME": "user",
        "PORT": "3001"
      },
      "console": "integratedTerminal"
    },
    {
      "name": "Order Service",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/services/order/index.js",
      "env": {
        "SERVICE_NAME": "order",
        "PORT": "3002"
      },
      "console": "integratedTerminal"
    },
    {
      "name": "All Services",
      "type": "node",
      "request": "launch",
      "program": "${workspaceFolder}/docker-compose.js",
      "args": ["up"],
      "console": "integratedTerminal"
    }
  ],
  "compounds": [
    {
      "name": "User & Order Services",
      "configurations": ["User Service", "Order Service"],
      "stopAll": true
    }
  ]
}
```

### 3. Electron 应用调试

```javascript
// Electron 主进程调试
{
  "name": "Electron: Main Process",
  "type": "node",
  "request": "launch",
  "cwd": "${workspaceFolder}",
  "runtimeExecutable": "${workspaceFolder}/node_modules/.bin/electron",
  "windows": {
    "runtimeExecutable": "${workspaceFolder}/node_modules/.bin/electron.cmd"
  },
  "args": [".", "--inspect=5858"],
  "outputCapture": "std"
}

// Electron 渲染进程调试
{
  "name": "Electron: Renderer Process",
  "type": "chrome",
  "request": "attach",
  "port": 9223,
  "webRoot": "${workspaceFolder}",
  "timeout": 30000
}

// 同时调试主进程和渲染进程
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Electron: All",
      "type": "node",
      "request": "launch",
      "cwd": "${workspaceFolder}",
      "runtimeExecutable": "${workspaceFolder}/node_modules/.bin/electron",
      "args": [".", "--remote-debugging-port=9223"],
      "windows": {
        "runtimeExecutable": "${workspaceFolder}/node_modules/.bin/electron.cmd"
      },
      "outputCapture": "std"
    }
  ]
}
```

### 4. Docker 容器调试

```javascript
// 远程 Docker 容器调试
{
  "name": "Debug in Docker",
  "type": "node",
  "request": "attach",
  "address": "localhost",
  "port": 9229,
  "localRoot": "${workspaceFolder}",
  "remoteRoot": "/app",
  "protocol": "inspector",
  "restart": true,
  "sourceMaps": true
}

// Docker Compose 调试
{
  "name": "Docker Compose Debug",
  "type": "node",
  "request": "attach",
  "address": "localhost",
  "port": 9229,
  "localRoot": "${workspaceFolder}",
  "remoteRoot": "/usr/src/app",
  "protocol": "inspector",
  "restart": true,
  "timeout": 30000,
  "preLaunchTask": "docker-up"
}

// .vscode/tasks.json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "docker-up",
      "type": "shell",
      "command": "docker-compose",
      "args": ["up", "-d", "--build"],
      "problemMatcher": []
    }
  ]
}
```

### 5. 性能分析调试

```javascript
// Node.js 性能分析
{
  "name": "Node.js Performance Profiling",
  "type": "node",
  "request": "launch",
  "program": "${workspaceFolder}/app.js",
  "runtimeArgs": [
    "--prof",
    "--no-logfile-per-isolation"
  ],
  "console": "integratedTerminal"
}

// 内存使用分析
{
  "name": "Node.js Memory Profiling",
  "type": "node",
  "request": "launch",
  "program": "${workspaceFolder}/app.js",
  "runtimeArgs": [
    "--inspect=9229"
  ],
  "console": "integratedTerminal",
  "sourceMaps": true
}
```

## 注意事项

### 1. Source Maps 配置

```javascript
// Source Maps 是调试压缩代码的关键

// 常见问题
// 1. Source Maps 路径不匹配
// 解决方案: 使用 sourceMapPathOverrides

{
  "sourceMapPathOverrides": {
    "webpack:///*": "${webRoot}/*",
    "webpack:///src/*": "${webRoot}/src/*",
    "webpack://~/*": "${webRoot}/node_modules/*"
  }
}

// 2. TypeScript 调试
// 确保 tsconfig.json 正确配置
{
  "compilerOptions": {
    "sourceMap": true,
    "inlineSourceMap": false,
    "sourceRoot": "/"
  }
}
```

### 2. 断点使用技巧

```javascript
// 高效使用断点的技巧

// 1. 条件断点
// 右键点击断点，设置条件
// 示例：只在 user.id > 100 时暂停
user.id > 100

// 2. 日志断点
// 不暂停代码，只输出信息
// 右键点击断点，编辑断点
// 输入表达式: console.log('Current value:', variable)

// 3. 触发断点
// 在抛出特定异常时暂停
{
  "name": "Debug with Exception Breakpoints",
  "type": "node",
  "request": "launch",
  "program": "${workspaceFolder}/app.js",
  "exceptionBreakpoints": [
    "uncaught",
    "caught"
  ]
}
```

### 3. 调试性能优化

```javascript
// 提高调试性能的方法

// 1. 减少断点数量
// 只在关键位置设置断点
// 使用条件断点减少暂停次数

// 2. 优化 Source Maps
{
  "sourceMaps": true,
  "outFiles": ["${workspaceFolder}/dist/**/*.js"],
  "skipFiles": [
    "<node_internals>/**",
    "node_modules/**"
  ]
}

// 3. 使用调试控制台
// 避免在代码中添加 console.log
// 使用调试控制台执行表达式
```

### 4. 常见问题解决

```javascript
// 1. 断点不命中
// 检查 Source Maps 配置
// 确认文件路径正确
// 验证代码行号匹配

// 2. 调试器无法连接
{
  // 检查端口是否被占用
  "port": 9229,
  // 增加超时时间
  "timeout": 30000
}

// 3. 变量显示不正确
// 确保 sourceMaps 启用
// 检查变量作用域
// 使用 "Debug: Show Variables" 命令
```

## 总结

VS Code 调试配置是前端开发中重要的技能，通过合理的配置可以实现：

1. **高效调试**: 精确定位和修复代码问题
2. **多场景支持**: 支持前端、后端、测试等多种调试场景
3. **团队协作**: 统一的调试配置便于团队协作
4. **性能分析**: 支持性能分析和内存分析

**核心要点**:
- 理解 launch.json 的配置结构和属性
- 掌握常用变量和表达式的使用
- 熟悉不同类型项目的调试配置
- 学会使用高级调试功能如条件断点、日志断点
- 注意 Source Maps 配置和性能优化

**最佳实践**:
- 为不同环境创建不同的调试配置
- 使用项目级的调试配置文件
- 结合 tasks.json 实现自动化工作流
- 定期更新和维护调试配置
- 与团队共享调试配置

通过掌握 VS Code 调试配置，你将获得强大的调试能力，能够更高效地开发和调试代码，显著提升开发体验和代码质量。