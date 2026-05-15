---
title: Git Stash
published: 2024-12-02
description: '暂存工作区的详细介绍和学习笔记'
image: ''
tags: ["Git","工具"]
category: '开发工具'
draft: false
lang: 'zh-CN'
---

## 概述

Git Stash 是一个强大的工具，用于临时保存当前工作目录的修改，让你可以在不提交的情况下切换分支或进行其他操作。这在需要频繁切换上下文、临时处理紧急任务或保存实验性修改时特别有用。

## 核心概念

### Stash 的作用

- **临时保存**：不提交的情况下保存工作区修改
- **上下文切换**：在不丢失修改的情况下切换分支
- **任务中断**：处理紧急任务时保存当前进度
- **实验性修改**：保存不确定的修改，随时可以恢复或丢弃

### Stash 的存储结构

每个 stash 实际上是一个 commit，包含：
- 工作区的修改
- 暂存区的修改
- 提交信息和时间戳

### Stash 的生命周期

```
工作区修改 → Stash → 保存到栈 → 恢复/丢弃
```

## 基本用法

### 基本 Stash 操作

```bash
# 保存当前修改到 stash（包括工作区和暂存区）
git stash

# 保存修改并添加描述信息
git stash save "描述信息"

# 只保存工作区的修改（不包括暂存区）
git stash push -m "只保存工作区"

# 只保存暂存区的修改
git stash push --staged -m "只保存暂存区"

# 保存特定文件的修改
git stash push src/file1.js src/file2.js

# 保存包括未跟踪文件的所有修改
git stash push -u -m "包含未跟踪文件"

# 保存所有修改（包括忽略文件）
git stash push -a -m "包含所有文件"
```

### 查看 Stash 列表

```bash
# 查看所有 stash
git stash list

# 查看最近的 stash
git stash show stash@{0}

# 查看 stash 的详细内容
git stash show -p stash@{0}

# 查看 stash 的统计信息
git stash show --stat stash@{0}
```

### 恢复 Stash

```bash
# 恢复最近的 stash 并从列表中删除
git stash pop

# 恢复指定的 stash
git stash pop stash@{1}

# 恢复 stash 但不从列表中删除
git stash apply

# 恢复指定 stash 但不从列表中删除
git stash apply stash@{1}

# 只恢复 stash 中的工作区修改
git stash apply --index

# 恢复到新分支
git stash branch new-branch-name stash@{0}
```

### 删除 Stash

```bash
# 删除最近的 stash
git stash drop

# 删除指定的 stash
git stash drop stash@{1}

# 删除所有 stash
git stash clear

# 删除指定范围的 stash（需要 Git 2.32+）
git stash drop stash@{0}..stash@{2}
```

## 实际应用

### 场景一：紧急任务处理

```bash
# 1. 正在开发功能，突然需要处理紧急 bug
git status
# modified:   src/auth.js
# modified:   src/user.js

# 2. 保存当前进度
git stash push -m "开发中：用户认证功能（进度 80%）"

# 3. 查看保存的 stash
git stash list
# stash@{0}: On main: 开发中：用户认证功能（进度 80%）

# 4. 切换到主分支处理紧急任务
git checkout main
git checkout -b hotfix/critical-bug

# 5. 修复 bug 并提交
# ... 修复代码 ...
git add .
git commit -m "fix: resolve critical authentication bug"

# 6. 切换回原来的分支
git checkout feature/user-auth

# 7. 恢复之前保存的进度
git stash pop

# 8. 继续开发
git status
# modified:   src/auth.js
# modified:   src/user.js
```

### 场景二：实验性功能测试

```bash
# 1. 有一些不确定的修改想测试
# ... 进行实验性修改 ...

# 2. 保存实验性修改
git stash push -m "实验：新的算法实现"

# 3. 进行其他开发
git add .
git commit -m "feat: add user profile feature"

# 4. 测试后发现实验性修改有用，恢复它
git stash apply stash@{0}

# 5. 进一步完善实验性功能
# ... 完善代码 ...

# 6. 提交完善后的代码
git add .
git commit -m "feat: implement optimized algorithm"

# 7. 删除原来的 stash
git stash drop stash@{0}
```

### 场景三：多任务并行开发

```bash
# 1. 正在开发用户功能
git checkout -b feature/user-management
# ... 进行用户功能开发 ...

# 2. 需要同时开发订单功能，保存用户功能进度
git stash push -m "用户功能：基础 CRUD 完成"

# 3. 开发订单功能
git checkout -b feature/order-management
# ... 订单功能开发 ...
git add .
git commit -m "feat: implement order creation"

# 4. 需要回到用户功能
git checkout feature/user-management
git stash pop

# 5. 继续用户功能开发
# ... 继续开发 ...
git add .
git commit -m "feat: complete user management features"

# 6. 再次切换到订单功能
git checkout feature/order-management
# ... 继续订单功能开发 ...
```

### 场景四：复杂的 Stash 管理

```bash
# 1. 多个任务的 stash
git stash push -m "任务1：用户登录功能"
git stash push -m "任务2：权限管理"
git stash push -m "任务3：数据导出功能"

# 2. 查看所有 stash
git stash list
# stash@{0}: On main: 任务3：数据导出功能
# stash@{1}: On main: 任务2：权限管理
# stash@{2}: On main: 任务1：用户登录功能

# 3. 选择恢复特定的 stash
git stash apply stash@{1}

# 4. 在新分支上恢复 stash
git stash branch feature/permission-system stash@{1}

# 5. 查看每个 stash 的详细内容
git stash show -p stash@{0}
git stash show -p stash@{1}
git stash show -p stash@{2}

# 6. 根据内容决定保留或删除
git stash drop stash@{2}  # 任务1已完成，删除
```

### 场景五：处理特定文件的 Stash

```bash
# 1. 有多个文件被修改
git status
# modified:   src/auth.js
# modified:   src/user.js
# modified:   src/payment.js

# 2. 只保存特定文件的修改
git stash push src/payment.js -m "支付功能实验"

# 3. 其他文件继续开发
git add src/auth.js src/user.js
git commit -m "feat: improve auth and user modules"

# 4. 恢复支付功能的实验性修改
git stash pop

# 5. 或者只保存暂存区的修改
git add src/auth.js
git commit -m "feat: add auth improvements"
git stash push --staged -m "暂存区的修改"

# 6. 恢复暂存区的修改
git stash apply --index
```

### 场景六：调试和错误追踪

```bash
# 1. 遇到复杂的 bug，需要多次尝试不同的修复方案
# ... 尝试第一种修复方案 ...
git stash push -m "修复方案1：增加错误处理"

# ... 尝试第二种修复方案 ...
git stash push -m "修复方案2：修改数据结构"

# ... 尝试第三种修复方案 ...
git stash push -m "修复方案3：调整算法逻辑"

# 2. 测试发现方案2最有效
git stash apply stash@{1}

# 3. 进一步完善方案2
# ... 完善代码 ...

# 4. 提交最终方案
git add .
git commit -m "fix: resolve complex data processing issue"

# 5. 删除其他方案的 stash
git stash drop stash@{0}
git stash drop stash@{2}
```

## 注意事项

1. **不要丢失重要修改**：在 drop stash 前确保不再需要这些修改

2. **描述信息很重要**：使用清晰的描述信息，方便后续识别和管理

3. **定期清理**：定期清理不再需要的 stash，避免堆积

4. **冲突处理**：恢复 stash 时可能遇到冲突，需要手动解决

5. **包含文件范围**：根据需要选择是否包含未跟踪文件和忽略文件

6. **分支切换**：stash 可以在不同分支间切换，但要注意文件路径差异

7. **敏感信息**：不要 stash 包含敏感信息的文件

8. **大文件注意**：stash 会保存完整的文件内容，大文件会增加仓库大小

## 高级技巧

### Stash 与工作流集成

```bash
# 在 Git Hook 中自动使用 stash
#!/bin/bash
# pre-commit hook
if git diff --quiet && git diff --cached --quiet; then
    echo "没有需要提交的修改"
    exit 1
fi

# 检查是否有未完成的测试
if ! npm test; then
    echo "测试失败，保存修改以便调试"
    git stash push -m "测试失败保存的修改"
    exit 1
fi
```

### 定期自动 Stash

```bash
# 定期保存工作状态（如每小时）
#!/bin/bash
while true; do
    if ! git diff --quiet; then
        git stash push -m "自动保存 $(date '+%Y-%m-%d %H:%M:%S')"
    fi
    sleep 3600  # 1小时
done
```

### Stash 与分支管理结合

```bash
# 自动创建分支并恢复 stash
git_stash_branch() {
    local stash_num=$1
    local branch_name=$2
    git stash branch "$branch_name" stash@{$stash_num}
}

# 使用示例
git_stash_branch 0 "feature/quick-fix"
```

### 查看和比较 Stash

```bash
# 比较两个 stash 的差异
git diff stash@{0} stash@{1}

# 比较当前工作区和 stash 的差异
git diff stash@{0}

# 查看某个文件的在 stash 中的版本
git show stash@{0}:src/file.js

# 将 stash 中的文件恢复到新位置
git show stash@{0}:old/path/file.js > new/path/file.js
```

### Stash 性能优化

```bash
# 对于大型项目，只 stash 必要的文件
git stash push src/important-file.js -m "关键修改"

# 使用 --include-untracked 和 --all 要谨慎
# 只在确实需要时使用这些选项
```

## 最佳实践

### Stash 使用规范

1. **清晰的命名**：使用有意义的描述信息，如"功能：用户认证（进度 60%）"
2. **及时清理**：定期清理过期的 stash
3. **不要滥用**：对于确定性的修改，应该直接提交而不是 stash
4. **团队协作**：在团队协作中，尽量避免长期使用 stash
5. **文档记录**：对于重要的 stash，可以在项目中记录其用途

### Stash 与 Commit 的选择

```bash
# 使用 Stash 的情况：
- 临时切换任务
- 实验性修改
- 不确定的功能实现
- 调试过程中的尝试

# 使用 Commit 的情况：
- 功能性的修改
- 已经测试通过的代码
- 需要分享给团队的修改
- 重要的里程碑
```

### Stash 管理建议

```bash
# 定期清理过期 stash
git_clean_old_stashes() {
    # 删除超过 7 天的 stash
    git stash list --date=local | awk '/[0-9]+ days ago/ {print $1}' | \
    sed 's/:$//' | while read stash; do
        echo "删除过期 stash: $stash"
        git stash drop "$stash"
    done
}

# 设置别名方便使用
git config --global alias.stash-list 'stash list --date=local'
git config --global alias.stash-show 'stash show -p'
```

## 总结

Git Stash 是一个灵活实用的工具，主要应用场景包括：

- 临时保存工作进度，处理紧急任务
- 保存实验性修改，随时恢复或丢弃
- 多任务并行开发时切换上下文
- 调试过程中保存不同的尝试方案

掌握 Stash 的使用可以大大提高开发效率，特别是在复杂的项目和多任务环境中。通过合理使用 Stash，你可以更好地管理代码状态，避免丢失重要的修改，并保持开发流程的灵活性。

记住，Stash 虽然强大，但不应该替代正常的提交流程。对于稳定和重要的修改，还是应该及时提交并遵循项目的分支管理策略。