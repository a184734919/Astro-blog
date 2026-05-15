---
title: Shell 脚本进阶
published: 2025-04-05
description: '自动化脚本编写的详细介绍和学习笔记'
image: ''
tags: ["Shell","脚本"]
category: '开发工具'
draft: false
lang: 'zh-CN'
---

## 概述

Shell 脚本进阶是在掌握了基础知识后，学习更高级的脚本编程技术。进阶内容包括字符串处理、正则表达式、进程管理、信号处理、调试技巧等高级主题。掌握这些技术可以让你编写出更强大、更专业的自动化脚本。

## 核心概念

### 脚本编程模式

- **命令式编程**：按顺序执行命令
- **函数式编程**：使用函数和组合
- **面向过程**：使用函数和变量
- **事件驱动**：响应系统事件和信号

### 脚本优化原则

- **性能优化**：减少不必要的操作和循环
- **内存管理**：避免内存泄漏和过度使用
- **代码重用**：使用函数和模块化设计
- **错误处理**：健壮的错误处理机制
- **日志记录**：详细的日志和调试信息

## 基本用法

### 高级字符串处理

```bash
#!/bin/bash

# 字符串长度
text="Hello, World!"
length=${#text}
echo "String length: $length"

# 字符串截取
echo "Substring (0-5): ${text:0:5}"    # Hello
echo "Substring (7-12): ${text:7:5}"  # World
echo "Substring from 7: ${text:7}"    # World!

# 字符串删除
filename="document.backup.txt"
echo "Remove prefix: ${filename#document.}"   # backup.txt
echo "Remove suffix: ${filename%.txt}"        # document.backup
echo "Remove longest prefix: ${filename##*.}" # txt
echo "Remove longest suffix: ${filename%%.*}" # document

# 字符串替换
text="Hello, World! Hello, Universe!"
echo "Replace first: ${text/Hello/Hi}"           # Hi, World! Hello, Universe!
echo "Replace all: ${text//Hello/Hi}"            # Hi, World! Hi, Universe!
echo "Replace prefix: ${text/#Hello/Hi}"         # Hi, World! Hello, Universe!
echo "Replace suffix: ${text/Universe!/Cosmos!}" # Hello, World! Hello, Cosmos!

# 字符串查找和匹配
text="The quick brown fox jumps over the lazy dog"
if [[ $text == *"fox"* ]]; then
    echo "Found 'fox' in the text"
fi

# 大小写转换
text="Hello, World!"
echo "Uppercase: ${text^^}"
echo "Lowercase: ${text,,}"
echo "Capitalize: ${text^}"

# 字符串分割
text="apple,banana,orange,grape"
IFS=',' read -ra fruits <<< "$text"
for fruit in "${fruits[@]}"; do
    echo "- $fruit"
done

# 字符串连接
first_name="John"
last_name="Doe"
full_name="${first_name} ${last_name}"
echo "Full name: $full_name"

# 字符串格式化
name="Alice"
age=25
formatted=$(printf "Name: %-10s Age: %3d\n" "$name" "$age")
echo "$formatted"

# 多行字符串
multi_line="This is a
multi-line
string"
echo "$multi_line"

# 使用 heredoc
cat << EOF
This is a heredoc example.
It can contain multiple lines.
And preserve formatting.
EOF

# 字符串去空格
text="  Hello, World!  "
trimmed_text=$(echo "$text" | xargs)
echo "Trimmed: '$trimmed_text'"

# 字符串反转
reverse=$(echo "$text" | rev)
echo "Reversed: $reverse"
```

### 正则表达式和模式匹配

```bash
#!/bin/bash

# 基本匹配
email="user@example.com"
if [[ $email =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Valid email address"
else
    echo "Invalid email address"
fi

# 电话号码验证
phone="123-456-7890"
if [[ $phone =~ ^[0-9]{3}-[0-9]{3}-[0-9]{4}$ ]]; then
    echo "Valid phone number"
else
    echo "Invalid phone number"
fi

# IP 地址验证
ip="192.168.1.1"
if [[ $ip =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]]; then
    echo "Valid IP address format"
else
    echo "Invalid IP address"
fi

# URL 验证
url="https://example.com/path?query=value"
if [[ $url =~ ^https?://[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}(/.*)?$ ]]; then
    echo "Valid URL"
else
    echo "Invalid URL"
fi

# 使用 grep 进行模式匹配
text="Hello World 123"
if echo "$text" | grep -q "World"; then
    echo "Found 'World' in text"
fi

# 提取匹配内容
text="Price: $19.99"
price=$(echo "$text" | grep -oP '\$\d+\.\d{2}')
echo "Extracted price: $price"

# 批量文件重命名
for file in image_*.jpg; do
    new_name=$(echo "$file" | sed 's/image_/photo_/')
    mv "$file" "$new_name"
done

# 查找特定模式
grep -r "TODO" *.py
grep -i "error" /var/log/syslog
grep -v "comment" file.txt

# 高级模式匹配
text="The price is $19.99 and the discount is 10%"
if [[ $text =~ \$[0-9]+\.[0-9]{2} ]]; then
    echo "Found price: ${BASH_REMATCH[0]}"
fi

# 多条件匹配
pattern="(error|warning|critical)"
text="This is a warning message"
if [[ $text =~ $pattern ]]; then
    echo "Found pattern: ${BASH_REMATCH[1]}"
fi
```

### 进程管理和信号处理

```bash
#!/bin/bash

# 获取进程信息
process_name="nginx"
pid=$(pgrep -f "$process_name")
if [ -n "$pid" ]; then
    echo "Process $process_name is running with PID: $pid"
else
    echo "Process $process_name is not running"
fi

# 查看进程详情
ps aux | grep "$process_name"
ps -p "$pid" -o pid,ppid,cmd,%mem,%cpu

# 进程状态监控
monitor_process() {
    local process_name=$1
    local max_memory=80  # MB
    local max_cpu=80     # %
    
    while true; do
        pid=$(pgrep -f "$process_name")
        if [ -n "$pid" ]; then
            memory=$(ps -p "$pid" -o %mem --no-headers | awk '{print int($1*100)}')
            cpu=$(ps -p "$pid" -o %cpu --no-headers | awk '{print int($1)}')
            
            echo "[$(date)] $process_name (PID: $pid) - Memory: ${memory}MB, CPU: ${cpu}%"
            
            if [ "$memory" -gt "$max_memory" ]; then
                echo "Warning: High memory usage (${memory}MB > ${max_memory}MB)"
            fi
            
            if [ "$cpu" -gt "$max_cpu" ]; then
                echo "Warning: High CPU usage (${cpu}% > ${max_cpu}%)"
            fi
        fi
        sleep 5
    done
}

# 信号处理
cleanup() {
    echo "Cleaning up..."
    kill $(jobs -p) 2>/dev/null
    exit 0
}

trap cleanup SIGINT SIGTERM

# 后台进程管理
start_background_process() {
    local command=$1
    local log_file=$2
    
    echo "Starting background process: $command"
    nohup $command > "$log_file" 2>&1 &
    local pid=$!
    echo "Process started with PID: $pid"
    
    # 保存 PID
    echo $pid > "${log_file}.pid"
    
    return $pid
}

# 停止后台进程
stop_background_process() {
    local log_file=$1
    
    if [ -f "${log_file}.pid" ]; then
        pid=$(cat "${log_file}.pid")
        echo "Stopping process with PID: $pid"
        kill $pid
        rm "${log_file}.pid"
    else
        echo "PID file not found"
    fi
}

# 使用示例
# start_background_process "python server.py" "server.log"
# stop_background_process "server.log"

# 进程等待和超时
wait_with_timeout() {
    local pid=$1
    local timeout=$2
    local count=0
    
    while kill -0 $pid 2>/dev/null; do
        if [ $count -ge $timeout ]; then
            echo "Timeout reached, killing process $pid"
            kill -9 $pid
            return 1
        fi
        sleep 1
        ((count++))
    done
    
    return 0
}

# 进程池管理
process_pool() {
    local max_processes=$1
    local command=$2
    local items=("${@:3}")
    local running=0
    
    for item in "${items[@]}"; do
        # 如果达到最大进程数，等待一个进程完成
        while [ $running -ge $max_processes ]; do
            wait -n
            ((running--))
        done
        
        # 启动新进程
        $command "$item" &
        ((running++))
    done
    
    # 等待所有进程完成
    wait
}

# 使用示例
# process_pool 4 "process_item" item1 item2 item3 item4 item5 item6
```

### 错误处理和调试

```bash
#!/bin/bash

# 错误处理选项
set -e          # 遇到错误立即退出
set -u          # 使用未定义变量时报错
set -o pipefail # 管道中任何命令失败都会导致整个管道失败
set -x          # 显示执行的每个命令（调试用）

# 自定义错误处理
error_handler() {
    local line_no=$1
    echo "Error occurred in script at line $line_no"
    # 清理操作
    cleanup
    exit 1
}

trap 'error_handler $LINENO' ERR

# 函数错误处理
safe_command() {
    local command=$1
    local error_msg=$2
    
    if ! $command; then
        echo "Error: $error_msg"
        return 1
    fi
    return 0
}

# 使用示例
if ! safe_command "mkdir /test/directory" "Failed to create directory"; then
    echo "Directory creation failed"
    exit 1
fi

# 调试函数
debug() {
    if [ "$DEBUG" = "true" ]; then
        echo "[DEBUG] $1"
    fi
}

# 使用示例
DEBUG=true
debug "This is a debug message"

# 日志函数
log() {
    local level=$1
    local message=$2
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    
    echo "[$timestamp] [$level] $message"
    
    # 写入日志文件
    echo "[$timestamp] [$level] $message" >> /var/log/script.log
}

# 使用示例
log "INFO" "Script started"
log "ERROR" "Failed to connect to database"

# 性能测量
time_command() {
    local start_time=$(date +%s)
    
    $@
    
    local end_time=$(date +%s)
    local duration=$((end_time - start_time))
    
    echo "Command completed in $duration seconds"
}

# 使用示例
time_command sleep 5

# 资源监控
monitor_resources() {
    echo "=== Resource Usage ==="
    echo "Memory:"
    free -h
    echo -e "\nDisk Usage:"
    df -h
    echo -e "\nCPU Load:"
    uptime
    echo "====================="
}

# 调试技巧
debug_script() {
    local script=$1
    
    # 使用 bash -x 调试
    echo "Running script in debug mode:"
    bash -x "$script"
    
    # 检查语法
    echo "Checking syntax:"
    bash -n "$script"
    
    # 查找常见问题
    echo "Checking for common issues:"
    grep -n "echo " "$script" | head -10
    grep -n "if " "$script" | head -10
}

# 使用示例
# debug_script "myscript.sh"
```

### 高级文件操作

```bash
#!/bin/bash

# 文件锁定
lock_file="/tmp/mylockfile"

acquire_lock() {
    if (set -o noclobber; echo "$$" > "$lock_file") 2> /dev/null; then
        echo "Lock acquired"
        return 0
    else
        echo "Lock already held by $(cat $lock_file)"
        return 1
    fi
}

release_lock() {
    if [ -f "$lock_file" ]; then
        rm "$lock_file"
        echo "Lock released"
    fi
}

# 使用示例
if acquire_lock; then
    # 执行需要锁定的操作
    sleep 10
    release_lock
fi

# 配置文件解析
parse_config() {
    local config_file=$1
    
    while IFS='=' read -r key value; do
        # 跳过注释和空行
        [[ $key =~ ^#.*$ ]] && continue
        [[ -z $key ]] && continue
        
        # 移除空格和引号
        key=$(echo "$key" | xargs)
        value=$(echo "$value" | xargs | sed 's/^["'\'']\(.*\)["'\'']$/\1/')
        
        # 设置变量
        eval "$key='$value'"
    done < "$config_file"
}

# 使用示例
# parse_config "config.ini"
# echo "$DATABASE_HOST"

# 批量文件处理
process_files() {
    local pattern=$1
    local action=$2
    
    for file in $pattern; do
        if [ -f "$file" ]; then
            echo "Processing: $file"
            $action "$file"
        fi
    done
}

# 文件比较
compare_files() {
    local file1=$1
    local file2=$2
    
    if [ ! -f "$file1" ] || [ ! -f "$file2" ]; then
        echo "One or both files not found"
        return 1
    fi
    
    if cmp -s "$file1" "$file2"; then
        echo "Files are identical"
        return 0
    else
        echo "Files are different"
        diff "$file1" "$file2"
        return 1
    fi
}

# 文件完整性检查
check_file_integrity() {
    local file=$1
    local expected_checksum=$2
    
    actual_checksum=$(md5sum "$file" | awk '{print $1}')
    
    if [ "$actual_checksum" = "$expected_checksum" ]; then
        echo "File integrity check passed"
        return 0
    else
        echo "File integrity check failed"
        echo "Expected: $expected_checksum"
        echo "Actual: $actual_checksum"
        return 1
    fi
}

# 递归文件操作
recursive_operation() {
    local directory=$1
    local operation=$2
    
    for item in "$directory"/*; do
        if [ -f "$item" ]; then
            $operation "$item"
        elif [ -d "$item" ]; then
            recursive_operation "$item" "$operation"
        fi
    done
}

# 使用示例
# recursive_operation "/path/to/dir" "process_file"

# 文件监控
monitor_file() {
    local file=$1
    local action=$2
    
    inotifywait -m -e modify,create,delete "$file" | while read event; do
        echo "File event: $event"
        $action "$file"
    done
}

# 使用示例（需要安装 inotify-tools）
# monitor_file "/path/to/file" "handle_change"
```

### 网络编程和API调用

```bash
#!/bin/bash

# HTTP GET 请求
http_get() {
    local url=$1
    local response=$(curl -s -w "\n%{http_code}" "$url")
    local body=$(echo "$response" | head -n -1)
    local status_code=$(echo "$response" | tail -n 1)
    
    echo "Status Code: $status_code"
    echo "Response Body: $body"
    
    return $status_code
}

# HTTP POST 请求
http_post() {
    local url=$1
    local data=$2
    
    curl -X POST \
         -H "Content-Type: application/json" \
         -d "$data" \
         "$url"
}

# API 认证
api_request() {
    local url=$1
    local token=$2
    
    curl -H "Authorization: Bearer $token" "$url"
}

# JSON 处理（需要 jq）
parse_json() {
    local json=$1
    local field=$2
    
    echo "$json" | jq -r ".$field"
}

# 使用示例
# response='{"name":"John","age":30}'
# name=$(parse_json "$response" "name")
# echo "Name: $name"

# 网络连接测试
test_connection() {
    local host=$1
    local port=$2
    local timeout=5
    
    if timeout $timeout bash -c "cat < /dev/null > /dev/tcp/$host/$port" 2>/dev/null; then
        echo "Connection to $host:$port successful"
        return 0
    else
        echo "Connection to $host:$port failed"
        return 1
    fi
}

# 使用示例
# test_connection "example.com" 80

# DNS 查询
dns_lookup() {
    local domain=$1
    
    echo "A records:"
    dig +short A "$domain"
    
    echo "MX records:"
    dig +short MX "$domain"
    
    echo "NS records:"
    dig +short NS "$domain"
}

# 端口扫描
scan_ports() {
    local host=$1
    local start_port=$2
    local end_port=$3
    
    echo "Scanning ports $start_port-$end_port on $host..."
    
    for port in $(seq $start_port $end_port); do
        if timeout 1 bash -c "echo > /dev/tcp/$host/$port" 2>/dev/null; then
            echo "Port $port is open"
        fi
    done
}

# 使用示例
# scan_ports "localhost" 1 1000

# 网络性能测试
network_test() {
    local host=$1
    local count=4
    
    echo "=== Network Test to $host ==="
    
    echo "Ping test:"
    ping -c $count "$host"
    
    echo "Download speed test:"
    time curl -o /dev/null "http://$host/10MB.test"
    
    echo "DNS resolution time:"
    time dig "$host"
    
    echo "Trace route:"
    traceroute "$host"
}
```

## 实际应用

### 场景一：自动化部署脚本

```bash
#!/bin/bash

# 自动化部署脚本
# 用途：自动化部署 Web 应用

set -e  # 遇到错误立即退出

# 配置变量
APP_NAME="myapp"
APP_DIR="/var/www/$APP_NAME"
REPO_URL="https://github.com/user/$APP_NAME.git"
BRANCH="main"
BACKUP_DIR="/backup/$APP_NAME"
LOG_FILE="/var/log/deploy.log"

# 函数定义
log() {
    local level=$1
    local message=$2
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $message" | tee -a "$LOG_FILE"
}

backup_current_version() {
    log "INFO" "Creating backup..."
    
    if [ -d "$APP_DIR" ]; then
        backup_name="backup_$(date +%Y%m%d_%H%M%S).tar.gz"
        tar -czf "$BACKUP_DIR/$backup_name" -C "$(dirname "$APP_DIR")" "$(basename "$APP_DIR")"
        log "INFO" "Backup created: $backup_name"
    else
        log "WARN" "No existing version to backup"
    fi
}

clone_repository() {
    log "INFO" "Cloning repository..."
    
    if [ -d "$APP_DIR" ]; then
        cd "$APP_DIR"
        git fetch origin
        git checkout $BRANCH
        git pull origin $BRANCH
    else
        git clone -b $BRANCH "$REPO_URL" "$APP_DIR"
        cd "$APP_DIR"
    fi
    
    log "INFO" "Repository cloned/updated"
}

install_dependencies() {
    log "INFO" "Installing dependencies..."
    
    cd "$APP_DIR"
    
    # 安装 Node.js 依赖
    if [ -f "package.json" ]; then
        npm install --production
        log "INFO" "Node.js dependencies installed"
    fi
    
    # 安装 Python 依赖
    if [ -f "requirements.txt" ]; then
        pip install -r requirements.txt
        log "INFO" "Python dependencies installed"
    fi
    
    # 安装 PHP 依赖
    if [ -f "composer.json" ]; then
        composer install --no-dev
        log "INFO" "PHP dependencies installed"
    fi
}

run_build() {
    log "INFO" "Building application..."
    
    cd "$APP_DIR"
    
    # 运行构建脚本
    if [ -f "build.sh" ]; then
        chmod +x build.sh
        ./build.sh
    elif [ -f "package.json" ] && grep -q '"build"' package.json; then
        npm run build
    fi
    
    log "INFO" "Build completed"
}

run_tests() {
    log "INFO" "Running tests..."
    
    cd "$APP_DIR"
    
    # 运行测试
    if [ -f "test.sh" ]; then
        chmod +x test.sh
        ./test.sh
    elif [ -f "package.json" ] && grep -q '"test"' package.json; then
        npm test
    fi
    
    log "INFO" "Tests completed"
}

update_permissions() {
    log "INFO" "Updating permissions..."
    
    chown -R www-data:www-data "$APP_DIR"
    find "$APP_DIR" -type f -exec chmod 644 {} \;
    find "$APP_DIR" -type d -exec chmod 755 {} \;
    
    # 设置可执行权限
    find "$APP_DIR" -name "*.sh" -exec chmod +x {} \;
    
    log "INFO" "Permissions updated"
}

restart_services() {
    log "INFO" "Restarting services..."
    
    # 重启 Nginx
    systemctl reload nginx
    
    # 重启应用服务
    if systemctl is-active --quiet "$APP_NAME"; then
        systemctl restart "$APP_NAME"
    fi
    
    log "INFO" "Services restarted"
}

health_check() {
    log "INFO" "Performing health check..."
    
    local max_attempts=10
    local attempt=1
    
    while [ $attempt -le $max_attempts ]; do
        if curl -f http://localhost:3000/health > /dev/null 2>&1; then
            log "INFO" "Health check passed"
            return 0
        fi
        
        log "WARN" "Health check failed, attempt $attempt/$max_attempts"
        sleep 5
        ((attempt++))
    done
    
    log "ERROR" "Health check failed after $max_attempts attempts"
    return 1
}

rollback() {
    log "ERROR" "Deployment failed, initiating rollback..."
    
    local latest_backup=$(ls -t "$BACKUP_DIR"/backup_*.tar.gz | head -1)
    
    if [ -n "$latest_backup" ]; then
        tar -xzf "$latest_backup" -C "$(dirname "$APP_DIR")"
        restart_services
        log "INFO" "Rollback completed"
    else
        log "ERROR" "No backup found for rollback"
    fi
}

# 主部署流程
main() {
    log "INFO" "Starting deployment of $APP_NAME"
    
    # 创建必要的目录
    mkdir -p "$BACKUP_DIR"
    mkdir -p "$(dirname "$LOG_FILE")"
    
    # 备份当前版本
    backup_current_version
    
    # 克隆或更新代码
    clone_repository
    
    # 安装依赖
    install_dependencies
    
    # 构建应用
    run_build
    
    # 运行测试
    if [ "$SKIP_TESTS" != "true" ]; then
        run_tests
    fi
    
    # 更新权限
    update_permissions
    
    # 重启服务
    restart_services
    
    # 健康检查
    if health_check; then
        log "INFO" "Deployment completed successfully"
        exit 0
    else
        rollback
        exit 1
    fi
}

# 执行主函数
main
```

### 场景二：系统监控告警脚本

```bash
#!/bin/bash

# 系统监控告警脚本
# 用途：监控系统资源并发送告警

# 配置
ALERT_EMAIL="admin@example.com"
DISCORD_WEBHOOK="https://discord.com/api/webhooks/..."
SLACK_WEBHOOK="https://hooks.slack.com/services/..."
TELEGRAM_BOT_TOKEN="your_bot_token"
TELEGRAM_CHAT_ID="your_chat_id"

# 阈值设置
CPU_THRESHOLD=80
MEMORY_THRESHOLD=80
DISK_THRESHOLD=90
LOAD_THRESHOLD=2.0

# 日志文件
LOG_FILE="/var/log/system_monitor.log"

# 函数定义
log() {
    local level=$1
    local message=$2
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $message" | tee -a "$LOG_FILE"
}

send_email() {
    local subject=$1
    local body=$2
    
    echo "$body" | mail -s "$subject" "$ALERT_EMAIL"
    log "INFO" "Email sent to $ALERT_EMAIL"
}

send_discord() {
    local message=$1
    
    curl -X POST -H "Content-Type: application/json" \
         -d "{\"content\": \"$message\"}" \
         "$DISCORD_WEBHOOK"
    
    log "INFO" "Discord message sent"
}

send_slack() {
    local message=$1
    
    curl -X POST -H "Content-Type: application/json" \
         -d "{\"text\": \"$message\"}" \
         "$SLACK_WEBHOOK"
    
    log "INFO" "Slack message sent"
}

send_telegram() {
    local message=$1
    
    curl -X POST "https://api.telegram.org/bot$TELEGRAM_BOT_TOKEN/sendMessage" \
         -d "chat_id=$TELEGRAM_CHAT_ID" \
         -d "text=$message"
    
    log "INFO" "Telegram message sent"
}

send_alert() {
    local severity=$1
    local subject=$2
    local message=$3
    
    # 根据严重程度选择通知方式
    case $severity in
        "CRITICAL")
            send_email "$subject" "$message"
            send_discord "🚨 **$subject**\n$message"
            send_slack "🚨 *$subject*\n$message"
            send_telegram "🚨 $subject\n$message"
            ;;
        "WARNING")
            send_discord "⚠️ **$subject**\n$message"
            send_slack "⚠️ *$subject*\n$message"
            ;;
        "INFO")
            log "INFO" "$message"
            ;;
    esac
}

check_cpu() {
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d '%' -f 1)
    
    if (( $(echo "$cpu_usage > $CPU_THRESHOLD" | bc -l) )); then
        local message="CPU usage is critical: ${cpu_usage}%"
        log "ERROR" "$message"
        send_alert "CRITICAL" "CPU Alert" "$message"
        return 1
    elif (( $(echo "$cpu_usage > $((CPU_THRESHOLD - 10))" | bc -l) )); then
        local message="CPU usage is high: ${cpu_usage}%"
        log "WARN" "$message"
        send_alert "WARNING" "CPU Warning" "$message"
    fi
    
    return 0
}

check_memory() {
    local memory_usage=$(free | grep Mem | awk '{print ($3/$2)*100}')
    
    if (( $(echo "$memory_usage > $MEMORY_THRESHOLD" | bc -l) )); then
        local message="Memory usage is critical: ${memory_usage}%"
        log "ERROR" "$message"
        send_alert "CRITICAL" "Memory Alert" "$message"
        return 1
    elif (( $(echo "$memory_usage > $((MEMORY_THRESHOLD - 10))" | bc -l) )); then
        local message="Memory usage is high: ${memory_usage}%"
        log "WARN" "$message"
        send_alert "WARNING" "Memory Warning" "$message"
    fi
    
    return 0
}

check_disk() {
    local disk_usage=$(df -h / | awk 'NR==2 {print $5}' | cut -d '%' -f 1)
    
    if [ "$disk_usage" -gt "$DISK_THRESHOLD" ]; then
        local message="Disk usage is critical: ${disk_usage}%"
        log "ERROR" "$message"
        send_alert "CRITICAL" "Disk Alert" "$message"
        return 1
    elif [ "$disk_usage" -gt "$((DISK_THRESHOLD - 5))" ]; then
        local message="Disk usage is high: ${disk_usage}%"
        log "WARN" "$message"
        send_alert "WARNING" "Disk Warning" "$message"
    fi
    
    return 0
}

check_load() {
    local load_average=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | cut -d ',' -f 1)
    
    if (( $(echo "$load_average > $LOAD_THRESHOLD" | bc -l) )); then
        local message="System load is high: $load_average"
        log "ERROR" "$message"
        send_alert "CRITICAL" "Load Alert" "$message"
        return 1
    fi
    
    return 0
}

check_services() {
    local services=("nginx" "mysql" "redis" "mongodb")
    local failed_services=()
    
    for service in "${services[@]}"; do
        if ! systemctl is-active --quiet "$service"; then
            failed_services+=("$service")
        fi
    done
    
    if [ ${#failed_services[@]} -gt 0 ]; then
        local message="The following services are not running: ${failed_services[*]}"
        log "ERROR" "$message"
        send_alert "CRITICAL" "Service Alert" "$message"
        
        # 尝试重启服务
        for service in "${failed_services[@]}"; do
            log "INFO" "Attempting to restart $service..."
            systemctl restart "$service"
            
            if systemctl is-active --quiet "$service"; then
                log "INFO" "$service restarted successfully"
            else
                log "ERROR" "Failed to restart $service"
            fi
        done
        
        return 1
    fi
    
    return 0
}

check_network() {
    local test_host="8.8.8.8"
    
    if ! ping -c 1 -W 5 "$test_host" > /dev/null 2>&1; then
        local message="Network connectivity issue: cannot reach $test_host"
        log "ERROR" "$message"
        send_alert "CRITICAL" "Network Alert" "$message"
        return 1
    fi
    
    return 0
}

generate_report() {
    log "INFO" "Generating system report..."
    
    echo "=== System Report ===" | tee -a "$LOG_FILE"
    echo "Date: $(date)" | tee -a "$LOG_FILE"
    echo "Hostname: $(hostname)" | tee -a "$LOG_FILE"
    echo "" | tee -a "$LOG_FILE"
    
    echo "CPU Usage:" | tee -a "$LOG_FILE"
    top -bn1 | grep "Cpu(s)" | tee -a "$LOG_FILE"
    echo "" | tee -a "$LOG_FILE"
    
    echo "Memory Usage:" | tee -a "$LOG_FILE"
    free -h | tee -a "$LOG_FILE"
    echo "" | tee -a "$LOG_FILE"
    
    echo "Disk Usage:" | tee -a "$LOG_FILE"
    df -h | tee -a "$LOG_FILE"
    echo "" | tee -a "$LOG_FILE"
    
    echo "System Load:" | tee -a "$LOG_FILE"
    uptime | tee -a "$LOG_FILE"
    echo "" | tee -a "$LOG_FILE"
    
    echo "Running Services:" | tee -a "$LOG_FILE"
    systemctl list-units --type=service --state=running | tee -a "$LOG_FILE"
    echo "" | tee -a "$LOG_FILE"
    
    echo "=== End of Report ===" | tee -a "$LOG_FILE"
}

# 主监控循环
main() {
    log "INFO" "System monitoring started"
    
    while true; do
        # 执行各项检查
        check_cpu
        check_memory
        check_disk
        check_load
        check_services
        check_network
        
        # 每小时生成一次报告
        if [ $(date +%M) -eq 0 ]; then
            generate_report
        fi
        
        # 等待 5 分钟后再次检查
        sleep 300
    done
}

# 执行主函数
main
```

### 场景三：数据库备份和迁移脚本

```bash
#!/bin/bash

# 数据库备份和迁移脚本
# 用途：自动化数据库备份、恢复和迁移

set -e  # 遇到错误立即退出

# 配置变量
DB_TYPE="mysql"  # mysql, postgresql, mongodb
DB_HOST="localhost"
DB_PORT=3306
DB_NAME="myapp"
DB_USER="backup_user"
DB_PASS="secure_password"
BACKUP_DIR="/backup/database"
RETENTION_DAYS=30
S3_BUCKET="s3://my-backups/database"

# 日志文件
LOG_FILE="/var/log/db_backup.log"

# 函数定义
log() {
    local level=$1
    local message=$2
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $message" | tee -a "$LOG_FILE"
}

create_backup_dir() {
    mkdir -p "$BACKUP_DIR"
    log "INFO" "Backup directory created: $BACKUP_DIR"
}

backup_mysql() {
    local backup_file="$BACKUP_DIR/${DB_NAME}_$(date +%Y%m%d_%H%M%S).sql.gz"
    
    log "INFO" "Starting MySQL backup for $DB_NAME"
    
    # 使用 mysqldump 备份数据库
    mysqldump -h "$DB_HOST" -P "$DB_PORT" -u "$DB_USER" -p"$DB_PASS" \
              --single-transaction --routines --triggers \
              "$DB_NAME" | gzip > "$backup_file"
    
    if [ $? -eq 0 ]; then
        local size=$(du -h "$backup_file" | cut -f1)
        log "INFO" "MySQL backup completed: $backup_file ($size)"
        return 0
    else
        log "ERROR" "MySQL backup failed"
        return 1
    fi
}

backup_postgresql() {
    local backup_file="$BACKUP_DIR/${DB_NAME}_$(date +%Y%m%d_%H%M%S).sql.gz"
    
    log "INFO" "Starting PostgreSQL backup for $DB_NAME"
    
    # 使用 pg_dump 备份数据库
    PGPASSWORD="$DB_PASS" pg_dump -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" \
              -d "$DB_NAME" --no-owner --no-acl | gzip > "$backup_file"
    
    if [ $? -eq 0 ]; then
        local size=$(du -h "$backup_file" | cut -f1)
        log "INFO" "PostgreSQL backup completed: $backup_file ($size)"
        return 0
    else
        log "ERROR" "PostgreSQL backup failed"
        return 1
    fi
}

backup_mongodb() {
    local backup_file="$BACKUP_DIR/${DB_NAME}_$(date +%Y%m%d_%H%M%S).tar.gz"
    local temp_dir="/tmp/mongodb_backup_$(date +%Y%m%d_%H%M%S)"
    
    log "INFO" "Starting MongoDB backup for $DB_NAME"
    
    mkdir -p "$temp_dir"
    
    # 使用 mongodump 备份数据库
    mongodump --host "$DB_HOST" --port "$DB_PORT" \
              --username "$DB_USER" --password "$DB_PASS" \
              --db "$DB_NAME" --out "$temp_dir"
    
    if [ $? -eq 0 ]; then
        # 压缩备份文件
        tar -czf "$backup_file" -C "$temp_dir" .
        rm -rf "$temp_dir"
        
        local size=$(du -h "$backup_file" | cut -f1)
        log "INFO" "MongoDB backup completed: $backup_file ($size)"
        return 0
    else
        log "ERROR" "MongoDB backup failed"
        rm -rf "$temp_dir"
        return 1
    fi
}

backup_database() {
    case $DB_TYPE in
        "mysql")
            backup_mysql
            ;;
        "postgresql")
            backup_postgresql
            ;;
        "mongodb")
            backup_mongodb
            ;;
        *)
            log "ERROR" "Unsupported database type: $DB_TYPE"
            return 1
            ;;
    esac
}

upload_to_s3() {
    local backup_file=$1
    
    log "INFO" "Uploading backup to S3: $backup_file"
    
    # 使用 aws CLI 上传到 S3
    aws s3 cp "$backup_file" "$S3_BUCKET/"
    
    if [ $? -eq 0 ]; then
        log "INFO" "Backup uploaded to S3 successfully"
        return 0
    else
        log "ERROR" "Failed to upload backup to S3"
        return 1
    fi
}

cleanup_old_backups() {
    log "INFO" "Cleaning up old backups (older than $RETENTION_DAYS days)"
    
    # 删除本地旧备份
    find "$BACKUP_DIR" -name "${DB_NAME}_*.sql.gz" -mtime +$RETENTION_DAYS -delete
    find "$BACKUP_DIR" -name "${DB_NAME}_*.tar.gz" -mtime +$RETENTION_DAYS -delete
    
    # 删除 S3 上的旧备份
    aws s3 ls "$S3_BUCKET/" --recursive | while read -r line; do
        file_date=$(echo "$line" | awk '{print $1}' | sed 's/-//g')
        current_date=$(date +%Y%m%d)
        
        if [ $((current_date - file_date)) -gt $RETENTION_DAYS ]; then
            file_key=$(echo "$line" | awk '{print $4}')
            aws s3 rm "$S3_BUCKET/$file_key"
            log "INFO" "Deleted old backup from S3: $file_key"
        fi
    done
}

verify_backup() {
    local backup_file=$1
    
    log "INFO" "Verifying backup: $backup_file"
    
    # 检查文件是否存在且不为空
    if [ ! -f "$backup_file" ] || [ ! -s "$backup_file" ]; then
        log "ERROR" "Backup file is empty or does not exist"
        return 1
    fi
    
    # 检查文件完整性
    case $DB_TYPE in
        "mysql"|"postgresql")
            # 检查 SQL 文件是否有效
            if gzip -t "$backup_file" 2>/dev/null; then
                log "INFO" "Backup integrity check passed"
                return 0
            else
                log "ERROR" "Backup file is corrupted"
                return 1
            fi
            ;;
        "mongodb")
            # 检查 tar 文件是否有效
            if tar -tzf "$backup_file" > /dev/null 2>&1; then
                log "INFO" "Backup integrity check passed"
                return 0
            else
                log "ERROR" "Backup file is corrupted"
                return 1
            fi
            ;;
    esac
}

restore_mysql() {
    local backup_file=$1
    
    log "INFO" "Restoring MySQL database from: $backup_file"
    
    # 解压并恢复数据库
    gunzip -c "$backup_file" | mysql -h "$DB_HOST" -P "$DB_PORT" -u "$DB_USER" -p"$DB_PASS" "$DB_NAME"
    
    if [ $? -eq 0 ]; then
        log "INFO" "MySQL database restored successfully"
        return 0
    else
        log "ERROR" "MySQL database restore failed"
        return 1
    fi
}

restore_postgresql() {
    local backup_file=$1
    
    log "INFO" "Restoring PostgreSQL database from: $backup_file"
    
    # 解压并恢复数据库
    gunzip -c "$backup_file" | PGPASSWORD="$DB_PASS" psql -h "$DB_HOST" -p "$DB_PORT" -U "$DB_USER" -d "$DB_NAME"
    
    if [ $? -eq 0 ]; then
        log "INFO" "PostgreSQL database restored successfully"
        return 0
    else
        log "ERROR" "PostgreSQL database restore failed"
        return 1
    fi
}

restore_mongodb() {
    local backup_file=$1
    local temp_dir="/tmp/mongodb_restore_$(date +%Y%m%d_%H%M%S)"
    
    log "INFO" "Restoring MongoDB database from: $backup_file"
    
    mkdir -p "$temp_dir"
    
    # 解压备份文件
    tar -xzf "$backup_file" -C "$temp_dir"
    
    # 恢复数据库
    mongorestore --host "$DB_HOST" --port "$DB_PORT" \
                 --username "$DB_USER" --password "$DB_PASS" \
                 --db "$DB_NAME" "$temp_dir/$DB_NAME"
    
    if [ $? -eq 0 ]; then
        log "INFO" "MongoDB database restored successfully"
        rm -rf "$temp_dir"
        return 0
    else
        log "ERROR" "MongoDB database restore failed"
        rm -rf "$temp_dir"
        return 1
    fi
}

restore_database() {
    local backup_file=$1
    
    if [ ! -f "$backup_file" ]; then
        log "ERROR" "Backup file not found: $backup_file"
        return 1
    fi
    
    case $DB_TYPE in
        "mysql")
            restore_mysql "$backup_file"
            ;;
        "postgresql")
            restore_postgresql "$backup_file"
            ;;
        "mongodb")
            restore_mongodb "$backup_file"
            ;;
        *)
            log "ERROR" "Unsupported database type: $DB_TYPE"
            return 1
            ;;
    esac
}

list_backups() {
    log "INFO" "Listing available backups:"
    
    echo "Local backups:"
    ls -lh "$BACKUP_DIR"/${DB_NAME}_* 2>/dev/null || echo "No local backups found"
    
    echo -e "\nS3 backups:"
    aws s3 ls "$S3_BUCKET/" --recursive --human-readable | grep "$DB_NAME" || echo "No S3 backups found"
}

# 主函数
main() {
    local operation=$1
    local backup_file=$2
    
    log "INFO" "Database backup script started"
    create_backup_dir
    
    case $operation in
        "backup")
            if backup_file=$(backup_database); then
                verify_backup "$backup_file"
                upload_to_s3 "$backup_file"
                cleanup_old_backups
            else
                log "ERROR" "Backup operation failed"
                exit 1
            fi
            ;;
        "restore")
            if [ -z "$backup_file" ]; then
                log "ERROR" "Please specify backup file to restore"
                exit 1
            fi
            
            restore_database "$backup_file"
            ;;
        "list")
            list_backups
            ;;
        *)
            echo "Usage: $0 {backup|restore|list} [backup_file]"
            echo "  backup  - Create a new database backup"
            echo "  restore - Restore database from backup file"
            echo "  list    - List available backups"
            exit 1
            ;;
    esac
    
    log "INFO" "Database backup script completed"
}

# 执行主函数
main "$@"
```

### 场景四：自动化测试和CI/CD脚本

```bash
#!/bin/bash

# 自动化测试和CI/CD脚本
# 用途：集成自动化测试、代码质量检查和部署流程

set -e  # 遇到错误立即退出

# 配置变量
PROJECT_NAME="myproject"
PROJECT_DIR="/workspace/$PROJECT_NAME"
TEST_REPORTS_DIR="$PROJECT_DIR/test-reports"
COVERAGE_DIR="$PROJECT_DIR/coverage"
ARTIFACTS_DIR="$PROJECT_DIR/artifacts"

# Git 配置
GIT_BRANCH=${GIT_BRANCH:-"main"}
GIT_COMMIT=${GIT_COMMIT:-"$(git rev-parse HEAD)"}

# 通知配置
SLACK_WEBHOOK=${SLACK_WEBHOOK:-""}
EMAIL_RECIPIENTS=${EMAIL_RECIPIENTS:-""}

# 函数定义
log() {
    local level=$1
    local message=$2
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    echo "[$timestamp] [$level] $message"
}

setup_environment() {
    log "INFO" "Setting up test environment..."
    
    # 创建必要的目录
    mkdir -p "$TEST_REPORTS_DIR"
    mkdir -p "$COVERAGE_DIR"
    mkdir -p "$ARTIFACTS_DIR"
    
    # 安装依赖
    cd "$PROJECT_DIR"
    
    if [ -f "package.json" ]; then
        log "INFO" "Installing Node.js dependencies..."
        npm ci
    fi
    
    if [ -f "requirements.txt" ]; then
        log "INFO" "Installing Python dependencies..."
        pip install -r requirements.txt
        pip install pytest pytest-cov pytest-html
    fi
    
    if [ -f "Gemfile" ]; then
        log "INFO" "Installing Ruby dependencies..."
        bundle install
    fi
    
    log "INFO" "Environment setup completed"
}

code_quality_check() {
    log "INFO" "Running code quality checks..."
    
    cd "$PROJECT_DIR"
    
    # ESLint 检查
    if [ -f ".eslintrc.json" ] || [ -f ".eslintrc.js" ]; then
        log "INFO" "Running ESLint..."
        npm run lint -- --format json --output-file "$TEST_REPORTS_DIR/eslint-report.json" || true
        
        if npm run lint; then
            log "INFO" "ESLint checks passed"
        else
            log "ERROR" "ESLint checks failed"
            return 1
        fi
    fi
    
    # Prettier 检查
    if [ -f ".prettierrc" ] || [ -f ".prettierrc.json" ]; then
        log "INFO" "Running Prettier..."
        npm run format:check || true
        
        if npm run format:check; then
            log "INFO" "Prettier checks passed"
        else
            log "ERROR" "Prettier checks failed"
            return 1
        fi
    fi
    
    # Pylint 检查
    if [ -f "pylintrc" ] || [ -f ".pylintrc" ]; then
        log "INFO" "Running Pylint..."
        pylint --output-format=json --output="$TEST_REPORTS_DIR/pylint-report.json" src/ || true
    fi
    
    # 安全检查
    log "INFO" "Running security checks..."
    
    if [ -f "package.json" ]; then
        npm audit --audit-level=high || true
    fi
    
    if [ -f "requirements.txt" ]; then
        safety check --json > "$TEST_REPORTS_DIR/safety-report.json" || true
    fi
    
    log "INFO" "Code quality checks completed"
}

unit_tests() {
    log "INFO" "Running unit tests..."
    
    cd "$PROJECT_DIR"
    
    # JavaScript/TypeScript 测试
    if [ -f "package.json" ] && grep -q '"test"' package.json; then
        log "INFO" "Running Jest tests..."
        
        npm test -- \
            --coverage \
            --coverageReporters=json \
            --coverageReporters=html \
            --reporters=default \
            --reporters=json-summary \
            --outputFile="$TEST_REPORTS_DIR/jest-report.json"
        
        # 移动覆盖率报告
        if [ -d "coverage" ]; then
            mv coverage/* "$COVERAGE_DIR/"
        fi
    fi
    
    # Python 测试
    if [ -f "requirements.txt" ]; then
        log "INFO" "Running Pytest tests..."
        
        pytest \
            --cov=src \
            --cov-report=json:"$TEST_REPORTS_DIR/coverage.json" \
            --cov-report=html:"$COVERAGE_DIR" \
            --html="$TEST_REPORTS_DIR/pytest-report.html" \
            --self-contained-html \
            --junitxml="$TEST_REPORTS_DIR/junit-report.xml" \
            -v
    fi
    
    log "INFO" "Unit tests completed"
}

integration_tests() {
    log "INFO" "Running integration tests..."
    
    cd "$PROJECT_DIR"
    
    # 启动测试服务
    log "INFO" "Starting test services..."
    docker-compose -f docker-compose.test.yml up -d
    
    # 等待服务就绪
    log "INFO" "Waiting for services to be ready..."
    sleep 10
    
    # 运行集成测试
    if [ -f "package.json" ] && grep -q '"test:integration"' package.json; then
        npm run test:integration
    fi
    
    # 清理测试服务
    log "INFO" "Cleaning up test services..."
    docker-compose -f docker-compose.test.yml down
    
    log "INFO" "Integration tests completed"
}

build_project() {
    log "INFO" "Building project..."
    
    cd "$PROJECT_DIR"
    
    # 构建项目
    if [ -f "package.json" ] && grep -q '"build"' package.json; then
        npm run build
    fi
    
    if [ -f "build.sh" ]; then
        chmod +x build.sh
        ./build.sh
    fi
    
    # 收集构建产物
    if [ -d "dist" ]; then
        tar -czf "$ARTIFACTS_DIR/dist.tar.gz" -C dist .
    fi
    
    if [ -d "build" ]; then
        tar -czf "$ARTIFACTS_DIR/build.tar.gz" -C build .
    fi
    
    log "INFO" "Project build completed"
}

generate_report() {
    log "INFO" "Generating test report..."
    
    local report_file="$TEST_REPORTS_DIR/test-report.html"
    
    cat > "$report_file" << EOF
<!DOCTYPE html>
<html>
<head>
    <title>Test Report - $PROJECT_NAME</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .header { background: #f0f0f0; padding: 20px; margin-bottom: 20px; }
        .section { margin-bottom: 30px; }
        .success { color: green; }
        .error { color: red; }
        .warning { color: orange; }
        table { border-collapse: collapse; width: 100%; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f2f2f2; }
    </style>
</head>
<body>
    <div class="header">
        <h1>Test Report - $PROJECT_NAME</h1>
        <p>Branch: $GIT_BRANCH</p>
        <p>Commit: $GIT_COMMIT</p>
        <p>Date: $(date)</p>
    </div>
    
    <div class="section">
        <h2>Test Results</h2>
        $(cat "$TEST_REPORTS_DIR/junit-report.xml" 2>/dev/null | grep -o 'tests="[0-9]*"' | head -1 || echo "No test results available")
    </div>
    
    <div class="section">
        <h2>Code Coverage</h2>
        $(cat "$TEST_REPORTS_DIR/coverage-summary.json" 2>/dev/null || echo "No coverage data available")
    </div>
    
    <div class="section">
        <h2>Code Quality</h2>
        $(cat "$TEST_REPORTS_DIR/eslint-report.json" 2>/dev/null | jq '.' || echo "No ESLint data available")
    </div>
</body>
</html>
EOF
    
    log "INFO" "Test report generated: $report_file"
}

send_notification() {
    local status=$1
    local message=$2
    
    log "INFO" "Sending notification: $status"
    
    # Slack 通知
    if [ -n "$SLACK_WEBHOOK" ]; then
        local color="good"
        if [ "$status" = "FAILURE" ]; then
            color="danger"
        elif [ "$status" = "WARNING" ]; then
            color="warning"
        fi
        
        curl -X POST -H "Content-Type: application/json" \
             -d "{
                 \"attachments\": [{
                     \"color\": \"$color\",
                     \"title\": \"CI/CD Pipeline - $status\",
                     \"text\": \"$message\",
                     \"fields\": [{
                         \"title\": \"Project\",
                         \"value\": \"$PROJECT_NAME\",
                         \"short\": true
                     }, {
                         \"title\": \"Branch\",
                         \"value\": \"$GIT_BRANCH\",
                         \"short\": true
                     }, {
                         \"title\": \"Commit\",
                         \"value\": \"$GIT_COMMIT\",
                         \"short\": true
                     }]
                 }]
             }" \
             "$SLACK_WEBHOOK"
    fi
    
    # Email 通知
    if [ -n "$EMAIL_RECIPIENTS" ]; then
        echo "$message" | mail -s "CI/CD Pipeline - $status: $PROJECT_NAME" "$EMAIL_RECIPIENTS"
    fi
}

# 主CI/CD流程
main() {
    local stage=${1:-"all"}
    
    log "INFO" "Starting CI/CD pipeline for $PROJECT_NAME"
    log "INFO" "Stage: $stage"
    
    case $stage in
        "setup")
            setup_environment
            ;;
        "quality")
            code_quality_check
            ;;
        "unit")
            unit_tests
            ;;
        "integration")
            integration_tests
            ;;
        "build")
            build_project
            ;;
        "report")
            generate_report
            ;;
        "all")
            # 执行完整的 CI/CD 流程
            if setup_environment && \
               code_quality_check && \
               unit_tests && \
               integration_tests && \
               build_project; then
                generate_report
                send_notification "SUCCESS" "CI/CD pipeline completed successfully"
                log "INFO" "CI/CD pipeline completed successfully"
                exit 0
            else
                generate_report
                send_notification "FAILURE" "CI/CD pipeline failed"
                log "ERROR" "CI/CD pipeline failed"
                exit 1
            fi
            ;;
        *)
            echo "Usage: $0 {setup|quality|unit|integration|build|report|all}"
            echo "  setup        - Set up test environment"
            echo "  quality      - Run code quality checks"
            echo "  unit         - Run unit tests"
            echo "  integration  - Run integration tests"
            echo "  build        - Build project"
            echo "  report       - Generate test report"
            echo "  all          - Run complete CI/CD pipeline"
            exit 1
            ;;
    esac
}

# 执行主函数
main "$@"
```

## 注意事项

1. **安全性考虑**：
   - 不要在脚本中硬编码敏感信息
   - 使用环境变量或配置文件存储密码
   - 验证所有用户输入，防止注入攻击
   - 使用最小权限原则运行脚本

2. **错误处理**：
   - 实现全面的错误处理机制
   - 提供有意义的错误信息
   - 实现回滚机制
   - 记录详细的日志信息

3. **性能优化**：
   - 避免不必要的循环和重复计算
   - 使用并行处理加速操作
   - 优化文件操作和网络请求
   - 实现缓存机制

4. **可维护性**：
   - 使用模块化设计
   - 编写清晰的文档
   - 保持代码一致性
   - 实现配置文件

5. **兼容性**：
   - 测试脚本在不同环境中的兼容性
   - 考虑不同操作系统的差异
   - 使用可移植的命令
   - 提供降级方案

6. **监控和日志**：
   - 实现详细的日志记录
   - 提供性能监控
   - 设置告警机制
   - 定期审计脚本运行情况

## 总结

Shell 脚本进阶技术让你能够编写更专业、更强大的自动化脚本。通过掌握这些高级特性，你可以：

- **高级文本处理**：使用正则表达式和字符串操作处理复杂数据
- **进程管理**：控制和管理多个进程的执行
- **网络编程**：实现网络通信和API调用
- **系统集成**：与系统服务和外部工具深度集成
- **自动化部署**：实现完整的CI/CD流程
- **监控告警**：建立完善的监控和告警系统

记住，优秀的Shell脚本不仅仅是能工作的代码，它应该：
- **健壮可靠**：能够处理各种异常情况
- **安全高效**：遵循安全原则，性能优化
- **易于维护**：代码结构清晰，文档完善
- **功能完整**：满足实际业务需求
- **可扩展性**：便于后续功能扩展

继续深入学习和实践，你会发现Shell脚本在自动化领域有着无限的可能性。从简单的脚本到复杂的自动化系统，Shell脚本都能为你提供强大的支持。