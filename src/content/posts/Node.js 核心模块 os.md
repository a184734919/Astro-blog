---
title: Node.js 核心模块 os
published: 2023-02-06
description: '操作系统信息获取的详细介绍和学习笔记'
image: ''
tags: ["Node.js","核心模块","操作系统"]
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 的 `os` 模块提供了与操作系统相关的基本实用程序和方法，用于获取系统信息、网络接口、CPU、内存等信息。这些信息对于系统监控、性能优化、跨平台开发、负载均衡等场景非常重要。

`os` 模块是 Node.js 的内置模块，无需安装即可直接使用：

```javascript
const os = require('os');
```

## 核心概念

### 系统信息获取
`os` 模块提供了多种获取操作系统信息的方法，包括系统平台、架构、版本等基本信息。

### 硬件资源监控
可以获取 CPU 核心信息、内存使用情况、系统负载等硬件资源相关数据，这对于性能监控和资源分配至关重要。

### 网络接口管理
能够获取所有网络接口的详细信息，包括 IP 地址、MAC 地址、网络掩码等，用于网络配置和管理。

### 路径和用户信息
提供跨平台的路径处理和用户信息获取，确保应用在不同操作系统上的一致性。

### 系统常量
包含系统相关的常量，如信号常量、错误码常量、优先级常量等，用于系统编程。

## 基本用法

### 1. 系统基本信息

#### 操作系统和架构信息

```javascript
const os = require('os');

// 操作系统平台
console.log('操作系统平台:', os.platform());
// 'darwin' - macOS
// 'linux' - Linux
// 'win32' - Windows
// 'freebsd' - FreeBSD
// 'openbsd' - OpenBSD
// 'sunos' - Solaris

// 操作系统版本
console.log('操作系统版本:', os.release());
// '18.7.0' - macOS 版本号
// '5.4.0-42-generic' - Linux 内核版本
// '10.0.19041' - Windows 版本号

// 操作系统类型
console.log('操作系统类型:', os.type());
// 'Darwin' - macOS
// 'Linux' - Linux
// 'Windows_NT' - Windows

// CPU 架构
console.log('CPU 架构:', os.arch());
// 'x64' - 64位 Intel/AMD
// 'arm' - ARM (32位)
// 'arm64' - ARM (64位)
// 'ia32' - Intel 32位
// 'mips' - MIPS
// 'mipsel' - MIPS Little Endian
// 'ppc' - PowerPC
// 'ppc64' - PowerPC 64位
// 's390' - IBM System/390
// 's390x' - IBM System/390 64位

// 操作系统主机名
console.log('主机名:', os.hostname());
// 'MacBook-Pro.local' 或 'DESKTOP-XXXXXXX'
```

#### 系统常量

```javascript
const os = require('os');

// 信号常量 - 用于进程控制
console.log('SIGINT:', os.constants.signals.SIGINT);    // 2 - 中断信号 (Ctrl+C)
console.log('SIGTERM:', os.constants.signals.SIGTERM);   // 15 - 终止信号
console.log('SIGKILL:', os.constants.signals.SIGKILL);   // 9 - 强制终止信号
console.log('SIGSTOP:', os.constants.signals.SIGSTOP);   // 19 - 暂停信号
console.log('SIGCONT:', os.constants.signals.SIGCONT);   // 18 - 继续执行信号

// 错误常量 - 用于错误处理
console.log('EEXIST:', os.constants.errno.EEXIST);       // 文件已存在
console.log('ENOENT:', os.constants.errno.ENOENT);       // 文件不存在
console.log('EACCES:', os.constants.errno.EACCES);       // 权限被拒绝
console.log('ENOMEM:', os.constants.errno.ENOMEM);       // 内存不足

// 优先级常量 - 用于进程优先级设置
console.log('PRIORITY_LOW:', os.constants.priority.PRIORITY_LOW);     // 19 - 低优先级
console.log('PRIORITY_NORMAL:', os.constants.priority.PRIORITY_NORMAL); // 0 - 普通优先级
console.log('PRIORITY_HIGH:', os.constants.priority.PRIORITY_HIGH);   // -10 - 高优先级

// 信号处理示例
process.on('SIGINT', () => {
  console.log('收到 SIGINT 信号，优雅退出...');
  // 执行清理工作
  process.exit(0);
});

// 设置进程优先级 (需要管理员权限)
try {
  os.setPriority(os.constants.priority.PRIORITY_LOW);
  console.log('进程优先级已设置为低');
} catch (error) {
  console.log('无法设置进程优先级 (可能需要管理员权限)');
}
```

### 2. 内存信息

#### 基础内存获取

```javascript
const os = require('os');

// 系统总内存（字节）
const totalMemory = os.totalmem();
console.log('总内存 (字节):', totalMemory);
console.log('总内存 (GB):', (totalMemory / 1024 / 1024 / 1024).toFixed(2));

// 系统空闲内存（字节）
const freeMemory = os.freemem();
console.log('空闲内存 (字节):', freeMemory);
console.log('空闲内存 (GB):', (freeMemory / 1024 / 1024 / 1024).toFixed(2));

// 内存使用情况计算
const usedMemory = totalMemory - freeMemory;
const usagePercent = ((usedMemory / totalMemory) * 100);

console.log('已用内存 (GB):', (usedMemory / 1024 / 1024 / 1024).toFixed(2));
console.log('内存使用率:', usagePercent.toFixed(2), '%');

// 内存状态检查
function getMemoryStatus() {
  const total = os.totalmem();
  const free = os.freemem();
  const used = total - free;
  const percent = (used / total) * 100;

  let status = '正常';
  if (percent > 80) status = '警告';
  if (percent > 90) status = '严重';

  return {
    total: (total / 1024 / 1024 / 1024).toFixed(2) + ' GB',
    used: (used / 1024 / 1024 / 1024).toFixed(2) + ' GB',
    free: (free / 1024 / 1024 / 1024).toFixed(2) + ' GB',
    percent: percent.toFixed(2) + '%',
    status
  };
}

console.log('内存状态:', getMemoryStatus());
```

#### 内存监控工具

```javascript
const os = require('os');

class MemoryMonitor {
  constructor(threshold = 80) {
    this.threshold = threshold;
    this.monitoring = false;
    this.intervalId = null;
  }

  startMonitoring(interval = 5000) {
    if (this.monitoring) {
      console.log('监控已在运行中');
      return;
    }

    this.monitoring = true;
    console.log(`开始内存监控 (阈值: ${this.threshold}%)`);

    this.intervalId = setInterval(() => {
      const status = this.getMemoryStatus();
      this.report(status);

      if (parseFloat(status.percent) > this.threshold) {
        this.alert(status);
      }
    }, interval);
  }

  stopMonitoring() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
      this.intervalId = null;
    }
    this.monitoring = false;
    console.log('内存监控已停止');
  }

  getMemoryStatus() {
    const total = os.totalmem();
    const free = os.freemem();
    const used = total - free;
    const percent = (used / total) * 100;

    return {
      total: total,
      used: used,
      free: free,
      percent: percent,
      formatted: {
        total: this.formatBytes(total),
        used: this.formatBytes(used),
        free: this.formatBytes(free),
        percent: percent.toFixed(2) + '%'
      }
    };
  }

  formatBytes(bytes) {
    if (bytes === 0) return '0 B';
    const k = 1024;
    const sizes = ['B', 'KB', 'MB', 'GB', 'TB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  }

  report(status) {
    console.log(`[${new Date().toLocaleTimeString()}] 内存状态: ${status.formatted.used} / ${status.formatted.total} (${status.formatted.percent})`);
  }

  alert(status) {
    console.warn(`⚠️  内存警告: 使用率 ${status.formatted.percent} 超过阈值 ${this.threshold}%`);
    // 这里可以添加告警逻辑，如发送通知、记录日志等
  }
}

// 使用示例
const monitor = new MemoryMonitor(80);
monitor.startMonitoring(3000);

// 30 秒后停止监控
setTimeout(() => {
  monitor.stopMonitoring();
}, 30000);
```

### 3. CPU 信息

#### CPU 详细信息

```javascript
const os = require('os');

// 获取 CPU 核心信息
const cpus = os.cpus();

console.log(`CPU 核心数: ${cpus.length}`);
console.log(`CPU 型号: ${cpus[0].model}`);
console.log(`CPU 速度: ${cpus[0].speed} MHz`);

// 详细显示每个核心的信息
cpus.forEach((cpu, index) => {
  console.log(`\n核心 ${index + 1}:`);
  console.log(`  型号: ${cpu.model}`);
  console.log(`  速度: ${cpu.speed} MHz`);

  // CPU 时间统计
  console.log(`  CPU 时间统计:`);
  console.log(`    user: ${cpu.times.user} ms - 用户态时间`);
  console.log(`    nice: ${cpu.times.nice} ms - 用户态低优先级时间`);
  console.log(`    sys: ${cpu.times.sys} ms - 内核态时间`);
  console.log(`    idle: ${cpu.times.idle} ms - 空闲时间`);
  console.log(`    irq: ${cpu.times.irq} ms - 中断处理时间`);
});
```

#### CPU 使用率计算

```javascript
const os = require('os');

class CPUMonitor {
  constructor() {
    this.previousTimes = this.getCPUTimes();
    this.previousTimestamp = Date.now();
  }

  getCPUTimes() {
    const cpus = os.cpus();
    return cpus.map(cpu => ({ ...cpu.times }));
  }

  getCPUUsage() {
    const currentTimes = this.getCPUTimes();
    const currentTimestamp = Date.now();
    const timeDiff = currentTimestamp - this.previousTimestamp;

    let totalUsage = 0;

    this.previousTimes.forEach((prevTimes, index) => {
      const currTimes = currentTimes[index];

      // 计算各个时间的增量
      const prevTotal = Object.values(prevTimes).reduce((sum, time) => sum + time, 0);
      const currTotal = Object.values(currTimes).reduce((sum, time) => sum + time, 0);
      const totalDiff = currTotal - prevTotal;

      if (totalDiff > 0) {
        const idleDiff = currTimes.idle - prevTimes.idle;
        const usage = 1 - (idleDiff / totalDiff);
        totalUsage += usage;
      }
    });

    this.previousTimes = currentTimes;
    this.previousTimestamp = currentTimestamp;

    // 返回平均使用率
    return {
      overall: (totalUsage / os.cpus().length * 100).toFixed(2),
      cores: os.cpus().length,
      perCore: currentTimes.map((currTimes, index) => {
        const prevTimes = this.previousTimes[index];
        const prevTotal = Object.values(prevTimes).reduce((sum, time) => sum + time, 0);
        const currTotal = Object.values(currTimes).reduce((sum, time) => sum + time, 0);
        const totalDiff = currTotal - prevTotal;

        if (totalDiff > 0) {
          const idleDiff = currTimes.idle - prevTimes.idle;
          const usage = 1 - (idleDiff / totalDiff);
          return (usage * 100).toFixed(2);
        }
        return '0.00';
      })
    };
  }

  startMonitoring(interval = 1000, callback) {
    console.log('开始 CPU 监控...');
    this.intervalId = setInterval(() => {
      const usage = this.getCPUUsage();
      if (callback) {
        callback(usage);
      } else {
        console.log(`CPU 使用率: ${usage.overall}%`);
      }
    }, interval);
  }

  stopMonitoring() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
      console.log('CPU 监控已停止');
    }
  }
}

// 使用示例
const cpuMonitor = new CPUMonitor();

cpuMonitor.startMonitoring(2000, (usage) => {
  console.log(`[${new Date().toLocaleTimeString()}] CPU 总使用率: ${usage.overall}%`);
  console.log(`核心数: ${usage.cores}`);
  console.log(`各核心使用率: [${usage.perCore.join(', ')}%]`);
});

// 10 秒后停止监控
setTimeout(() => {
  cpuMonitor.stopMonitoring();
}, 10000);
```

#### CPU 平均负载

```javascript
const os = require('os');

// 获取平均负载（1、5、15 分钟）
const loadavg = os.loadavg();

console.log('系统负载:');
console.log(`  1 分钟平均负载: ${loadavg[0]}`);
console.log(`  5 分钟平均负载: ${loadavg[1]}`);
console.log(`  15 分钟平均负载: ${loadavg[2]}`);

// 计算负载百分比（基于 CPU 核心数）
const cpuCount = os.cpus().length;

function getLoadInfo() {
  const loadavg = os.loadavg();
  const cpuCount = os.cpus().length;

  return {
    load1: loadavg[0],
    load5: loadavg[1],
    load15: loadavg[2],
    cpuCount: cpuCount,
    load1Percent: ((loadavg[0] / cpuCount) * 100).toFixed(2),
    load5Percent: ((loadavg[1] / cpuCount) * 100).toFixed(2),
    load15Percent: ((loadavg[2] / cpuCount) * 100).toFixed(2),
    status: getLoadStatus(loadavg[0] / cpuCount)
  };
}

function getLoadStatus(loadRatio) {
  if (loadRatio < 0.7) return '正常';
  if (loadRatio < 1.0) return '偏高';
  if (loadRatio < 2.0) return '高';
  return '严重';
}

const loadInfo = getLoadInfo();
console.log('负载详细信息:', JSON.stringify(loadInfo, null, 2));

// 负载监控
class LoadMonitor {
  constructor(threshold = 1.0) {
    this.threshold = threshold;
  }

  startMonitoring(interval = 5000) {
    console.log(`开始负载监控 (阈值: ${this.threshold})`);

    return setInterval(() => {
      const info = getLoadInfo();
      const currentLoad = parseFloat(info.load1Percent) / 100;

      console.log(`[${new Date().toLocaleTimeString()}] 负载: ${info.load1} (${info.load1Percent}%) - ${info.status}`);

      if (currentLoad > this.threshold) {
        console.warn(`⚠️  负载警告: 当前负载 ${info.load1} 超过阈值 ${this.threshold}`);
      }
    }, interval);
  }
}

const loadMonitor = new LoadMonitor(1.0);
const loadInterval = loadMonitor.startMonitoring(3000);

// 15 秒后停止监控
setTimeout(() => {
  clearInterval(loadInterval);
  console.log('负载监控已停止');
}, 15000);
```

### 4. 网络接口

#### 网络接口详细信息

```javascript
const os = require('os');

const networkInterfaces = os.networkInterfaces();

console.log('网络接口信息:');
console.log(JSON.stringify(networkInterfaces, null, 2));

// 格式化显示网络接口
for (const [interfaceName, addresses] of Object.entries(networkInterfaces)) {
  console.log(`\n接口: ${interfaceName}`);

  addresses.forEach((address, index) => {
    console.log(`  地址 ${index + 1}:`);
    console.log(`    IP 地址: ${address.address}`);
    console.log(`    地址族: ${address.family}`);  // 'IPv4' 或 'IPv6'
    console.log(`    内部接口: ${address.internal}`); // true/false
    console.log(`    MAC 地址: ${address.mac}`);
    console.log(`    子网掩码: ${address.netmask}`);
    console.log(`    CIDR 表示: ${address.cidr}`);
    console.log(`    前缀长度: ${address.prefixlen}`);
  });
}
```

#### 获取本地 IP 地址工具

```javascript
const os = require('os');

class NetworkHelper {
  // 获取本地 IPv4 地址（非内部）
  static getLocalIPv4() {
    const interfaces = os.networkInterfaces();
    for (const interfaceName of Object.keys(interfaces)) {
      for (const address of interfaces[interfaceName]) {
        // 跳过内部网络接口和 IPv6
        if (address.family === 'IPv4' && !address.internal) {
          return {
            interface: interfaceName,
            address: address.address,
            mac: address.mac,
            netmask: address.netmask
          };
        }
      }
    }
    return null;
  }

  // 获取所有外部 IPv4 地址
  static getAllExternalIPs() {
    const interfaces = os.networkInterfaces();
    const externalIPs = [];

    for (const [interfaceName, addresses] of Object.entries(interfaces)) {
      addresses.forEach(address => {
        if (!address.internal && address.family === 'IPv4') {
          externalIPs.push({
            interface: interfaceName,
            address: address.address,
            mac: address.mac,
            netmask: address.netmask
          });
        }
      });
    }

    return externalIPs;
  }

  // 获取 IPv6 地址
  static getIPv6Addresses() {
    const interfaces = os.networkInterfaces();
    const ipv6Addresses = [];

    for (const [interfaceName, addresses] of Object.entries(interfaces)) {
      addresses.forEach(address => {
        if (!address.internal && address.family === 'IPv6') {
          ipv6Addresses.push({
            interface: interfaceName,
            address: address.address,
            scopeid: address.scopeid
          });
        }
      });
    }

    return ipv6Addresses;
  }

  // 检查网络连接
  static hasNetworkConnection() {
    const interfaces = os.networkInterfaces();
    return Object.values(interfaces).some(addrs =>
      addrs.some(addr => !addr.internal && addr.family === 'IPv4')
    );
  }
}

// 使用示例
console.log('本地 IPv4 地址:', NetworkHelper.getLocalIPv4());
console.log('所有外部 IP:', NetworkHelper.getAllExternalIPs());
console.log('IPv6 地址:', NetworkHelper.getIPv6Addresses());
console.log('网络连接状态:', NetworkHelper.hasNetworkConnection() ? '已连接' : '未连接');
```

### 5. 路径信息

#### 系统路径获取

```javascript
const os = require('os');
const path = require('path');

// 用户主目录
console.log('用户主目录:', os.homedir());
// macOS/Linux: /Users/username 或 /home/username
// Windows: C:\Users\username

// 临时文件目录
console.log('临时文件目录:', os.tmpdir());
// macOS: /var/folders/...
// Linux: /tmp
// Windows: C:\Users\username\AppData\Local\Temp

// 当前用户信息
const userInfo = os.userInfo();
console.log('用户信息:', userInfo);
// {
//   uid: 501,
//   gid: 20,
//   username: 'john',
//   homedir: '/Users/john',
//   shell: '/bin/zsh'
// }

// 使用用户信息
console.log('用户名:', userInfo.username);
console.log('用户 ID:', userInfo.uid);
console.log('组 ID:', userInfo.gid);
console.log('主目录:', userInfo.homedir);
console.log('默认 Shell:', userInfo.shell);
```

#### 跨平台路径管理

```javascript
const os = require('os');
const path = require('path');

class PathManager {
  constructor(appName) {
    this.appName = appName;
    this.platform = os.platform();
  }

  // 获取应用数据目录
  getAppDataDir() {
    const homeDir = os.homedir();

    switch (this.platform) {
      case 'win32':
        return path.join(homeDir, 'AppData', 'Roaming', this.appName);
      case 'darwin':
        return path.join(homeDir, 'Library', 'Application Support', this.appName);
      default: // Linux and others
        return path.join(homeDir, '.config', this.appName);
    }
  }

  // 获取缓存目录
  getCacheDir() {
    const homeDir = os.homedir();

    switch (this.platform) {
      case 'win32':
        return path.join(homeDir, 'AppData', 'Local', this.appName, 'Cache');
      case 'darwin':
        return path.join(homeDir, 'Library', 'Caches', this.appName);
      default:
        return path.join(homeDir, '.cache', this.appName);
    }
  }

  // 获取日志目录
  getLogDir() {
    const homeDir = os.homedir();

    switch (this.platform) {
      case 'win32':
        return path.join(homeDir, 'AppData', 'Local', this.appName, 'Logs');
      case 'darwin':
        return path.join(homeDir, 'Library', 'Logs', this.appName);
      default:
        return path.join(homeDir, '.local', 'state', this.appName, 'logs');
    }
  }

  // 获取临时文件目录
  getTempDir() {
    return path.join(os.tmpdir(), this.appName);
  }

  // 获取配置文件路径
  getConfigFile(filename = 'config.json') {
    return path.join(this.getAppDataDir(), filename);
  }

  // 初始化所有目录
  initializeDirectories() {
    const fs = require('fs');
    const directories = [
      this.getAppDataDir(),
      this.getCacheDir(),
      this.getLogDir(),
      this.getTempDir()
    ];

    directories.forEach(dir => {
      if (!fs.existsSync(dir)) {
        fs.mkdirSync(dir, { recursive: true });
        console.log(`创建目录: ${dir}`);
      }
    });

    return directories;
  }
}

// 使用示例
const pathManager = new PathManager('MyApp');

console.log('应用数据目录:', pathManager.getAppDataDir());
console.log('缓存目录:', pathManager.getCacheDir());
console.log('日志目录:', pathManager.getLogDir());
console.log('临时目录:', pathManager.getTempDir());
console.log('配置文件:', pathManager.getConfigFile());

// 初始化目录
const directories = pathManager.initializeDirectories();
console.log('初始化的目录:', directories);
```

### 6. 其他实用功能

#### 系统运行时间

```javascript
const os = require('os');

// 系统运行时间（秒）
const uptime = os.uptime();

console.log('系统运行时间 (秒):', uptime);
console.log('系统运行时间 (分钟):', (uptime / 60).toFixed(2));
console.log('系统运行时间 (小时):', (uptime / 3600).toFixed(2));
console.log('系统运行时间 (天):', (uptime / 86400).toFixed(2));

// 格式化运行时间
function formatUptime(seconds) {
  const days = Math.floor(seconds / 86400);
  const hours = Math.floor((seconds % 86400) / 3600);
  const minutes = Math.floor((seconds % 3600) / 60);
  const secs = Math.floor(seconds % 60);

  const parts = [];
  if (days > 0) parts.push(`${days}天`);
  if (hours > 0) parts.push(`${hours}小时`);
  if (minutes > 0) parts.push(`${minutes}分钟`);
  if (secs > 0 || parts.length === 0) parts.push(`${secs}秒`);

  return parts.join(' ');
}

console.log('格式化运行时间:', formatUptime(uptime));

// 获取启动时间
function getSystemStartTime() {
  const uptimeSeconds = os.uptime();
  const startTime = new Date(Date.now() - uptimeSeconds * 1000);
  return startTime;
}

console.log('系统启动时间:', getSystemStartTime().toLocaleString());
```

#### 进程优先级管理

```javascript
const os = require('os');

// 获取当前进程优先级
const currentPriority = os.getPriority();
console.log('当前进程优先级:', currentPriority);

// 进程优先级常量
console.log('低优先级:', os.constants.priority.PRIORITY_LOW);     // 19
console.log('普通优先级:', os.constants.priority.PRIORITY_NORMAL); // 0
console.log('高优先级:', os.constants.priority.PRIORITY_HIGH);    // -10

// 设置进程优先级 (需要特定权限)
function setProcessPriority(priority) {
  try {
    os.setPriority(priority);
    console.log(`进程优先级已设置为: ${os.getPriority()}`);
    return true;
  } catch (error) {
    console.error('设置优先级失败:', error.message);
    console.log('可能需要管理员/root 权限');
    return false;
  }
}

// 使用示例
// setProcessPriority(os.constants.priority.PRIORITY_LOW);

// 安全的优先级设置
class ProcessPriorityManager {
  static setLowPriority() {
    try {
      os.setPriority(os.constants.priority.PRIORITY_LOW);
      console.log('进程已设置为低优先级');
      return true;
    } catch (error) {
      console.error('无法设置低优先级:', error.message);
      return false;
    }
  }

  static setNormalPriority() {
    try {
      os.setPriority(os.constants.priority.PRIORITY_NORMAL);
      console.log('进程已设置为普通优先级');
      return true;
    } catch (error) {
      console.error('无法设置普通优先级:', error.message);
      return false;
    }
  }

  static setHighPriority() {
    try {
      os.setPriority(os.constants.priority.PRIORITY_HIGH);
      console.log('进程已设置为高优先级');
      return true;
    } catch (error) {
      console.error('无法设置高优先级:', error.message);
      return false;
    }
  }

  static getCurrentPriority() {
    return os.getPriority();
  }

  static getPriorityName(priority) {
    if (priority === os.constants.priority.PRIORITY_LOW) return '低';
    if (priority === os.constants.priority.PRIORITY_NORMAL) return '普通';
    if (priority === os.constants.priority.PRIORITY_HIGH) return '高';
    return '未知';
  }
}

// 使用示例
console.log('当前优先级:', ProcessPriorityManager.getPriorityName(ProcessPriorityManager.getCurrentPriority()));

// 尝试设置优先级 (可能会失败，取决于权限)
// ProcessPriorityManager.setLowPriority();
```

## 实际应用

### 1. 完整的系统监控工具

```javascript
const os = require('os');

class SystemMonitor {
  constructor(options = {}) {
    this.options = {
      updateInterval: options.updateInterval || 5000,
      memoryThreshold: options.memoryThreshold || 80,
      cpuThreshold: options.cpuThreshold || 80,
      loadThreshold: options.loadThreshold || 1.0,
      ...options
    };

    this.monitoring = false;
    this.intervalId = null;
    this.previousCpuTimes = null;
    this.previousTimestamp = null;

    this.stats = {
      startTime: Date.now(),
      samples: 0,
      alerts: 0
    };
  }

  start() {
    if (this.monitoring) {
      console.log('监控已在运行中');
      return;
    }

    this.monitoring = true;
    this.previousCpuTimes = this.getCPUTimes();
    this.previousTimestamp = Date.now();

    console.log('系统监控已启动');
    console.log(`更新间隔: ${this.options.updateInterval}ms`);
    console.log(`内存阈值: ${this.options.memoryThreshold}%`);
    console.log(`CPU 阈值: ${this.options.cpuThreshold}%`);
    console.log(`负载阈值: ${this.options.loadThreshold}`);

    this.intervalId = setInterval(() => {
      this.collectAndReport();
    }, this.options.updateInterval);
  }

  stop() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
      this.intervalId = null;
    }
    this.monitoring = false;
    console.log('系统监控已停止');
    this.printSummary();
  }

  getCPUTimes() {
    const cpus = os.cpus();
    return cpus.map(cpu => ({ ...cpu.times }));
  }

  getCPUUsage() {
    const currentTimes = this.getCPUTimes();
    const currentTimestamp = Date.now();

    let totalUsage = 0;

    this.previousCpuTimes.forEach((prevTimes, index) => {
      const currTimes = currentTimes[index];
      const prevTotal = Object.values(prevTimes).reduce((sum, time) => sum + time, 0);
      const currTotal = Object.values(currTimes).reduce((sum, time) => sum + time, 0);
      const totalDiff = currTotal - prevTotal;

      if (totalDiff > 0) {
        const idleDiff = currTimes.idle - prevTimes.idle;
        const usage = 1 - (idleDiff / totalDiff);
        totalUsage += usage;
      }
    });

    this.previousCpuTimes = currentTimes;
    this.previousTimestamp = currentTimestamp;

    return (totalUsage / os.cpus().length * 100).toFixed(2);
  }

  getMemoryInfo() {
    const total = os.totalmem();
    const free = os.freemem();
    const used = total - free;

    return {
      total: this.formatBytes(total),
      used: this.formatBytes(used),
      free: this.formatBytes(free),
      percent: ((used / total) * 100).toFixed(2)
    };
  }

  getLoadInfo() {
    const loadavg = os.loadavg();
    const cpuCount = os.cpus().length;

    return {
      load1: loadavg[0].toFixed(2),
      load5: loadavg[1].toFixed(2),
      load15: loadavg[2].toFixed(2),
      cpuCount: cpuCount,
      load1Percent: ((loadavg[0] / cpuCount) * 100).toFixed(2)
    };
  }

  getNetworkInfo() {
    const interfaces = os.networkInterfaces();
    const externalIPs = [];

    for (const [interfaceName, addresses] of Object.entries(interfaces)) {
      addresses.forEach(address => {
        if (!address.internal && address.family === 'IPv4') {
          externalIPs.push({
            interface: interfaceName,
            address: address.address,
            mac: address.mac
          });
        }
      });
    }

    return externalIPs;
  }

  getSystemInfo() {
    return {
      platform: os.platform(),
      arch: os.arch(),
      release: os.release(),
      hostname: os.hostname(),
      uptime: formatUptime(os.uptime()),
      startTime: getSystemStartTime().toLocaleString(),
      cpus: {
        count: os.cpus().length,
        model: os.cpus()[0]?.model || 'Unknown'
      }
    };
  }

  formatBytes(bytes) {
    if (bytes === 0) return '0 B';
    const k = 1024;
    const sizes = ['B', 'KB', 'MB', 'GB', 'TB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  }

  collectAndReport() {
    const report = {
      timestamp: new Date().toISOString(),
      system: this.getSystemInfo(),
      cpu: this.getCPUUsage(),
      memory: this.getMemoryInfo(),
      load: this.getLoadInfo(),
      network: this.getNetworkInfo()
    };

    this.stats.samples++;

    this.printReport(report);
    this.checkThresholds(report);
  }

  printReport(report) {
    console.log('\n' + '='.repeat(50));
    console.log(`系统监控报告 - ${new Date().toLocaleString()}`);
    console.log('='.repeat(50));
    console.log('系统信息:');
    console.log(`  平台: ${report.system.platform} ${report.system.arch}`);
    console.log(`  版本: ${report.system.release}`);
    console.log(`  主机: ${report.system.hostname}`);
    console.log(`  运行时间: ${report.system.uptime}`);
    console.log(`  启动时间: ${report.system.startTime}`);
    console.log(`  CPU: ${report.system.cpus.model} (${report.system.cpus.count} 核心)`);
    console.log('\n资源使用:');
    console.log(`  CPU 使用率: ${report.cpu}%`);
    console.log(`  内存使用: ${report.memory.used} / ${report.memory.total} (${report.memory.percent}%)`);
    console.log(`  系统负载: ${report.load.load1} (1分钟) - ${report.load.load1Percent}%`);

    if (report.network.length > 0) {
      console.log('\n网络接口:');
      report.network.forEach(net => {
        console.log(`  ${net.interface}: ${net.address} (${net.mac})`);
      });
    }
    console.log('='.repeat(50));
  }

  checkThresholds(report) {
    let alerts = [];

    // 内存检查
    if (parseFloat(report.memory.percent) > this.options.memoryThreshold) {
      alerts.push(`内存使用率 ${report.memory.percent}% 超过阈值 ${this.options.memoryThreshold}%`);
    }

    // CPU 检查
    if (parseFloat(report.cpu) > this.options.cpuThreshold) {
      alerts.push(`CPU 使用率 ${report.cpu}% 超过阈值 ${this.options.cpuThreshold}%`);
    }

    // 负载检查
    const loadRatio = parseFloat(report.load.load1Percent) / 100;
    if (loadRatio > this.options.loadThreshold) {
      alerts.push(`系统负载 ${report.load.load1} 超过阈值 ${this.options.loadThreshold}`);
    }

    // 输出告警
    if (alerts.length > 0) {
      this.stats.alerts++;
      console.warn('\n⚠️  系统告警:');
      alerts.forEach(alert => {
        console.warn(`  - ${alert}`);
      });
    }
  }

  printSummary() {
    const duration = Date.now() - this.stats.startTime;
    console.log('\n监控统计:');
    console.log(`  监控时长: ${formatUptime(duration / 1000)}`);
    console.log(`  采样次数: ${this.stats.samples}`);
    console.log(`  告警次数: ${this.stats.alerts}`);
  }
}

// 辅助函数
function formatUptime(seconds) {
  const days = Math.floor(seconds / 86400);
  const hours = Math.floor((seconds % 86400) / 3600);
  const minutes = Math.floor((seconds % 3600) / 60);
  const secs = Math.floor(seconds % 60);

  const parts = [];
  if (days > 0) parts.push(`${days}天`);
  if (hours > 0) parts.push(`${hours}小时`);
  if (minutes > 0) parts.push(`${minutes}分钟`);
  if (secs > 0 || parts.length === 0) parts.push(`${secs}秒`);

  return parts.join(' ');
}

function getSystemStartTime() {
  const uptimeSeconds = os.uptime();
  return new Date(Date.now() - uptimeSeconds * 1000);
}

// 使用示例
const monitor = new SystemMonitor({
  updateInterval: 3000,
  memoryThreshold: 75,
  cpuThreshold: 70,
  loadThreshold: 0.8
});

monitor.start();

// 30 秒后停止监控
setTimeout(() => {
  monitor.stop();
}, 30000);
```

### 2. 跨平台应用配置管理器

```javascript
const os = require('os');
const path = require('path');
const fs = require('fs');

class AppConfigManager {
  constructor(appName) {
    this.appName = appName;
    this.platform = os.platform();
    this.userInfo = os.userInfo();
    this.directories = this.initializeDirectories();
  }

  initializeDirectories() {
    const directories = {
      appData: this.getAppDataDir(),
      cache: this.getCacheDir(),
      logs: this.getLogDir(),
      temp: this.getTempDir(),
      userData: this.getUserDataDir()
    };

    // 创建目录
    Object.values(directories).forEach(dir => {
      if (!fs.existsSync(dir)) {
        try {
          fs.mkdirSync(dir, { recursive: true });
          console.log(`创建目录: ${dir}`);
        } catch (error) {
          console.error(`创建目录失败 ${dir}:`, error.message);
        }
      }
    });

    return directories;
  }

  getAppDataDir() {
    const homeDir = os.homedir();
    switch (this.platform) {
      case 'win32':
        return path.join(homeDir, 'AppData', 'Roaming', this.appName);
      case 'darwin':
        return path.join(homeDir, 'Library', 'Application Support', this.appName);
      default:
        return path.join(homeDir, '.config', this.appName);
    }
  }

  getCacheDir() {
    const homeDir = os.homedir();
    switch (this.platform) {
      case 'win32':
        return path.join(homeDir, 'AppData', 'Local', this.appName, 'Cache');
      case 'darwin':
        return path.join(homeDir, 'Library', 'Caches', this.appName);
      default:
        return path.join(homeDir, '.cache', this.appName);
    }
  }

  getLogDir() {
    const homeDir = os.homedir();
    switch (this.platform) {
      case 'win32':
        return path.join(homeDir, 'AppData', 'Local', this.appName, 'Logs');
      case 'darwin':
        return path.join(homeDir, 'Library', 'Logs', this.appName);
      default:
        return path.join(homeDir, '.local', 'state', this.appName, 'logs');
    }
  }

  getTempDir() {
    return path.join(os.tmpdir(), this.appName);
  }

  getUserDataDir() {
    return path.join(os.homedir(), this.appName);
  }

  saveConfig(config, filename = 'config.json') {
    const configPath = path.join(this.directories.appData, filename);
    try {
      fs.writeFileSync(configPath, JSON.stringify(config, null, 2), 'utf8');
      console.log(`配置已保存: ${configPath}`);
      return true;
    } catch (error) {
      console.error('保存配置失败:', error.message);
      return false;
    }
  }

  loadConfig(filename = 'config.json') {
    const configPath = path.join(this.directories.appData, filename);
    try {
      if (fs.existsSync(configPath)) {
        const data = fs.readFileSync(configPath, 'utf8');
        return JSON.parse(data);
      }
      return this.getDefaultConfig();
    } catch (error) {
      console.error('加载配置失败:', error.message);
      return this.getDefaultConfig();
    }
  }

  getDefaultConfig() {
    return {
      version: '1.0.0',
      appName: this.appName,
      platform: this.platform,
      username: this.userInfo.username,
      createdAt: new Date().toISOString(),
      settings: {
        theme: 'light',
        language: 'zh-CN',
        autoUpdate: true
      }
    };
  }

  getSystemInfo() {
    return {
      platform: os.platform(),
      arch: os.arch(),
      release: os.release(),
      hostname: os.hostname(),
      uptime: os.uptime(),
      homedir: os.homedir(),
      tmpdir: os.tmpdir(),
      userInfo: this.userInfo,
      directories: this.directories,
      cpus: {
        count: os.cpus().length,
        model: os.cpus()[0]?.model || 'Unknown'
      },
      memory: {
        total: os.totalmem(),
        free: os.freemem()
      }
    };
  }
}

// 使用示例
const configManager = new AppConfigManager('MyApp');

// 显示系统信息
console.log('系统信息:');
console.log(JSON.stringify(configManager.getSystemInfo(), null, 2));

// 保存配置
const appConfig = {
  server: {
    host: 'localhost',
    port: 3000
  },
  features: {
    debug: true,
    logging: true
  }
};

configManager.saveConfig(appConfig);

// 加载配置
const loadedConfig = configManager.loadConfig();
console.log('加载的配置:', loadedConfig);
```

### 3. 智能服务器配置优化器

```javascript
const os = require('os');

class ServerConfigOptimizer {
  constructor() {
    this.systemInfo = this.collectSystemInfo();
  }

  collectSystemInfo() {
    return {
      platform: os.platform(),
      arch: os.arch(),
      cpuCount: os.cpus().length,
      cpuModel: os.cpus()[0]?.model || 'Unknown',
      totalMemory: os.totalmem(),
      freeMemory: os.freemem(),
      uptime: os.uptime(),
      hostname: os.hostname()
    };
  }

  optimizeWorkerCount(options = {}) {
    const {
      memoryPerWorker = 512 * 1024 * 1024, // 默认每个 worker 512MB
      maxWorkers = null,
      minWorkers = 1
    } = options;

    const cpuCount = this.systemInfo.cpuCount;
    const totalMemory = this.systemInfo.totalMemory;
    const freeMemory = this.systemInfo.freeMemory;

    // 基于 CPU 的最大 worker 数量
    const maxByCPU = cpuCount;

    // 基于内存的最大 worker 数量
    const maxByMemory = Math.floor(totalMemory / memoryPerWorker);

    // 基于可用内存的最大 worker 数量
    const maxByFreeMemory = Math.floor(freeMemory / memoryPerWorker);

    // 取最小值作为基础
    let optimalWorkerCount = Math.min(maxByCPU, maxByMemory, maxByFreeMemory);

    // 应用限制
    if (maxWorkers !== null) {
      optimalWorkerCount = Math.min(optimalWorkerCount, maxWorkers);
    }

    optimalWorkerCount = Math.max(optimalWorkerCount, minWorkers);

    return {
      recommended: optimalWorkerCount,
      byCPU: maxByCPU,
      byMemory: maxByMemory,
      byFreeMemory: maxByFreeMemory,
      reasoning: {
        cpu: `CPU 核心数: ${cpuCount}`,
        memory: `总内存可支持: ${maxByMemory} 个 worker`,
        freeMemory: `可用内存支持: ${maxByFreeMemory} 个 worker`
      }
    };
  }

  optimizeClusterSettings(options = {}) {
    const {
      workerCount = this.optimizeWorkerCount().recommended,
      maxConnections = null,
      maxMemoryUsage = 0.7 // 默认最大内存使用率 70%
    } = options;

    const totalMemory = this.systemInfo.totalMemory;
    const maxMemory = totalMemory * maxMemoryUsage;

    // 计算最大连接数 (每 MB 内存一个连接)
    const maxConnectionsByMemory = Math.floor(maxMemory / (1024 * 1024));
    const calculatedMaxConnections = maxConnections || maxConnectionsByMemory;

    return {
      workers: workerCount,
      maxConnections: calculatedMaxConnections,
      maxMemoryUsage: maxMemoryUsage,
      maxMemoryBytes: maxMemory,
      memoryPerWorker: Math.floor(maxMemory / workerCount),
      recommendedConfig: {
        workerTimeout: 30000,
        workerRetries: 3,
        loadBalancing: 'round-robin',
        gracefulShutdown: true
      }
    };
  }

  optimizeServerPort(portRange = { min: 3000, max: 9000 }) {
    const net = require('net');

    return new Promise((resolve, reject) => {
      const server = net.createServer();

      server.on('error', (err) => {
        if (err.code === 'EADDRINUSE') {
          // 端口已被占用，尝试下一个
          server.close();
          this.optimizeServerPort({
            min: portRange.min + 1,
            max: portRange.max
          }).then(resolve).catch(reject);
        } else {
          reject(err);
        }
      });

      server.listen({ port: portRange.min, exclusive: true }, () => {
        const port = server.address().port;
        server.close(() => {
          resolve(port);
        });
      });
    });
  }

  getPerformanceRecommendations() {
    const recommendations = [];
    const cpuCount = this.systemInfo.cpuCount;
    const memoryUsage = (this.systemInfo.totalMemory - this.systemInfo.freeMemory) / this.systemInfo.totalMemory;

    // CPU 推荐
    if (cpuCount >= 8) {
      recommendations.push({
        type: 'cpu',
        level: 'optimal',
        message: 'CPU 核心数充足，可以考虑启用更多 worker 进程'
      });
    } else if (cpuCount >= 4) {
      recommendations.push({
        type: 'cpu',
        level: 'good',
        message: 'CPU 核心数适中，建议启用 cluster 模式'
      });
    } else {
      recommendations.push({
        type: 'cpu',
        level: 'warning',
        message: 'CPU 核心数较少，注意监控 CPU 使用率'
      });
    }

    // 内存推荐
    if (memoryUsage > 0.8) {
      recommendations.push({
        type: 'memory',
        level: 'critical',
        message: '内存使用率过高，建议增加内存或优化应用'
      });
    } else if (memoryUsage > 0.6) {
      recommendations.push({
        type: 'memory',
        level: 'warning',
        message: '内存使用率偏高，建议监控内存使用情况'
      });
    } else {
      recommendations.push({
        type: 'memory',
        level: 'optimal',
        message: '内存使用率正常'
      });
    }

    return recommendations;
  }

  generateOptimizedConfig(options = {}) {
    const workerOptimization = this.optimizeWorkerCount(options.workerOptions);
    const clusterOptimization = this.optimizeClusterSettings({
      ...options.clusterOptions,
      workerCount: workerOptimization.recommended
    });

    return {
      system: this.systemInfo,
      worker: workerOptimization,
      cluster: clusterOptimization,
      recommendations: this.getPerformanceRecommendations(),
      generatedAt: new Date().toISOString()
    };
  }
}

// 使用示例
const optimizer = new ServerConfigOptimizer();

console.log('系统信息:', optimizer.systemInfo);

// 优化 worker 数量
const workerOptimization = optimizer.optimizeWorkerCount({
  memoryPerWorker: 256 * 1024 * 1024, // 每个 worker 256MB
  maxWorkers: 8
});
console.log('Worker 优化:', workerOptimization);

// 优化集群设置
const clusterOptimization = optimizer.optimizeClusterSettings({
  maxMemoryUsage: 0.75
});
console.log('集群优化:', clusterOptimization);

// 性能推荐
const recommendations = optimizer.getPerformanceRecommendations();
console.log('性能推荐:');
recommendations.forEach(rec => {
  console.log(`  [${rec.level.toUpperCase()}] ${rec.type}: ${rec.message}`);
});

// 生成完整配置
const optimizedConfig = optimizer.generateOptimizedConfig({
  workerOptions: {
    memoryPerWorker: 512 * 1024 * 1024
  },
  clusterOptions: {
    maxMemoryUsage: 0.8
  }
});

console.log('优化后的配置:', JSON.stringify(optimizedConfig, null, 2));
```

### 4. 健康检查和告警系统

```javascript
const os = require('os');
const EventEmitter = require('events');

class HealthCheckSystem extends EventEmitter {
  constructor(thresholds = {}) {
    super();
    this.thresholds = {
      cpu: thresholds.cpu || 80,
      memory: thresholds.memory || 85,
      load: thresholds.load || 1.0,
      disk: thresholds.disk || 90,
      ...thresholds
    };

    this.checkInterval = null;
    this.history = {
      cpu: [],
      memory: [],
      load: [],
      maxHistoryLength: 60 // 保存最近60次检查结果
    };

    this.previousCpuTimes = null;
    this.previousTimestamp = null;
  }

  startMonitoring(interval = 5000) {
    if (this.checkInterval) {
      console.log('健康检查已在运行中');
      return;
    }

    this.previousCpuTimes = this.getCPUTimes();
    this.previousTimestamp = Date.now();

    console.log('健康检查系统已启动');
    console.log('阈值设置:', this.thresholds);

    this.checkInterval = setInterval(() => {
      this.performHealthCheck();
    }, interval);

    // 立即执行一次检查
    this.performHealthCheck();
  }

  stopMonitoring() {
    if (this.checkInterval) {
      clearInterval(this.checkInterval);
      this.checkInterval = null;
    }
    console.log('健康检查系统已停止');
  }

  getCPUTimes() {
    const cpus = os.cpus();
    return cpus.map(cpu => ({ ...cpu.times }));
  }

  getCPUUsage() {
    const currentTimes = this.getCPUTimes();
    const currentTimestamp = Date.now();

    let totalUsage = 0;

    this.previousCpuTimes.forEach((prevTimes, index) => {
      const currTimes = currentTimes[index];
      const prevTotal = Object.values(prevTimes).reduce((sum, time) => sum + time, 0);
      const currTotal = Object.values(currTimes).reduce((sum, time) => sum + time, 0);
      const totalDiff = currTotal - prevTotal;

      if (totalDiff > 0) {
        const idleDiff = currTimes.idle - prevTimes.idle;
        const usage = 1 - (idleDiff / totalDiff);
        totalUsage += usage;
      }
    });

    this.previousCpuTimes = currentTimes;
    this.previousTimestamp = currentTimestamp;

    return (totalUsage / os.cpus().length * 100).toFixed(2);
  }

  checkCPU() {
    const usage = parseFloat(this.getCPUUsage());
    const status = usage <= this.thresholds.cpu;

    this.updateHistory('cpu', { usage, status, timestamp: Date.now() });

    return {
      status: status ? 'healthy' : 'unhealthy',
      usage: usage.toFixed(2),
      threshold: this.thresholds.cpu,
      message: status ? 'CPU 使用率正常' : `CPU 使用率过高 (${usage.toFixed(2)}%)`
    };
  }

  checkMemory() {
    const total = os.totalmem();
    const free = os.freemem();
    const used = total - free;
    const usagePercent = ((used / total) * 100).toFixed(2);

    const status = parseFloat(usagePercent) <= this.thresholds.memory;

    this.updateHistory('memory', { usage: parseFloat(usagePercent), status, timestamp: Date.now() });

    return {
      status: status ? 'healthy' : 'unhealthy',
      total: this.formatBytes(total),
      used: this.formatBytes(used),
      free: this.formatBytes(free),
      usagePercent: `${usagePercent}%`,
      threshold: `${this.thresholds.memory}%`,
      message: status ? '内存使用正常' : `内存使用率过高 (${usagePercent}%)`
    };
  }

  checkLoad() {
    const loadavg = os.loadavg();
    const cpuCount = os.cpus().length;
    const load1 = loadavg[0];
    const loadRatio = load1 / cpuCount;

    const status = loadRatio <= this.thresholds.load;

    this.updateHistory('load', { load: load1, status, timestamp: Date.now() });

    return {
      status: status ? 'healthy' : 'unhealthy',
      load1: load1.toFixed(2),
      load5: loadavg[1].toFixed(2),
      load15: loadavg[2].toFixed(2),
      cpuCount: cpuCount,
      loadRatio: (loadRatio * 100).toFixed(2),
      threshold: (this.thresholds.load * 100).toFixed(2),
      message: status ? '系统负载正常' : `系统负载过高 (${load1.toFixed(2)})`
    };
  }

  updateHistory(type, data) {
    if (!this.history[type]) {
      this.history[type] = [];
    }
    this.history[type].push(data);

    // 保持历史记录长度
    if (this.history[type].length > this.history.maxHistoryLength) {
      this.history[type].shift();
    }
  }

  performHealthCheck() {
    const results = {
      timestamp: new Date().toISOString(),
      overall: 'healthy',
      checks: {}
    };

    // 执行各项检查
    results.checks.cpu = this.checkCPU();
    results.checks.memory = this.checkMemory();
    results.checks.load = this.checkLoad();

    // 网络检查
    results.checks.network = this.checkNetwork();

    // 磁盘检查 (简化版本)
    results.checks.disk = this.checkDisk();

    // 计算整体健康状态
    const hasUnhealthy = Object.values(results.checks).some(check => check.status === 'unhealthy');
    results.overall = hasUnhealthy ? 'unhealthy' : 'healthy';

    // 发出事件
    this.emit('healthCheck', results);

    // 如果有异常，发出告警事件
    if (results.overall === 'unhealthy') {
      this.emit('alert', results);
    }

    return results;
  }

  checkNetwork() {
    const interfaces = os.networkInterfaces();
    const hasNetwork = Object.values(interfaces).some(addrs =>
      addrs.some(addr => !addr.internal && addr.family === 'IPv4')
    );

    return {
      status: hasNetwork ? 'healthy' : 'unhealthy',
      hasNetwork,
      message: hasNetwork ? '网络连接正常' : '无网络连接'
    };
  }

  checkDisk() {
    // 简化版本，实际应用中需要使用专门的库来获取磁盘信息
    return {
      status: 'healthy',
      message: '磁盘检查需要额外实现'
    };
  }

  formatBytes(bytes) {
    if (bytes === 0) return '0 B';
    const k = 1024;
    const sizes = ['B', 'KB', 'MB', 'GB', 'TB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  }

  getHealthSummary() {
    return {
      thresholds: this.thresholds,
      history: this.history,
      recentChecks: {
        cpu: this.history.cpu.slice(-5),
        memory: this.history.memory.slice(-5),
        load: this.history.load.slice(-5)
      }
    };
  }
}

// 使用示例
const healthCheck = new HealthCheckSystem({
  cpu: 75,
  memory: 80,
  load: 0.8,
  disk: 85
});

// 监听健康检查结果
healthCheck.on('healthCheck', (results) => {
  console.log(`\n[${results.timestamp}] 健康检查结果:`);
  console.log(`整体状态: ${results.overall.toUpperCase()}`);

  Object.entries(results.checks).forEach(([name, check]) => {
    const statusIcon = check.status === 'healthy' ? '✓' : '✗';
    console.log(`  ${statusIcon} ${name}: ${check.message}`);
  });
});

// 监听告警
healthCheck.on('alert', (results) => {
  console.warn('\n⚠️  系统健康告警!');
  console.warn('异常项目:');
  Object.entries(results.checks).forEach(([name, check]) => {
    if (check.status === 'unhealthy') {
      console.warn(`  - ${name}: ${check.message}`);
    }
  });
});

// 开始监控
healthCheck.startMonitoring(5000);

// 30 秒后停止监控
setTimeout(() => {
  healthCheck.stopMonitoring();

  // 获取健康摘要
  const summary = healthCheck.getHealthSummary();
  console.log('\n健康检查摘要:', JSON.stringify(summary, null, 2));
}, 30000);
```

## 注意事项

### 1. 平台兼容性

不同操作系统的行为和返回值可能不同，需要做适当的兼容处理：

```javascript
const os = require('os');

// 跨平台路径分隔符
const path = require('path');
console.log('路径分隔符:', path.sep); // Windows: '\', Unix: '/'

// 跨平台换行符
console.log('换行符:', JSON.stringify(os.EOL)); // Windows: '\r\n', Unix: '\n'

// 跨平台用户目录
function getAppDataPath(appName) {
  const homeDir = os.homedir();
  switch (os.platform()) {
    case 'win32':
      return path.join(homeDir, 'AppData', 'Roaming', appName);
    case 'darwin':
      return path.join(homeDir, 'Library', 'Application Support', appName);
    default:
      return path.join(homeDir, '.config', appName);
  }
}

// 检查特定功能是否可用
function isWindows() {
  return os.platform() === 'win32';
}

function isUnix() {
  return ['darwin', 'linux', 'freebsd', 'openbsd'].includes(os.platform());
}
```

### 2. 性能考虑

频繁调用 `os` 模块方法可能影响性能，应适当缓存结果：

```javascript
const os = require('os');

// 不好的做法：每次函数调用都获取 CPU 信息
function getBadCPUInfo() {
  return os.cpus(); // 重复获取相同数据
}

// 好的做法：缓存不常变化的信息
class SystemInfoCache {
  constructor() {
    this.cache = {};
    this.cacheDuration = 60 * 1000; // 缓存60秒
  }

  getCachedCPUInfo() {
    const now = Date.now();
    if (!this.cache.cpus || (now - this.cache.cpus.timestamp) > this.cacheDuration) {
      this.cache.cpus = {
        data: os.cpus(),
        timestamp: now
      };
    }
    return this.cache.cpus.data;
  }

  getCachedMemoryInfo() {
    const now = Date.now();
    if (!this.cache.memory || (now - this.cache.memory.timestamp) > this.cacheDuration) {
      this.cache.memory = {
        data: {
          total: os.totalmem(),
          free: os.freemem()
        },
        timestamp: now
      };
    }
    return this.cache.memory.data;
  }
}

const cache = new SystemInfoCache();
console.log('缓存的CPU信息:', cache.getCachedCPUInfo());
```

### 3. 权限问题

某些操作需要特定权限，需要做适当的错误处理：

```javascript
const os = require('os');

// 设置进程优先级需要管理员权限
function safeSetPriority(priority) {
  try {
    os.setPriority(priority);
    console.log(`优先级已设置为: ${os.getPriority()}`);
    return true;
  } catch (error) {
    if (error.code === 'EPERM') {
      console.error('权限不足，无法设置进程优先级');
      console.error('请使用管理员/root权限运行此程序');
    } else {
      console.error('设置优先级失败:', error.message);
    }
    return false;
  }
}

// 获取用户信息可能在某些环境下受限
function safeGetUserInfo() {
  try {
    return os.userInfo();
  } catch (error) {
    console.warn('无法获取用户信息:', error.message);
    return {
      uid: -1,
      gid: -1,
      username: 'unknown',
      homedir: os.homedir(),
      shell: 'unknown'
    };
  }
}
```

### 4. 数据精度和内存管理

处理大数值时要注意精度问题，避免内存泄漏：

```javascript
const os = require('os');

// 使用 BigInt 处理大数值
const totalMemory = os.totalmem();
const totalMemoryBigInt = BigInt(totalMemory);

console.log('普通数字:', totalMemory);
console.log('BigInt:', totalMemoryBigInt.toString());

// 监控工具中避免内存泄漏
class MemoryEfficientMonitor {
  constructor() {
    this.maxHistorySize = 100; // 限制历史记录大小
    this.history = [];
  }

  addData(data) {
    this.history.push(data);

    // 限制历史记录大小
    if (this.history.length > this.maxHistorySize) {
      this.history.shift(); // 移除最旧的数据
    }
  }

  clearHistory() {
    this.history = [];
  }

  getAverageCPU() {
    if (this.history.length === 0) return 0;
    const sum = this.history.reduce((acc, item) => acc + item.cpu, 0);
    return (sum / this.history.length).toFixed(2);
  }
}
```

### 5. 系统资源限制

注意系统资源限制，避免创建过多的进程或文件描述符：

```javascript
const os = require('os');

// 计算合适的 worker 数量
function calculateOptimalWorkerCount() {
  const cpuCount = os.cpus().length;
  const totalMemory = os.totalmem();
  const freeMemory = os.freemem();

  // 基于CPU核心数
  const maxByCPU = cpuCount;

  // 基于内存（假设每个worker需要512MB）
  const maxByMemory = Math.floor(freeMemory / (512 * 1024 * 1024));

  // 取较小值，最少保留1个worker
  return Math.max(1, Math.min(maxByCPU, maxByMemory));
}

// 检查系统负载
function isSystemOverloaded() {
  const loadavg = os.loadavg();
  const cpuCount = os.cpus().length;
  const loadRatio = loadavg[0] / cpuCount;

  // 如果负载超过CPU核心数的80%，认为系统过载
  return loadRatio > 0.8;
}

// 内存使用率检查
function getMemoryWarning() {
  const total = os.totalmem();
  const free = os.freemem();
  const used = total - free;
  const usagePercent = (used / total) * 100;

  if (usagePercent > 90) {
    return { level: 'critical', message: '内存严重不足', usagePercent };
  } else if (usagePercent > 80) {
    return { level: 'warning', message: '内存使用率偏高', usagePercent };
  } else {
    return { level: 'normal', message: '内存使用正常', usagePercent };
  }
}
```

## 最佳实践

### 1. 系统信息缓存

```javascript
class SystemInfoManager {
  constructor() {
    this.cache = {};
    this.cacheConfig = {
      static: { ttl: Infinity },    // 不变的信息
      dynamic: { ttl: 60000 },      // 1分钟更新
      realtime: { ttl: 1000 }       // 1秒更新
    };
  }

  get(type, getter) {
    const now = Date.now();
    const config = this.cacheConfig[type];

    if (!this.cache[type] || (now - this.cache[type].timestamp) > config.ttl) {
      this.cache[type] = {
        data: getter(),
        timestamp: now
      };
    }

    return this.cache[type].data;
  }

  // 使用示例
  getPlatform() {
    return this.get('static', () => ({
      platform: os.platform(),
      arch: os.arch(),
      type: os.type()
    }));
  }

  getMemoryInfo() {
    return this.get('dynamic', () => ({
      total: os.totalmem(),
      free: os.freemem(),
      used: os.totalmem() - os.freemem()
    }));
  }

  getCPUUsage() {
    return this.get('realtime', () => this.calculateCPUUsage());
  }

  calculateCPUUsage() {
    // CPU 使用率计算逻辑
    return 'CPU calculation here';
  }
}
```

### 2. 健康检查和告警

```javascript
class HealthMonitor {
  constructor() {
    this.thresholds = {
      cpu: 80,
      memory: 85,
      load: 1.0
    };
    this.alertHistory = [];
  }

  async checkHealth() {
    const results = {
      timestamp: new Date().toISOString(),
      healthy: true,
      checks: {}
    };

    // CPU 检查
    results.checks.cpu = await this.checkCPU();

    // 内存检查
    results.checks.memory = await this.checkMemory();

    // 负载检查
    results.checks.load = await this.checkLoad();

    // 网络检查
    results.checks.network = await this.checkNetwork();

    // 计算整体健康状态
    results.healthy = Object.values(results.checks).every(check => check.healthy);

    // 处理告警
    if (!results.healthy) {
      await this.handleAlert(results);
    }

    return results;
  }

  async handleAlert(healthResult) {
    const alerts = Object.entries(healthResult.checks)
      .filter(([_, check]) => !check.healthy)
      .map(([name, check]) => ({ type: name, message: check.message }));

    // 防止重复告警
    const newAlerts = alerts.filter(alert =>
      !this.alertHistory.some(history =>
        history.type === alert.type &&
        (Date.now() - history.timestamp) < 300000 // 5分钟内不重复告警
      )
    );

    if (newAlerts.length > 0) {
      // 发送告警通知
      await this.sendAlerts(newAlerts);

      // 记录告警历史
      newAlerts.forEach(alert => {
        this.alertHistory.push({
          ...alert,
          timestamp: Date.now()
        });
      });
    }
  }
}
```

### 3. 资源限制和优雅降级

```javascript
class ResourceManager {
  constructor() {
    this.limits = {
      maxMemory: 0.8,     // 最大内存使用率80%
      maxCPU: 0.9,        // 最大CPU使用率90%
      maxLoad: 1.5        // 最大负载
    };
    this.degraded = false;
  }

  checkResources() {
    const memoryUsage = this.getMemoryUsage();
    const cpuUsage = this.getCPUUsage();
    const systemLoad = this.getSystemLoad();

    const overloaded = {
      memory: memoryUsage > this.limits.maxMemory,
      cpu: cpuUsage > this.limits.maxCPU,
      load: systemLoad > this.limits.maxLoad
    };

    return { memoryUsage, cpuUsage, systemLoad, overloaded };
  }

  shouldDegradedService() {
    const resources = this.checkResources();
    const needsDegradation = Object.values(resources.overloaded).some(Boolean);

    if (needsDegradation && !this.degraded) {
      this.enableDegradation();
    } else if (!needsDegradation && this.degraded) {
      this.disableDegradation();
    }

    return needsDegradation;
  }

  enableDegradation() {
    this.degraded = true;
    console.log('启用服务降级模式');

    // 降级策略
    this.disableNonCriticalFeatures();
    this.increaseCacheTTL();
    this.reduceLogLevel();
  }

  disableDegradation() {
    this.degraded = false;
    console.log('恢复正常服务模式');

    // 恢复正常配置
    this.enableAllFeatures();
    this.resetCacheTTL();
    this.resetLogLevel();
  }
}
```

### 4. 错误处理和恢复

```javascript
class RobustSystemMonitor {
  constructor() {
    this.recoveryAttempts = {};
    this.maxRecoveryAttempts = 3;
  }

  async safeExecute(operation, fallback) {
    try {
      return await operation();
    } catch (error) {
      console.error('操作执行失败:', error);
      if (fallback) {
        try {
          return await fallback();
        } catch (fallbackError) {
          console.error('fallback 执行失败:', fallbackError);
          throw error;
        }
      }
      throw error;
    }
  }

  async safeGetSystemInfo() {
    return await this.safeExecute(
      () => this.getSystemInfo(),
      () => this.getFallbackSystemInfo()
    );
  }

  async attemptRecovery(component, recoveryFunction) {
    const attempts = this.recoveryAttempts[component] || 0;

    if (attempts >= this.maxRecoveryAttempts) {
      console.error(`${component} 恢复尝试次数已达上限`);
      return false;
    }

    try {
      await recoveryFunction();
      this.recoveryAttempts[component] = 0; // 重置尝试次数
      return true;
    } catch (error) {
      this.recoveryAttempts[component] = attempts + 1;
      console.error(`${component} 恢复失败 (尝试 ${attempts + 1}/${this.maxRecoveryAttempts}):`, error);
      return false;
    }
  }
}
```

## 总结

Node.js 的 `os` 模块提供了丰富的系统信息获取能力，是系统监控、资源管理和跨平台开发的重要工具。通过本文的学习，我们掌握了：

1. **系统信息获取**：包括操作系统平台、架构、版本等基本信息
2. **硬件资源监控**：CPU、内存、系统负载等实时监控
3. **网络接口管理**：获取网络配置和连接状态
4. **跨平台路径处理**：处理不同操作系统的路径差异
5. **实用工具函数**：系统运行时间、用户信息、进程优先级等

通过合理使用这些功能，我们可以：
- 构建高效的系统监控工具
- 实现智能的资源管理
- 开发跨平台的应用程序
- 优化系统性能和资源使用
- 实现健康检查和告警系统

在实际开发中，建议：
- 根据具体需求选择合适的方法
- 注意平台差异和兼容性
- 合理使用缓存减少系统调用
- 实现完善的错误处理机制
- 遵循性能优化和资源管理的最佳实践

掌握 `os` 模块的使用对于开发高质量、高性能的 Node.js 应用程序至关重要。