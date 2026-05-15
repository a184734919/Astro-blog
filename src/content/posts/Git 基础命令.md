---
title: Git 基础命令
published: 2024-11-01
description: 'Git 常用命令大全的详细介绍和学习笔记'
image: ''
tags: ["Git","工具"]
category: '开发工具'
draft: false
lang: 'zh-CN'
---

## 概述

Git 是目前世界上最先进的分布式版本控制系统，它能够高效地处理从小到大的各种项目。掌握 Git 基础命令是每个开发者必备的技能，本文将详细介绍 Git 的常用命令及其实际应用。

## 核心概念

### Git 的三个工作区域

1. **工作区（Working Directory）**：你在电脑里能看到的目录
2. **暂存区（Staging Area）**：也叫索引区，一般存放在 `.git/index` 文件中
3. **仓库区（Repository）**：工作区有一个隐藏目录 `.git`，这是 Git 的版本库

### 文件状态

- **已修改（modified）**：文件被修改但还没有保存到数据库中
- **已暂存（staged）**：对一个已修改文件的当前版本做了标记
- **已提交（committed）**：数据已经安全地保存在本地数据库中

## 基本用法

### 初始化和配置

```bash
# 初始化新的 Git 仓库
git init

# 克隆远程仓库
git clone <repository-url>

# 配置用户信息（全局）
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# 配置用户信息（当前仓库）
git config user.name "Your Name"
git config user.email "your.email@example.com"

# 查看配置信息
git config --list
```

### 文件操作命令

```bash
# 查看文件状态
git status

# 添加文件到暂存区
git add <file-name>          # 添加指定文件
git add .                    # 添加当前目录所有文件
git add *.js                 # 添加所有 .js 文件
git add -A                   # 添加所有变化的文件

# 移除文件
git rm <file-name>           # 删除文件并暂存
git rm --cached <file-name>  # 从暂存区移除但保留工作区文件

# 重命名文件
git mv <old-name> <new-name>
```

### 提交操作

```bash
# 提交暂存区的更改
git commit -m "commit message"

# 跳过暂存区直接提交
git commit -am "commit message"

# 修改最后一次提交
git commit --amend

# 修改提交信息
git commit --amend -m "new commit message"
```

### 查看历史和差异

```bash
# 查看提交历史
git log
git log --oneline           # 单行显示
git log --graph --oneline   # 图形化显示
git log -5                  # 显示最近5次提交
git log --since="2 days ago" # 显示最近2天的提交

# 查看文件差异
git diff                    # 工作区和暂存区差异
git diff --staged           # 暂存区和仓库区差异
git diff HEAD               # 工作区和仓库区差异
git diff <commit-hash>      # 与指定提交的差异
git diff <branch1> <branch2> # 两个分支的差异

# 查看指定文件的修改历史
git log --follow <file-name>
git blame <file-name>       # 查看文件每一行的修改者
```

### 撤销操作

```bash
# 撤销工作区的修改
git checkout -- <file-name>
git restore <file-name>     # Git 2.23+ 新命令

# 撤销暂存区的修改
git reset HEAD <file-name>
git restore --staged <file-name> # Git 2.23+ 新命令

# 回退到指定版本（保留工作区修改）
git reset --soft <commit-hash>

# 回退到指定版本（保留工作区和暂存区修改）
git reset --mixed <commit-hash>  # 默认选项

# 回退到指定版本（删除所有修改）
git reset --hard <commit-hash>

# 撤销指定提交（创建新提交）
git revert <commit-hash>
```

## 实际应用

### 场景一：开始新项目

```bash
# 1. 创建项目目录
mkdir my-project
cd my-project

# 2. 初始化 Git 仓库
git init

# 3. 创建 .gitignore 文件
echo "node_modules/" > .gitignore
echo ".env" >> .gitignore

# 4. 创建项目文件
# ... 编写代码 ...

# 5. 添加文件到暂存区
git add .

# 6. 提交代码
git commit -m "Initial commit"

# 7. 关联远程仓库
git remote add origin https://github.com/username/my-project.git

# 8. 推送到远程仓库
git push -u origin main
```

### 场景二：日常开发工作流

```bash
# 1. 拉取最新代码
git pull origin main

# 2. 创建功能分支
git checkout -b feature/user-auth

# 3. 开发功能
# ... 编写代码 ...

# 4. 查看修改状态
git status

# 5. 查看具体修改内容
git diff

# 6. 添加修改文件
git add src/auth.js

# 7. 提交代码
git commit -m "feat: add user authentication"

# 8. 推送分支
git push origin feature/user-auth
```

### 场景三：修改已提交的代码

```bash
# 发现最后一个提交有错误

# 方法1：如果还没有推送到远程
git add fixed-file.js
git commit --amend -m "corrected commit message"

# 方法2：如果已经推送到远程，使用 revert
git revert HEAD
git push origin feature-branch
```

### 场景四：查看特定提交的修改

```bash
# 查看特定提交的详细信息
git show <commit-hash>

# 查看特定提交修改的文件
git show --name-only <commit-hash>

# 查看特定提交修改的文件和行数
git show --stat <commit-hash>
```

## 注意事项

1. **提交前检查**：每次提交前都要检查 `git status` 和 `git diff`，确保只提交需要的内容

2. **避免使用 `git add .`**：在团队协作中，建议使用 `git add <specific-file>` 来精确控制要提交的文件

3. **谨慎使用 `--amend`**：如果你已经推送了提交，不要使用 `--amend`，这会改变提交历史

4. **定期提交**：小而频繁的提交比大而少的提交更容易管理和回溯

5. **提交信息规范**：使用清晰、描述性的提交信息，遵循项目的提交规范

6. **忽略文件**：正确配置 `.gitignore` 文件，避免提交不必要的文件

7. **备份重要分支**：在进行危险操作前，先创建备份分支

## 总结

Git 基础命令是日常开发中最常用的操作，掌握这些命令可以大大提高开发效率。重点包括：

- 理解 Git 的三个工作区域和文件状态
- 熟练使用 `git add`、`git commit`、`git push` 等基本操作
- 掌握查看历史和差异的命令
- 学会各种撤销操作的适用场景
- 遵循最佳实践，养成良好的 Git 使用习惯

通过不断练习和实践，这些命令会成为你的肌肉记忆，让你在版本控制方面更加得心应手。