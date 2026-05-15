---
title: Node.js 核心模块 child_process
published: 2023-02-10
description: '子进程创建和管理的详细介绍和学习笔记'
image: ''
tags: ["Node.js","核心模块","进程管理"]
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 的 `child_process` 模块提供了创建子进程的能力，允许 Node.js 应用程序执行系统命令、运行脚本、与其他进程通信等。这是 Node.js 中实现多进程、任务并行化、系统集成的重要功能，使得 Node.js 单线程应用能够充分利用多核 CPU，处理 CPU 密集型任务，以及与操作系统深度集成。

## 核心概念

### 子进程创建方法

`child_process` 模块提供了多种创建子进程的方式：

1. **spawn()** - 创建子进程，以流的形式输出数据，适合长时间运行的进程
2. **exec()** - 创建 shell 并执行命令，缓冲输出，适合简单命令
3. **execFile()** - 直接执行文件，更安全，适合执行已知程序
4. **fork()** - 创建 Node.js 子进程，支持 IPC 通信，适合 Node.js 模块间通信
5. **spawnSync()** - 同步版本的 spawn，会阻塞主线程
6. **execSync()** - 同步版本的 exec，会阻塞主线程

### 标准流和 IPC

子进程有三个标准流：stdin（标准输入）、stdout（标准输出）、stderr（标准错误），以及 IPC（进程间通信）通道。

### 进程状态管理

子进程有不同的状态：启动、运行、停止、退出等，需要正确处理状态转换和清理。

## 基本用法

### 1. spawn() - 流式输出

#### 基础用法

```javascript
const { spawn } = require('child_process');

// 创建子进程
const ls = spawn('ls', ['-la', '/']);

// 监听标准输出
ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

// 监听标准错误
ls.stderr.on('data', (data) => {
  console.error(`stderr: ${data}`);
});

// 监听进程退出事件
ls.on('close', (code) => {
  console.log(`子进程退出，退出码: ${code}`);
});

// 监听错误事件
ls.on('error', (error) => {
  console.error('进程启动失败:', error);
});

// 监听进程退出事件
ls.on('exit', (code, signal) => {
  console.log(`进程退出，退出码: ${code}, 信号: ${signal}`);
});
```

#### 带选项的 spawn

```javascript
const { spawn } = require('child_process');

const process = spawn('npm', ['run', 'build'], {
  // 工作目录
  cwd: '/path/to/project',

  // 环境变量（继承当前环境并添加自定义变量）
  env: {
    ...process.env,
    NODE_ENV: 'production',
    CUSTOM_VAR: 'value'
  },

  // 标准输入/输出/错误处理
  // 'inherit' - 继承父进程的流
  // 'pipe' - 创建管道
  // 'ignore' - 忽略流
  // Stream 对象 - 连接到指定的流
  stdio: ['inherit', 'pipe', 'pipe'],

  // 是否使用 shell 执行命令
  shell: false,

  // 是否分离进程（创建守护进程）
  detached: false,

  // 用户 ID (需要 root 权限)
  uid: 1000,

  // 组 ID
  gid: 1000,

  // 是否为子进程创建新的 IPC 通道
  ipc: false,

  // 超时时间（毫秒）
  timeout: 0
});

// 监听错误
process.on('error', (error) => {
  console.error('进程启动失败:', error);
});

// 监听消息（当 IPC 为 true 时）
process.on('message', (message) => {
  console.log('收到子进程消息:', message);
});

// 向子进程发送消息
process.send({ hello: 'parent' });
```

#### 流处理和管道操作

```javascript
const { spawn } = require('child_process');

// 创建多个进程并连接管道
const grep = spawn('grep', ['text']);
const cat = spawn('cat', ['file.txt']);

// 连接进程管道
cat.stdout.pipe(grep.stdin);

// 处理最终输出
grep.stdout.on('data', (data) => {
  console.log(data.toString());
});

// 监听错误
cat.stderr.on('data', (data) => {
  console.error('cat 错误:', data.toString());
});

grep.stderr.on('data', (data) => {
  console.error('grep 错误:', data.toString());
});

// 链式管道操作：ps | sort | head
const ps = spawn('ps', ['aux']);
const sort = spawn('sort', ['-nk3']); // 按第3列数字排序
const head = spawn('head', ['-n5']);  // 显示前5行

ps.stdout.pipe(sort.stdin);
sort.stdout.pipe(head.stdin);
head.stdout.on('data', (data) => {
  console.log(data.toString());
});

// 处理错误
[ps, sort, head].forEach(proc => {
  proc.on('error', (error) => {
    console.error('进程错误:', error);
  });
});
```

### 2. exec() - Shell 执行

#### 基础用法

```javascript
const { exec } = require('child_process');

// 执行 shell 命令
exec('ls -la /', (error, stdout, stderr) => {
  if (error) {
    console.error(`执行出错: ${error}`);
    return;
  }

  console.log('标准输出:');
  console.log(stdout);

  if (stderr) {
    console.error('标准错误:');
    console.error(stderr);
  }
});
```

#### 带选项的 exec

```javascript
const { exec } = require('child_process');

exec('ls -la /', {
  // 工作目录
  cwd: '/',

  // 环境变量
  env: { ...process.env, LANG: 'en_US.UTF-8' },

  // 超时时间（毫秒）
  timeout: 5000,

  // 缓冲区最大大小（字节）
  maxBuffer: 1024 * 1024, // 1MB

  // 杀死进程的信号
  killSignal: 'SIGTERM',

  // 编码格式
  encoding: 'utf8',

  // 是否使用 shell
  shell: '/bin/bash'

}, (error, stdout, stderr) => {
  if (error) {
    // error.code - 退出码
    // error.signal - 终止信号
    // error.killed - 是否被杀死
    console.error(`执行失败: ${error.message}`);
    console.error(`退出码: ${error.code}`);
    console.error(`信号: ${error.signal}`);
    return;
  }

  console.log('执行成功');
  console.log('输出:', stdout);
});
```

#### 使用 Promise 封装

```javascript
const { exec } = require('child_process');
const { promisify } = require('util');

const execAsync = promisify(exec);

async function runCommand() {
  try {
    const { stdout, stderr } = await execAsync('node --version');
    console.log('Node.js 版本:', stdout.trim());

    if (stderr) {
      console.warn('标准错误输出:', stderr);
    }

    return stdout.trim();
  } catch (error) {
    console.error('命令执行失败:', error);
    throw error;
  }
}

// 使用示例
runCommand()
  .then(version => console.log('版本:', version))
  .catch(error => console.error('错误:', error));
```

#### 执行复杂命令

```javascript
const { exec } = require('child_process');

// 执行包含管道和重定向的命令
const command = `
  cat file.txt |
  grep "pattern" |
  sed 's/old/new/g' |
  tee output.txt |
  wc -l
`;

exec(command, (error, stdout, stderr) => {
  if (error) {
    console.error('命令执行失败:', error);
    return;
  }

  console.log('行数:', stdout.trim());
});

// 执行带变量的命令
const fileName = 'example.txt';
const searchPattern = 'test';

exec(`grep "${searchPattern}" ${fileName}`, (error, stdout, stderr) => {
  if (error) {
    console.error('搜索失败:', error);
    return;
  }

  console.log('搜索结果:', stdout);
});
```

### 3. execFile() - 直接执行文件

#### 基础用法

```javascript
const { execFile } = require('child_process');

// 直接执行文件（比 exec 更安全，避免 shell 注入）
execFile('node', ['--version'], (error, stdout, stderr) => {
  if (error) {
    console.error(`执行出错: ${error}`);
    return;
  }
  console.log('Node.js 版本:', stdout.trim());
});
```

#### 执行脚本文件

```javascript
const { execFile } = require('child_process');

// 执行 Python 脚本
execFile('python3', ['script.py', 'arg1', 'arg2'], {
  cwd: '/path/to/scripts',
  env: {
    PYTHONPATH: '/custom/path',
    PYTHON_VERSION: '3.8'
  }
}, (error, stdout, stderr) => {
  if (error) {
    console.error('脚本执行失败:', error);
    return;
  }
  console.log('脚本输出:', stdout);
  if (stderr) {
    console.error('脚本错误:', stderr);
  }
});

// 执行 Shell 脚本
execFile('./deploy.sh', ['production'], {
  cwd: '/path/to/project'
}, (error, stdout, stderr) => {
  if (error) {
    console.error('部署失败:', error);
    return;
  }
  console.log('部署成功:', stdout);
});
```

#### 带选项的 execFile

```javascript
const { execFile } = require('child_process');

execFile('git', ['status', '--short'], {
  // 工作目录
  cwd: '/path/to/repo',

  // 环境变量
  env: {
    GIT_AUTHOR_NAME: 'CI Bot',
    GIT_AUTHOR_EMAIL: 'ci@example.com'
  },

  // 超时时间
  timeout: 10000,

  // 最大缓冲区大小
  maxBuffer: 1024 * 1024 * 10, // 10MB

  // 杀死进程的信号
  killSignal: 'SIGKILL',

  // 编码格式
  encoding: 'utf8',

  // 设置用户组
  uid: 1000,
  gid: 1000

}, (error, stdout, stderr) => {
  if (error) {
    console.error('Git 状态检查失败:', error);
    return;
  }

  console.log('Git 状态:', stdout);
  if (stderr) {
    console.error('Git 错误:', stderr);
  }
});
```

### 4. fork() - Node.js 子进程

#### 基础用法

```javascript
// parent.js
const { fork } = require('child_process');

const child = fork('./child.js');

// 发送消息给子进程
child.send({ message: 'Hello from parent' });

// 接收子进程的消息
child.on('message', (msg) => {
  console.log('来自子进程的消息:', msg);
});

// 监听子进程退出
child.on('exit', (code, signal) => {
  console.log(`子进程退出，退出码: ${code}, 信号: ${signal}`);
});

// 监听子进程错误
child.on('error', (error) => {
  console.error('子进程错误:', error);
});

// child.js
process.on('message', (msg) => {
  console.log('来自父进程的消息:', msg);

  // 处理消息
  const result = {
    response: 'Hello from child',
    original: msg,
    processed: true
  };

  // 发送响应给父进程
  process.send(result);
});

// 模拟长时间任务
setTimeout(() => {
  process.exit(0);
}, 2000);
```

#### 带参数的 fork

```javascript
const { fork } = require('child_process');

// 使用配置对象创建 Worker
const worker = fork('./worker.js', ['arg1', 'arg2'], {
  // 工作目录
  cwd: '/path/to/project',

  // 环境变量
  env: {
    ...process.env,
    WORKER_TYPE: 'calculation',
    DEBUG: 'true'
  },

  // 是否分离进程
  detached: false,

  // 是否静默模式（不连接IPC）
  silent: false,

  // 执行参数
  execArgv: ['--max-old-space-size=4096'],

  // 自定义 IPC 通道
  ipc: true
});

worker.on('message', (message) => {
  console.log('Worker 消息:', message);
});

// 发送任务
worker.send({
  type: 'calculate',
  data: [1, 2, 3, 4, 5]
});

// worker.js
console.log('Worker 参数:', process.argv);

process.on('message', (message) => {
  console.log('环境变量:', process.env.WORKER_TYPE);

  if (message.type === 'calculate') {
    const result = message.data.reduce((sum, num) => sum + num, 0);
    process.send({ type: 'result', value: result });
  }
});
```

#### 高级 IPC 通信

```javascript
// parent.js
const { fork } = require('child_process');

class WorkerManager {
  constructor(scriptPath, workerCount = 4) {
    this.scriptPath = scriptPath;
    this.workerCount = workerCount;
    this.workers = [];
    this.taskQueue = [];
    this.activeTasks = new Map();
  }

  start() {
    for (let i = 0; i < this.workerCount; i++) {
      const worker = fork(this.scriptPath);

      worker.on('message', (msg) => this.handleWorkerMessage(worker, msg));
      worker.on('error', (err) => this.handleWorkerError(worker, err));
      worker.on('exit', (code, signal) => this.handleWorkerExit(worker, code, signal));

      this.workers.push({
        worker,
        id: worker.pid,
        busy: false,
        tasksCompleted: 0
      });
    }

    console.log(`已启动 ${this.workerCount} 个 worker`);
  }

  handleWorkerMessage(worker, msg) {
    const { taskId, result, error } = msg;
    const task = this.activeTasks.get(taskId);

    if (task) {
      this.activeTasks.delete(taskId);
      const workerInfo = this.workers.find(w => w.id === worker.pid);
      if (workerInfo) {
        workerInfo.busy = false;
        workerInfo.tasksCompleted++;
      }

      if (error) {
        task.reject(new Error(error));
      } else {
        task.resolve(result);
      }

      this.processQueue();
    }
  }

  handleWorkerError(worker, err) {
    console.error(`Worker ${worker.pid} 错误:`, err);
  }

  handleWorkerExit(worker, code, signal) {
    console.log(`Worker ${worker.pid} 退出，退出码: ${code}, 信号: ${signal}`);

    // 重启 worker
    const index = this.workers.findIndex(w => w.id === worker.pid);
    if (index !== -1) {
      this.workers[index].worker = fork(this.scriptPath);
      this.workers[index].id = this.workers[index].worker.pid;
      this.workers[index].busy = false;
    }
  }

  async executeTask(data) {
    const taskId = `${Date.now()}_${Math.random()}`;
    return new Promise((resolve, reject) => {
      this.taskQueue.push({ taskId, data, resolve, reject });
      this.processQueue();
    });
  }

  processQueue() {
    const availableWorker = this.workers.find(w => !w.busy);
    if (availableWorker && this.taskQueue.length > 0) {
      const task = this.taskQueue.shift();
      availableWorker.busy = true;
      this.activeTasks.set(task.taskId, task);
      availableWorker.worker.send({
        taskId: task.taskId,
        data: task.data
      });
    }
  }

  getStats() {
    return {
      totalWorkers: this.workers.length,
      busyWorkers: this.workers.filter(w => w.busy).length,
      pendingTasks: this.taskQueue.length,
      activeTasks: this.activeTasks.size,
      tasksCompleted: this.workers.reduce((sum, w) => sum + w.tasksCompleted, 0)
    };
  }

  shutdown() {
    return Promise.all(
      this.workers.map(({ worker }) => {
        return new Promise(resolve => {
          worker.on('exit', resolve);
          worker.kill();
        });
      })
    );
  }
}

// 使用示例
const manager = new WorkerManager('./worker.js', 4);
manager.start();

async function runTasks() {
  const tasks = Array.from({ length: 10 }, (_, i) => i + 1);

  for (const task of tasks) {
    const result = await manager.executeTask({ task, data: `Task ${task}` });
    console.log('任务完成:', result);
    console.log('统计信息:', manager.getStats());
  }

  await manager.shutdown();
  console.log('所有任务完成');
}

runTasks();

// worker.js
process.on('message', async ({ taskId, data }) => {
  try {
    console.log(`Worker ${process.pid} 处理任务: ${taskId}`);

    // 模拟工作
    await new Promise(resolve => setTimeout(resolve, 1000));

    const result = {
      taskId,
      data,
      processedBy: process.pid,
      timestamp: Date.now()
    };

    process.send({ taskId, result });
  } catch (error) {
    process.send({ taskId, error: error.message });
  }
});
```

### 5. 同步方法

#### spawnSync

```javascript
const { spawnSync } = require('child_process');

// 同步执行命令（阻塞主线程）
const result = spawnSync('ls', ['-la', '/']);

console.log('状态码:', result.status);
console.log('信号:', result.signal);
console.log('输出:', result.stdout.toString());
console.log('错误:', result.stderr.toString());
console.log('PID:', result.pid);

// 带选项的 spawnSync
const result2 = spawnSync('npm', ['version'], {
  cwd: '/path/to/project',
  env: { ...process.env, NODE_ENV: 'production' },
  input: '',  // 要发送给子进程的标准输入
  timeout: 5000,
  killSignal: 'SIGTERM',
  maxBuffer: 1024 * 1024,
  encoding: 'utf8',
  windowsHide: true
});

if (result2.error) {
  console.error('执行失败:', result2.error);
} else if (result2.status !== 0) {
  console.error('进程非正常退出，退出码:', result2.status);
} else {
  console.log('执行成功:', result2.stdout.toString());
}
```

#### execSync

```javascript
const { execSync } = require('child_process');

try {
  // 同步执行 shell 命令
  const output = execSync('node --version', {
    encoding: 'utf8',
    timeout: 5000,
    maxBuffer: 1024 * 1024,
    killSignal: 'SIGTERM',
    cwd: process.cwd(),
    env: process.env
  });

  console.log('Node.js 版本:', output.trim());
} catch (error) {
  console.error('命令执行失败:', error);
  console.error('退出码:', error.status);
  console.error('信号:', error.signal);
}

// 执行带管道的命令
const result = execSync('cat package.json | grep name', {
  encoding: 'utf8',
  shell: '/bin/bash'
});

console.log('结果:', result);
```

#### 使用同步方法的注意事项

```javascript
const { execSync, spawnSync } = require('child_process');

// 注意：同步方法会阻塞主线程，影响应用性能

// ❌ 不好的做法：在高并发场景使用同步方法
app.get('/api/check', (req, res) => {
  const result = execSync('git status'); // 阻塞请求处理
  res.json({ status: result.toString() });
});

// ✅ 好的做法：使用异步方法
app.get('/api/check', async (req, res) => {
  try {
    const result = await execPromise('git status');
    res.json({ status: result });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// 同步方法适用场景：
// 1. 脚本启动时的一次性操作
// 2. 初始化配置
// 3. 不需要高并发的命令

// 启动时检查版本
function checkNodeVersion() {
  try {
    const version = execSync('node --version', { encoding: 'utf8' }).trim();
    console.log('Node.js 版本:', version);

    const minVersion = '14.0.0';
    if (version.startsWith('v')) {
      const actualVersion = version.slice(1);
      if (actualVersion < minVersion) {
        console.error(`Node.js 版本过低，需要 ${minVersion} 或更高`);
        process.exit(1);
      }
    }
  } catch (error) {
    console.error('无法检查 Node.js 版本:', error);
    process.exit(1);
  }
}

checkNodeVersion();
```

## 实际应用

### 1. 文件压缩和解压工具

```javascript
const { spawn } = require('child_process');
const path = require('path');
const fs = require('fs');

class FileCompressor {
  static async compressFile(sourcePath, outputPath) {
    return new Promise((resolve, reject) => {
      // 检查源文件是否存在
      if (!fs.existsSync(sourcePath)) {
        reject(new Error(`源文件不存在: ${sourcePath}`));
        return;
      }

      const gzip = spawn('gzip', ['-c', sourcePath]);
      const writeStream = fs.createWriteStream(outputPath);

      // 连接输出
      gzip.stdout.pipe(writeStream);

      // 处理错误
      gzip.on('error', reject);
      writeStream.on('error', reject);

      // 完成
      writeStream.on('finish', () => {
        const originalSize = fs.statSync(sourcePath).size;
        const compressedSize = fs.statSync(outputPath).size;
        const ratio = ((1 - compressedSize / originalSize) * 100).toFixed(2);

        console.log(`文件压缩完成: ${outputPath}`);
        console.log(`原始大小: ${this.formatBytes(originalSize)}`);
        console.log(`压缩大小: ${this.formatBytes(compressedSize)}`);
        console.log(`压缩率: ${ratio}%`);

        resolve({
          originalSize,
          compressedSize,
          ratio: parseFloat(ratio)
        });
      });

      // 检查退出码
      gzip.on('close', (code) => {
        if (code !== 0) {
          reject(new Error(`压缩失败，退出码: ${code}`));
        }
      });
    });
  }

  static async decompressFile(sourcePath, outputPath) {
    return new Promise((resolve, reject) => {
      if (!fs.existsSync(sourcePath)) {
        reject(new Error(`压缩文件不存在: ${sourcePath}`));
        return;
      }

      const gunzip = spawn('gunzip', ['-c', sourcePath]);
      const writeStream = fs.createWriteStream(outputPath);

      gunzip.stdout.pipe(writeStream);

      gunzip.on('error', reject);
      writeStream.on('error', reject);

      writeStream.on('finish', () => {
        console.log(`文件解压完成: ${outputPath}`);
        resolve();
      });

      gunzip.on('close', (code) => {
        if (code !== 0) {
          reject(new Error(`解压失败，退出码: ${code}`));
        }
      });
    });
  }

  static async compressDirectory(sourceDir, outputPath) {
    return new Promise((resolve, reject) => {
      const tar = spawn('tar', [
        '-czf', outputPath,
        '-C', path.dirname(sourceDir),
        path.basename(sourceDir)
      ]);

      tar.on('error', reject);
      tar.on('close', (code) => {
        if (code === 0) {
          console.log(`目录压缩完成: ${outputPath}`);
          resolve();
        } else {
          reject(new Error(`目录压缩失败，退出码: ${code}`));
        }
      });
    });
  }

  static async decompressDirectory(sourcePath, outputDir) {
    return new Promise((resolve, reject) => {
      const tar = spawn('tar', ['-xzf', sourcePath, '-C', outputDir]);

      tar.on('error', reject);
      tar.on('close', (code) => {
        if (code === 0) {
          console.log(`目录解压完成: ${outputDir}`);
          resolve();
        } else {
          reject(new Error(`目录解压失败，退出码: ${code}`));
        }
      });
    });
  }

  static formatBytes(bytes) {
    if (bytes === 0) return '0 B';
    const k = 1024;
    const sizes = ['B', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
  }

  static async getFileCompressionInfo(filePath) {
    return new Promise((resolve, reject) => {
      const gzip = spawn('gzip', ['-l', filePath]);

      let output = '';

      gzip.stdout.on('data', (data) => {
        output += data.toString();
      });

      gzip.on('error', reject);
      gzip.on('close', (code) => {
        if (code === 0) {
          // 解析 gzip 输出
          const lines = output.split('\n');
          if (lines.length >= 2) {
            const parts = lines[1].trim().split(/\s+/);
            if (parts.length >= 5) {
              resolve({
                compressed: parseInt(parts[0]),
                uncompressed: parseInt(parts[1]),
                ratio: parts[2],
                uncompressedName: parts[3]
              });
            }
          }
          resolve(null);
        } else {
          reject(new Error(`获取压缩信息失败，退出码: ${code}`));
        }
      });
    });
  }
}

// 使用示例
async function compressionExample() {
  try {
    // 创建测试文件
    const testFile = 'test-large-file.txt';
    const compressedFile = testFile + '.gz';
    const decompressedFile = 'test-decompressed.txt';

    // 创建大文件
    const largeContent = 'Hello, World! '.repeat(1000000);
    fs.writeFileSync(testFile, largeContent);

    // 压缩文件
    await FileCompressor.compressFile(testFile, compressedFile);

    // 获取压缩信息
    const info = await FileCompressor.getFileCompressionInfo(compressedFile);
    console.log('压缩信息:', info);

    // 解压文件
    await FileCompressor.decompressFile(compressedFile, decompressedFile);

    // 验证内容
    const original = fs.readFileSync(testFile, 'utf8');
    const decompressed = fs.readFileSync(decompressedFile, 'utf8');
    console.log('内容匹配:', original === decompressed);

    // 清理测试文件
    fs.unlinkSync(testFile);
    fs.unlinkSync(compressedFile);
    fs.unlinkSync(decompressedFile);

  } catch (error) {
    console.error('压缩/解压示例失败:', error);
  }
}

// 目录压缩示例
async function directoryCompressionExample() {
  try {
    // 创建测试目录
    const testDir = 'test-directory';
    const compressedDir = testDir + '.tar.gz';
    const outputDir = 'output-directory';

    if (!fs.existsSync(testDir)) {
      fs.mkdirSync(testDir);
      fs.writeFileSync(path.join(testDir, 'file1.txt'), 'Content 1');
      fs.writeFileSync(path.join(testDir, 'file2.txt'), 'Content 2');
    }

    // 压缩目录
    await FileCompressor.compressDirectory(testDir, compressedDir);

    // 解压目录
    if (!fs.existsSync(outputDir)) {
      fs.mkdirSync(outputDir);
    }
    await FileCompressor.decompressDirectory(compressedDir, outputDir);

    console.log('目录压缩/解压完成');

  } catch (error) {
    console.error('目录压缩示例失败:', error);
  }
}

// 运行示例
compressionExample();
directoryCompressionExample();
```

### 2. Git 操作工具

```javascript
const { exec } = require('child_process');
const { promisify } = require('util');

const execAsync = promisify(exec);

class GitOperations {
  constructor(repoPath = process.cwd()) {
    this.repoPath = repoPath;
  }

  async executeGitCommand(command, options = {}) {
    try {
      const { stdout, stderr } = await execAsync(command, {
        cwd: this.repoPath,
        env: {
          ...process.env,
          GIT_AUTHOR_NAME: 'Git Bot',
          GIT_AUTHOR_EMAIL: 'gitbot@example.com'
        },
        ...options
      });

      // 处理警告信息
      if (stderr && stderr.includes('warning')) {
        console.warn('Git 警告:', stderr);
      }

      return stdout.trim();
    } catch (error) {
      if (error.stdout) {
        return error.stdout.trim();
      }
      throw new Error(`Git 命令执行失败: ${error.message}`);
    }
  }

  // 状态检查
  async getStatus(format = 'short') {
    return this.executeGitCommand(`git status --${format}`);
  }

  async getCurrentBranch() {
    return this.executeGitCommand('git rev-parse --abbrev-ref HEAD');
  }

  async getCurrentCommit() {
    return this.executeGitCommand('git rev-parse HEAD');
  }

  async getCurrentCommitMessage() {
    return this.executeGitCommand('git log -1 --pretty=%B');
  }

  async getRemoteBranches() {
    const branches = await this.executeGitCommand('git branch -r');
    return branches.split('\n').map(branch => branch.trim());
  }

  // 分支操作
  async getLocalBranches() {
    const branches = await this.executeGitCommand('git branch');
    return branches.split('\n').map(branch => branch.replace('*', '').trim());
  }

  async createBranch(branchName, checkout = true) {
    if (checkout) {
      return this.executeGitCommand(`git checkout -b ${branchName}`);
    } else {
      return this.executeGitCommand(`git branch ${branchName}`);
    }
  }

  async switchBranch(branchName) {
    return this.executeGitCommand(`git checkout ${branchName}`);
  }

  async deleteBranch(branchName, force = false) {
    const flag = force ? '-D' : '-d';
    return this.executeGitCommand(`git branch ${flag} ${branchName}`);
  }

  // 提交操作
  async getStagedFiles() {
    const files = await this.executeGitCommand('git diff --cached --name-only');
    return files.split('\n').filter(file => file.trim());
  }

  async getModifiedFiles() {
    const files = await this.executeGitCommand('git diff --name-only');
    return files.split('\n').filter(file => file.trim());
  }

  async addFiles(files) {
    const fileList = Array.isArray(files) ? files.join(' ') : files;
    return this.executeGitCommand(`git add ${fileList}`);
  }

  async addAll() {
    return this.executeGitCommand('git add -A');
  }

  async commit(message, options = {}) {
    const { allowEmpty = false, noVerify = false } = options;
    let command = 'git commit -m';

    if (allowEmpty) command += ' --allow-empty';
    if (noVerify) command += ' --no-verify';

    return this.executeGitCommand(`${command} "${message}"`);
  }

  async amendCommit(message) {
    return this.executeGitCommand(`git commit --amend -m "${message}"`);
  }

  // 远程操作
  async getRemoteUrl(remote = 'origin') {
    return this.executeGitCommand(`git remote get-url ${remote}`);
  }

  async pull(remote = 'origin', branch = null) {
    const target = branch ? `${remote} ${branch}` : remote;
    return this.executeGitCommand(`git pull ${target}`);
  }

  async push(remote = 'origin', branch = null, force = false) {
    const target = branch ? `${remote} ${branch}` : remote;
    const forceFlag = force ? '--force' : '';
    return this.executeGitCommand(`git push ${forceFlag} ${target}`);
  }

  async fetch(remote = 'origin') {
    return this.executeGitCommand(`git fetch ${remote}`);
  }

  // 日志操作
  async getLog(limit = 10, format = '%h - %s (%an, %ar)') {
    return this.executeGitCommand(`git log -${limit} --pretty=format:"${format}"`);
  }

  async getFileHistory(filePath, limit = 10) {
    return this.executeGitCommand(`git log -${limit} --pretty=format:"%h %s" -- ${filePath}`);
  }

  async getFileDiff(filePath, commit1, commit2 = null) {
    if (commit2) {
      return this.executeGitCommand(`git diff ${commit1} ${commit2} ${filePath}`);
    } else {
      return this.executeGitCommand(`git diff ${commit1} ${filePath}`);
    }
  }

  async getStagedDiff() {
    return this.executeGitCommand('git diff --cached');
  }

  // 仓库操作
  async clone(repoUrl, targetDir, options = {}) {
    const { branch = null, depth = null } = options;
    let command = `git clone ${repoUrl} ${targetDir}`;

    if (branch) command += ` -b ${branch}`;
    if (depth) command += ` --depth ${depth}`;

    return this.executeGitCommand(command);
  }

  async isRepository() {
    try {
      await this.executeGitCommand('git rev-parse --is-inside-work-tree');
      return true;
    } catch (error) {
      return false;
    }
  }

  async getRepositoryInfo() {
    const [branch, commit, remoteUrl] = await Promise.all([
      this.getCurrentBranch().catch(() => 'unknown'),
      this.getCurrentCommit().catch(() => 'unknown'),
      this.getRemoteUrl().catch(() => 'unknown')
    ]);

    return {
      path: this.repoPath,
      branch,
      commit,
      remoteUrl,
      isRepository: await this.isRepository()
    };
  }

  // 标签操作
  async getTags() {
    const tags = await this.executeGitCommand('git tag');
    return tags.split('\n').filter(tag => tag.trim());
  }

  async createTag(tagName, message, annotated = true) {
    if (annotated) {
      return this.executeGitCommand(`git tag -a ${tagName} -m "${message}"`);
    } else {
      return this.executeGitCommand(`git tag ${tagName}`);
    }
  }

  async deleteTag(tagName) {
    return this.executeGitCommand(`git tag -d ${tagName}`);
  }

  async pushTags(remote = 'origin') {
    return this.executeGitCommand(`git push ${remote} --tags`);
  }

  // 合并操作
  async merge(branch, options = {}) {
    const { noCommit = false, noFastForward = false, squash = false } = options;
    let command = 'git merge';

    if (noCommit) command += ' --no-commit';
    if (noFastForward) command += ' --no-ff';
    if (squash) command += ' --squash';

    return this.executeGitCommand(`${command} ${branch}`);
  }

  async abortMerge() {
    return this.executeGitCommand('git merge --abort');
  }

  async continueMerge() {
    return this.executeGitCommand('git merge --continue');
  }
}

// 使用示例
async function gitExample() {
  try {
    const git = new GitOperations('/path/to/your/repo');

    // 检查仓库信息
    const repoInfo = await git.getRepositoryInfo();
    console.log('仓库信息:', repoInfo);

    // 获取状态
    const status = await git.getStatus();
    console.log('Git 状态:', status);

    // 获取当前分支
    const branch = await git.getCurrentBranch();
    console.log('当前分支:', branch);

    // 获取日志
    const log = await git.getLog(5);
    console.log('最近提交:');
    console.log(log);

    // 检查修改的文件
    const modifiedFiles = await git.getModifiedFiles();
    console.log('修改的文件:', modifiedFiles);

    if (modifiedFiles.length > 0) {
      // 添加文件并提交
      await git.addFiles(modifiedFiles);
      await git.commit('自动提交的更改');
      console.log('文件已提交');

      // 推送更改
      // await git.push('origin', branch);
    }

  } catch (error) {
    console.error('Git 操作失败:', error);
  }
}

gitExample();
```

### 3. 进程池管理器

```javascript
const { spawn } = require('child_process');
const EventEmitter = require('events');

class ProcessPool extends EventEmitter {
  constructor(options = {}) {
    super();
    this.maxProcesses = options.maxProcesses || 4;
    this.processes = [];
    this.queue = [];
    this.activeProcesses = 0;
    this.options = {
      timeout: options.timeout || 30000,
      retries: options.retries || 3,
      ...options
    };
  }

  execute(command, args, options = {}) {
    return new Promise((resolve, reject) => {
      const task = {
        command,
        args,
        options: { ...this.options, ...options },
        resolve,
        reject,
        attempts: 0
      };
      this.queue.push(task);
      this.processQueue();
    });
  }

  processQueue() {
    while (this.activeProcesses < this.maxProcesses && this.queue.length > 0) {
      const task = this.queue.shift();
      this.runTask(task);
    }
  }

  runTask(task) {
    const { command, args, options, resolve, reject, attempts } = task;
    this.activeProcesses++;

    const process = spawn(command, args, options);
    let stdout = '';
    let stderr = '';
    let timeoutId = null;

    // 设置超时
    if (options.timeout > 0) {
      timeoutId = setTimeout(() => {
        process.kill();
        reject(new Error(`命令执行超时 (${options.timeout}ms)`));
      }, options.timeout);
    }

    process.stdout.on('data', (data) => {
      stdout += data.toString();
      this.emit('stdout', data);
    });

    process.stderr.on('data', (data) => {
      stderr += data.toString();
      this.emit('stderr', data);
    });

    process.on('close', (code, signal) => {
      this.activeProcesses--;
      if (timeoutId) clearTimeout(timeoutId);

      if (code === 0) {
        resolve({ stdout, stderr, code, signal });
      } else {
        // 重试逻辑
        if (attempts < this.options.retries) {
          console.log(`命令执行失败，重试 (${attempts + 1}/${this.options.retries})`);
          task.attempts++;
          this.queue.unshift(task);
        } else {
          reject(new Error(`进程退出，退出码: ${code}, 信号: ${signal}\n${stderr}`));
        }
      }

      this.processQueue();
    });

    process.on('error', (error) => {
      this.activeProcesses--;
      if (timeoutId) clearTimeout(timeoutId);
      reject(error);
      this.processQueue();
    });

    this.processes.push({
      pid: process.pid,
      process,
      command,
      args,
      startTime: Date.now(),
      options
    });

    this.emit('spawn', process);
  }

  getActiveProcesses() {
    return this.processes.filter(p => !p.process.killed);
  }

  killProcess(pid, signal = 'SIGTERM') {
    const processInfo = this.processes.find(p => p.pid === pid);
    if (processInfo && !processInfo.process.killed) {
      processInfo.process.kill(signal);
      return true;
    }
    return false;
  }

  killAll(signal = 'SIGTERM') {
    let killedCount = 0;
    this.processes.forEach(({ process }) => {
      if (!process.killed) {
        process.kill(signal);
        killedCount++;
      }
    });
    return killedCount;
  }

  getStats() {
    const activeProcesses = this.getActiveProcesses();
    const now = Date.now();

    return {
      maxProcesses: this.maxProcesses,
      activeProcesses: this.activeProcesses,
      queueLength: this.queue.length,
      processes: activeProcesses.map(p => ({
        pid: p.pid,
        command: p.command,
        runtime: now - p.startTime,
        options: p.options
      }))
    };
  }

  async executeBatch(commands) {
    const promises = commands.map(({ command, args, options }) =>
      this.execute(command, args, options)
    );
    return Promise.all(promises);
  }

  async executeParallel(commands) {
    return this.executeBatch(commands);
  }

  async executeSequential(commands) {
    const results = [];
    for (const { command, args, options } of commands) {
      try {
        const result = await this.execute(command, args, options);
        results.push({ success: true, result });
      } catch (error) {
        results.push({ success: false, error });
      }
    }
    return results;
  }
}

// 使用示例
const pool = new ProcessPool({
  maxProcesses: 3,
  timeout: 5000,
  retries: 2
});

// 监听事件
pool.on('spawn', (process) => {
  console.log(`进程启动: PID ${process.pid}`);
});

pool.on('stdout', (data) => {
  console.log('输出:', data.toString().trim());
});

pool.on('stderr', (data) => {
  console.error('错误:', data.toString().trim());
});

async function runProcessPoolExample() {
  try {
    // 并行执行多个命令
    const commands = [
      { command: 'node', args: ['--version'] },
      { command: 'npm', args: ['--version'] },
      { command: 'git', args: ['--version'] },
      { command: 'node', args: ['-e', 'console.log("Hello")'] }
    ];

    console.log('开始并行执行命令...');
    const results = await pool.executeBatch(commands);

    console.log('\n执行结果:');
    results.forEach((result, index) => {
      if (result.success) {
        console.log(`命令 ${index + 1} 成功:`, result.result.stdout.trim());
      } else {
        console.error(`命令 ${index + 1} 失败:`, result.error.message);
      }
    });

    // 查看统计信息
    console.log('\n进程池统计:', pool.getStats());

  } catch (error) {
    console.error('进程池执行失败:', error);
  }
}

// 顺序执行示例
async function runSequentialExample() {
  try {
    const commands = [
      { command: 'echo', args: ['Step 1'] },
      { command: 'sleep', args: ['1'] },
      { command: 'echo', args: ['Step 2'] },
      { command: 'sleep', args: ['1'] },
      { command: 'echo', args: ['Step 3'] }
    ];

    console.log('开始顺序执行命令...');
    const results = await pool.executeSequential(commands);

    console.log('\n执行结果:');
    results.forEach((result, index) => {
      if (result.success) {
        console.log(`步骤 ${index + 1} 成功`);
      } else {
        console.error(`步骤 ${index + 1} 失败:`, result.error.message);
      }
    });

  } catch (error) {
    console.error('顺序执行失败:', error);
  }
}

// 运行示例
runProcessPoolExample().then(() => {
  console.log('\n\n顺序执行示例:');
  return runSequentialExample();
});
```

### 4. 文件转换服务

```javascript
const { spawn } = require('child_process');
const path = require('path');
const fs = require('fs');

class FileConverter {
  static convertImage(inputPath, outputPath, format = 'png', options = {}) {
    return new Promise((resolve, reject) => {
      const args = [inputPath];

      // 添加转换选项
      if (options.resize) {
        args.push('-resize', options.resize);
      }

      if (options.quality) {
        args.push('-quality', options.quality.toString());
      }

      args.push(path.format(outputPath, format));

      const magick = spawn('convert', args);

      magick.on('error', reject);
      magick.on('close', (code) => {
        if (code === 0) {
          resolve(outputPath);
        } else {
          reject(new Error(`图片转换失败，退出码: ${code}`));
        }
      });
    });
  }

  static convertVideo(inputPath, outputPath, format = 'mp4', options = {}) {
    return new Promise((resolve, reject) => {
      const args = ['-i', inputPath];

      // 添加转换选项
      if (options.codec) {
        args.push('-c:v', options.codec);
      }

      if (options.bitrate) {
        args.push('-b:v', options.bitrate);
      }

      if (options.resolution) {
        args.push('-s', options.resolution);
      }

      if (options.audioCodec) {
        args.push('-c:a', options.audioCodec);
      }

      args.push(outputPath);

      const ffmpeg = spawn('ffmpeg', args);

      ffmpeg.on('error', reject);
      ffmpeg.on('close', (code) => {
        if (code === 0) {
          resolve(outputPath);
        } else {
          reject(new Error(`视频转换失败，退出码: ${code}`));
        }
      });
    });
  }

  static extractAudio(inputPath, outputPath, format = 'mp3', options = {}) {
    return new Promise((resolve, reject) => {
      const args = ['-i', inputPath, '-vn'];

      if (options.bitrate) {
        args.push('-ab', options.bitrate);
      }

      args.push(outputPath);

      const ffmpeg = spawn('ffmpeg', args);

      ffmpeg.on('error', reject);
      ffmpeg.on('close', (code) => {
        if (code === 0) {
          resolve(outputPath);
        } else {
          reject(new Error(`音频提取失败，退出码: ${code}`));
        }
      });
    });
  }

  static convertPDF(inputPath, outputPath, options = {}) {
    return new Promise((resolve, reject) => {
      const args = [inputPath, outputPath];

      const libreoffice = spawn('libreoffice', ['--headless', '--convert-to', 'pdf', ...args]);

      libreoffice.on('error', reject);
      libreoffice.on('close', (code) => {
        if (code === 0) {
          resolve(outputPath);
        } else {
          reject(new Error(`PDF转换失败，退出码: ${code}`));
        }
      });
    });
  }

  static convertMarkdown(inputPath, outputPath, format = 'html', options = {}) {
    return new Promise((resolve, reject) => {
      const pandoc = spawn('pandoc', [
        '-f', 'markdown',
        '-t', format,
        '-o', outputPath,
        inputPath
      ]);

      pandoc.on('error', reject);
      pandoc.on('close', (code) => {
        if (code === 0) {
          resolve(outputPath);
        } else {
          reject(new Error(`Markdown转换失败，退出码: ${code}`));
        }
      });
    });
  }
}

// 使用示例
async function fileConversionExample() {
  try {
    // 图片转换
    // await FileConverter.convertImage('input.jpg', 'output.png', 'png', {
    //   resize: '800x600',
    //   quality: 90
    // });

    // 视频转换
    // await FileConverter.convertVideo('input.avi', 'output.mp4', 'mp4', {
    //   codec: 'libx264',
    //   bitrate: '1000k',
    //   resolution: '1280x720'
    // });

    // 音频提取
    // await FileConverter.extractAudio('video.mp4', 'audio.mp3', 'mp3', {
    //   bitrate: '192k'
    // });

    // Markdown 转换
    // await FileConverter.convertMarkdown('README.md', 'README.html', 'html');

    console.log('文件转换示例完成（需要实际文件）');

  } catch (error) {
    console.error('文件转换失败:', error);
  }
}

fileConversionExample();
```

## 注意事项

### 1. 资源管理

子进程需要正确管理资源，避免资源泄漏和僵尸进程：

```javascript
const { spawn } = require('child_process');

class ProcessManager {
  constructor() {
    this.processes = new Map();
  }

  createProcess(command, args, options = {}) {
    const process = spawn(command, args, options);

    this.processes.set(process.pid, process);

    // 监听进程退出
    process.on('exit', (code, signal) => {
      console.log(`进程 ${process.pid} 退出，退出码: ${code}, 信号: ${signal}`);
      this.processes.delete(process.pid);
    });

    process.on('error', (error) => {
      console.error(`进程 ${process.pid} 错误:`, error);
      this.processes.delete(process.pid);
    });

    return process;
  }

  cleanup() {
    // 杀死所有进程
    this.processes.forEach((process, pid) => {
      console.log(`清理进程 ${pid}`);
      process.kill('SIGTERM');
    });
    this.processes.clear();
  }
}

// 使用示例
const manager = new ProcessManager();

// 主进程退出时清理
process.on('exit', () => {
  manager.cleanup();
});

// 处理信号
process.on('SIGTERM', () => {
  console.log('收到 SIGTERM 信号，开始清理...');
  manager.cleanup();
  process.exit(0);
});

process.on('SIGINT', () => {
  console.log('收到 SIGINT 信号，开始清理...');
  manager.cleanup();
  process.exit(0);
});
```

### 2. 安全性考虑

避免命令注入攻击，不要直接拼接用户输入到命令中：

```javascript
const { spawn, exec } = require('child_process');

// ❌ 不安全：命令注入风险
function unsafeExecute(userInput) {
  exec(`echo ${userInput}`, (error, stdout, stderr) => {
    if (error) {
      console.error('执行失败:', error);
      return;
    }
    console.log('输出:', stdout);
  });
}

// 用户输入: '; rm -rf /' 会删除根目录

// ✅ 安全：使用参数数组
function safeExecute(userInput) {
  const args = [userInput];
  const process = spawn('echo', args);

  process.stdout.on('data', (data) => {
    console.log('输出:', data.toString());
  });

  process.on('error', (error) => {
    console.error('执行失败:', error);
  });
}

// ✅ 安全：使用 execFile
function safeExecFile(userInput) {
  execFile('echo', [userInput], (error, stdout, stderr) => {
    if (error) {
      console.error('执行失败:', error);
      return;
    }
    console.log('输出:', stdout);
  });
}

// ✅ 安全：参数验证和清理
function sanitizeInput(input) {
  // 移除危险字符
  return input.replace(/[;&|`$()]/g, '');
}

function safeExecuteWithSanitization(userInput) {
  const safeInput = sanitizeInput(userInput);
  exec(`echo ${safeInput}`, (error, stdout, stderr) => {
    if (error) {
      console.error('执行失败:', error);
      return;
    }
    console.log('输出:', stdout);
  });
}
```

### 3. 错误处理

正确处理子进程的各种错误情况：

```javascript
const { spawn } = require('child_process');

class RobustProcessExecutor {
  async executeWithRetry(command, args, options = {}) {
    const {
      maxRetries = 3,
      retryDelay = 1000,
      timeout = 5000
    } = options;

    let lastError;

    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        return await this.executeWithTimeout(command, args, { ...options, timeout });
      } catch (error) {
        lastError = error;
        console.error(`执行失败 (尝试 ${attempt}/${maxRetries}):`, error.message);

        if (attempt < maxRetries) {
          console.log(`等待 ${retryDelay}ms 后重试...`);
          await this.delay(retryDelay);
        }
      }
    }

    throw new Error(`命令执行失败，已重试 ${maxRetries} 次: ${lastError.message}`);
  }

  executeWithTimeout(command, args, options = {}) {
    return new Promise((resolve, reject) => {
      const { timeout } = options;
      const process = spawn(command, args, options);

      let stdout = '';
      let stderr = '';
      let timeoutId = null;

      // 设置超时
      if (timeout > 0) {
        timeoutId = setTimeout(() => {
          process.kill('SIGTERM');
          reject(new Error(`命令执行超时 (${timeout}ms)`));
        }, timeout);
      }

      process.stdout.on('data', (data) => {
        stdout += data.toString();
      });

      process.stderr.on('data', (data) => {
        stderr += data.toString();
      });

      process.on('close', (code, signal) => {
        if (timeoutId) clearTimeout(timeoutId);

        if (code === 0) {
          resolve({ stdout, stderr, code, signal });
        } else if (signal === 'SIGTERM') {
          // 超时错误已经处理
          return;
        } else {
          reject(new Error(`进程退出，退出码: ${code}\n${stderr}`));
        }
      });

      process.on('error', (error) => {
        if (timeoutId) clearTimeout(timeoutId);
        reject(new Error(`进程启动失败: ${error.message}`));
      });
    });
  }

  delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// 使用示例
const executor = new RobustProcessExecutor();

executor.executeWithRetry('npm', ['install'], {
  maxRetries: 3,
  retryDelay: 2000,
  timeout: 60000
})
.then(result => console.log('安装成功'))
.catch(error => console.error('安装失败:', error));
```

### 4. 性能优化

合理使用进程池，避免频繁创建和销毁进程：

```javascript
const { spawn } = require('child_process');

class OptimizedProcessPool {
  constructor(options = {}) {
    this.maxProcesses = options.maxProcesses || 4;
    this.minProcesses = options.minProcesses || 1;
    this.idleTimeout = options.idleTimeout || 30000; // 30秒

    this.processes = [];
    this.queue = [];
    this.activeCount = 0;
    this.processCreationTime = new Map();
  }

  async execute(command, args, options = {}) {
    return new Promise((resolve, reject) => {
      this.queue.push({ command, args, options, resolve, reject });
      this.processQueue();
    });
  }

  processQueue() {
    if (this.queue.length === 0) {
      this.checkIdleProcesses();
      return;
    }

    const availableProcess = this.processes.find(p => !p.busy);
    if (!availableProcess && this.processes.length < this.maxProcesses) {
      // 创建新进程
      this.createNewProcess().then(process => {
        this.assignTask(process);
      });
    } else if (availableProcess) {
      this.assignTask(availableProcess);
    }
  }

  async createNewProcess() {
    const task = this.queue[0];
    const { command, args, options } = task;

    return new Promise((resolve, reject) => {
      const process = spawn(command, args, options);
      const processInfo = {
        process,
        busy: true,
        createdAt: Date.now()
      };

      this.processes.push(processInfo);
      this.activeCount++;
      this.processCreationTime.set(process.pid, Date.now());

      process.on('error', (error) => {
        this.removeProcess(processInfo);
        reject(error);
      });

      process.on('exit', () => {
        this.removeProcess(processInfo);
      });

      resolve(processInfo);
    });
  }

  assignTask(processInfo) {
    if (this.queue.length === 0) {
      processInfo.busy = false;
      return;
    }

    const task = this.queue.shift();
    const { command, args, options, resolve, reject } = task;

    // 如果进程已关闭，重新创建
    if (processInfo.process.killed) {
      this.createNewProcess().then(newProcessInfo => {
        this.assignTask(newProcessInfo);
      });
      return;
    }

    processInfo.busy = true;
    let stdout = '';
    let stderr = '';

    processInfo.process.stdout.on('data', (data) => {
      stdout += data.toString();
    });

    processInfo.process.stderr.on('data', (data) => {
      stderr += data.toString();
    });

    processInfo.process.on('close', (code) => {
      processInfo.busy = false;
      this.processQueue();

      if (code === 0) {
        resolve({ stdout, stderr, code });
      } else {
        reject(new Error(`进程退出，退出码: ${code}\n${stderr}`));
      }
    });
  }

  removeProcess(processInfo) {
    const index = this.processes.indexOf(processInfo);
    if (index !== -1) {
      this.processes.splice(index, 1);
      this.processCreationTime.delete(processInfo.process.pid);
    }
  }

  checkIdleProcesses() {
    const now = Date.now();
    const idleProcesses = this.processes.filter(p =>
      !p.busy && (now - p.createdAt) > this.idleTimeout
    );

    idleProcesses.forEach(processInfo => {
      console.log(`关闭空闲进程 ${processInfo.process.pid}`);
      processInfo.process.kill();
    });
  }

  getStats() {
    return {
      totalProcesses: this.processes.length,
      activeProcesses: this.processes.filter(p => p.busy).length,
      idleProcesses: this.processes.filter(p => !p.busy).length,
      queueLength: this.queue.length,
      maxProcesses: this.maxProcesses,
      minProcesses: this.minProcesses
    };
  }

  shutdown() {
    return Promise.all(
      this.processes.map(processInfo => {
        return new Promise(resolve => {
          processInfo.process.on('exit', resolve);
          processInfo.process.kill('SIGTERM');
        });
      })
    );
  }
}

// 使用示例
const optimizedPool = new OptimizedProcessPool({
  maxProcesses: 4,
  minProcesses: 1,
  idleTimeout: 30000
});

// 执行任务
for (let i = 0; i < 10; i++) {
  optimizedPool.execute('echo', [`Task ${i}`])
    .then(result => console.log('任务完成:', result.stdout.trim()))
    .catch(error => console.error('任务失败:', error.message));
}

// 定期查看统计信息
setInterval(() => {
  console.log('进程池统计:', optimizedPool.getStats());
}, 5000);
```

### 5. 平台兼容性

处理不同操作系统间的差异：

```javascript
const { spawn } = require('child_process');
const os = require('os');
const path = require('path');

class CrossPlatformExecutor {
  constructor() {
    this.platform = os.platform();
    this.isWindows = this.platform === 'win32';
    this.isUnix = ['darwin', 'linux', 'freebsd', 'openbsd'].includes(this.platform);
  }

  execute(command, args, options = {}) {
    const resolvedCommand = this.resolveCommand(command);
    const resolvedArgs = this.resolveArgs(args);
    const resolvedOptions = this.resolveOptions(options);

    return this.spawnProcess(resolvedCommand, resolvedArgs, resolvedOptions);
  }

  resolveCommand(command) {
    // Windows 下可能需要使用完整路径或命令扩展
    if (this.isWindows) {
      const commonWindowsCommands = ['node', 'npm', 'git', 'python', 'python3'];
      if (commonWindowsCommands.includes(command)) {
        return command + '.exe';
      }
    }
    return command;
  }

  resolveArgs(args) {
    if (this.isWindows) {
      return args.map(arg => {
        // Windows 参数中的特殊字符需要转义
        if (arg.includes(' ') || arg.includes('&') || arg.includes('|')) {
          return `"${arg}"`;
        }
        return arg;
      });
    }
    return args;
  }

  resolveOptions(options = {}) {
    const resolvedOptions = { ...options };

    // Windows 下可能需要设置 shell
    if (this.isWindows && !resolvedOptions.shell) {
      resolvedOptions.shell = true;
    }

    // 设置路径分隔符
    if (resolvedOptions.cwd) {
      resolvedOptions.cwd = resolvedOptions.cwd.replace(/\//g, path.sep);
    }

    return resolvedOptions;
  }

  spawnProcess(command, args, options) {
    return new Promise((resolve, reject) => {
      const process = spawn(command, args, options);

      let stdout = '';
      let stderr = '';

      process.stdout.on('data', (data) => {
        stdout += data.toString();
      });

      process.stderr.on('data', (data) => {
        stderr += data.toString();
      });

      process.on('close', (code) => {
        if (code === 0) {
          resolve({ stdout, stderr, code });
        } else {
          reject(new Error(`进程退出，退出码: ${code}\n${stderr}`));
        }
      });

      process.on('error', (error) => {
        reject(new Error(`进程启动失败: ${error.message}`));
      });
    });
  }

  // 跨平台的环境变量设置
  getEnvironmentVariables(customEnv = {}) {
    const baseEnv = { ...process.env };

    // 添加平台特定的环境变量
    if (this.isWindows) {
      baseEnv.PATH = baseEnv.Path || baseEnv.PATH;
    } else {
      baseEnv.PATH = baseEnv.PATH;
    }

    return { ...baseEnv, ...customEnv };
  }

  // 跨平台的临时目录
  getTempDir() {
    return os.tmpdir();
  }

  // 跨平台的用户目录
  getUserHome() {
    return os.homedir();
  }
}

// 使用示例
const executor = new CrossPlatformExecutor();

executor.execute('node', ['--version'])
  .then(result => console.log('Node.js 版本:', result.stdout.trim()))
  .catch(error => console.error('执行失败:', error.message));

// 获取平台信息
console.log('运行平台:', executor.platform);
console.log('是否为 Windows:', executor.isWindows);
console.log('是否为 Unix:', executor.isUnix);
```

## 最佳实践

### 1. 选择合适的方法

根据具体场景选择合适的子进程创建方法：

```javascript
const { spawn, exec, execFile, fork } = require('child_process');

// 场景1: 执行简单命令，需要完整输出
exec('ls -la', (error, stdout, stderr) => {
  if (error) {
    console.error('执行失败:', error);
    return;
  }
  console.log(stdout);
});

// 场景2: 执行长时间运行的任务，需要实时处理输出
const process = spawn('tail', ['-f', '/var/log/app.log']);

process.stdout.on('data', (data) => {
  console.log('日志:', data.toString());
});

// 场景3: 执行已知程序，避免 shell 注入
execFile('node', ['--version'], (error, stdout, stderr) => {
  if (error) {
    console.error('执行失败:', error);
    return;
  }
  console.log('Node.js 版本:', stdout.trim());
});

// 场景4: Node.js 模块间通信
const worker = fork('./worker.js');

worker.send({ task: 'heavyCalculation' });

worker.on('message', (result) => {
  console.log('计算结果:', result);
});
```

### 2. 资源清理

确保子进程资源得到正确清理：

```javascript
class ManagedProcess {
  constructor(command, args, options = {}) {
    this.command = command;
    this.args = args;
    this.options = options;
    this.process = null;
    this.isRunning = false;
  }

  start() {
    if (this.isRunning) {
      throw new Error('进程已在运行中');
    }

    this.process = spawn(this.command, this.args, this.options);
    this.isRunning = true;

    // 监听进程退出
    this.process.on('exit', (code, signal) => {
      this.isRunning = false;
      console.log(`进程退出，退出码: ${code}, 信号: ${signal}`);
    });

    this.process.on('error', (error) => {
      this.isRunning = false;
      console.error('进程错误:', error);
    });

    return this.process;
  }

  stop(signal = 'SIGTERM') {
    if (!this.isRunning || !this.process) {
      return false;
    }

    this.process.kill(signal);
    return true;
  }

  isAlive() {
    return this.isRunning && this.process && !this.process.killed;
  }

  getPID() {
    return this.process ? this.process.pid : null;
  }
}

// 使用示例
const managedProcess = new ManagedProcess('node', ['long-running.js']);

managedProcess.start();

// 优雅关闭
process.on('SIGTERM', () => {
  managedProcess.stop();
  setTimeout(() => {
    if (managedProcess.isAlive()) {
      managedProcess.stop('SIGKILL');
    }
  }, 5000);
});
```

### 3. 错误处理和重试

实现完善的错误处理和重试机制：

```javascript
class ResilientProcessExecutor {
  async execute(command, args, options = {}) {
    const {
      maxRetries = 3,
      retryDelay = 1000,
      timeout = 30000,
      backoff = true
    } = options;

    let lastError;

    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        return await this.executeOnce(command, args, { ...options, timeout });
      } catch (error) {
        lastError = error;

        if (attempt < maxRetries) {
          const delay = backoff ? retryDelay * attempt : retryDelay;
          console.log(`执行失败 (尝试 ${attempt}/${maxRetries}), ${delay}ms 后重试...`);
          await this.sleep(delay);
        }
      }
    }

    throw new Error(`命令执行失败，已重试 ${maxRetries} 次: ${lastError.message}`);
  }

  async executeOnce(command, args, options = {}) {
    return new Promise((resolve, reject) => {
      const { timeout } = options;
      const process = spawn(command, args, options);

      let stdout = '';
      let stderr = '';
      let timeoutId = null;
      let timedOut = false;

      if (timeout > 0) {
        timeoutId = setTimeout(() => {
          timedOut = true;
          process.kill('SIGTERM');
        }, timeout);
      }

      process.stdout.on('data', (data) => {
        stdout += data.toString();
      });

      process.stderr.on('data', (data) => {
        stderr += data.toString();
      });

      process.on('close', (code) => {
        if (timeoutId) clearTimeout(timeoutId);

        if (timedOut) {
          reject(new Error(`命令执行超时 (${timeout}ms)`));
        } else if (code === 0) {
          resolve({ stdout, stderr, code });
        } else {
          reject(new Error(`进程退出，退出码: ${code}\n${stderr}`));
        }
      });

      process.on('error', (error) => {
        if (timeoutId) clearTimeout(timeoutId);
        reject(new Error(`进程启动失败: ${error.message}`));
      });
    });
  }

  sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// 使用示例
const executor = new ResilientProcessExecutor();

executor.execute('npm', ['install'], {
  maxRetries: 3,
  retryDelay: 2000,
  timeout: 60000,
  backoff: true
})
.then(result => console.log('安装成功'))
.catch(error => console.error('安装失败:', error));
```

### 4. 环境隔离

使用独立的环境变量和权限：

```javascript
const { spawn } = require('child_process');

class SecureProcessExecutor {
  execute(command, args, options = {}) {
    const secureOptions = this.secureOptions(options);

    return new Promise((resolve, reject) => {
      const process = spawn(command, args, secureOptions);

      let stdout = '';
      let stderr = '';

      process.stdout.on('data', (data) => {
        stdout += data.toString();
      });

      process.stderr.on('data', (data) => {
        stderr += data.toString();
      });

      process.on('close', (code) => {
        if (code === 0) {
          resolve({ stdout, stderr, code });
        } else {
          reject(new Error(`进程退出，退出码: ${code}\n${stderr}`));
        }
      });

      process.on('error', (error) => {
        reject(new Error(`进程启动失败: ${error.message}`));
      });
    });
  }

  secureOptions(options = {}) {
    const baseEnv = {
      PATH: process.env.PATH,
      HOME: process.env.HOME
    };

    // 只传递必要的环境变量
    const secureEnv = options.env ? { ...baseEnv, ...options.env } : baseEnv;

    // 移除敏感环境变量
    delete secureEnv.PASSWORD;
    delete secureEnv.API_KEY;
    delete secureEnv.SECRET;

    return {
      ...options,
      env: secureEnv,
      // 限制进程权限
      uid: options.uid || process.getuid(),
      gid: options.gid || process.getgid(),
      // 设置工作目录
      cwd: options.cwd || process.cwd(),
      // 设置资源限制
      // ulimit: options.ulimit
    };
  }

  // 创建沙盒环境
  executeInSandbox(command, args, sandboxOptions = {}) {
    const {
      memoryLimit = '512m',
      cpuLimit = 1,
      networkDisabled = false
    } = sandboxOptions;

    const options = {
      // Linux 下可以使用 cgroups 限制资源
      // 这里是示例，实际实现需要更复杂的配置
      env: {
        NODE_OPTIONS: `--max-old-space-size=${parseInt(memoryLimit)}`
      }
    };

    return this.execute(command, args, options);
  }
}

// 使用示例
const secureExecutor = new SecureProcessExecutor();

secureExecutor.execute('node', ['--version'], {
  env: {
    NODE_ENV: 'production'
  }
})
.then(result => console.log('版本:', result.stdout.trim()))
.catch(error => console.error('执行失败:', error));
```

## 总结

Node.js 的 `child_process` 模块提供了强大的子进程管理能力，是构建复杂应用的重要工具。通过本文的学习，我们掌握了：

1. **多种子进程创建方法**：spawn、exec、execFile、fork 等
2. **流式数据处理**：实时处理子进程的输出和输入
3. **进程间通信**：使用 IPC 机制进行父子进程通信
4. **进程生命周期管理**：启动、监控、停止和清理子进程
5. **错误处理和重试**：构建健壮的进程管理系统
6. **进程池和并发控制**：提高资源利用率和性能

通过合理使用这些功能，我们可以：
- 执行系统命令和外部程序
- 实现 CPU 密集型任务的并行处理
- 构建多进程的 Web 服务器
- 实现复杂的系统集成和工作流自动化
- 提高应用的稳定性和性能

在实际开发中，建议：
- 根据具体场景选择合适的方法
- 重视安全性和资源管理
- 实现完善的错误处理和重试机制
- 注意平台兼容性和性能优化
- 遵循进程管理的最佳实践

掌握 `child_process` 模块的使用对于开发高性能、高可用的 Node.js 应用程序至关重要。