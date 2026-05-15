---
title: Git Commit 规范
published: 2024-09-22
description: '提交信息规范和工具的详细介绍和学习笔记'
image: ''
tags: ["Git","规范"]
category: '前端工具'
draft: false
lang: 'zh-CN'
---

## 概述

Git Commit 规范是团队协作开发中至关重要的实践，统一的提交信息格式可以提高代码可读性、简化代码审查过程，并且方便自动化工具的使用。本文将详细介绍主流的提交规范及其在项目中的应用。

## 核心概念

### 提交信息的重要性

- **代码历史清晰**：通过提交信息快速了解项目演进
- **自动化工具集成**：支持自动生成 changelog、版本管理等
- **团队协作效率**：统一的格式便于团队成员理解和协作
- **问题追踪**：关联问题跟踪系统，方便问题定位和解决

### 主流提交规范

1. **Conventional Commits**：最广泛使用的提交规范
2. **Angular 约定**：Angular 项目的提交规范
3. **自定义规范**：团队根据项目需求定制的规范

## 基本用法

### Conventional Commits 规范格式

```
<type>(<scope>): <subject>

<body>

<footer>
```

### 提交类型（Type）

```bash
# 常用提交类型
feat:     # 新功能
fix:      # 修复 bug
docs:     # 文档变更
style:    # 代码格式（不影响代码运行的变动）
refactor: # 重构（既不是新增功能，也不是修改 bug 的代码变动）
perf:     # 性能优化
test:     # 增加测试
chore:    # 构建过程或辅助工具的变动
ci:       # CI 配置文件和脚本的变动
revert:   # 回退之前的 commit
build:    # 影响构建系统或外部依赖的变更
```

### 提交范围（Scope）

```bash
# 范围表示提交影响的模块或组件
feat(auth): add user login functionality
fix(ui): resolve button display issue
docs(readme): update installation instructions

# 常见范围
auth, ui, api, db, config, utils, components, etc.
```

### 提交示例

```bash
# 新功能
feat: add user authentication system

# 修复 bug
fix: resolve login page redirect issue

# 文档更新
docs: update API documentation

# 性能优化
perf: optimize database query performance

# 重构代码
refactor: simplify user validation logic

# 测试相关
test: add unit tests for payment module

# 构建/工具
chore: update package.json dependencies

# CI 配置
ci: configure automated testing pipeline

# 回退提交
revert: "fix: resolve login issue"

# 带范围的提交
feat(api): implement user profile endpoints
fix(ui): resolve responsive design issues
refactor(utils): optimize data processing functions
```

## 实际应用

### 场景一：完整的提交信息示例

```bash
# 1. 新功能的完整提交
feat(auth): add OAuth2 authentication support

- Add Google OAuth provider
- Add Facebook OAuth provider
- Implement token refresh mechanism
- Update user model to store OAuth data

Closes #123

# 2. 修复问题的完整提交
fix(api): resolve JSON parsing error in user endpoints

The issue occurred when user data contained special characters.
Updated JSON parser configuration to handle edge cases.

Fixes #456

# 3. 重构的完整提交
refactor(database): optimize query performance

- Replaced multiple queries with batch operations
- Added database indexes for frequently accessed fields
- Implemented query result caching
- Reduced database round trips by 60%

Performance improvement: 2x faster user data retrieval

# 4. 文档更新的完整提交
docs(readme): update installation and setup guide

- Add Docker installation instructions
- Update system requirements
- Add troubleshooting section
- Improve code examples for better clarity

Related to #789
```

### 场景二：项目级别的提交规范

```bash
# 1. 功能模块开发
feat(user): implement complete user management system

feat(user): add user registration functionality
feat(user): add user login and logout
feat(user): implement user profile management
feat(user): add user permission system

# 2. bug 修复流程
fix(user): resolve email validation issues
fix(user): fix session timeout handling
fix(user): resolve user avatar upload problems

# 3. 性能优化流程
perf(api): optimize database queries
perf(frontend): reduce bundle size by 30%
perf(backend): implement response caching

# 4. 测试补充
test(auth): add comprehensive authentication tests
test(api): add integration tests for user endpoints
test(ui): add component tests for user interface
```

### 场景三：使用 Commitizen 工具

```bash
# 1. 安装 Commitizen
npm install -g commitizen
npm install -g cz-conventional-changelog

# 2. 项目中配置
npm install -D commitizen cz-conventional-changelog
echo '{"path":"cz-conventional-changelog"}' > .czrc

# 3. 使用 Commitizen 提交
git add .
git cz  # 代替 git commit

# 4. 交互式提交过程
? Select the type of change that you're committing:
  feat:     A new feature
  fix:      A bug fix
  docs:     Documentation only changes
  ❯ style:   Changes that do not affect the meaning of the code
  refactor: A code change that neither fixes a bug nor adds a feature
  perf:     A code change that improves performance
  test:     Adding missing tests
  chore:    Changes to the build process or auxiliary tools
  ci:       Changes to CI configuration files and scripts

# 5. 填写提交信息
? What is the scope of this change (e.g. component or file name): (press enter to skip)
auth

? Write a short, imperative tense description of the change (max 82 chars):
 (0/82) add login functionality

? Provide a longer description of the change: (press enter to skip)
Implement user login with email and password support.
Include session management and security features.

? Are there any breaking changes? No
? Does this change affect any open issues? No
```

### 场景四：Husky 和 Commitlint 的集成

```bash
# 1. 安装依赖
npm install -D husky @commitlint/cli @commitlint/config-conventional

# 2. 初始化 Husky
npx husky-init
npm install

# 3. 配置 commitlint
echo "export default {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [2, 'always', [
      'feat', 'fix', 'docs', 'style', 'refactor', 'perf', 'test', 'chore', 'ci', 'build', 'revert'
    ]],
    'type-case': [2, 'always', 'lower-case'],
    'type-empty': [2, 'never'],
    'scope-empty': [1, 'never'],
    'scope-case': [2, 'always', 'kebab-case'],
    'subject-empty': [2, 'never'],
    'subject-case': [2, 'never', ['lower-case', 'upper-case']],
    'subject-full-stop': [2, 'never', '.'],
    'body-leading-blank': [1, 'always'],
    'footer-leading-blank': [1, 'always']
  }
};" > commitlint.config.js

# 4. 配置 git hooks
echo "npx --no -- commitlint --edit \$1" > .husky/commit-msg
chmod +x .husky/commit-msg

# 5. 测试配置
git add .
git commit -m "invalid commit message"
# 错误：不符合规范的提交将被拒绝

git commit -m "feat: add user authentication"
# 成功：符合规范的提交将被接受
```

### 场景五：自动化生成 Changelog

```bash
# 1. 安装 conventional-changelog
npm install -g conventional-changelog-cli

# 2. 初始化 changelog
conventional-changelog -p angular -i CHANGELOG.md -s

# 3. 手动更新 changelog
conventional-changelog -p angular -i CHANGELOG.md -s -r 0

# 4. 自动化集成到 package.json
{
  "scripts": {
    "changelog": "conventional-changelog -p angular -i CHANGELOG.md -s -r 0"
  }
}

# 5. 运行脚本生成 changelog
npm run changelog

# 生成的 CHANGELOG.md 示例：
# # Changelog
# All notable changes to this project will be documented in this file.
# The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
# and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).
#
# ## [1.0.0] - 2024-01-15
#
# ### Added
# - User authentication system
# - User profile management
# - OAuth2 integration
#
# ### Fixed
# - Login page redirect issue
# - Email validation problems
#
# ### Performance
# - Database query optimization
# - Frontend bundle size reduction
```

### 场景六：团队提交规范配置

```bash
# 1. 创建团队提交规范配置文件
cat > .commitlintrc.js << 'EOF'
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [2, 'always', [
      'feat', 'fix', 'docs', 'style', 'refactor', 'perf', 'test', 'chore', 'ci', 'build', 'revert'
    ]],
    'type-case': [2, 'always', 'lower-case'],
    'type-empty': [2, 'never'],
    'scope-empty': [0], // 允许空的 scope
    'scope-enum': [2, 'always', [
      'auth', 'user', 'api', 'ui', 'db', 'config', 'utils', 'components'
    ]],
    'subject-empty': [2, 'never'],
    'subject-case': [0], // 不限制 subject 的大小写
    'body-leading-blank': [1, 'always'],
    'footer-leading-blank': [1, 'always'],
    'max-header-length': [2, 'always', 100],
    'max-body-line-length': [2, 'always', 100]
  }
};
EOF

# 2. 创建提交指南
cat > COMMIT_GUIDE.md << 'EOF'
# Git 提交规范指南

## 提交格式
```
<type>(<scope>): <subject>

<body>

<footer>
```

## 提交类型
- `feat`: 新功能
- `fix`: 修复 bug
- `docs`: 文档变更
- `style`: 代码格式
- `refactor`: 重构
- `perf`: 性能优化
- `test`: 测试
- `chore`: 构建/工具
- `ci`: CI 配置
- `build`: 构建系统
- `revert`: 回退

## 提交范围
- `auth`: 认证相关
- `user`: 用户相关
- `api`: API 相关
- `ui`: 用户界面
- `db`: 数据库
- `config`: 配置
- `utils`: 工具函数
- `components`: 组件

## 示例
```
feat(auth): add OAuth2 login support

- Add Google OAuth
- Add Facebook OAuth
- Implement token refresh

Closes #123
```
EOF

# 3. 在 README 中添加链接
echo "提交规范：[查看指南](./COMMIT_GUIDE.md)" >> README.md
```

## 注意事项

1. **提交信息简洁明了**：Subject 应该简洁明了，描述做了什么而不是为什么

2. **使用祈使句**：Subject 应该使用祈使句开头，如 "add" 而不是 "adds" 或 "added"

3. **限制长度**：Subject 限制在 50-72 个字符，每行 body 限制在 72 个字符

4. **关联 Issue**：在 footer 中关联相关的 Issue，使用 "Closes" 或 "Fixes"

5. **避免合并提交**：尽量避免直接使用 "Merge branch" 作为提交信息

6. **区分变更类型**：正确使用不同的提交类型，不要混淆

7. **提供上下文**：Body 应该提供足够的上下文信息，但不要过于详细

8. **团队统一**：整个团队应遵循相同的提交规范

## 高级技巧

### 自定义提交规范

```bash
# 1. 创建自定义提交规范
cat > custom-conventional-commits.js << 'EOF'
module.exports = {
  types: [
    { value: 'feat', name: 'feat:     A new feature', title: 'Features' },
    { value: 'fix', name: 'fix:      A bug fix', title: 'Bug Fixes' },
    { value: 'docs', name: 'docs:     Documentation only changes', title: 'Documentation' },
    { value: 'style', name: 'style:    Changes that do not affect the meaning of the code', title: 'Styles' },
    { value: 'refactor', name: 'refactor: A code change that neither fixes a bug nor adds a feature', title: 'Code Refactoring' },
    { value: 'perf', name: 'perf:     A code change that improves performance', title: 'Performance Improvements' },
    { value: 'test', name: 'test:     Adding missing tests', title: 'Tests' },
    { value: 'chore', name: 'chore:    Changes to the build process or auxiliary tools', title: 'Chores' },
    { value: 'ci', name: 'ci:       Changes to CI configuration files and scripts', title: 'Continuous Integration' },
    { value: 'build', name: 'build:    Changes that affect the build system or external dependencies', title: 'Builds' },
    { value: 'revert', name: 'revert:   Reverts a previous commit', title: 'Reverts' }
  ],
  scopes: [
    { name: 'auth' },
    { name: 'user' },
    { name: 'api' },
    { name: 'ui' },
    { name: 'db' },
    { name: 'config' },
    { name: 'utils' },
    { name: 'components' }
  ],
  messages: {
    type: 'Select the type of change that you\'re committing:',
    scope: '\nWhat is the scope of this change (e.g. component or file name)?',
    subject: '\nWrite a short, imperative tense description of the change:\n',
    body: 'Provide a longer description of the change:\n',
    breaking: 'List any BREAKING CHANGES:\n',
    footer: 'Is this a BREAKING CHANGE? Are there any related issues? (e.g. "fix #123")\n',
    confirmCommit: 'Are you sure you want to proceed with the commit above?'
  }
};
EOF

# 2. 配置使用自定义规范
cat > .czrc << 'EOF'
{
  "path": "./custom-conventional-commits.js"
}
EOF

# 3. 测试自定义规范
git add .
git cz
```

### 自动化工具集成

```bash
# 1. 在 CI/CD 中验证提交信息
# .github/workflows/commitlint.yml
cat > .github/workflows/commitlint.yml << 'EOF'
name: Commitlint
on: [push, pull_request]
jobs:
  commitlint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0
      - uses: wagoid/commitlint-github-action@v5
EOF

# 2. 预提交钩子
cat > .husky/pre-commit << 'EOF'
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# 运行代码检查
npm run lint:fix

# 运行测试
npm run test
EOF

chmod +x .husky/pre-commit

# 3. 提交信息钩子
cat > .husky/commit-msg << 'EOF'
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# 验证提交信息格式
npx commitlint --edit $1

# 检查是否关联了 issue
grep -E "(Fixes|Closes|Resolves) #[0-9]+" $1 || \
  echo "Warning: Consider linking to an issue in the commit message"
EOF

chmod +x .husky/commit-msg
```

## 最佳实践

### 提交规范检查清单

```bash
# 提交前检查清单
- [ ] 提交类型正确
- [ ] 范围（如有）准确
- [ ] Subject 简洁明了
- [ ] 使用祈使句开头
- [ ] 长度符合要求
- [ ] Body 提供足够上下文
- [ ] 关联相关的 Issue
- [ ] 通过 lint-staged 检查
- [ ] 通过 commitlint 验证
```

### 团队协作建议

1. **制定规范**：制定团队统一的提交规范
2. **工具支持**：配置自动化工具强制执行规范
3. **培训指导**：对团队成员进行培训和指导
4. **定期审查**：定期审查提交信息质量
5. **持续改进**：根据实际情况优化规范

### 工具推荐

```bash
# 必备工具
commitizen          # 交互式提交工具
commitlint          # 提交信息检查
husky              # Git hooks 管理
lint-staged        # 暂存文件检查
conventional-changelog  # 自动生成 changelog

# 可选工具
semantic-release   # 自动版本发布
standard-version   # 版本管理工具
@commitlint/cli    # Commitlint 命令行工具
cz-customizable    # 可定制的 Commitizen 适配器
```

## 总结

Git Commit 规范是提高代码质量和团队协作效率的重要实践。通过采用统一的提交规范，我们可以：

- 提高代码历史的可读性和可维护性
- 支持自动化工具的使用
- 简化代码审查过程
- 便于问题追踪和版本管理

要成功实施提交规范，需要：

1. 选择合适的规范（如 Conventional Commits）
2. 配置自动化工具（Commitizen、Commitlint、Husky）
3. 培训团队成员理解和使用规范
4. 定期检查和优化规范实施效果

记住，提交规范不仅是一种格式要求，更是一种团队协作的沟通方式。通过良好的提交规范，可以大大提高项目的可维护性和团队协作效率。