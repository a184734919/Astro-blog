---
title: Linux 文件操作
published: 2025-03-17
description: '文件和目录操作的详细介绍和学习笔记'
image: ''
tags: ["Linux","服务器"]
category: '开发工具'
draft: false
lang: 'zh-CN'
---

## 概述

Linux 文件操作系统是 Linux 系统中最基础也最重要的操作之一。Linux 将所有内容都视为文件，包括常规文件、目录、设备文件等。掌握文件操作命令是进行系统管理、开发调试和日常工作的必备技能。

## 核心概念

### 文件系统结构

Linux 文件系统采用树形结构，根目录为 `/`：

```
/ (根目录)
├── bin/       # 常用命令
├── dev/       # 设备文件
├── etc/       # 配置文件
├── home/      # 用户主目录
├── lib/       # 系统库
├── tmp/       # 临时文件
├── usr/       # 用户程序
├── var/       # 可变数据
└── opt/       # 可选软件包
```

### 路径概念

- **绝对路径**：从根目录开始的完整路径，如 `/home/user/documents`
- **相对路径**：相对于当前工作目录的路径，如 `documents/file.txt`
- **特殊路径**：
  - `.`：当前目录
  - `..`：上级目录
  - `~`：当前用户的主目录
  - `-`：上一个工作目录

### 文件类型

Linux 系统中主要有以下文件类型：

- **常规文件** (-)：文本文件、二进制文件等
- **目录文件** (d)：包含其他文件和目录
- **符号链接** (l)：指向其他文件的引用
- **设备文件**：块设备(b)和字符设备(c)
- **命名管道** (p)：进程间通信
- **套接字** (s)：网络通信

## 基本用法

### 查看目录和文件

```bash
# 列出目录内容
ls                    # 列出当前目录内容
ls -la               # 列出所有文件（包括隐藏文件），显示详细信息
ls -lh               # 显示文件大小（人可读格式）
ls -lt               # 按修改时间排序
ls -lr               # 按修改时间反向排序
ls -R                # 递归列出所有子目录

# 查看目录树结构
tree                 # 显示目录树结构
tree -L 2            # 显示 2 级深度
tree -d              # 只显示目录

# 查看当前路径
pwd                  # 显示当前工作目录的完整路径
pwd -P               # 显示物理路径（解析符号链接）

# 查找文件位置
whereis command      # 查找命令的二进制、源码和手册文件位置
which command        # 查找命令的路径
locate filename      # 快速查找文件（需要 updatedb 数据库）
```

### 目录操作

```bash
# 创建目录
mkdir dirname                      # 创建单个目录
mkdir -p dir1/dir2/dir3            # 创建多级目录
mkdir -m 755 dirname               # 创建目录并设置权限
mkdir {dir1,dir2,dir3}             # 创建多个目录

# 删除目录
rmdir empty_dir                    # 删除空目录
rmdir -p dir1/dir2/empty_dir       # 删除多级空目录
rm -rf directory                   # 强制删除目录及其内容

# 切换目录
cd /path/to/directory             # 切换到指定目录
cd ~                             # 切换到用户主目录
cd ..                            # 切换到上级目录
cd -                             # 切换到上一个工作目录
cd ~/Documents                   # 切换到 Documents 目录

# 复制目录
cp -r source_dir destination_dir     # 递归复制目录
cp -a source_dir destination_dir     # 保留所有属性复制目录
cp -r dir1/ dir2/                    # 将 dir1 复制到 dir2 中

# 移动/重命名目录
mv old_dir new_dir                   # 重命名目录
mv dir1/ /path/to/destination/      # 移动目录到指定位置
```

### 文件操作

```bash
# 创建文件
touch filename                      # 创建空文件
touch {file1,file2,file3}          # 创建多个文件
echo "content" > filename           # 创建文件并写入内容
cat > filename << EOF               # 多行内容写入文件
Line 1
Line 2
EOF

# 查看文件内容
cat filename                        # 显示文件内容
cat -n filename                     # 显示行号
less filename                       # 分页查看文件
more filename                       # 分页查看文件（向后翻页）
head -n 10 filename                 # 查看文件前 10 行
tail -n 10 filename                 # 查看文件后 10 行
tail -f filename                    # 实时查看文件内容

# 查看文件类型
file filename                       # 识别文件类型
file -i filename                    # 显示 MIME 类型

# 复制文件
cp file1 file2                      # 复制文件
cp -p file1 file2                   # 保留文件属性复制
cp -i file1 file2                   # 覆盖前提示
cp file1 file2 file3 destination/   # 复制多个文件到目录

# 移动/重命名文件
mv old_file new_file                # 重命名文件
mv file /path/to/destination/       # 移动文件

# 删除文件
rm filename                         # 删除文件
rm -f filename                      # 强制删除（不提示）
rm -i filename                      # 删除前确认
rm *.log                            # 删除所有 .log 文件
rm -rf directory/                   # 强制删除目录及其内容

# 比较文件
diff file1 file2                    # 比较两个文件的差异
diff -u file1 file2                 # 统一格式显示差异
diff -r dir1/ dir2/                 # 递归比较目录

# 文件内容处理
sort filename                       # 对文件内容排序
sort -n filename                    # 数字排序
sort -r filename                    # 反向排序
sort -u filename                    # 去重排序
wc -l filename                      # 统计行数
wc -w filename                      # 统计单词数
wc -c filename                      # 统计字符数

# 文本搜索
grep "pattern" filename             # 在文件中搜索
grep -r "pattern" directory/        # 递归搜索目录
grep -i "pattern" filename          # 忽略大小写
grep -v "pattern" filename          # 反向匹配
grep -n "pattern" filename          # 显示行号
grep -c "pattern" filename          # 统计匹配次数
```

### 文件权限和属性

```bash
# 查看文件权限
ls -la filename                     # 查看文件详细权限
stat filename                       # 查看文件完整属性

# 修改文件权限
chmod 755 filename                  # 数字方式设置权限
chmod u+rwx,g+rx,o+r filename       # 符号方式设置权限
chmod +x script.sh                  # 添加执行权限
chmod -w filename                   # 移除写权限

# 修改文件所有者
chown user:group filename           # 修改所有者和组
chown user filename                 # 只修改所有者
chown :group filename               # 只修改组
chown -R user:group directory/      # 递归修改目录

# 修改文件时间戳
touch -t 202501151200 filename      # 修改文件时间为 2025-01-15 12:00
touch -d "2 days ago" filename      # 修改文件时间为 2 天前

# 文件属性设置
chattr +i filename                  # 设置文件为不可修改
chattr -i filename                  # 取消不可修改属性
chattr +a filename                  # 设置文件只能追加内容
```

### 符号链接和硬链接

```bash
# 创建符号链接（软链接）
ln -s /path/to/file link_name       # 创建文件链接
ln -s /path/to/directory link_dir   # 创建目录链接
ln -sf /path/to/new_file link_name  # 强制覆盖已有链接

# 创建硬链接
ln original_file hard_link          # 创建硬链接

# 查看链接
ls -la link_name                    # 查看链接信息
readlink link_name                  # 查看链接指向的路径
readlink -f link_name               # 查看绝对路径

# 删除链接
rm link_name                        # 删除链接（不影响原文件）
```

### 文件查找和搜索

```bash
# find 命令
find /path -name filename           # 按文件名查找
find /path -name "*.txt"            # 使用通配符查找
find /path -type f -name "pattern"  # 查找普通文件
find /path -type d -name "pattern"  # 查找目录
find /path -size +100M              # 查找大于 100MB 的文件
find /path -size -1M                # 查找小于 1MB 的文件
find /path -mtime -7                # 查找 7 天内修改的文件
find /path -mtime +30               # 查找 30 天前修改的文件
find /path -user username           # 查找指定用户的文件
find /path -group groupname         # 查找指定组的文件
find /path -perm 777                # 查找权限为 777 的文件
find /path -empty                   # 查找空文件或空目录
find /path -executable              # 查找可执行文件

# find 的执行操作
find /path -name "*.log" -delete    # 查找并删除文件
find /path -name "*.sh" -exec chmod +x {} \;  # 查找并设置权限
find /path -name "*.bak" -exec rm {} \;        # 查找并删除文件
find /path -type f -exec file {} \;            # 查找并识别文件类型

# locate 命令（快速查找）
updatedb                            # 更新文件数据库
locate filename                     # 快速查找文件
locate -i filename                  # 忽略大小写查找
```

## 实际应用

### 场景一：项目文件管理

```bash
# 1. 创建项目结构
mkdir -p ~/projects/webapp/{src,dist,tests,docs}
cd ~/projects/webapp

# 2. 创建基础文件
touch src/{main.js,styles.css,index.html}
touch tests/{test.spec.js}
touch README.md package.json

# 3. 查看项目结构
tree -L 2
ls -la src/

# 4. 复制模板文件
cp ~/templates/package.json .
cp ~/templates/.gitignore .

# 5. 设置文件权限
chmod +x scripts/build.sh
chmod 644 *.html *.js *.css

# 6. 创建备份
tar -czf backup_$(date +%Y%m%d).tar.gz src/
```

### 场景二：日志文件管理

```bash
# 1. 查看当前日志
tail -f /var/log/nginx/access.log    # 实时查看日志
tail -n 100 /var/log/syslog          # 查看最近 100 行
head -n 50 /var/log/auth.log         # 查看前 50 行

# 2. 搜索日志内容
grep "error" /var/log/syslog         # 搜索错误信息
grep -i "failed" /var/log/auth.log   # 忽略大小写搜索
grep "ERROR" *.log | wc -l           # 统计错误数量

# 3. 日志分析
awk '{print $1}' access.log | sort | uniq -c | sort -rn  # 统计 IP 访问次数
grep "404" access.log | wc -l       # 统计 404 错误数量

# 4. 日志备份和清理
tar -czf logs_backup_$(date +%Y%m%d).tar.gz /var/log/*.log
find /var/log -name "*.log" -mtime +30 -exec gzip {} \;
find /var/log -name "*.gz" -mtime +90 -delete

# 5. 日志轮转
logrotate -f /etc/logrotate.conf    # 手动执行日志轮转
```

### 场景三：批量文件处理

```bash
# 1. 批量重命名文件
for file in *.txt; do
    mv "$file" "${file%.txt}.md"
done

# 2. 批量修改权限
find ~/projects -type f -name "*.sh" -exec chmod +x {} \;
find ~/projects -type d -exec chmod 755 {} \;
find ~/projects -type f -exec chmod 644 {} \;

# 3. 批量替换文件内容
find . -name "*.html" -exec sed -i 's/old_text/new_text/g' {} \;

# 4. 批量删除文件
find . -name "*.tmp" -delete
find . -name "*.bak" -mtime +7 -delete

# 5. 批量压缩文件
for dir in */; do
    tar -czf "${dir%/}.tar.gz" "$dir"
done
```

### 场景四：磁盘空间管理

```bash
# 1. 查看磁盘使用情况
df -h                              # 查看磁盘使用情况
du -sh /var/*                      # 查看 /var 目录下各子目录大小
du -h --max-depth=1 /home/user     # 查看用户目录大小

# 2. 查找大文件
find /home -type f -size +100M -exec ls -lh {} \;
find /var/log -type f -size +50M

# 3. 清理缓存文件
sudo apt-get clean                 # 清理 apt 缓存
sudo apt-get autoremove            # 删除不需要的包
rm -rf ~/.cache/*                  # 清理用户缓存
rm -rf /tmp/*                      # 清理临时文件

# 4. 清理旧日志
find /var/log -name "*.log.*" -mtime +30 -delete
find /var/log -name "*.gz" -mtime +60 -delete

# 5. 清理重复文件
fdupes -r ~/downloads/             # 查找重复文件
fdupes -d ~/downloads/             # 删除重复文件
```

## 注意事项

1. **删除操作安全**：
   - 使用 `rm -rf` 前要确认路径正确
   - 重要的删除操作先使用 `ls` 查看内容
   - 考虑使用 `trash-cli` 替代 `rm`，可以恢复误删文件

2. **文件权限**：
   - 不要随意修改系统文件的权限
   - 网站文件和脚本要设置正确的执行权限
   - 敏感配置文件要限制访问权限

3. **磁盘空间**：
   - 定期清理临时文件和日志
   - 注意监控磁盘使用情况
   - 大文件操作前检查可用空间

4. **文件命名**：
   - 避免使用特殊字符和空格
   - 使用有意义的文件名
   - 保持命名规范一致性

5. **符号链接**：
   - 相对路径符号链接要注意移动后的有效性
   - 删除符号链接不会影响原文件
   - 循环链接会导致问题

6. **备份策略**：
   - 重要文件定期备份
   - 删除前先备份
   - 使用版本控制系统管理重要代码

## 总结

Linux 文件操作是系统管理的基础技能，掌握这些命令可以高效地管理文件和目录。重点包括：

- **目录操作**：熟练使用 `ls`, `cd`, `mkdir`, `rm` 等命令
- **文件操作**：掌握文件的创建、复制、移动、删除等操作
- **内容查看**：使用 `cat`, `less`, `head`, `tail` 等命令查看文件内容
- **文件搜索**：使用 `find` 和 `locate` 命令查找文件
- **权限管理**：了解和设置文件权限和所有者
- **批量操作**：结合循环和条件判断进行批量文件处理
- **安全意识**：在删除和修改操作前要谨慎确认

通过不断实践，这些文件操作命令会成为你日常工作的得力助手，大幅提升工作效率。记住，文件操作是 Linux 系统的基础，熟练掌握这些命令是成为高级用户的必经之路。