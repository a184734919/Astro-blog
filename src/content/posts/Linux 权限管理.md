---
title: Linux 权限管理
published: 2025-03-23
description: '文件权限设置的详细介绍和学习笔记'
image: ''
tags: ["Linux","服务器"]
category: '开发工具'
draft: false
lang: 'zh-CN'
---

## 概述

Linux 权限管理是系统安全的核心机制，它控制着用户对文件和目录的访问权限。理解并正确设置文件权限对于系统安全、数据保护和多用户协作至关重要。Linux 采用基于用户的权限模型，通过精细的权限控制确保系统安全。

## 核心概念

### 用户和用户组

Linux 权限系统基于三个基本概念：

- **用户（User）**：系统的账户，每个用户都有唯一的用户 ID（UID）
- **用户组（Group）**：用户的集合，便于权限管理
- **其他（Others）**：既不是文件所有者也不在文件所属组的用户

### 权限类型

每个文件有三种基本权限：

- **读权限（r，Read）**：查看文件内容或列出目录内容
- **写权限（w，Write）**：修改文件内容或在目录中创建/删除文件
- **执行权限（x，Execute）**：执行文件或进入目录

### 权限表示法

Linux 权限有两种表示方式：

#### 1. 符号表示法

```bash
drwxr-xr-x 2 user group 4096 Jan 15 10:30 directory
-rw-r--r-- 1 user group  123 Jan 15 10:30 file.txt
-rwxr-x--- 1 user group  456 Jan 15 10:30 script.sh
```

- 第一个字符表示文件类型：
  - `-`：普通文件
  - `d`：目录
  - `l`：符号链接
  - `b`：块设备
  - `c`：字符设备
- 接下来的 9 个字符分为三组：
  - 前 3 个：所有者权限
  - 中间 3 个：所属组权限
  - 后 3 个：其他用户权限

#### 2. 数字表示法

每个权限对应的数字：
- `r` = 4
- `w` = 2
- `x` = 1
- `-` = 0

常见权限组合：
- `7` = rwx (4+2+1) = 读写执行
- `6` = rw- (4+2+0) = 读写
- `5` = r-x (4+0+1) = 读执行
- `4` = r-- (4+0+0) = 只读
- `0` = --- (0+0+0) = 无权限

示例：
- `755` = rwxr-xr-x
- `644` = rw-r--r--
- `600` = rw-------
- `777` = rwxrwxrwx

### 特殊权限

除了基本权限外，Linux 还有三种特殊权限：

#### 1. SetUID (SUID)
- 用数字 `4` 表示
- 设置在可执行文件上，用户执行该程序时，会以文件所有者的权限运行
- 常见于 `passwd`、`sudo` 等命令
- 在权限表示中显示为 `s`（替代用户位的 `x`）

#### 2. SetGID (SGID)
- 用数字 `2` 表示
- 设置在目录上，新创建的文件会继承目录的所属组
- 设置在可执行文件上，执行时以文件所属组的权限运行
- 在权限表示中显示为 `s`（替代组位的 `x`）

#### 3. Sticky Bit
- 用数字 `1` 表示
- 主要设置在目录上，只有文件所有者才能删除目录中的文件
- 常见于 `/tmp` 目录
- 在权限表示中显示为 `t`（替代其他位的 `x`）

特殊权限的数字表示：
- `4755` = rwsr-xr-x (SetUID)
- `2755` = rwxr-sr-x (SetGID)
- `1755` = rwxr-xr-t (Sticky Bit)
- `7755` = rwsr-sr-t (所有特殊权限)

## 基本用法

### 查看文件权限

```bash
# 查看文件权限
ls -la filename
ls -ld directory/

# 查看文件详细属性
stat filename

# 查看目录权限
ls -la | grep "^d"
find /path -type d -exec ls -ld {} \;

# 查找特定权限的文件
find /path -perm 777
find /path -perm 644
find /path -perm -4000  # 查找 SetUID 文件
find /path -perm -2000  # 查找 SetGID 文件
```

### 修改文件权限（chmod）

#### 使用数字方式

```bash
# 基本权限设置
chmod 644 file.txt          # rw-r--r--
chmod 755 script.sh         # rwxr-xr-x
chmod 700 private_key       # rw-------
chmod 777 public_dir        # rwxrwxrwx

# 递归设置权限
chmod -R 755 /var/www/html/
chmod -R 644 /var/www/html/*.html

# 特殊权限
chmod 4755 /usr/bin/passwd  # SetUID
chmod 2755 /usr/bin/write   # SetGID
chmod 1777 /tmp             # Sticky Bit
```

#### 使用符号方式

```bash
# 基本操作符
# + 添加权限
# - 删除权限
# = 设置精确权限

# 操作对象
# u (user) 所有者
# g (group) 所属组
# o (others) 其他用户
# a (all) 所有用户

# 添加权限
chmod +x script.sh          # 为所有用户添加执行权限
chmod u+x script.sh         # 为所有者添加执行权限
chmod g+rw file.txt         # 为所属组添加读写权限
chmod o+r file.txt          # 为其他用户添加读权限

# 删除权限
chmod -x script.sh          # 删除执行权限
chmod go-w file.txt         # 删除组和其他用户的写权限

# 设置精确权限
chmod u=rwx,g=rx,o=r file.txt
chmod a=rx script.sh

# 组合操作
chmod u+x,g+w,o-w file.txt
chmod a-x,u+w file.txt

# 递归操作
chmod -R g+w /var/www/html/
chmod -R o-rwx /home/user/private/
```

### 修改文件所有者（chown）

```bash
# 修改所有者
chown user file.txt
chown user:group file.txt

# 只修改所属组
chown :group file.txt
chown user: file.txt

# 递归修改
chown -R user:group /var/www/html/
chown -R user /home/user/projects/

# 同时修改权限和所有者
chown user:group file.txt && chmod 644 file.txt

# 查看用户信息
id username
groups username
```

### 修改文件所属组（chgrp）

```bash
# 修改所属组
chgrp group file.txt
chgrp -R group /path/to/directory/

# 查看组信息
groups
cat /etc/group
```

### 默认权限设置（umask）

```bash
# 查看 umask 值
umask
umask -S

# 设置 umask
umask 022          # 默认值，新文件权限 644，目录权限 755
umask 077          # 新文件权限 600，目录权限 700

# 临时设置（当前会话有效）
umask 077

# 永久设置（添加到 ~/.bashrc 或 ~/.bash_profile）
echo "umask 077" >> ~/.bashrc

# umask 计算规则
# 文件默认权限 = 666 - umask
# 目录默认权限 = 777 - umask

# 示例
umask 022
# 新文件：666 - 022 = 644 (rw-r--r--)
# 新目录：777 - 022 = 755 (rwxr-xr-x)

umask 077
# 新文件：666 - 077 = 600 (rw-------)
# 新目录：777 - 077 = 700 (rwx------)
```

### 特殊权限设置

```bash
# SetUID
chmod 4755 script.sh          # 数字方式
chmod u+s script.sh           # 符号方式

# SetGID
chmod 2755 directory/         # 数字方式
chmod g+s directory/          # 符号方式

# Sticky Bit
chmod 1777 /tmp/              # 数字方式
chmod +t /tmp/                # 符号方式

# 移除特殊权限
chmod 755 script.sh           # 移除 SetUID
chmod 755 directory/          # 移除 SetGID
chmod 777 /tmp/               # 移除 Sticky Bit

# 查找特殊权限文件
find / -perm -4000 -type f    # SetUID 文件
find / -perm -2000 -type f    # SetGID 文件
find / -perm -1000 -type d    # Sticky Bit 目录
```

## 实际应用

### 场景一：Web 服务器配置

```bash
# 1. 创建网站目录结构
sudo mkdir -p /var/www/example.com/{public,private,logs}
sudo mkdir -p /var/www/example.com/public/{css,js,images}

# 2. 设置正确的所有者和权限
sudo chown -R www-data:www-data /var/www/example.com/
sudo chmod -R 755 /var/www/example.com/
sudo chmod -R 644 /var/www/example.com/public/*
sudo chmod -R 600 /var/www/example.com/private/*

# 3. 设置上传目录权限
sudo mkdir -p /var/www/example.com/public/uploads
sudo chmod -R 775 /var/www/example.com/public/uploads/
sudo chown -R www-data:www-data /var/www/example.com/public/uploads/

# 4. 配置日志目录权限
sudo chmod -R 750 /var/www/example.com/logs/
sudo touch /var/www/example.com/logs/access.log
sudo touch /var/www/example.com/logs/error.log
sudo chmod 640 /var/www/example.com/logs/*.log

# 5. 设置 SSL 证书权限
sudo mkdir -p /etc/ssl/example.com
sudo chmod 700 /etc/ssl/example.com/
sudo cp cert.key cert.crt /etc/ssl/example.com/
sudo chmod 600 /etc/ssl/example.com/cert.key
sudo chmod 644 /etc/ssl/example.com/cert.crt

# 6. 验证权限设置
ls -la /var/www/example.com/
find /var/www/example.com/ -type f -exec ls -la {} \;
```

### 场景二：开发环境配置

```bash
# 1. 创建项目目录
mkdir -p ~/projects/webapp
cd ~/projects/webapp

# 2. 设置项目文件权限
chmod 644 *.html *.css *.js
chmod 755 scripts/
chmod +x scripts/*.sh

# 3. 配置 SSH 密钥权限
mkdir -p ~/.ssh
chmod 700 ~/.ssh/
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# 4. 配置数据库连接文件
echo "DB_PASSWORD=secret" > .env
chmod 600 .env

# 5. 设置 Git 钩子权限
mkdir -p .git/hooks
chmod +x .git/hooks/pre-commit

# 6. 设置共享目录权限
mkdir -p ~/shared
chmod 770 ~/shared
sudo chown :developers ~/shared
```

### 场景三：系统安全加固

```bash
# 1. 查找并审核 SetUID 文件
find / -perm -4000 -type f -exec ls -la {} \;

# 2. 移除不必要的 SetUID 权限
sudo chmod 755 /usr/bin/ping          # 如果不需要普通用户使用
sudo chmod 755 /usr/bin/mount
sudo chmod 755 /usr/bin/umount

# 3. 保护重要配置文件
sudo chmod 600 /etc/shadow
sudo chmod 644 /etc/passwd
sudo chmod 644 /etc/group

# 4. 限制用户主目录权限
chmod 700 /home/user/
chmod 755 /home/

# 5. 保护系统日志
sudo chmod 640 /var/log/auth.log
sudo chmod 640 /var/log/syslog

# 6. 设置临时目录权限
sudo chmod 1777 /tmp
sudo chmod 1777 /var/tmp

# 7. 限制 cron 权限
sudo chmod 600 /etc/crontab
sudo chmod 700 /etc/cron.d/
```

### 场景四：多用户协作环境

```bash
# 1. 创建共享项目组
sudo groupadd developers
sudo groupadd designers

# 2. 创建用户并添加到组
sudo useradd -m -G developers,developers alice
sudo useradd -m -G designers,developers bob

# 3. 创建项目目录
sudo mkdir -p /projects/{webapp,mobileapp}
sudo chown -R :developers /projects/webapp/
sudo chown -R :designers /projects/mobileapp/

# 4. 设置 SetGID，确保新文件继承组权限
sudo chmod 2775 /projects/webapp/
sudo chmod 2775 /projects/mobileapp/

# 5. 设置用户主目录权限
sudo chmod 755 /home/alice/
sudo chmod 755 /home/bob/

# 6. 配置共享文档目录
sudo mkdir -p /shared/documents
sudo chown -R :developers /shared/documents/
sudo chmod 2770 /shared/documents/

# 7. 设置默认权限
echo "umask 002" | sudo tee -a /home/alice/.bashrc
echo "umask 002" | sudo tee -a /home/bob/.bashrc
```

## 注意事项

1. **安全最佳实践**：
   - 避免使用 777 权限，这会带来严重的安全风险
   - 敏感文件（如密钥、配置）应该设置为最严格的权限
   - 定期审计系统中的特殊权限文件

2. **权限设置原则**：
   - 最小权限原则：只给予必要的最小权限
   - 按需分配：根据用户需求分配权限
   - 定期审查：定期检查和更新权限设置

3. **目录与文件权限区别**：
   - 目录需要执行权限才能进入
   - 目录的写权限允许创建和删除文件
   - 删除文件需要对父目录有写权限

4. **特殊权限使用**：
   - SetUID 只在必要时使用，且要确保程序安全
   - SetGID 适合用于协作目录
   - Sticky Bit 适合用于公共目录（如 /tmp）

5. **权限继承**：
   - 新文件不会自动继承目录的执行权限
   - 使用 umask 控制新文件的默认权限
   - SetGID 可以让新文件继承目录的组权限

6. **权限问题排查**：
   - 使用 `ls -la` 检查权限设置
   - 检查文件所有者和所属组
   - 验证用户是否在正确的组中
   - 查看 SELinux 或 AppArmor 的影响

## 总结

Linux 权限管理是系统安全的基础，正确设置权限可以保护系统安全和数据完整性。重点包括：

- **权限模型**：理解用户、组、其他三种权限对象
- **权限类型**：掌握读、写、执行三种基本权限
- **权限表示**：熟悉符号和数字两种权限表示法
- **权限命令**：熟练使用 `chmod`、`chown`、`chgrp` 命令
- **特殊权限**：了解 SetUID、SetGID、Sticky Bit 的用途
- **安全实践**：遵循最小权限原则，避免过度授权
- **故障排查**：掌握权限问题的诊断和解决方法

通过合理设置权限，可以确保系统的安全性、数据的保密性，同时支持多用户协作环境。记住，权限管理是一个持续的过程，需要定期审计和调整，以适应不断变化的安全需求和组织架构。