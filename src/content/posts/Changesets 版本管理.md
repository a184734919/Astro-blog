---
title: Changesets 版本管理
published: 2024-12-02
description: 'Changesets 发布流程的详细介绍和学习笔记'
image: ''
tags: ["工具","版本"]
category: '前端工具'
draft: false
lang: 'zh-CN'
---

## Changesets 概述

Changesets 是一个功能强大的版本管理工具，主要用于 JavaScript/TypeScript 项目的自动化版本发布。它在 monorepo 环境下特别有用，能够智能地管理多个包的版本依赖关系。

### 为什么选择 Changesets

- **Monorepo 友好**：原生支持 monorepo 架构
- **智能版本控制**：基于代码变更自动确定版本号
- **变更日志生成**：自动生成规范的 CHANGELOG.md
- **依赖管理**：自动处理包之间的依赖关系
- **团队协作**：提供清晰的工作流程
- **灵活性**：支持多种发布策略

### 核心概念

1. **Changeset**：描述变更的元数据文件
2. **Versioning**：语义化版本控制（SemVer）
3. **Publishing**：自动发布到 npm
4. **Dependencies**：依赖关系管理

## 安装和配置

### 基础安装

```bash
# 安装 Changesets
npm install @changesets/cli

# 初始化 Changesets
npx changeset init
```

### 生成的配置文件

```json
// .changeset/config.json
{
  "$schema": "https://unpkg.com/@changesets/config@3.0.0/schema.json",
  "changelog": "@changesets/cli/changelog",
  "commit": false,
  "fixed": [],
  "linked": [],
  "access": "restricted",
  "baseBranch": "main",
  "updateInternalDependencies": "patch",
  "ignore": []
}
```

### 配置选项详解

```json
{
  // 更改日志生成器
  "changelog": "@changesets/cli/changelog",
  
  // 是否在版本更新时创建 git commit
  "commit": false,
  
  // 固定版本更新的包组（一起更新版本）
  "fixed": [
    ["packages/ui", "packages/theme"]
  ],
  
  // 链接的包组（一起发布，但版本独立）
  "linked": [
    ["packages/client", "packages/server"]
  ],
  
  // npm 访问权限：restricted（私有）或 public（公开）
  "access": "restricted",
  
  // 主分支名称
  "baseBranch": "main",
  
  // 内部依赖更新策略：patch 或 minor
  "updateInternalDependencies": "patch",
  
  // 忽略的包（不参与版本管理）
  "ignore": [
    "packages/internal",
    "packages/tools"
  ]
}
```

## 基础工作流程

### 1. 创建 Changeset

```bash
# 创建新的 changeset
npx changeset

# 指定具体的包
npx changeset add

# 查看所有未发布的 changesets
npx changeset status

# 查看详细状态
npx changeset status --verbose
```

### 2. Changeset 文件结构

```markdown
# .changeset/filename.md
---
"@company/package-a": minor
"@company/package-b": patch
---

添加新功能并修复了一些 bug。
```

### 3. 版本类型说明

- **patch**: 修复 bug，向后兼容
- **minor**: 新功能，向后兼容
- **major**: 破坏性变更

### 4. 版本更新

```bash
# 消费 changesets 并更新版本号
npx changeset version

# 查看版本变更信息
npx changeset version --verbose

# 确认版本更新
npx changeset pre enter <tag>
```

### 5. 发布包

```bash
# 发布所有已更新的包
npx changeset publish

# 发布特定包
npx changeset publish --tag latest

# 预发布
npx changeset publish --tag beta
```

## 完整的发布流程

### 标准发布流程

```bash
# 1. 开发者创建 changeset
npx changeset

# 交互式选择：
# ? Which packages would you like to include? 
# ◉ package-a
# ◉ package-b
# ? Which packages should have a major bump?
# ◯ package-a
# ● package-b
# ? What is the summary for this change?
# > 添加用户认证功能并修复登录问题

# 2. 提交 changeset 文件
git add .changeset/filename.md
git commit -m "feat: add user authentication"

# 3. 推送到远程仓库
git push origin feature/user-auth

# 4. 创建 Pull Request
# PR 包含 changeset 文件，会被自动检测

# 5. 合并 PR 到主分支
# 在主分支上执行版本更新
npx changeset version

# 这会：
# - 更新 package.json 版本号
# - 更新内部依赖
# - 更新或创建 CHANGELOG.md
# - 删除已使用的 changeset 文件

# 6. 提交版本更新
git add .
git commit -m "chore: version packages"

# 7. 发布到 npm
npx changeset publish

# 8. 推送标签
git push --follow-tags
```

### 脚本化发布流程

```json
// package.json
{
  "scripts": {
    "changeset": "changeset",
    "version-packages": "changeset version",
    "release": "changeset publish",
    "ci:version": "changeset version && git push --follow-tags",
    "ci:release": "changeset publish",
    "clean:changesets": "changeset clean"
  }
}
```

```bash
# 使用脚本
npm run changeset          # 创建 changeset
npm run version-packages   # 更新版本
npm run release           # 发布
```

## 高级功能

### 1. Monorepo 管理

```json
// .changeset/config.json
{
  "fixed": [
    ["packages/ui", "packages/theme", "packages/components"]
  ],
  "linked": [
    ["packages/client", "packages/server"]
  ],
  "updateInternalDependencies": "patch",
  "ignore": [
    "packages/docs",
    "packages/examples"
  ]
}
```

### 2. 自定义变更日志

```javascript
// .changeset/config.json
{
  "changelog": "./scripts/custom-changelog.js"
}
```

```javascript
// scripts/custom-changelog.js
const getDependencyChangelogs = require('@changesets/get-dependents-graph').default;
const { getInfo } = require('@changesets/get-github-info');

module.exports = async function customChangelog(changesets, options) {
  const changelog = [];
  
  for (const changeset of changesets) {
    const commit = await getInfo({
      repo: `${options.repo.owner}/${options.repo.repo}`,
      commit: changeset.commit
    });
    
    changelog.push({
      ...changeset,
      commitUrl: commit ? commit.href : null
    });
  }
  
  return changelog;
};
```

### 3. 预发布版本

```bash
# 进入预发布模式
npx changeset pre enter beta

# 正常开发流程
npx changeset add
npx changeset version

# 退出预发布模式
npx changeset pre exit

# 发布预发布版本
npx changeset publish --tag beta
```

```json
// package.json 中的预发布版本
{
  "version": "1.2.3-beta.1",
  "publishConfig": {
    "tag": "beta"
  }
}
```

### 4. 版本策略控制

```javascript
// .changeset/config.json
{
  "bumpVersionsWithWorkspaceProtocolOnly": true,
  "updateInternalDependencies": "patch",
  "privatePackages": {
    "version": true,
    "tag": true
  }
}
```

## CI/CD 集成

### GitHub Actions 配置

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    branches:
      - main

jobs:
  release:
    name: Release
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          registry-url: 'https://registry.npmjs.org'

      - name: Install dependencies
        run: npm ci

      - name: Build packages
        run: npm run build

      - name: Create Release Pull Request or Publish
        id: changesets
        uses: changesets/action@v1
        with:
          publish: npm run release
          version: npm run version-packages
          commit: 'chore: version packages'
          title: 'chore: version packages'
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}

      - name: Docker Build and Push
        if: steps.changesets.outputs.published == 'true'
        run: |
          echo "Building and pushing Docker images..."
```

### GitLab CI 配置

```yaml
# .gitlab-ci.yml
stages:
  - version
  - publish

version_packages:
  stage: version
  script:
    - npm ci
    - npx changeset version
    - git config user.email "ci@example.com"
    - git config user.name "CI"
    - git add .
    - git commit -m "chore: version packages"
    - git push origin main
  only:
    - main
  when: manual

publish_packages:
  stage: publish
  script:
    - npm ci
    - npm run build
    - npx changeset publish
  only:
    - main
  when: on_success
```

## 环境配置

### npm 配置

```bash
# 设置 npm 认证令牌
npm config set //registry.npmjs.org/:_authToken YOUR_NPM_TOKEN

# 或者在环境变量中设置
export NPM_TOKEN="your-npm-token"

# 私有 registry 配置
npm config set @company:registry https://npm.company.com
npm config set //npm.company.com/:_authToken YOUR_TOKEN
```

### GitHub Actions 环境变量

```yaml
- name: Configure npm
  run: |
    echo "//registry.npmjs.org/:_authToken=${{ secrets.NPM_TOKEN }}" > .npmrc
    echo "@company:registry=https://npm.company.com" >> .npmrc
    echo "//npm.company.com/:_authToken=${{ secrets.COMPANY_NPM_TOKEN }}" >> .npmrc
```

## 最佳实践

### 1. 团队协作规范

```markdown
# Changesets 使用指南

## 创建 Changeset

每次代码变更都需要创建 changeset：

```bash
npx changeset
```

## Changeset 类型选择

- **patch**: Bug 修复，向后兼容
- **minor**: 新功能，向后兼容
- **major**: 破坏性变更

## Commit Message 规范

changeset 的描述应该：
- 清晰简洁
- 说明变更的原因
- 列出主要的功能点

## 版本发布流程

1. 开发者在功能分支创建 changeset
2. 提交 PR 到主分支
3. 合并后自动触发版本更新
4. 审查版本变更
5. 发布到 npm
```

### 2. 项目结构组织

```
my-monorepo/
├── .changeset/
│   ├── config.json
│   ├── README.md
│   └── *.md              # changeset 文件
├── packages/
│   ├── package-a/
│   │   ├── package.json
│   │   ├── CHANGELOG.md
│   │   └── src/
│   ├── package-b/
│   └── package-c/
├── package.json
└── tsconfig.json
```

### 3. 版本管理策略

```json
// .changeset/config.json
{
  // 对于紧密相关的包，使用 fixed 策略
  "fixed": [
    ["packages/ui", "packages/theme", "packages/components"]
  ],
  
  // 对于独立发布但功能相关的包，使用 linked 策略
  "linked": [
    ["packages/client", "packages/server"]
  ],
  
  // 内部依赖更新策略
  "updateInternalDependencies": "patch",
  
  // 忽略内部工具包
  "ignore": [
    "packages/internal",
    "packages/eslint-config"
  ]
}
```

### 4. 自动化脚本

```javascript
// scripts/check-changesets.js
const { getPackages } = require('@manypkg/get-packages');
const { read } = require('@changesets/config');
const fs = require('fs');
const path = require('path');

async function checkChangesets() {
  const packages = await getPackages(__dirname);
  const config = await read(process.cwd(), packages);
  const changesetDir = path.join(__dirname, '.changeset');
  
  if (!fs.existsSync(changesetDir)) {
    console.error('No changesets directory found');
    process.exit(1);
  }
  
  const files = fs.readdirSync(changesetDir)
    .filter(file => file.endsWith('.md'));
  
  if (files.length === 0) {
    console.error('No changesets found. Please run `npx changeset`');
    process.exit(1);
  }
  
  console.log(`Found ${files.length} changesets`);
  files.forEach(file => {
    console.log(`- ${file}`);
  });
}

checkChangesets().catch(error => {
  console.error('Error:', error);
  process.exit(1);
});
```

```json
// package.json
{
  "scripts": {
    "pre-commit": "node scripts/check-changesets.js",
    "version": "changeset version",
    "release": "changeset publish"
  }
}
```

## 常见问题和解决方案

### 问题 1：Changeset 未被识别

```bash
# 问题：changeset 文件存在但未生效

# 检查文件名格式
# 正确：.changeset/cool-dolphins-jog.md
# 错误：.changeset/cool dolphins jog.md (含空格)

# 解决方案：重命名文件
mv .changeset/old-name.md .changeset/correct-name.md

# 检查文件格式
# 必须包含 JSON frontmatter 和描述
---
"@company/package": minor
---

描述内容
```

### 问题 2：版本更新不正确

```bash
# 问题：版本号没有按预期更新

# 检查内部依赖配置
# .changeset/config.json
{
  "updateInternalDependencies": "patch" // 或 "minor"
}

# 手动触发版本更新
npx changeset version --verbose

# 检查包依赖关系
npm list package-name
```

### 问题 3：发布失败

```bash
# 问题：npm 发布失败

# 检查 npm 认证
npm whoami

# 检查包名是否已存在
npm view package-name

# 检查包访问权限
# 私有包
"publishConfig": {
  "access": "restricted"
}

# 公开包
"publishConfig": {
  "access": "public"
}

# 清理未发布的 changesets
npx changeset pre exit
npx changeset clean
```

### 问题 4：依赖关系问题

```bash
# 问题：包之间的依赖版本不匹配

# 解决方案：使用 workspace 协议
{
  "dependencies": {
    "@company/ui": "workspace:*"
  }
}

# 或指定相对路径
{
  "dependencies": {
    "@company/ui": "file:../ui"
  }
}
```

## 高级使用场景

### 1. 自定义版本计算

```javascript
// scripts/custom-version.js
const { read, getPackages } = require('@changesets/config');
const { getReleases } = require('@changesets/git');
const { applyReleasePlan } = require('@changesets/apply-release-plan');

async function customVersion() {
  const cwd = process.cwd();
  const packages = await getPackages(cwd);
  const config = await read(cwd, packages);
  
  // 自定义版本计算逻辑
  const customVersionType = (changeset) => {
    if (changeset.summary.includes('BREAKING')) {
      return 'major';
    }
    if (changeset.summary.includes('feature')) {
      return 'minor';
    }
    return 'patch';
  };
  
  const releases = await getReleases(cwd);
  const releasePlan = await getReleasePlan(cwd, config, packages);
  
  // 应用自定义版本计算
  releasePlan.releases.forEach(release => {
    release.type = customVersionType(release.changesets[0]);
  });
  
  await applyReleasePlan(releasePlan, packages, config);
  
  console.log('Custom version calculation completed');
}

customVersion().catch(console.error);
```

### 2. 多 Registry 发布

```javascript
// scripts/publish-multi-registry.js
const { publish } = require('@changesets/cli/dist/publish');
const npmPublish = require('libnpmpublish');
const fs = require('fs');

async function publishMultiRegistry() {
  const cwd = process.cwd();
  const packages = await getPackages(cwd);
  
  for (const pkg of packages) {
    const packageJson = JSON.parse(
      fs.readFileSync(path.join(pkg.dir, 'package.json'), 'utf-8')
    );
    
    // 根据包名选择 registry
    let registry = 'https://registry.npmjs.org';
    let token = process.env.NPM_TOKEN;
    
    if (packageJson.name.startsWith('@company/')) {
      registry = 'https://npm.company.com';
      token = process.env.COMPANY_NPM_TOKEN;
    }
    
    // 发布到指定的 registry
    await npmPublish(pkg.dir, {
      registry,
      token
    });
    
    console.log(`Published ${packageJson.name} to ${registry}`);
  }
}

publishMultiRegistry().catch(console.error);
```

### 3. 变更日志定制

```javascript
// scripts/custom-changelog.js
const { getInfo } = require('@changesets/get-github-info');

module.exports = async function customChangelog(
  changesets,
  options
) {
  const changelogLines = [];
  
  // 按类型分组
  const groupedChangesets = {
    major: [],
    minor: [],
    patch: []
  };
  
  changesets.forEach(changeset => {
    const type = changeset.releases[0]?.type || 'patch';
    groupedChangesets[type].push(changeset);
  });
  
  // 生成分类变更日志
  for (const [type, csList] of Object.entries(groupedChangesets)) {
    if (csList.length === 0) continue;
    
    changelogLines.push(`\n## ${type.toUpperCase()}\n`);
    
    for (const changeset of csList) {
      const commit = await getInfo({
        repo: options.repo,
        commit: changeset.commit
      });
      
      const packages = changeset.releases
        .map(r => `**@${r.name}**`)
        .join(', ');
      
      changelogLines.push(`- ${changeset.summary} (${packages})`);
      
      if (commit) {
        changelogLines.push(`  ([${commit.pull}](https://github.com/${options.repo}/pull/${commit.pull}))`);
      }
    }
  }
  
  return changelogLines.join('\n');
};
```

## 监控和报告

### 发布统计

```javascript
// scripts/publish-stats.js
const { getPackages } = require('@manypkg/get-packages');
const axios = require('axios');

async function getPublishStats() {
  const packages = await getPackages(process.cwd());
  const stats = [];
  
  for (const pkg of packages.packages) {
    const packageJson = JSON.parse(
      require('fs').readFileSync(
        require('path').join(pkg.dir, 'package.json'),
        'utf-8'
      )
    );
    
    try {
      const response = await axios.get(
        `https://registry.npmjs.org/${packageJson.name}`
      );
      
      const latestVersion = response.data['dist-tags'].latest;
      const downloads = await axios.get(
        `https://api.npmjs.org/downloads/point/last-month/${packageJson.name}`
      );
      
      stats.push({
        name: packageJson.name,
        version: latestVersion,
        downloads: downloads.data.downloads
      });
    } catch (error) {
      console.error(`Error fetching stats for ${packageJson.name}:`, error.message);
    }
  }
  
  console.table(stats);
  return stats;
}

getPublishStats().catch(console.error);
```

## 总结

Changesets 是一个功能强大且灵活的版本管理工具，特别适合 monorepo 项目。通过本篇详细指南，你已经了解了：

- Changesets 的核心概念和优势
- 安装配置和基础使用
- 完整的发布工作流程
- 高级功能和自定义配置
- CI/CD 集成方案
- 团队协作最佳实践
- 常见问题的解决方案
- 高级使用场景

在实际项目中，建议：

1. 建立明确的 Changesets 使用规范
2. 合理配置 fixed 和 linked 策略
3. 集成到 CI/CD 流程中自动化发布
4. 定期审查和优化版本管理流程
5. 使用自定义脚本增强功能
6. 监控发布状态和包使用情况

掌握 Changesets 能够显著提升版本发布效率，特别是在复杂的 monorepo 项目中，它是维护多包依赖关系的理想选择。