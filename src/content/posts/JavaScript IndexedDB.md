---
title: JavaScript IndexedDB
published: 2022-12-24
description: '浏览器数据库详解的详细介绍和学习笔记'
image: ''
tags: ["存储"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

IndexedDB 是浏览器提供的内置数据库系统，支持存储大量结构化数据。与 LocalStorage 和 SessionStorage 不同，IndexedDB 是异步的、事务性的，能够存储复杂的对象和二进制数据，适合需要离线存储大量数据的应用场景。

## 核心概念

### 数据库结构

- **Database（数据库）**：最高级别的容器
- **Object Store（对象存储）**：类似于表，存储数据记录
- **Index（索引）**：用于快速检索数据
- **Transaction（事务）**：保证数据操作的原子性
- **Cursor（游标）**：遍历对象存储中的记录

### 与其他存储方式的对比

| 特性 | LocalStorage | SessionStorage | IndexedDB |
|------|--------------|----------------|-----------|
| 存储容量 | ~5MB | ~5MB | ~50MB-2GB+ |
| 数据类型 | 仅字符串 | 仅字符串 | 多种类型 |
| 操作方式 | 同步 | 同步 | 异步 |
| 查询能力 | 无 | 无 | 索引查询 |
| 事务支持 | 无 | 无 | 支持 |
| 适用场景 | 简单数据存储 | 临时数据 | 复杂数据存储 |

## 基本用法

### 打开数据库

```javascript
// 打开数据库，如果不存在则创建
const request = indexedDB.open('MyDatabase', 1);

request.onerror = function(event) {
  console.error('数据库打开失败:', event.target.error);
};

request.onsuccess = function(event) {
  const db = event.target.result;
  console.log('数据库打开成功:', db);
  console.log('数据库名称:', db.name);
  console.log('数据库版本:', db.version);
};

request.onupgradeneeded = function(event) {
  const db = event.target.result;
  console.log('数据库升级:', event.oldVersion, '->', event.newVersion);

  // 在这里创建对象存储和索引
  createObjectStores(db);
};
```

### 创建对象存储和索引

```javascript
function createObjectStores(db) {
  // 创建用户对象存储
  if (!db.objectStoreNames.contains('users')) {
    const usersStore = db.createObjectStore('users', { keyPath: 'id' });

    // 创建普通索引
    usersStore.createIndex('name', 'name', { unique: false });
    usersStore.createIndex('email', 'email', { unique: true });

    // 创建复合索引
    usersStore.createIndex('name-email', ['name', 'email'], { unique: false });

    // 创建多值索引（数组字段）
    usersStore.createIndex('tags', 'tags', { multiEntry: true });

    console.log('users 对象存储创建成功');
  }

  // 创建文章对象存储（使用自增键）
  if (!db.objectStoreNames.contains('articles')) {
    const articlesStore = db.createObjectStore('articles', {
      keyPath: 'id',
      autoIncrement: true
    });

    articlesStore.createIndex('title', 'title', { unique: false });
    articlesStore.createIndex('authorId', 'authorId', { unique: false });
    articlesStore.createIndex('publishedAt', 'publishedAt', { unique: false });

    console.log('articles 对象存储创建成功');
  }
}
```

### 添加数据

```javascript
function addUser(db, userData) {
  const transaction = db.transaction(['users'], 'readwrite');
  const store = transaction.objectStore('users');

  const request = store.add(userData);

  request.onsuccess = function() {
    console.log('用户添加成功:', userData.id);
  };

  request.onerror = function(event) {
    console.error('用户添加失败:', event.target.error);
  };
}

// 使用示例
const user = {
  id: 1,
  name: '张三',
  email: 'zhangsan@example.com',
  age: 25,
  tags: ['developer', 'javascript'],
  createdAt: new Date()
};

addUser(db, user);
```

### 读取数据

```javascript
// 通过主键读取
function getUserById(db, id) {
  const transaction = db.transaction(['users'], 'readonly');
  const store = transaction.objectStore('users');
  const request = store.get(id);

  request.onsuccess = function() {
    const user = request.result;
    if (user) {
      console.log('找到用户:', user);
    } else {
      console.log('用户不存在');
    }
  };
}

// 通过索引读取
function getUserByEmail(db, email) {
  const transaction = db.transaction(['users'], 'readonly');
  const store = transaction.objectStore('users');
  const index = store.index('email');
  const request = index.get(email);

  request.onsuccess = function() {
    const user = request.result;
    if (user) {
      console.log('找到用户:', user);
    } else {
      console.log('用户不存在');
    }
  };
}

// 使用游标读取所有数据
function getAllUsers(db) {
  const transaction = db.transaction(['users'], 'readonly');
  const store = transaction.objectStore('users');
  const request = store.getAll();

  request.onsuccess = function() {
    const users = request.result;
    console.log('所有用户:', users);
  };
}
```

### 更新数据

```javascript
function updateUser(db, userData) {
  const transaction = db.transaction(['users'], 'readwrite');
  const store = transaction.objectStore('users');

  const request = store.put(userData);

  request.onsuccess = function() {
    console.log('用户更新成功:', userData.id);
  };

  request.onerror = function(event) {
    console.error('用户更新失败:', event.target.error);
  };
}

// 使用示例
const updatedUser = {
  id: 1,
  name: '张三（更新）',
  email: 'zhangsan@example.com',
  age: 26,
  tags: ['developer', 'javascript', 'react'],
  createdAt: new Date(),
  updatedAt: new Date()
};

updateUser(db, updatedUser);
```

### 删除数据

```javascript
// 通过主键删除
function deleteUserById(db, id) {
  const transaction = db.transaction(['users'], 'readwrite');
  const store = transaction.objectStore('users');
  const request = store.delete(id);

  request.onsuccess = function() {
    console.log('用户删除成功:', id);
  };

  request.onerror = function(event) {
    console.error('用户删除失败:', event.target.error);
  };
}

// 清空对象存储
function clearUsers(db) {
  const transaction = db.transaction(['users'], 'readwrite');
  const store = transaction.objectStore('users');
  const request = store.clear();

  request.onsuccess = function() {
    console.log('用户表清空成功');
  };
}

// 使用游标删除符合条件的记录
function deleteUsersByAge(db, minAge) {
  const transaction = db.transaction(['users'], 'readwrite');
  const store = transaction.objectStore('users');
  const request = store.openCursor();

  request.onsuccess = function(event) {
    const cursor = event.target.result;
    if (cursor) {
      const user = cursor.value;
      if (user.age < minAge) {
        cursor.delete();
        console.log('删除用户:', user.id);
      }
      cursor.continue();
    }
  };
}
```

## 高级操作

### 使用游标

```javascript
// 遍历所有记录
function iterateUsers(db) {
  const transaction = db.transaction(['users'], 'readonly');
  const store = transaction.objectStore('users');
  const request = store.openCursor();

  request.onsuccess = function(event) {
    const cursor = event.target.result;
    if (cursor) {
      console.log('当前用户:', cursor.value);
      cursor.continue(); // 继续下一条记录
    } else {
      console.log('遍历完成');
    }
  };
}

// 带条件的游标遍历
function iterateUsersWithCondition(db, callback) {
  const transaction = db.transaction(['users'], 'readonly');
  const store = transaction.objectStore('users');
  const request = store.openCursor();

  request.onsuccess = function(event) {
    const cursor = event.target.result;
    if (cursor) {
      const shouldContinue = callback(cursor.value);
      if (shouldContinue !== false) {
        cursor.continue();
      }
    }
  };
}

// 使用示例：查找年龄大于25的用户
iterateUsersWithCondition(db, user => {
  if (user.age > 25) {
    console.log('符合条件的用户:', user);
  }
  return true; // 继续遍历
});
```

### 索引查询

```javascript
// 使用索引查询单个记录
function getUserByName(db, name) {
  const transaction = db.transaction(['users'], 'readonly');
  const store = transaction.objectStore('users');
  const index = store.index('name');
  const request = index.get(name);

  request.onsuccess = function() {
    const user = request.result;
    if (user) {
      console.log('找到用户:', user);
    }
  };
}

// 使用索引查询多个记录
function getUsersByTag(db, tag) {
  const transaction = db.transaction(['users'], 'readonly');
  const store = transaction.objectStore('users');
  const index = store.index('tags');
  const request = index.getAll(tag);

  request.onsuccess = function() {
    const users = request.result;
    console.log('标签为', tag, '的用户:', users);
  };
}

// 使用游标查询索引范围
function getUsersInAgeRange(db, minAge, maxAge) {
  const transaction = db.transaction(['users'], 'readonly');
  const store = transaction.objectStore('users');
  const index = store.index('age');
  const range = IDBKeyRange.bound(minAge, maxAge);
  const request = index.openCursor(range);

  request.onsuccess = function(event) {
    const cursor = event.target.result;
    if (cursor) {
      console.log('年龄在范围内的用户:', cursor.value);
      cursor.continue();
    }
  };
}
```

### 键范围查询

```javascript
// 创建键范围
const lowerBound = IDBKeyRange.lowerBound(10);      // >= 10
const upperBound = IDBKeyRange.upperBound(20);      // <= 20
const bound = IDBKeyRange.bound(10, 20);           // 10 <= x <= 20
const boundExclusive = IDBKeyRange.bound(10, 20, true, true); // 10 < x < 20
const only = IDBKeyRange.only(15);                  // === 15

// 使用键范围查询
function getUsersByAgeRange(db, minAge, maxAge) {
  const transaction = db.transaction(['users'], 'readonly');
  const store = transaction.objectStore('users');
  const ageIndex = store.index('age');
  const range = IDBKeyRange.bound(minAge, maxAge);

  const request = ageIndex.openCursor(range);

  request.onsuccess = function(event) {
    const cursor = event.target.result;
    if (cursor) {
      console.log('用户:', cursor.value);
      cursor.continue();
    }
  };
}

// 分页查询
function getPaginatedUsers(db, page = 1, pageSize = 10) {
  const transaction = db.transaction(['users'], 'readonly');
  const store = transaction.objectStore('users');
  const request = store.openCursor(null, 'prev');

  let skip = (page - 1) * pageSize;
  let count = 0;
  const users = [];

  request.onsuccess = function(event) {
    const cursor = event.target.result;
    if (cursor) {
      if (skip > 0) {
        skip--;
        cursor.continue();
      } else if (count < pageSize) {
        users.push(cursor.value);
        count++;
        cursor.continue();
      } else {
        console.log('第', page, '页用户:', users);
      }
    }
  };
}
```

### 复合索引查询

```javascript
// 使用复合索引
function getUserByNameAndEmail(db, name, email) {
  const transaction = db.transaction(['users'], 'readonly');
  const store = transaction.objectStore('users');
  const index = store.index('name-email');
  const request = index.get([name, email]);

  request.onsuccess = function() {
    const user = request.result;
    if (user) {
      console.log('找到用户:', user);
    }
  };
}

// 复合索引范围查询
function getUsersByNameRange(db, startName, endName) {
  const transaction = db.transaction(['users'], 'readonly');
  const store = transaction.objectStore('users');
  const index = store.index('name-email');
  const range = IDBKeyRange.bound(
    [startName, ''],
    [endName, '']
  );
  const request = index.openCursor(range);

  request.onsuccess = function(event) {
    const cursor = event.target.result;
    if (cursor) {
      console.log('用户:', cursor.value);
      cursor.continue();
    }
  };
}
```

## 实际应用

### 封装 IndexedDB 工具类

```javascript
class IDBHelper {
  constructor(dbName, version = 1) {
    this.dbName = dbName;
    this.version = version;
    this.db = null;
  }

  async open() {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, this.version);

      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.db = request.result;
        resolve(this.db);
      };

      request.onupgradeneeded = (event) => {
        this.db = event.target.result;
        if (this.onupgradeneeded) {
          this.onupgradeneeded(event);
        }
      };
    });
  }

  async get(storeName, key) {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction(storeName, 'readonly');
      const store = transaction.objectStore(storeName);
      const request = store.get(key);

      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }

  async getAll(storeName) {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction(storeName, 'readonly');
      const store = transaction.objectStore(storeName);
      const request = store.getAll();

      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }

  async add(storeName, data) {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction(storeName, 'readwrite');
      const store = transaction.objectStore(storeName);
      const request = store.add(data);

      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }

  async put(storeName, data) {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction(storeName, 'readwrite');
      const store = transaction.objectStore(storeName);
      const request = store.put(data);

      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }

  async delete(storeName, key) {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction(storeName, 'readwrite');
      const store = transaction.objectStore(storeName);
      const request = store.delete(key);

      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }

  async clear(storeName) {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction(storeName, 'readwrite');
      const store = transaction.objectStore(storeName);
      const request = store.clear();

      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }

  async count(storeName) {
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction(storeName, 'readonly');
      const store = transaction.objectStore(storeName);
      const request = store.count();

      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
}

// 使用示例
const dbHelper = new IDBHelper('MyDatabase', 1);

dbHelper.onupgradeneeded = (event) => {
  const db = event.target.result;
  createObjectStores(db);
};

dbHelper.open().then(async () => {
  // 添加数据
  await dbHelper.add('users', {
    id: 1,
    name: '张三',
    email: 'zhangsan@example.com'
  });

  // 读取数据
  const user = await dbHelper.get('users', 1);
  console.log('用户:', user);

  // 获取所有数据
  const users = await dbHelper.getAll('users');
  console.log('所有用户:', users);

  // 统计数据
  const count = await dbHelper.count('users');
  console.log('用户数量:', count);
});
```

### 离线数据同步

```javascript
class OfflineDataManager {
  constructor(dbName) {
    this.helper = new IDBHelper(dbName);
    this.pendingStore = 'pending-operations';
  }

  async init() {
    this.helper.onupgradeneeded = (event) => {
      const db = event.target.result;

      if (!db.objectStoreNames.contains(this.pendingStore)) {
        const store = db.createObjectStore(this.pendingStore, {
          keyPath: 'id',
          autoIncrement: true
        });
        store.createIndex('operationType', 'operationType', { unique: false });
        store.createIndex('timestamp', 'timestamp', { unique: false });
      }
    };

    await this.helper.open();
  }

  // 保存离线操作
  async saveOfflineOperation(operation) {
    const data = {
      operation,
      timestamp: Date.now(),
      synced: false
    };
    return await this.helper.add(this.pendingStore, data);
  }

  // 同步离线数据
  async syncOfflineData() {
    const operations = await this.helper.getAll(this.pendingStore);
    const pendingOps = operations.filter(op => !op.synced);

    for (const op of pendingOps) {
      try {
        await this.executeOperation(op.operation);

        // 标记为已同步
        op.synced = true;
        await this.helper.put(this.pendingStore, op);

        console.log('操作同步成功:', op.id);
      } catch (error) {
        console.error('操作同步失败:', op.id, error);
      }
    }

    // 清理已同步的操作
    const syncedOps = operations.filter(op => op.synced);
    for (const op of syncedOps) {
      await this.helper.delete(this.pendingStore, op.id);
    }
  }

  async executeOperation(operation) {
    // 根据操作类型执行对应的 API 请求
    switch (operation.type) {
      case 'create':
        return await this.apiCreate(operation.data);
      case 'update':
        return await this.apiUpdate(operation.id, operation.data);
      case 'delete':
        return await this.apiDelete(operation.id);
      default:
        throw new Error('未知操作类型:', operation.type);
    }
  }

  async apiCreate(data) {
    // 模拟 API 调用
    return new Promise((resolve) => {
      setTimeout(() => resolve({ id: Date.now() }), 1000);
    });
  }

  async apiUpdate(id, data) {
    // 模拟 API 调用
    return new Promise((resolve) => {
      setTimeout(() => resolve({ success: true }), 1000);
    });
  }

  async apiDelete(id) {
    // 模拟 API 调用
    return new Promise((resolve) => {
      setTimeout(() => resolve({ success: true }), 1000);
    });
  }
}

// 使用示例
const offlineManager = new OfflineDataManager('OfflineDataDB');
offlineManager.init().then(async () => {
  // 保存离线操作
  await offlineManager.saveOfflineOperation({
    type: 'create',
    data: { name: '新数据', value: 100 }
  });

  // 在线时同步数据
  if (navigator.onLine) {
    await offlineManager.syncOfflineData();
  }

  // 监听网络状态
  window.addEventListener('online', async () => {
    console.log('网络已恢复，开始同步...');
    await offlineManager.syncOfflineData();
  });
});
```

### 数据导入导出

```javascript
// 导出数据
async function exportDatabase(dbName, exportData = {}) {
  const request = indexedDB.open(dbName);
  return new Promise((resolve, reject) => {
    request.onsuccess = async (event) => {
      const db = event.target.result;

      for (const storeName of db.objectStoreNames) {
        const transaction = db.transaction(storeName, 'readonly');
        const store = transaction.objectStore(storeName);
        const getAllRequest = store.getAll();

        await new Promise((resolve) => {
          getAllRequest.onsuccess = () => {
            exportData[storeName] = getAllRequest.result;
            resolve();
          };
        });
      }

      db.close();
      resolve(exportData);
    };

    request.onerror = () => reject(request.error);
  });
}

// 导入数据
async function importDatabase(dbName, version, importData) {
  const request = indexedDB.open(dbName, version);
  return new Promise((resolve, reject) => {
    request.onupgradeneeded = (event) => {
      const db = event.target.result;

      // 清空现有数据
      for (const storeName of db.objectStoreNames) {
        db.deleteObjectStore(storeName);
      }

      // 重新创建对象存储
      for (const storeName of Object.keys(importData)) {
        const sampleData = importData[storeName][0];
        if (sampleData) {
          const keyPath = Object.keys(sampleData)[0];
          db.createObjectStore(storeName, { keyPath });
        }
      }
    };

    request.onsuccess = async (event) => {
      const db = event.target.result;

      for (const storeName of Object.keys(importData)) {
        const transaction = db.transaction(storeName, 'readwrite');
        const store = transaction.objectStore(storeName);

        for (const data of importData[storeName]) {
          await new Promise((resolve) => {
            const addRequest = store.add(data);
            addRequest.onsuccess = resolve;
          });
        }
      }

      db.close();
      resolve();
    };

    request.onerror = () => reject(request.error);
  });
}

// 使用示例
exportDatabase('MyDatabase').then(exportedData => {
  console.log('导出的数据:', exportedData);

  // 保存到文件
  const blob = new Blob([JSON.stringify(exportedData)], { type: 'application/json' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'database-export.json';
  a.click();
});
```

## 注意事项

### 性能优化

1. **批量操作**
   - 使用事务进行批量操作
   - 避免频繁创建事务

2. **索引优化**
   - 只为必要的字段创建索引
   - 复合索引字段顺序很重要

3. **数据清理**
   - 定期清理过期数据
   - 避免存储过大对象

4. **内存管理**
   - 及时关闭数据库连接
   - 使用游标时分批处理数据

### 错误处理

```javascript
// 完整的错误处理
async function safeOperation() {
  try {
    const result = await dbHelper.add('users', userData);
    console.log('操作成功:', result);
  } catch (error) {
    console.error('操作失败:', error);

    // 根据错误类型处理
    if (error.name === 'QuotaExceededError') {
      console.warn('存储空间不足');
      // 清理旧数据或提示用户
    } else if (error.name === 'ConstraintError') {
      console.warn('数据约束冲突');
      // 处理唯一键冲突
    } else {
      // 其他错误
      console.error('未知错误:', error);
    }
  }
}
```

### 浏览器兼容性

```javascript
// 检测浏览器支持
if (!window.indexedDB) {
  console.error('浏览器不支持 IndexedDB');
  // 降级使用其他存储方案
} else {
  // 使用 IndexedDB
}

// 使用前缀兼容旧浏览器
const indexedDB = window.indexedDB ||
                  window.webkitIndexedDB ||
                  window.mozIndexedDB ||
                  window.msIndexedDB;
```

### 安全考虑

1. **数据敏感度**
   - 不要存储敏感信息
   - 考虑数据加密

2. **清理策略**
   - 提供清除数据的功能
   - 实现数据过期机制

3. **跨域限制**
   - 遵守同源策略
   - 注意隐私保护

## 总结

IndexedDB 是强大的浏览器端数据库解决方案，具有以下特点：

- 大容量存储（可达数GB）
- 支持复杂对象和二进制数据
- 异步操作，不阻塞主线程
- 事务支持，保证数据一致性
- 索引查询，提供高效检索

关键要点：
- 理解数据库结构和生命周期
- 掌握基本 CRUD 操作
- 合理使用索引和游标
- 封装工具类简化操作
- 做好错误处理和性能优化

IndexedDB 特别适合需要离线支持、大量数据存储的应用，如离线编辑器、数据缓存、离线游戏等场景。结合 Service Worker 可以构建完整的离线应用。