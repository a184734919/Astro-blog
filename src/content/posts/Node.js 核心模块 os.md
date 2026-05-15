---
title: Node.js 核心模块 os
published: 2023-02-06
description: '操作系统信息获取的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 的 `os` 模块提供了与操作系统相关的基本实用程序和方法，用于获取系统信息、网络接口、CPU、内存等信息。这些信息对于系统监控、性能优化、跨平台开发等场景非常有用。

## 核心概念

`os` 模块主要提供以下功能：
- **系统信息**：获取操作系统、架构、版本等基本信息
- **网络接口**：获取网络接口和地址信息
- **硬件信息**：获取 CPU、内存等硬件资源信息
- **路径信息**：获取系统路径信息（如临时目录、主目录等）
- **进程信息**：获取当前进程的用户信息

## 基本用法

### 1. 系统基本信息

#### 操作系统和架构

```javascript
const os = require('os');

// 操作系统平台
console.log(os.platform());  // 'darwin', 'linux', 'win32', 'freebsd', 等

// 操作系统版本
console.log(os.release());   // '18.7.0', '5.4.0-42-generic', 等

// 操作系统类型
console.log(os.type());      // 'Darwin', 'Linux', 'Windows_NT'

// CPU 架构
console.log(os.arch());      // 'x64', 'arm', 'arm64', 'ia32', 等

// 操作系统主机名
console.log(os.hostname());  // 'MacBook-Pro.local', 等
```

#### 系统常量

```javascript
const os = require('os');

// 信号常量
console.log(os.constants.signals.SIGINT);    // 2
console.log(os.constants.signals.SIGTERM);   // 15
console.log(os.constants.signals.SIGKILL);   // 9

// 错误常量
console.log(os.constants.errno.EEXIST);      // 文件已存在错误码

// 优先级常量
console.log(os.constants.priority.PRIORITY_LOW);     // 19
console.log(os.constants.priority.PRIORITY_NORMAL);   // 0
console.log(os.constants.priority.PRIORITY_HIGH);    // -10
```

### 2. 内存信息

#### 内存统计

```javascript
const os = require('os');

// 系统总内存（字节）
console.log(os.totalmem());  // 例如：17179869184 (16GB)

// 系统空闲内存（字节）
console.log(os.freemem());   // 例如：8589934592 (8GB)

// 内存使用情况
const total = os.totalmem();
const free = os.freemem();
const used = total - free;
const usagePercent = ((used / total) * 100).toFixed(2);

console.log(`总内存: ${(total / 1024 / 1024 / 1024).toFixed(2)} GB`);
console.log(`空闲内存: ${(free / 1024 / 1024 / 1024).toFixed(2)} GB`);
console.log(`已用内存: ${(used / 1024 / 1024 / 1024).toFixed(2)} GB`);
console.log(`内存使用率: ${usagePercent}%`);
```

### 3. CPU 信息

#### CPU 详细信息

```javascript
const os = require('os');

// 获取 CPU 信息
const cpus = os.cpus();

console.log(`CPU 核心数: ${cpus.length}`);

cpus.forEach((cpu, index) => {
  console.log(`\n核心 ${index + 1}:`);
  console.log(`  型号: ${cpu.model}`);
  console.log(`  速度: ${cpu.speed} MHz`);
  console.log(`  用户时间: ${cpu.times.user} ms`);
  console.log(`  系统时间: ${cpu.times.sys} ms`);
  console.log(`  空闲时间: ${cpu.times.idle} ms`);
  console.log(`  中断时间: ${cpu.times.irq} ms`);
});
```

#### CPU 平均负载

```javascript
const os = require('os');

// 获取平均负载（1、5、15 分钟）
const loadavg = os.loadavg();
console.log(`1分钟平均负载: ${loadavg[0]}`);
console.log(`5分钟平均负载: ${loadavg[1]}`);
console.log(`15分钟平均负载: ${loadavg[2]}`);

// 计算负载百分比（核心数相关）
const cpuCount = os.cpus().length;
const loadPercent = ((loadavg[0] / cpuCount) * 100).toFixed(2);
console.log(`当前负载百分比: ${loadPercent}%`);
```

### 4. 网络接口

#### 网络接口信息

```javascript
const os = require('os');

const networkInterfaces = os.networkInterfaces();

for (const [interfaceName, addresses] of Object.entries(networkInterfaces)) {
  console.log(`\n接口: ${interfaceName}`);

  addresses.forEach(address => {
    console.log(`  地址: ${address.address}`);
    console.log(`  家族: ${address.family}`);  // 'IPv4' 或 'IPv6'
    console.log(`  内部: ${address.internal}`); // true/false
    console.log(`  MAC: ${address.mac}`);
    console.log(`  网络掩码: ${address.netmask}`);
    console.log(`  前缀长度: ${address.prefixlen}`);
  });
}
```

#### 获取本地 IP 地址

```javascript
const os = require('os');

function getLocalIP() {
  const interfaces = os.networkInterfaces();
  for (const interfaceName of Object.keys(interfaces)) {
    for (const address of interfaces[interfaceName]) {
      if (address.family === 'IPv4' && !address.internal) {
        return address.address;
      }
    }
  }
  return '127.0.0.1';
}

console.log('本地 IP:', getLocalIP());
```

### 5. 路径信息

#### 系统路径

```javascript
const os = require('os');

// 用户主目录
console.log('主目录:', os.homedir());

// 临时文件目录
console.log('临时目录:', os.tmpdir());

// 当前用户的主目录（同 homedir）
console.log('用户目录:', os.userInfo().homedir);

// 系统默认的 shell
console.log('Shell:', os.userInfo().shell);
```

#### 用户信息

```javascript
const os = require('os');

// 当前用户信息
const userInfo = os.userInfo();

console.log('用户名:', userInfo.username);
console.log('用户 ID:', userInfo.uid);
console.log('组 ID:', userInfo.gid);
console.log('主目录:', userInfo.homedir);
console.log('Shell:', userInfo.shell);
console.log('环境变量:', userInfo.shell);

// 获取指定用户的组信息
const groupInfo = os.userInfo({ encoding: 'buffer' });
```

### 6. 文件系统信息

#### 目录信息

```javascript
const os = require('os');
const path = require('path');

// 获取系统特定的临时目录
const tempDir = os.tmpdir();
console.log('临时目录:', tempDir);

// 创建临时文件路径
const tempFilePath = path.join(tempDir, 'temp_file.txt');
console.log('临时文件路径:', tempFilePath);

// 获取用户主目录
const userHome = os.homedir();
console.log('用户主目录:', userHome);

// 获取用户配置目录
const configDir = path.join(userHome, '.config');
console.log('配置目录:', configDir);
```

### 7. 其他实用功能

#### 进程信息

```javascript
const os = require('os');

// 设置进程优先级（需要特定权限）
try {
  os.setPriority(os.constants.priority.PRIORITY_HIGH);
  console.log('优先级设置成功');
} catch (error) {
  console.log('权限不足，无法设置优先级');
}

// 获取进程优先级
const priority = os.getPriority();
console.log('当前进程优先级:', priority);
```

#### 系统运行时间

```javascript
const os = require('os');

// 系统运行时间（秒）
const uptime = os.uptime();
const hours = Math.floor(uptime / 3600);
const minutes = Math.floor((uptime % 3600) / 60);
const seconds = Math.floor(uptime % 60);

console.log(`系统已运行: ${hours} 小时 ${minutes} 分钟 ${seconds} 秒`);
```

## 实际应用

### 1. 系统监控工具

```javascript
const os = require('os');

class SystemMonitor {
  constructor() {
    this.interval = null;
  }

  startMonitoring(intervalMs = 5000) {
    this.interval = setInterval(() => {
      this.report();
    }, intervalMs);
  }

  stopMonitoring() {
    if (this.interval) {
      clearInterval(this.interval);
      this.interval = null;
    }
  }

  report() {
    const cpuUsage = this.getCPUUsage();
    const memUsage = this.getMemoryUsage();
    const load = os.loadavg();

    console.log('\n=== 系统状态 ===');
    console.log(`CPU 使用率: ${cpuUsage}%`);
    console.log(`内存使用: ${memUsage.usedPercent.toFixed(2)}%`);
    console.log(`1分钟负载: ${load[0]}`);
    console.log(`系统运行: ${this.formatUptime(os.uptime())}`);
  }

  getCPUUsage() {
    const cpus = os.cpus();
    let totalUser = 0;
    let totalSys = 0;
    let totalIdle = 0;

    cpus.forEach(cpu => {
      totalUser += cpu.times.user;
      totalSys += cpu.times.sys;
      totalIdle += cpu.times.idle;
    });

    const total = totalUser + totalSys + totalIdle;
    return ((totalUser + totalSys) / total * 100).toFixed(2);
  }

  getMemoryUsage() {
    const total = os.totalmem();
    const free = os.freemem();
    const used = total - free;

    return {
      total: total / 1024 / 1024 / 1024,
      used: used / 1024 / 1024 / 1024,
      free: free / 1024 / 1024 / 1024,
      usedPercent: (used / total) * 100
    };
  }

  formatUptime(seconds) {
    const hours = Math.floor(seconds / 3600);
    const minutes = Math.floor((seconds % 3600) / 60);
    const secs = Math.floor(seconds % 60);
    return `${hours}h ${minutes}m ${secs}s`;
  }
}

// 使用示例
const monitor = new SystemMonitor();
monitor.startMonitoring(5000);
```

### 2. 跨平台路径处理

```javascript
const os = require('os');
const path = require('path');

class PathManager {
  static getAppDataPath(appName) {
    const platform = os.platform();

    if (platform === 'win32') {
      return path.join(os.homedir(), 'AppData', 'Roaming', appName);
    } else if (platform === 'darwin') {
      return path.join(os.homedir(), 'Library', 'Application Support', appName);
    } else {
      return path.join(os.homedir(), '.config', appName);
    }
  }

  static getCachePath(appName) {
    const platform = os.platform();

    if (platform === 'win32') {
      return path.join(os.homedir(), 'AppData', 'Local', appName, 'Cache');
    } else if (platform === 'darwin') {
      return path.join(os.homedir(), 'Library', 'Caches', appName);
    } else {
      return path.join(os.homedir(), '.cache', appName);
    }
  }

  static getLogPath(appName) {
    const platform = os.platform();

    if (platform === 'win32') {
      return path.join(os.homedir(), 'AppData', 'Local', appName, 'Logs');
    } else if (platform === 'darwin') {
      return path.join(os.homedir(), 'Library', 'Logs', appName);
    } else {
      return path.join(os.homedir(), '.local', 'share', appName, 'logs');
    }
  }
}

console.log('应用数据路径:', PathManager.getAppDataPath('MyApp'));
console.log('缓存路径:', PathManager.getCachePath('MyApp'));
console.log('日志路径:', PathManager.getLogPath('MyApp'));
```

### 3. 资源健康检查

```javascript
const os = require('os');

class HealthChecker {
  static async checkSystemHealth() {
    const checks = {
      cpu: this.checkCPU(),
      memory: this.checkMemory(),
      disk: this.checkDisk(),
      network: this.checkNetwork()
    };

    const results = await Promise.all(Object.values(checks));
    const health = Object.keys(checks).reduce((acc, key, index) => {
      acc[key] = results[index];
      return acc;
    }, {});

    return {
      healthy: Object.values(health).every(check => check.healthy),
      timestamp: new Date().toISOString(),
      checks: health
    };
  }

  static checkCPU() {
    const cpus = os.cpus();
    const load = os.loadavg()[0];
    const cpuCount = cpus.length;
    const loadPercent = (load / cpuCount) * 100;

    return {
      healthy: loadPercent < 80,
      cores: cpuCount,
      loadAverage: load,
      loadPercent: loadPercent.toFixed(2),
      message: loadPercent >= 80 ? 'CPU 负载过高' : 'CPU 状态正常'
    };
  }

  static checkMemory() {
    const total = os.totalmem();
    const free = os.freemem();
    const used = total - free;
    const usedPercent = (used / total) * 100;

    return {
      healthy: usedPercent < 85,
      total: (total / 1024 / 1024 / 1024).toFixed(2),
      used: (used / 1024 / 1024 / 1024).toFixed(2),
      free: (free / 1024 / 1024 / 1024).toFixed(2),
      usedPercent: usedPercent.toFixed(2),
      message: usedPercent >= 85 ? '内存使用率过高' : '内存状态正常'
    };
  }

  static checkDisk() {
    // 这里需要使用 fs 模块获取磁盘信息
    // 简化示例
    return {
      healthy: true,
      message: '磁盘检查需要额外实现'
    };
  }

  static checkNetwork() {
    const interfaces = os.networkInterfaces();
    const hasNetwork = Object.values(interfaces).some(addrs =>
      addrs.some(addr => !addr.internal && addr.family === 'IPv4')
    );

    return {
      healthy: hasNetwork,
      message: hasNetwork ? '网络连接正常' : '无网络连接'
    };
  }
}

// 使用示例
HealthChecker.checkSystemHealth().then(health => {
  console.log(JSON.stringify(health, null, 2));
});
```

### 4. 智能服务器配置

```javascript
const os = require('os');

class ServerConfig {
  constructor() {
    this.cpuCount = os.cpus().length;
    this.memory = os.totalmem();
  }

  getOptimalWorkerCount() {
    // 基于 CPU 核心数和内存计算最佳 worker 数量
    const memPerWorker = 512 * 1024 * 1024; // 每个 worker 512MB
    const maxWorkersByMem = Math.floor(this.memory / memPerWorker);
    const maxWorkersByCPU = this.cpuCount * 2;

    return Math.min(maxWorkersByMem, maxWorkersByCPU);
  }

  getOptimalClusterSettings() {
    return {
      workers: this.getOptimalWorkerCount(),
      maxConnections: Math.floor(this.memory / 1024 / 1024), // 每 MB 内存一个连接
      timeout: 30000
    };
  }

  getSystemInfo() {
    return {
      platform: os.platform(),
      arch: os.arch(),
      hostname: os.hostname(),
      uptime: os.uptime(),
      loadavg: os.loadavg(),
      cpu: {
        count: this.cpuCount,
        model: os.cpus()[0]?.model || 'Unknown'
      },
      memory: {
        total: (this.memory / 1024 / 1024 / 1024).toFixed(2) + ' GB',
        free: (os.freemem() / 1024 / 1024 / 1024).toFixed(2) + ' GB'
      },
      network: this.getNetworkInfo()
    };
  }

  getNetworkInfo() {
    const interfaces = os.networkInterfaces();
    const publicIPs = [];

    for (const [name, addrs] of Object.entries(interfaces)) {
      for (const addr of addrs) {
        if (!addr.internal && addr.family === 'IPv4') {
          publicIPs.push({
            interface: name,
            address: addr.address,
            mac: addr.mac
          });
        }
      }
    }

    return publicIPs;
  }
}

const config = new ServerConfig();
console.log('系统信息:', config.getSystemInfo());
console.log('集群配置:', config.getOptimalClusterSettings());
```

## 注意事项

1. **平台差异**：不同操作系统返回的值可能不同，需要做兼容处理
2. **权限问题**：某些操作（如设置进程优先级）需要特定权限
3. **缓存问题**：`os.cpus()` 等信息是调用时的快照，可能会变化
4. **性能考虑**：频繁调用 `os` 方法可能影响性能，应适当缓存结果
5. **单位换算**：注意字节、KB、MB、GB 之间的换算关系

## 最佳实践

1. **跨平台兼容**：使用 `os.platform()` 判断平台，实现跨平台逻辑
2. **路径处理**：结合 `path` 模块处理路径，避免硬编码分隔符
3. **资源监控**：定期检查系统资源，避免过度消耗
4. **错误处理**：对可能失败的调用进行 try-catch 处理
5. **性能优化**：缓存不常变化的信息，减少系统调用

## 总结

`os` 模块是 Node.js 中获取系统信息的核心模块，提供了丰富的系统信息获取接口。通过合理使用这些接口，可以实现系统监控、资源管理、跨平台配置等功能。在实际项目中，应结合具体场景选择合适的方法，并注意平台差异和性能影响。