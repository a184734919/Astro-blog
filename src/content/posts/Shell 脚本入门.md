---
title: Shell 脚本入门
published: 2025-03-29
description: 'Shell 脚本基础的详细介绍和学习笔记'
image: ''
tags: ["Shell","脚本"]
category: '开发工具'
draft: false
lang: 'zh-CN'
---

## 概述

Shell 脚本是 Linux 系统自动化和任务批处理的强大工具。通过编写 Shell 脚本，可以将复杂的命令序列组合起来，实现自动化操作、系统管理、数据处理等功能。Shell 脚本入门是学习 Linux 系统管理和自动化的重要第一步。

## 核心概念

### 什么是 Shell

Shell 是用户与操作系统内核之间的接口，它接收用户输入的命令并传递给内核执行。常见的 Shell 包括：

- **Bash（Bourne Again Shell）**：最常用的 Shell，大多数 Linux 系统的默认 Shell
- **Zsh（Z Shell）**：功能强大的 Shell，提供丰富的特性
- **Fish**：用户友好的现代 Shell
- **sh**：传统的 Bourne Shell

### Shell 脚本的特点

- **解释型语言**：不需要编译，直接解释执行
- **脚本化**：将多个命令组合成一个脚本文件
- **自动化**：可以实现任务的自动化执行
- **跨平台**：在大多数 Unix-like 系统上运行
- **功能强大**：支持变量、函数、条件判断、循环等编程特性

### 脚本文件结构

```bash
#!/bin/bash

# 脚本注释
# 脚本描述：这是一个示例脚本
# 作者：Your Name
# 日期：2025-03-29

# 脚本主体
echo "Hello, World!"
```

- `#!/bin/bash`：Shebang，指定脚本解释器
- `#`：注释符号
- 脚本主体：实际的命令和逻辑

## 基本用法

### 创建和执行脚本

```bash
# 1. 创建脚本文件
touch myscript.sh

# 2. 编辑脚本文件
nano myscript.sh
# 或使用 vim
vim myscript.sh

# 3. 添加执行权限
chmod +x myscript.sh

# 4. 执行脚本
./myscript.sh

# 5. 使用 bash 执行（不需要执行权限）
bash myscript.sh

# 6. 使用 sh 执行
sh myscript.sh

# 7. 查看脚本内容
cat myscript.sh

# 8. 调试脚本
bash -x myscript.sh  # 显示执行的每个命令
```

### 变量

```bash
#!/bin/bash

# 变量定义
name="John"
age=25
is_student=true

# 变量使用
echo "Name: $name"
echo "Age: $age"
echo "Is student: $is_student"

# 变量拼接
greeting="Hello, $name!"
echo $greeting

# 变量默认值
echo ${name:-"Guest"}  # 如果 name 为空，使用 "Guest"

# 只读变量
readonly PI=3.14159
# PI=3.14  # 这会报错

# 删除变量
unset name
# echo $name  # 输出为空

# 环境变量
export PATH=/usr/local/bin:$PATH
export JAVA_HOME=/usr/lib/jvm/java-8

# 获取命令行参数
echo "脚本名称: $0"
echo "第一个参数: $1"
echo "第二个参数: $2"
echo "所有参数: $@"
echo "参数个数: $#"

# 特殊变量
echo "当前进程ID: $$"
echo "上一个命令的退出状态: $?"
echo "当前Shell的PID: $$"
```

### 输入输出

```bash
#!/bin/bash

# 基本输出
echo "Hello, World!"
echo -n "不换行输出"
echo "换行输出"

# 输出变量
name="Alice"
echo "My name is $name"

# 格式化输出
printf "Name: %s, Age: %d\n" "Bob" 30
printf "Price: %.2f\n" 19.99

# 用户输入
echo "What is your name?"
read name
echo "Nice to meet you, $name!"

# 带提示的输入
read -p "Enter your age: " age
echo "You are $age years old."

# 密码输入（不显示字符）
read -s -p "Enter your password: " password
echo  # 换行

# 超时输入
read -t 5 -p "Enter your choice (5 seconds): " choice
if [ -z "$choice" ]; then
    echo "Timeout!"
else
    echo "You chose: $choice"
fi

# 读取多个值
read -p "Enter name age city: " name age city
echo "Name: $name, Age: $age, City: $city"

# 重定向输出
echo "This goes to a file" > output.txt
echo "This appends to a file" >> output.txt

# 重定向输入
while read line; do
    echo "Read: $line"
done < input.txt
```

### 条件判断

```bash
#!/bin/bash

# if-else 语句
age=25
if [ $age -ge 18 ]; then
    echo "You are an adult."
else
    echo "You are a minor."
fi

# 多重条件判断
score=85
if [ $score -ge 90 ]; then
    echo "Grade: A"
elif [ $score -ge 80 ]; then
    echo "Grade: B"
elif [ $score -ge 70 ]; then
    echo "Grade: C"
else
    echo "Grade: D"
fi

# 字符串比较
name="Alice"
if [ "$name" = "Alice" ]; then
    echo "Hello, Alice!"
fi

if [ "$name" != "Bob" ]; then
    echo "You are not Bob."
fi

# 数字比较
num1=10
num2=20

if [ $num1 -eq $num2 ]; then
    echo "Numbers are equal"
elif [ $num1 -lt $num2 ]; then
    echo "$num1 is less than $num2"
else
    echo "$num1 is greater than $num2"
fi

# 逻辑运算
age=25
is_student=true

if [ $age -ge 18 ] && [ "$is_student" = true ]; then
    echo "Adult student"
fi

if [ $age -lt 18 ] || [ "$is_student" = true ]; then
    echo "Young or student"
fi

# 文件测试
file="test.txt"
if [ -f "$file" ]; then
    echo "File exists"
fi

if [ -d "/tmp" ]; then
    echo "Directory exists"
fi

if [ -r "$file" ]; then
    echo "File is readable"
fi

if [ -w "$file" ]; then
    echo "File is writable"
fi

if [ -x "$file" ]; then
    echo "File is executable"
fi

# case 语句
day="Monday"
case $day in
    "Monday")
        echo "Start of the work week"
        ;;
    "Friday")
        echo "End of the work week"
        ;;
    "Saturday" | "Sunday")
        echo "Weekend!"
        ;;
    *)
        echo "Regular day"
        ;;
esac
```

### 循环

```bash
#!/bin/bash

# for 循环
echo "Counting from 1 to 5:"
for i in {1..5}; do
    echo "Number: $i"
done

# 遍历数组
fruits=("apple" "banana" "orange")
echo "Fruits:"
for fruit in "${fruits[@]}"; do
    echo "- $fruit"
done

# 遍历文件
echo "Files in current directory:"
for file in *.txt; do
    echo "Processing: $file"
done

# C 风格的 for 循环
echo "Counting down:"
for ((i=10; i>0; i--)); do
    echo "$i"
done

# while 循环
echo "Counting up:"
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done

# 读取文件行
echo "Reading file line by line:"
while IFS= read -r line; do
    echo "Line: $line"
done < input.txt

# until 循环
echo "Counting until 5:"
count=0
until [ $count -eq 5 ]; do
    echo "Count: $count"
    ((count++))
done

# 循环控制
echo "Loop control demonstration:"
for i in {1..10}; do
    if [ $i -eq 3 ]; then
        continue  # 跳过本次迭代
    fi
    
    if [ $i -eq 7 ]; then
        break     # 退出循环
    fi
    
    echo "Number: $i"
done

# 嵌套循环
echo "Nested loop:"
for i in {1..3}; do
    for j in {1..3}; do
        echo "i=$i, j=$j"
    done
done
```

### 函数

```bash
#!/bin/bash

# 定义函数
greet() {
    echo "Hello, $1!"
}

# 调用函数
greet "Alice"

# 带返回值的函数
add() {
    local sum=$(($1 + $2))
    echo $sum
}

result=$(add 5 3)
echo "Result: $result"

# 使用 return 语句
multiply() {
    local product=$(($1 * $2))
    return $product
}

multiply 4 5
echo "Product: $?"

# 带默认参数的函数
backup_file() {
    local file=${1:-"default.txt"}
    local backup_dir=${2:-"/tmp/backups"}
    
    if [ -f "$file" ]; then
        cp "$file" "$backup_dir/"
        echo "Backed up $file to $backup_dir"
    else
        echo "File not found: $file"
    fi
}

backup_file "important.txt"
backup_file "data.txt" "/home/user/backups"

# 递归函数
factorial() {
    local n=$1
    if [ $n -le 1 ]; then
        echo 1
    else
        local prev=$(factorial $((n-1)))
        echo $((n * prev))
    fi
}

result=$(factorial 5)
echo "5! = $result"

# 局部变量
example() {
    local local_var="I am local"
    global_var="I am global"
    echo "Inside function: $local_var"
}

example
# echo $local_var  # 这会报错，因为 local_var 是局部变量
echo "Global variable: $global_var"
```

### 数组操作

```bash
#!/bin/bash

# 定义数组
fruits=("apple" "banana" "orange" "grape")

# 访问数组元素
echo "First fruit: ${fruits[0]}"
echo "Second fruit: ${fruits[1]}"

# 获取所有元素
echo "All fruits: ${fruits[@]}"
echo "All fruits: ${fruits[*]}"

# 获取数组长度
echo "Number of fruits: ${#fruits[@]}"

# 遍历数组
echo "Iterating over fruits:"
for fruit in "${fruits[@]}"; do
    echo "- $fruit"
done

# 添加元素
fruits+=("mango")
echo "After adding mango: ${fruits[@]}"

# 删除元素
unset fruits[1]
echo "After removing banana: ${fruits[@]}"

# 数组切片
echo "First two fruits: ${fruits[@]:0:2}"
echo "Last two fruits: ${fruits[@]: -2}"

# 数组排序
IFS=$'\n' sorted=($(sort <<<"${fruits[*]}"))
unset IFS
echo "Sorted fruits: ${sorted[@]}"

# 数组查找
if [[ " ${fruits[@]} " =~ " apple " ]]; then
    echo "Apple is in the array"
fi

# 关联数组（需要 Bash 4.0+）
declare -A ages
ages["Alice"]=25
ages["Bob"]=30
ages["Charlie"]=28

echo "Alice's age: ${ages[Alice]}"
echo "All ages: ${ages[@]}"

# 遍历关联数组
for name in "${!ages[@]}"; do
    echo "$name: ${ages[$name]}"
done
```

## 实际应用

### 场景一：系统信息收集脚本

```bash
#!/bin/bash

# 系统信息收集脚本
# 用途：收集系统基本信息并生成报告

echo "=== System Information Report ==="
echo "Generated: $(date)"
echo "================================="

# 系统基本信息
echo -e "\n[Basic Information]"
echo "Hostname: $(hostname)"
echo "OS: $(cat /etc/os-release | grep PRETTY_NAME | cut -d '"' -f 2)"
echo "Kernel: $(uname -r)"
echo "Uptime: $(uptime -p)"
echo "Architecture: $(uname -m)"

# CPU 信息
echo -e "\n[CPU Information]"
echo "CPU Model: $(lscpu | grep 'Model name' | cut -d ':' -f 2 | xargs)"
echo "CPU Cores: $(nproc)"
echo "CPU Usage: $(top -bn1 | grep 'Cpu(s)' | awk '{print $2}' | cut -d '%' -f 1)%"

# 内存信息
echo -e "\n[Memory Information]"
free -h | grep -E "Mem|Swap"

# 磁盘信息
echo -e "\n[Disk Information]"
df -h | grep -E "Filesystem|/dev/"

# 网络信息
echo -e "\n[Network Information]"
ip addr show | grep -E "inet |eth0|wlan0"

# 用户信息
echo -e "\n[User Information]"
echo "Current user: $(whoami)"
echo "Logged in users:"
who

# 进程信息
echo -e "\n[Process Information]"
echo "Total processes: $(ps aux | wc -l)"
echo "Top 5 CPU consuming processes:"
ps aux --sort=-%cpu | head -6

echo -e "\n=== Report Complete ==="
```

### 场景二：日志分析脚本

```bash
#!/bin/bash

# 日志分析脚本
# 用途：分析系统日志文件，提取关键信息

LOG_FILE="/var/log/syslog"
ERROR_KEYWORD="ERROR"
WARNING_KEYWORD="WARNING"

if [ ! -f "$LOG_FILE" ]; then
    echo "Log file not found: $LOG_FILE"
    exit 1
fi

echo "=== Log Analysis Report ==="
echo "Analyzing: $LOG_FILE"
echo "Time range: Last 24 hours"
echo "================================="

# 统计错误数量
error_count=$(grep -c "$ERROR_KEYWORD" "$LOG_FILE")
echo -e "\n[Error Statistics]"
echo "Total errors: $error_count"

# 统计警告数量
warning_count=$(grep -c "$WARNING_KEYWORD" "$LOG_FILE")
echo "Total warnings: $warning_count"

# 显示最近的错误
echo -e "\n[Recent Errors]"
grep "$ERROR_KEYWORD" "$LOG_FILE" | tail -5

# 按服务分类错误
echo -e "\n[Errors by Service]"
grep "$ERROR_KEYWORD" "$LOG_FILE" | awk '{print $5}' | sort | uniq -c | sort -rn

# 查找失败的登录尝试
echo -e "\n[Failed Login Attempts]"
grep "Failed password" "$LOG_FILE" | tail -10

# 内存使用警告
echo -e "\n[Memory Warnings]"
grep -i "memory" "$LOG_FILE" | grep -i "$WARNING_KEYWORD" | tail -5

# 磁盘空间警告
echo -e "\n[Disk Space Warnings]"
grep -i "disk" "$LOG_FILE" | grep -i "$WARNING_KEYWORD" | tail -5

# 生成摘要
echo -e "\n[Summary]"
echo "Total log entries: $(wc -l < "$LOG_FILE")"
echo "Error rate: $(echo "scale=2; $error_count * 100 / $(wc -l < "$LOG_FILE")" | bc)%"
echo "Warning rate: $(echo "scale=2; $warning_count * 100 / $(wc -l < "$LOG_FILE")" | bc)%"

echo -e "\n=== Analysis Complete ==="
```

### 场景三：文件备份脚本

```bash
#!/bin/bash

# 文件备份脚本
# 用途：备份指定目录到目标位置

SOURCE_DIR="/home/user/documents"
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_NAME="backup_${DATE}.tar.gz"

# 创建备份目录
mkdir -p "$BACKUP_DIR"

echo "=== Backup Process Started ==="
echo "Source: $SOURCE_DIR"
echo "Backup file: $BACKUP_DIR/$BACKUP_NAME"
echo "Time: $(date)"

# 检查源目录
if [ ! -d "$SOURCE_DIR" ]; then
    echo "Error: Source directory not found: $SOURCE_DIR"
    exit 1
fi

# 创建备份
echo "Creating backup..."
tar -czf "$BACKUP_DIR/$BACKUP_NAME" -C "$(dirname "$SOURCE_DIR")" "$(basename "$SOURCE_DIR")"

if [ $? -eq 0 ]; then
    echo "Backup created successfully!"
    
    # 显示备份文件大小
    backup_size=$(du -h "$BACKUP_DIR/$BACKUP_NAME" | cut -f1)
    echo "Backup size: $backup_size"
    
    # 保留最近 7 天的备份
    echo "Cleaning old backups..."
    find "$BACKUP_DIR" -name "backup_*.tar.gz" -mtime +7 -delete
    
    # 列出当前备份
    echo -e "\nCurrent backups:"
    ls -lh "$BACKUP_DIR"/backup_*.tar.gz
    
    echo -e "\n=== Backup Complete ==="
else
    echo "Error: Backup failed!"
    exit 1
fi
```

### 场景四：进程监控脚本

```bash
#!/bin/bash

# 进程监控脚本
# 用途：监控指定进程并在异常时发送通知

PROCESS_NAME="nginx"
LOG_FILE="/var/log/process_monitor.log"
MAX_CPU_USAGE=80
MAX_MEMORY_USAGE=80

echo "=== Process Monitor Started ===" | tee -a "$LOG_FILE"
echo "Monitoring: $PROCESS_NAME" | tee -a "$LOG_FILE"
echo "Time: $(date)" | tee -a "$LOG_FILE"

while true; do
    # 检查进程是否运行
    if ! pgrep -x "$PROCESS_NAME" > /dev/null; then
        echo "[$(date)] ERROR: Process $PROCESS_NAME is not running!" | tee -a "$LOG_FILE"
        
        # 尝试重启进程
        echo "[$(date)] Attempting to restart $PROCESS_NAME..." | tee -a "$LOG_FILE"
        systemctl start "$PROCESS_NAME"
        
        if [ $? -eq 0 ]; then
            echo "[$(date)] Successfully restarted $PROCESS_NAME" | tee -a "$LOG_FILE"
        else
            echo "[$(date)] Failed to restart $PROCESS_NAME" | tee -a "$LOG_FILE"
        fi
    else
        # 获取进程信息
        pid=$(pgrep -x "$PROCESS_NAME")
        cpu_usage=$(ps -p "$pid" -o %cpu --no-headers | awk '{print int($1)}')
        memory_usage=$(ps -p "$pid" -o %mem --no-headers | awk '{print int($1)}')
        
        echo "[$(date)] $PROCESS_NAME is running (PID: $pid, CPU: ${cpu_usage}%, Memory: ${memory_usage}%)" | tee -a "$LOG_FILE"
        
        # 检查资源使用情况
        if [ "$cpu_usage" -gt "$MAX_CPU_USAGE" ]; then
            echo "[$(date)] WARNING: High CPU usage ($cpu_usage%)" | tee -a "$LOG_FILE"
        fi
        
        if [ "$memory_usage" -gt "$MAX_MEMORY_USAGE" ]; then
            echo "[$(date)] WARNING: High memory usage ($memory_usage%)" | tee -a "$LOG_FILE"
        fi
    fi
    
    # 等待 60 秒后再次检查
    sleep 60
done
```

## 注意事项

1. **脚本安全**：
   - 避免在脚本中硬编码密码和敏感信息
   - 使用环境变量或配置文件存储敏感数据
   - 验证用户输入，防止命令注入攻击

2. **错误处理**：
   - 使用 `set -e` 在错误时退出脚本
   - 检查命令执行状态
   - 提供有意义的错误信息

3. **可读性**：
   - 使用有意义的变量名和函数名
   - 添加适当的注释
   - 保持代码格式整洁

4. **性能优化**：
   - 避免不必要的循环和重复计算
   - 使用内置命令而非外部命令
   - 合理使用管道和重定向

5. **兼容性**：
   - 考虑不同 Shell 的兼容性
   - 避免使用特定版本的特性
   - 测试脚本在不同系统上的运行情况

6. **调试技巧**：
   - 使用 `bash -x` 调试脚本
   - 添加调试输出
   - 分步骤测试脚本功能

## 总结

Shell 脚本入门是 Linux 系统管理和自动化的基础。通过掌握这些基础知识，你可以：

- **自动化任务**：将重复性任务自动化
- **系统管理**：简化系统配置和维护
- **数据处理**：处理和分析文本数据
- **批量操作**：执行批量文件操作
- **监控告警**：实现系统监控和通知

Shell 脚本的学习是一个渐进的过程，从简单的命令组合到复杂的逻辑控制。通过不断实践和项目经验，你将能够编写出功能强大、健壮可靠的 Shell 脚本，大大提升工作效率。

记住，好的脚本应该是：
- **可读性强**：清晰的代码结构和注释
- **健壮性好**：能够处理异常情况
- **安全性高**：避免安全漏洞
- **性能优化**：高效执行任务
- **易于维护**：便于后续修改和扩展

继续学习和实践，你会发现 Shell 脚本在日常工作中有着广泛的应用价值。