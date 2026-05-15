---
title: Node.js npm 包管理
published: 2023-02-28
description: 'npm 的使用和 package.json的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

npm（Node Package Manager）是 Node.js 的默认包管理器，也是世界上最大的软件注册表。通过 npm，我们可以轻松安装、更新、管理和共享 JavaScript 代码包。

## package.json 详解

### 基本信息

```json
{
  "name": "my-awesome-project",
  "version": "1.0.0",
  "description": "一个很棒的项目描述",
  "main": "index.js",
  "author": "Your Name <your.email@example.com>",
  "license": "MIT"
}
```

- `name`: 项目名称（必须小写，不能包含空格）
- `version`: 遵循语义化版本规范（semver）
- `description`: 项目描述
- `main`: 项目入口文件
- `author`: 作者信息
- `license`: 开源协议

### 脚本命令

```json
{
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest",
    "build": "webpack --mode production",
    "lint": "eslint .",
    "format": "prettier --write ."
  }
}
```

使用 npm 脚本：
```bash
npm start           # 运行 start 脚本
npm run dev         # 运行 dev 脚本
npm run test        # 运行 test 脚本
```

### 依赖管理

```json
{
  "dependencies": {
    "express": "^4.18.2",
    "lodash": "~4.17.21",
    "axios": "1.4.0"
  },
  "devDependencies": {
    "jest": "^29.5.0",
    "eslint": "^8.40.0",
    "nodemon": "^2.0.22"
  },
  "peerDependencies": {
    "react": ">=16.8.0"
  },
  "optionalDependencies": {
    "fsevents": "^2.3.2"
  }
}
```

- `dependencies`: 生产环境依赖
- `devDependencies`: 开发环境依赖
- `peerDependencies`: 对等依赖（插件类库需要）
- `optionalDependencies`: 可选依赖

### 版本号规则

遵循语义化版本（Semantic Versioning）：`主版本号.次版本号.修订号`

- `^1.2.3`: 兼容版本（主版本不变）
- `~1.2.3`: 补丁版本（主次版本不变）
- `1.2.3`: 精确版本
- `*` 或 `latest`: 最新版本
- `>1.2.3`, `>=1.2.3`, `<1.2.3`, `<=1.2.3`: 范围版本
- `1.2.3 - 2.3.4`: 版本范围

```json
{
  "dependencies": {
    "express": "^4.18.2",      // >=4.18.2 <5.0.0
    "lodash": "~4.17.21",      // >=4.17.21 <4.18.0
    "axios": "1.4.0",          // 精确 1.4.0
    "react": ">=16.8.0 <18.0.0"
  }
}
```

## npm 常用命令

### 初始化项目

```bash
# 创建 package.json（交互式）
npm init

# 使用默认配置创建
npm init -y

# 创建特定名称的项目
npm init -y --scope=mycompany
```

### 安装依赖

```bash
# 安装最新版本
npm install express

# 安装指定版本
npm install express@4.18.0

# 安装多个包
npm install express lodash axios

# 安装开发依赖
npm install --save-dev jest
# 或简写
npm install -D jest

# 全局安装
npm install -g nodemon

# 安装生产依赖（跳过 devDependencies）
npm install --production
```

### 其他安装选项

```bash
# 只安装 dependencies
npm install --only=prod

# 只安装 devDependencies
npm install --only=dev

# 从 package.json 安装所有依赖
npm install

# 精确安装（锁定版本）
npm install express --save-exact

# 安装但不保存到 package.json
npm install express --no-save
```

### 查看和搜索

```bash
# 查看已安装的包
npm list

# 查看全局安装的包
npm list -g --depth=0

# 查看包信息
npm info express

# 查看包的特定版本
npm info express@4.18.0

# 搜索包
npm search express

# 查看过时的包
npm outdated

# 查看包的文档
npm home express
npm docs express
```

### 更新和卸载

```bash
# 更新所有依赖（遵循 package.json 版本规则）
npm update

# 更新特定包
npm update express

# 更新到最新版本（忽略版本限制）
npm install express@latest

# 卸载包
npm uninstall express

# 卸载开发依赖
npm uninstall -D jest
```

### 脚本执行

```bash
# 运行脚本
npm run <script-name>

# 运行 start 脚本（可省略 run）
npm start

# 运行 test 脚本
npm test

# 传递参数
npm run build -- --mode production

# 并行执行多个脚本
npm run watch & npm run serve

# 使用 pre 和 post 钩子
npm run build  # 会先执行 prebuild，再执行 build，最后执行 postbuild
```

## npm 配置

### 查看配置

```bash
# 查看所有配置
npm config list

# 查看特定配置
npm config get registry

# 编辑配置文件
npm config edit

# 查看用户配置文件路径
npm config get userconfig
```

### 设置配置

```bash
# 设置镜像源
npm config set registry https://registry.npmmirror.com

# 设置全局安装路径
npm config set prefix /usr/local

# 保存为项目配置（创建 .npmrc）
npm config set init-author-name "Your Name"
```

### 常用配置

```bash
# 使用淘宝镜像
npm config set registry https://registry.npmmirror.com

# 或者临时使用
npm install express --registry=https://registry.npmmirror.com

# 验证配置
npm config get registry
```

### 项目级配置（.npmrc）

在项目根目录创建 `.npmrc` 文件：

```ini
# 使用特定镜像
registry=https://registry.npmmirror.com

# 安装时不保存精确版本
save-exact=false

# 安装时自动保存
save=true

# 保存前缀
save-prefix=^

# 忽略脚本
ignore-scripts=false

# 引用私有包
@mycompany:registry=https://npm.mycompany.com
```

## package-lock.json

### 作用

`package-lock.json` 锁定依赖的确切版本，确保团队成员安装相同的依赖树。

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "lockfileVersion": 2,
  "requires": true,
  "packages": {
    "node_modules/express": {
      "version": "4.18.2",
      "resolved": "https://registry.npmmirror.com/express/-/express-4.18.2.tgz",
      "integrity": "sha512-xxx...",
      "dependencies": {
        "accepts": "~1.3.8",
        "array-flatten": "1.1.1"
      }
    }
  }
}
```

### 使用场景

```bash
# 安装时自动生成 package-lock.json
npm install express

# 根据 package-lock.json 安装（默认行为）
npm ci

# 忽略 package-lock.json
npm install --no-package-lock
```

**npm ci vs npm install:**
- `npm ci`: 严格根据 package-lock.json 安装，更快，适合 CI/CD
- `npm install`: 可以更新 package-lock.json，适合开发环境

## npm 脚本进阶

### 生命周期脚本

```json
{
  "scripts": {
    "preinstall": "node check-node-version.js",
    "install": "node-gyp rebuild",
    "postinstall": "node setup.js",
    "prepublishOnly": "npm run build",
    "pretest": "npm run lint",
    "test": "jest",
    "posttest": "npm run coverage",
    "prestart": "npm run build",
    "start": "node index.js",
    "prestop": "node pre-stop.js",
    "stop": "node stop.js",
    "poststop": "node post-stop.js"
  }
}
```

### 环境变量

在 npm 脚本中使用环境变量：

```json
{
  "scripts": {
    "start": "NODE_ENV=production node index.js",
    "dev": "NODE_ENV=development nodemon index.js",
    "test": "NODE_ENV=test jest",
    "build:prod": "NODE_ENV=production npm run build"
  }
}
```

在代码中访问：

```javascript
console.log(process.env.NODE_ENV);
```

### 跨平台脚本

使用 `cross-env` 处理跨平台环境变量：

```bash
npm install --save-dev cross-env
```

```json
{
  "scripts": {
    "build": "cross-env NODE_ENV=production webpack --mode production"
  }
}
```

### 并行和串行执行

使用 `npm-run-all`：

```bash
npm install --save-dev npm-run-all
```

```json
{
  "scripts": {
    "lint": "eslint .",
    "test": "jest",
    "build": "webpack",
    "check": "npm-run-all lint test",
    "watch": "npm-run-all --parallel watch:*",
    "watch:css": "sass --watch src:dist",
    "watch:js": "webpack --watch"
  }
}
```

## 发布包到 npm

### 准备工作

```bash
# 登录 npm 账号
npm login

# 检查包名是否可用
npm search your-package-name

# 或访问 https://www.npmjs.com/package/your-package-name
```

### package.json 配置

```json
{
  "name": "@yourusername/your-package",
  "version": "1.0.0",
  "description": "包描述",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "files": [
    "dist"
  ],
  "scripts": {
    "build": "tsc",
    "prepublishOnly": "npm run build"
  },
  "keywords": [
    "keyword1",
    "keyword2"
  ],
  "author": "Your Name <your.email@example.com>",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "https://github.com/yourusername/your-package.git"
  },
  "homepage": "https://github.com/yourusername/your-package#readme",
  "bugs": {
    "url": "https://github.com/yourusername/your-package/issues"
  }
}
```

### 发布流程

```bash
# 构建项目
npm run build

# 检查包内容
npm pack --dry-run

# 发布到 npm
npm publish

# 发布到特定标签
npm publish --tag beta

# 发布私有包
npm publish --access restricted
```

### 更新包

```bash
# 更新版本
npm version patch   # 1.0.0 -> 1.0.1
npm version minor   # 1.0.0 -> 1.1.0
npm version major   # 1.0.0 -> 2.0.0

# 重新发布
npm publish
```

### 撤销发布（24小时内）

```bash
# 撤销特定版本
npm unpublish your-package-name@1.0.0

# 撤销整个包（危险操作）
npm unpublish your-package-name --force
```

## 私有 npm 包管理

### npm 私有包

```bash
# 创建作用域包
npm init --scope=@yourcompany

# 发布私有包
npm publish --access restricted

# 安装私有包
npm install @yourcompany/your-package
```

### 配置私有仓库

```bash
# 添加私有仓库
npm login --registry=https://npm.yourcompany.com

# 在 .npmrc 中配置
@yourcompany:registry=https://npm.yourcompany.com
registry=https://registry.npmmirror.com
```

## 常用 npm 工具

### nvm（Node 版本管理）

```bash
# 安装 Node 版本
nvm install 18.16.0

# 切换版本
nvm use 18.16.0

# 设置默认版本
nvm alias default 18.16.0

# 列出已安装版本
nvm ls
```

### npx（执行包）

```bash
# 直接执行包而不安装
npx create-react-app my-app

# 执行特定版本
npx create-react-app@5.0.1 my-app

# 使用本地 node_modules 中的命令
npx prettier --write .
```

### 其他实用工具

```bash
# npm-check：检查过时的包
npm install -g npm-check
npm-check -u

# npm-check-updates：更新 package.json
npm install -g npm-check-updates
ncu -u

# npm-scripts-info：查看脚本说明
npm install -g npm-scripts-info
```

## 最佳实践

### 1. 使用精确版本

```bash
# 生产环境使用精确版本
npm install express --save-exact
# 或在 package-lock.json 中锁定
```

### 2. 定期更新依赖

```bash
# 检查过时包
npm outdated

# 更新依赖
npm update

# 审计安全漏洞
npm audit
npm audit fix
```

### 3. 使用 .npmignore

```
# 测试文件
test/
*.test.js

# 配置文件
.env
.env.local

# 文档
*.md
!README.md

# IDE
.vscode/
.idea/

# 构建工具
.git/
node_modules/
```

### 4. 版本号管理

```bash
# 使用语义化版本
npm version patch  # 修复 bug
npm version minor  # 新功能（向后兼容）
npm version major  # 破坏性变更
```

### 5. 锁定依赖树

```bash
# 将 package-lock.json 提交到版本控制
git add package-lock.json
git commit -m "Update dependencies"
```

## 总结

npm 包管理是 Node.js 开发的核心技能：

1. **package.json**: 项目配置的核心文件
2. **依赖管理**: 区分生产、开发、对等依赖
3. **版本控制**: 遵循语义化版本规范
4. **npm 命令**: 熟练使用安装、更新、卸载等命令
5. **脚本管理**: 优雅地管理项目脚本
6. **包发布**: 了解如何发布和维护 npm 包
7. **私有包**: 支持企业级私有包管理

掌握 npm 包管理能够大大提高开发效率，让项目管理更加规范和便捷。