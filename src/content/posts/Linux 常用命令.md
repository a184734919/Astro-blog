---
title: Linux 常用命令
published: 2025-03-11
description: 'Linux 基础命令的详细介绍和学习笔记'
image: ''
tags: ["Linux","服务器"]
category: '开发工具'
draft: false
lang: 'zh-CN'
---

## 概述

Linux 常用命令是每个开发者必备的技能，无论是在服务器管理、开发调试还是日常系统使用中都扮演着重要角色。Linux 命令行界面（CLI）提供了强大而高效的系统操作能力，通过掌握这些命令，可以大幅提升工作效率。

## 核心概念

### 命令行结构

Linux 命令通常遵循以下结构：

```bash
命令名称 [选项] [参数]
```

- **命令名称**：要执行的程序或命令
- **选项**：修改命令行为的标志，通常以 `-` 或 `--` 开头
- **参数**：命令操作的对象，如文件名、目录名等

### 常用选项格式

- 短选项：`-a`, `-l`, `-h`（单个字符，可合并使用：`-alh`）
- 长选项：`--all`, `--list`, `--help`（完整单词）

### 帮助信息

```bash
# 查看命令帮助
命令 --help
man 命令          # 查看详细手册
info 命令         # 查看信息文档
```

## 基本用法

### 系统信息查看

```bash
# 查看系统信息
uname -a              # 显示系统内核信息
uname -r              # 显示内核版本
uname -m              # 显示系统架构

# 查看发行版信息
cat /etc/os-release   # 查看操作系统信息
lsb_release -a        # 查看 LSB 信息

# 查看主机信息
hostname              # 查看主机名
hostname -I           # 查看主机 IP 地址
whoami                # 查看当前用户
id                    # 查看用户详细信息

# 查看系统时间和负载
date                  # 查看当前日期和时间
uptime                # 查看系统运行时间和负载
top                   # 查看系统进程和资源使用情况
htop                  # 更友好的进程监控工具
```

### 用户和权限管理

```bash
# 用户切换
su - username         # 切换到指定用户
sudo command          # 以超级用户权限执行命令
sudo su               # 切换到 root 用户
exit                  # 退出当前用户会话

# 用户管理
useradd username      # 创建新用户
userdel username      # 删除用户
usermod -aG group username  # 将用户添加到组
passwd                # 修改当前用户密码
passwd username       # 修改指定用户密码

# 查看用户信息
who                   # 查看当前登录用户
w                     # 查看登录用户及其活动
last                  # 查看登录历史
lastlog               # 查看所有用户的最后登录时间
```

### 网络相关命令

```bash
# 网络连接测试
ping hostname         # 测试网络连通性
ping -c 4 google.com  # 发送 4 个 ping 包
traceroute hostname   # 追踪数据包路径
mtr hostname          # 结合 ping 和 traceroute 的工具

# 网络信息查看
ip addr               # 查看 IP 地址信息
ifconfig              # 查看网络接口（旧命令）
netstat -tuln         # 查看网络连接和监听端口
ss -tuln              # 现代化的网络连接查看工具
hostname -I           # 查看本机 IP 地址

# 网络下载
wget URL              # 下载文件
curl URL              # 获取 URL 内容
curl -O URL           # 下载文件并保存原文件名
scp file user@host:/path  # 安全复制文件到远程服务器

# 防火墙管理
ufw status            # 查看 UFW 防火墙状态
ufw enable            # 启用 UFW 防火墙
ufw disable           # 禁用 UFW 防火墙
ufw allow 22/tcp      # 开放 22 端口
```

### 进程管理

```bash
# 查看进程
ps aux                # 查看所有进程
ps -ef                # 查看完整格式的进程列表
ps -u username        # 查看指定用户的进程
pgrep process_name    # 按名称查找进程 ID

# 实时监控
top                   # 实时显示进程信息
htop                  # 更友好的进程监控
iotop                 # 监控磁盘 I/O
iftop                 # 监控网络流量

# 进程控制
kill PID              # 终止指定进程（默认 SIGTERM 信号）
kill -9 PID           # 强制终止进程（SIGKILL 信号）
killall process_name  # 按名称终止所有匹配的进程
pkill process_name    # 按名称模式匹配并终止进程
nohup command &       # 后台运行命令，忽略挂起信号
bg                    # 将作业放到后台运行
fg                    # 将后台作业恢复到前台
```

### 系统资源监控

```bash
# CPU 信息
lscpu                 # 查看 CPU 信息
cat /proc/cpuinfo     # 查看 CPU 详细信息
vmstat                # 虚拟内存统计
sar -u 1 5            # 每秒采集一次 CPU 数据，共 5 次

# 内存信息
free -h               # 查看内存使用情况（人可读格式）
cat /proc/meminfo     # 查看内存详细信息
vmstat -s             # 显示内存统计信息

# 磁盘信息
df -h                 # 查看磁盘使用情况
du -sh directory      # 查看目录大小
du -h --max-depth=1   # 查看当前目录下一级目录的大小
lsblk                 # 查看块设备信息
fdisk -l              # 查看磁盘分区信息

# 系统日志
journalctl            # 查看系统日志
journalctl -f         # 实时查看日志
journalctl -u service # 查看特定服务的日志
tail -f /var/log/syslog  # 实时查看系统日志
```

### 搜索和查找

```bash
# 文件查找
find /path -name filename       # 按文件名查找
find /path -type d -name dirname # 查找目录
find /path -size +100M          # 查找大于 100MB 的文件
find /path -mtime -7            # 查找 7 天内修改的文件
find /path -perm 777            # 查找权限为 777 的文件
find /path -user username       # 查找指定用户的文件

# 内容搜索
grep "pattern" file             # 在文件中搜索模式
grep -r "pattern" directory     # 递归搜索目录
grep -i "pattern" file          # 忽略大小写搜索
grep -v "pattern" file          # 反向匹配
grep -n "pattern" file          # 显示行号
grep -c "pattern" file          # 统计匹配行数

# 组合使用
find /path -name "*.log" -exec grep "error" {} \;  # 查找并搜索日志文件
```

### 压缩和解压

```bash
# tar 命令
tar -czf archive.tar.gz directory    # 压缩目录为 .tar.gz
tar -xzf archive.tar.gz              # 解压 .tar.gz 文件
tar -cjf archive.tar.bz2 directory   # 压缩为 .tar.bz2
tar -xjf archive.tar.bz2             # 解压 .tar.bz2 文件
tar -tzf archive.tar.gz              # 查看压缩包内容

# zip 命令
zip -r archive.zip directory         # 压缩目录为 .zip
unzip archive.zip                    # 解压 .zip 文件
unzip -l archive.zip                 # 查看 .zip 文件内容

# 其他压缩格式
gzip file                            # 压缩为 .gz
gunzip file.gz                       # 解压 .gz 文件
bzip2 file                           # 压缩为 .bz2
bunzip2 file.bz2                     # 解压 .bz2 文件
```

### 软件包管理

```bash
# Debian/Ubuntu 系统
apt update                           # 更新软件包列表
apt upgrade                          # 升级已安装的软件包
apt install package_name             # 安装软件包
apt remove package_name              # 删除软件包
apt purge package_name               # 删除软件包及配置文件
apt search package_name              # 搜索软件包
apt show package_name                # 显示软件包信息
apt autoremove                       # 删除不需要的依赖包

# CentOS/RHEL 系统
yum update                           # 更新软件包
yum install package_name             # 安装软件包
yum remove package_name              # 删除软件包
yum search package_name              # 搜索软件包
yum info package_name                # 显示软件包信息

# 通用方法
snap install package_name            # 使用 Snap 安装
flatpak install package_name         # 使用 Flatpak 安装
```

## 实际应用

### 场景一：系统性能诊断

```bash
# 1. 查看系统负载
uptime
# 输出示例: load average: 1.5, 1.8, 2.1

# 2. 查看内存使用情况
free -h
# 检查内存是否充足

# 3. 查看磁盘使用情况
df -h
# 检查磁盘空间是否足够

# 4. 查看进程状态
top
# 找出 CPU 和内存占用高的进程

# 5. 查看网络连接
netstat -tuln
# 检查网络连接和监听端口

# 6. 查看系统日志
journalctl -p err -n 50
# 查看最近的错误日志
```

### 场景二：查找和清理磁盘空间

```bash
# 1. 查找大文件
find / -type f -size +100M 2>/dev/null | head -20

# 2. 查看目录大小
du -sh /var/*
du -h --max-depth=1 /home

# 3. 清理系统缓存
sudo apt-get clean              # 清理 apt 缓存
sudo apt-get autoremove         # 删除不需要的包
sudo journalctl --vacuum-size=100M  # 清理系统日志

# 4. 清理旧的日志文件
find /var/log -name "*.log" -mtime +30 -exec rm {} \;

# 5. 清理临时文件
sudo rm -rf /tmp/*
sudo rm -rf /var/tmp/*
```

### 场景三：进程监控和管理

```bash
# 1. 查找特定进程
ps aux | grep nginx
pgrep -f "node server.js"

# 2. 查看进程详细信息
ps -p PID -f
cat /proc/PID/status

# 3. 监控进程资源使用
top -p PID
pidstat -p PID 1 10

# 4. 查看进程打开的文件
lsof -p PID
lsof -i :8080  # 查看占用端口的进程

# 5. 优雅终止进程
kill -15 PID     # SIGTERM 信号，允许进程清理
kill -9 PID      # SIGKILL 信号，强制终止

# 6. 后台运行进程
nohup python script.py > output.log 2>&1 &
screen -S session_name    # 创建 screen 会话
screen -r session_name    # 恢复 screen 会话
```

### 场景四：网络问题诊断

```bash
# 1. 测试网络连通性
ping -c 4 google.com
traceroute google.com

# 2. 检查 DNS 解析
nslookup google.com
dig google.com
host google.com

# 3. 检查端口连接
telnet host port
nc -zv host port

# 4. 查看网络流量
iftop
nethogs

# 5. 检查网络接口
ip addr show
ethtool eth0

# 6. 抓包分析
tcpdump -i eth0 -nn port 80
tcpdump -i eth0 -nn host 192.168.1.1
```

## 注意事项

1. **命令安全性**：
   - 在使用 `rm -rf` 等危险命令前，先确认路径正确
   - 使用 `sudo` 时要小心，避免误操作
   - 定期备份重要数据

2. **性能优化**：
   - 避免在生产环境使用开发工具
   - 合理设置系统资源限制
   - 定期清理日志和临时文件

3. **命令组合**：
   - 学会使用管道（`|`）和重定向（`>`, `>>`）
   - 熟练使用命令组合提高效率
   - 了解正则表达式在 grep 中的使用

4. **文档和帮助**：
   - 遇到问题时先查看 `--help` 和 `man`
   - 保存常用命令和脚本
   - 记录解决过的问题

5. **系统维护**：
   - 定期更新系统和软件包
   - 监控系统资源使用情况
   - 及时处理错误日志和异常

## 总结

Linux 常用命令是开发和运维工作的基础工具，掌握这些命令可以大幅提升工作效率。重点包括：

- **系统监控**：使用 `top`, `htop`, `df`, `free` 等命令监控系统状态
- **进程管理**：熟练使用 `ps`, `kill`, `pgrep` 等命令管理进程
- **文件操作**：掌握 `find`, `grep`, `tar` 等命令进行文件操作
- **网络诊断**：使用 `ping`, `netstat`, `tcpdump` 等命令诊断网络问题
- **用户管理**：了解用户和权限管理的基本操作
- **包管理**：掌握不同系统的软件包管理命令

通过不断实践和总结，这些命令会成为你工作中的得力助手。记住，Linux 命令行是一个强大的工具，善用它可以解决很多复杂的问题。