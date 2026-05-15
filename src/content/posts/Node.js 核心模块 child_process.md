---
title: Node.js 核心模块 child_process
published: 2023-02-10
description: '子进程创建和管理的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 的 `child_process` 模块提供了创建子进程的能力，允许 Node.js 应用程序执行系统命令、运行脚本、与其他进程通信等。这是 Node.js 中实现多进程、任务并行化、系统集成的重要功能。

## 核心概念

`child_process` 模块提供多种创建子进程的方式：
- **spawn()**：创建子进程，以流的形式输出数据
- **exec()**：创建 shell 并执行命令，缓冲输出
- **execFile()**：直接执行文件，更安全
- **fork()**：创建 Node.js 子进程，支持 IPC 通信
- **spawnSync()**：同步版本的 spawn
- **execSync()**：同步版本的 exec

## 基本用法

### 1. spawn() - 流式输出

```javascript
const { spawn } = require('child_process');

// 创建子进程
const ls = spawn('ls', ['-la', '/']);

// 监听输出
ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

ls.stderr.on('data', (data) => {
  console.error(`stderr: ${data}`);
});

// 监听退出事件
ls.on('close', (code) => {
  console.log(`子进程退出，退出码: ${code}`);
});
```

#### 带选项的 spawn

```javascript
const { spawn } = require('child_process');

const process = spawn('npm', ['run', 'build'], {
  cwd: '/path/to/project',      // 工作目录
  env: { ...process.env, NODE_ENV: 'production' }, // 环境变量
  stdio: ['inherit', 'pipe', 'pipe'], // 标准输入/输出/错误
  shell: false,                  // 是否使用 shell
  detached: false,               // 是否分离进程
  uid: 1000,                     // 用户 ID
  gid: 1000                      // 组 ID
});

process.on('error', (error) => {
  console.error('进程启动失败:', error);
});
```

### 2. exec() - Shell 执行

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

#### 使用 Promise 封装

```javascript
const { exec } = require('child_process');
const { promisify } = require('util');

const execAsync = promisify(exec);

async function runCommand() {
  try {
    const { stdout, stderr } = await execAsync('node --version');
    console.log('Node.js 版本:', stdout.trim());
  } catch (error) {
    console.error('命令执行失败:', error);
  }
}

runCommand();
```

### 3. execFile() - 直接执行文件

```javascript
const { execFile } = require('child_process');

// 直接执行文件（比 exec 更安全）
execFile('node', ['--version'], (error, stdout, stderr) => {
  if (error) {
    console.error(`执行出错: ${error}`);
    return;
  }
  console.log('Node.js 版本:', stdout.trim());
});
```

#### 执行 Python 脚本

```javascript
const { execFile } = require('child_process');

execFile('python3', ['script.py', 'arg1', 'arg2'], {
  cwd: '/path/to/scripts',
  env: { PYTHONPATH: '/custom/path' }
}, (error, stdout, stderr) => {
  if (error) {
    console.error('脚本执行失败:', error);
    return;
  }
  console.log('脚本输出:', stdout);
});
```

### 4. fork() - Node.js 子进程

```javascript
// parent.js
const { fork } = require('child_process');

const child = fork('child.js');

// 发送消息给子进程
child.send({ message: 'Hello from parent' });

// 接收子进程的消息
child.on('message', (msg) => {
  console.log('来自子进程的消息:', msg);
});

// 监听子进程退出
child.on('exit', (code) => {
  console.log(`子进程退出，退出码: ${code}`);
});

// child.js
process.on('message', (msg) => {
  console.log('来自父进程的消息:', msg);

  // 发送响应
  process.send({ response: 'Hello from child', data: 'some data' });

  // 模拟长时间任务
  setTimeout(() => {
    process.exit(0);
  }, 1000);
});
```

### 5. 同步方法

```javascript
const { spawnSync, execSync } = require('child_process');

// 同步 spawn
const result = spawnSync('ls', ['-la', '/']);
console.log('状态码:', result.status);
console.log('输出:', result.stdout.toString());
console.log('错误:', result.stderr.toString());

// 同步 exec
try {
  const output = execSync('node --version');
  console.log('Node.js 版本:', output.toString().trim());
} catch (error) {
  console.error('命令执行失败:', error);
}
```

### 6. 管道操作

```javascript
const { spawn } = require('child_process');

// 创建多个进程
const grep = spawn('grep', ['text']);
const cat = spawn('cat', ['file.txt']);

// 连接进程管道
cat.stdout.pipe(grep.stdin);

// 输出结果
grep.stdout.on('data', (data) => {
  console.log(data.toString());
});

// 链式管道操作
const ps = spawn('ps', ['aux']);
const sort = spawn('sort', ['-nk3']); // 按第3列数字排序
const head = spawn('head', ['-n5']);  // 显示前5行

ps.stdout.pipe(sort.stdin);
sort.stdout.pipe(head.stdin);
head.stdout.on('data', (data) => {
  console.log(data.toString());
});
```

## 实际应用

### 1. 文件压缩工具

```javascript
const { spawn } = require('child_process');
const path = require('path');

class FileCompressor {
  static async compressFile(sourcePath, outputPath) {
    return new Promise((resolve, reject) => {
      const gzip = spawn('gzip', ['-c', sourcePath]);

      const fs = require('fs');
      const writeStream = fs.createWriteStream(outputPath);

      // 连接输出
      gzip.stdout.pipe(writeStream);

      // 处理错误
      gzip.on('error', reject);
      writeStream.on('error', reject);

      // 完成
      writeStream.on('finish', () => {
        console.log(`文件压缩完成: ${outputPath}`);
        resolve(outputPath);
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
      const gunzip = spawn('gunzip', ['-c', sourcePath]);

      const fs = require('fs');
      const writeStream = fs.createWriteStream(outputPath);

      gunzip.stdout.pipe(writeStream);

      gunzip.on('error', reject);
      writeStream.on('error', reject);

      writeStream.on('finish', () => {
        console.log(`文件解压完成: ${outputPath}`);
        resolve(outputPath);
      });

      gunzip.on('close', (code) => {
        if (code !== 0) {
          reject(new Error(`解压失败，退出码: ${code}`));
        }
      });
    });
  }
}

// 使用示例
FileCompressor.compressFile('large-file.txt', 'compressed-file.gz')
  .then(() => FileCompressor.decompressFile('compressed-file.gz', 'restored-file.txt'))
  .catch(console.error);
```

### 2. 并行任务处理器

```javascript
const { fork } = require('child_process');

class TaskProcessor {
  constructor(workerCount = 4) {
    this.workers = [];
    this.workerCount = workerCount;
    this.taskQueue = [];
    this.activeTasks = new Map();
  }

  initialize() {
    for (let i = 0; i < this.workerCount; i++) {
      const worker = fork('worker.js');

      worker.on('message', (msg) => {
        this.handleWorkerMessage(worker, msg);
      });

      worker.on('exit', (code) => {
        console.log(`Worker ${worker.pid} 退出，退出码: ${code}`);
        this.restartWorker(worker);
      });

      this.workers.push({
        worker,
        id: worker.pid,
        busy: false
      });
    }
  }

  handleWorkerMessage(worker, msg) {
    const { taskId, result, error } = msg;

    if (taskId) {
      const task = this.activeTasks.get(taskId);
      if (task) {
        this.activeTasks.delete(taskId);

        if (error) {
          task.reject(new Error(error));
        } else {
          task.resolve(result);
        }

        this.markWorkerAvailable(worker);
        this.processQueue();
      }
    }
  }

  markWorkerAvailable(worker) {
    const workerInfo = this.workers.find(w => w.id === worker.pid);
    if (workerInfo) {
      workerInfo.busy = false;
    }
  }

  processQueue() {
    const availableWorker = this.workers.find(w => !w.busy);

    if (availableWorker && this.taskQueue.length > 0) {
      const task = this.taskQueue.shift();
      this.executeTask(availableWorker, task);
    }
  }

  executeTask(workerInfo, task) {
    const { taskId, data } = task;
    workerInfo.busy = true;

    return new Promise((resolve, reject) => {
      this.activeTasks.set(taskId, { resolve, reject });
      workerInfo.worker.send({ taskId, data });
    });
  }

  async submitTask(data) {
    const taskId = `${Date.now()}-${Math.random()}`;
    const task = { taskId, data };

    return new Promise((resolve, reject) => {
      task.resolve = resolve;
      task.reject = reject;
      this.taskQueue.push(task);
      this.processQueue();
    });
  }

  restartWorker(worker) {
    const index = this.workers.findIndex(w => w.id === worker.pid);
    if (index !== -1) {
      const newWorker = fork('worker.js');

      newWorker.on('message', (msg) => {
        this.handleWorkerMessage(newWorker, msg);
      });

      this.workers[index] = {
        worker: newWorker,
        id: newWorker.pid,
        busy: false
      };
    }
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

// worker.js
process.on('message', async ({ taskId, data }) => {
  try {
    // 处理任务
    const result = await processTask(data);
    process.send({ taskId, result });
  } catch (error) {
    process.send({ taskId, error: error.message });
  }
});

async function processTask(data) {
  // 模拟长时间任务
  await new Promise(resolve => setTimeout(resolve, 1000));
  return { processed: true, data };
}
```

### 3. Git 操作工具

```javascript
const { exec } = require('child_process');
const { promisify } = require('util');

const execAsync = promisify(exec);

class GitOperations {
  constructor(repoPath) {
    this.repoPath = repoPath;
  }

  async executeGitCommand(command) {
    try {
      const { stdout, stderr } = await execAsync(command, {
        cwd: this.repoPath
      });

      if (stderr && !stderr.includes('warning')) {
        console.warn('Git 警告:', stderr);
      }

      return stdout.trim();
    } catch (error) {
      throw new Error(`Git 命令执行失败: ${error.message}`);
    }
  }

  async getStatus() {
    return this.executeGitCommand('git status');
  }

  async getCurrentBranch() {
    return this.executeGitCommand('git rev-parse --abbrev-ref HEAD');
  }

  async getLatestCommit() {
    return this.executeGitCommand('git log -1 --pretty=format:"%h - %s (%an, %ar)"');
  }

  async pull(branch = 'main') {
    return this.executeGitCommand(`git pull origin ${branch}`);
  }

  async add(files) {
    const fileList = Array.isArray(files) ? files.join(' ') : files;
    return this.executeGitCommand(`git add ${fileList}`);
  }

  async commit(message) {
    return this.executeGitCommand(`git commit -m "${message}"`);
  }

  async push(branch) {
    return this.executeGitCommand(`git push origin ${branch}`);
  }

  async createBranch(branchName) {
    return this.executeGitCommand(`git checkout -b ${branchName}`);
  }

  async switchBranch(branchName) {
    return this.executeGitCommand(`git checkout ${branchName}`);
  }

  async getRemoteUrl() {
    return this.executeGitCommand('git config --get remote.origin.url');
  }

  async getFileHistory(filePath, limit = 10) {
    return this.executeGitCommand(`git log -${limit} --pretty=format:"%h %s" ${filePath}`);
  }

  async getFileDiff(filePath, commit1, commit2) {
    if (commit2) {
      return this.executeGitCommand(`git diff ${commit1} ${commit2} ${filePath}`);
    } else {
      return this.executeGitCommand(`git diff ${commit1} ${filePath}`);
    }
  }

  async clone(repoUrl, targetDir) {
    try {
      const { stdout } = await execAsync(`git clone ${repoUrl} ${targetDir}`);
      return stdout;
    } catch (error) {
      throw new Error(`克隆仓库失败: ${error.message}`);
    }
  }
}

// 使用示例
const git = new GitOperations('/path/to/repo');

git.getStatus().then(status => console.log('Git 状态:', status));
git.getCurrentBranch().then(branch => console.log('当前分支:', branch));
git.getLatestCommit().then(commit => console.log('最新提交:', commit));
```

### 4. 进程池

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
  }

  execute(command, args, options = {}) {
    return new Promise((resolve, reject) => {
      const task = { command, args, options, resolve, reject };
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
    const { command, args, options, resolve, reject } = task;
    this.activeProcesses++;

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
      this.activeProcesses--;

      if (code === 0) {
        resolve({ stdout, stderr, code });
      } else {
        reject(new Error(`进程退出，退出码: ${code}\n${stderr}`));
      }

      this.processQueue();
    });

    process.on('error', (error) => {
      this.activeProcesses--;
      reject(error);
      this.processQueue();
    });

    this.processes.push({
      pid: process.pid,
      process,
      startTime: Date.now()
    });
  }

  getActiveProcesses() {
    return this.processes.filter(p => p.process.killed === false);
  }

  killAll() {
    this.processes.forEach(({ process }) => {
      if (!process.killed) {
        process.kill('SIGTERM');
      }
    });
  }
}

// 使用示例
const pool = new ProcessPool({ maxProcesses: 3 });

async function runTasks() {
  try {
    const result1 = await pool.execute('node', ['--version']);
    console.log('结果1:', result1.stdout);

    const result2 = await pool.execute('npm', ['--version']);
    console.log('结果2:', result2.stdout);

    const result3 = await pool.execute('git', ['--version']);
    console.log('结果3:', result3.stdout);
  } catch (error) {
    console.error('任务执行失败:', error);
  }
}

runTasks();
```

## 注意事项

1. **资源管理**：及时清理子进程，避免僵尸进程和资源泄漏
2. **错误处理**：监听 `error` 事件，捕获进程启动和运行中的错误
3. **安全性**：避免直接拼接用户输入到命令中，防止命令注入攻击
4. **性能考虑**：频繁创建和销毁子进程开销较大，考虑使用进程池
5. **缓冲限制**：exec() 的输出缓冲有限制，大量输出可能被截断
6. **平台差异**：不同操作系统的命令和路径可能不同
7. **退出码**：正确处理进程的退出码，区分正常退出和异常退出

## 最佳实践

1. **选择合适的方法**：
   - 需要流式输出：使用 `spawn()`
   - 执行简单命令：使用 `exec()`
   - 执行文件：使用 `execFile()`
   - Node.js 子进程：使用 `fork()`

2. **安全执行命令**：
   ```javascript
   // 不安全（命令注入风险）
   const userInput = '; rm -rf /';
   exec(`echo ${userInput}`);

   // 安全
   const args = [userInput];
   spawn('echo', args);
   ```

3. **清理资源**：
   ```javascript
   const child = spawn('long-running-command');

   process.on('exit', () => {
     child.kill();
   });
   ```

4. **超时处理**：
   ```javascript
   const child = spawn('command');
   const timeout = setTimeout(() => {
     child.kill();
   }, 5000);

   child.on('close', () => {
     clearTimeout(timeout);
   });
   ```

5. **环境变量隔离**：
   ```javascript
   const child = spawn('command', [], {
     env: { PATH: process.env.PATH, CUSTOM_VAR: 'value' }
   });
   ```

## 总结

`child_process` 模块是 Node.js 中强大的多进程工具，提供了多种创建和管理子进程的方式。合理使用这些方法可以实现进程并行、系统集成、任务队列等功能。在实际开发中，应根据具体场景选择合适的方法，注意安全性和性能问题，确保进程的正确管理和资源清理。