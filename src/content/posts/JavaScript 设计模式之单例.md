---
title: JavaScript 设计模式之单例
published: 2022-08-12
description: '单例模式的实现和应用的详细介绍和学习笔记'
image: ''
tags: ["JS","设计模式"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

单例模式(Singleton Pattern)是一种创建型设计模式,它确保一个类只有一个实例,并提供一个全局访问点。在前端开发中,单例模式常用于管理全局状态、配置信息、工具类等。

## 核心概念

### 单例模式特点

1. **唯一性**: 保证一个类只有一个实例
2. **全局性**: 提供全局访问点
3. **延迟创建**: 在需要时才创建实例
4. **线程安全**: 确保多线程环境下的安全性

### 基本实现

```javascript
// 基础单例模式
class Singleton {
  constructor() {
    if (Singleton.instance) {
      return Singleton.instance;
    }
    Singleton.instance = this;
    this.name = 'Singleton Instance';
  }
  
  sayHello() {
    console.log(`Hello from ${this.name}`);
  }
}

// 使用
const instance1 = new Singleton();
const instance2 = new Singleton();

console.log(instance1 === instance2); // true
instance1.sayHello(); // Hello from Singleton Instance
```

## 实现方式

### 1. 静态属性实现

```javascript
class StaticSingleton {
  constructor() {
    if (StaticSingleton.instance) {
      return StaticSingleton.instance;
    }
    
    this.config = {
      apiUrl: 'https://api.example.com',
      timeout: 5000
    };
    
    StaticSingleton.instance = this;
  }
  
  getConfig() {
    return this.config;
  }
  
  updateConfig(newConfig) {
    this.config = { ...this.config, ...newConfig };
  }
}

// 使用
const config1 = new StaticSingleton();
const config2 = new StaticSingleton();

console.log(config1 === config2); // true
config1.updateConfig({ timeout: 10000 });
console.log(config2.getConfig().timeout); // 10000
```

### 2. 闭包实现

```javascript
const ClosureSingleton = (function() {
  let instance = null;
  
  function createInstance() {
    return {
      data: [],
      add(item) {
        this.data.push(item);
      },
      get() {
        return this.data;
      },
      clear() {
        this.data = [];
      }
    };
  }
  
  return {
    getInstance() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

// 使用
const dataStore1 = ClosureSingleton.getInstance();
const dataStore2 = ClosureSingleton.getInstance();

console.log(dataStore1 === dataStore2); // true
dataStore1.add('item1');
console.log(dataStore2.get()); // ['item1']
```

### 3. 模块模式实现

```javascript
const ModuleSingleton = {
  instance: null,
  
  getInstance() {
    if (!this.instance) {
      this.instance = {
        users: new Map(),
        
        addUser(id, user) {
          this.users.set(id, user);
        },
        
        getUser(id) {
          return this.users.get(id);
        },
        
        removeUser(id) {
          this.users.delete(id);
        },
        
        getAllUsers() {
          return Array.from(this.users.values());
        }
      };
    }
    
    return this.instance;
  }
};

// 使用
const userManager1 = ModuleSingleton.getInstance();
const userManager2 = ModuleSingleton.getInstance();

userManager1.addUser(1, { name: 'Alice' });
console.log(userManager2.getUser(1)); // { name: 'Alice' }
```

### 4. Object.create 实现

```javascript
const ObjectSingleton = {
  instance: null,
  
  init() {
    this.data = {};
    this.listeners = [];
  },
  
  getInstance() {
    if (!this.instance) {
      this.instance = Object.create(this);
      this.instance.init();
    }
    
    return this.instance;
  },
  
  set(key, value) {
    this.data[key] = value;
    this.notify({ type: 'SET', key, value });
  },
  
  get(key) {
    return this.data[key];
  },
  
  subscribe(listener) {
    this.listeners.push(listener);
    return () => {
      this.listeners = this.listeners.filter(l => l !== listener);
    };
  },
  
  notify(event) {
    this.listeners.forEach(listener => listener(event));
  }
};

// 使用
const store = ObjectSingleton.getInstance();
const unsubscribe = store.subscribe((event) => {
  console.log('Event:', event);
});

store.set('name', 'Alice'); // Event: { type: 'SET', key: 'name', value: 'Alice' }
unsubscribe();
```

## 实际应用

### 1. 全局配置管理

```javascript
class ConfigManager {
  constructor() {
    if (ConfigManager.instance) {
      return ConfigManager.instance;
    }
    
    this.config = {
      environment: 'development',
      apiUrl: '',
      timeout: 5000,
      retries: 3
    };
    
    ConfigManager.instance = this;
  }
  
  static getInstance() {
    if (!ConfigManager.instance) {
      ConfigManager.instance = new ConfigManager();
    }
    return ConfigManager.instance;
  }
  
  load(config) {
    this.config = { ...this.config, ...config };
  }
  
  get(key) {
    return this.config[key];
  }
  
  set(key, value) {
    this.config[key] = value;
  }
  
  isProduction() {
    return this.config.environment === 'production';
  }
}

// 应用初始化
ConfigManager.getInstance().load({
  environment: process.env.NODE_ENV || 'development',
  apiUrl: process.env.API_URL || 'http://localhost:3000'
});

// 在其他地方使用
const config = ConfigManager.getInstance();
console.log(config.isProduction());
console.log(config.get('apiUrl'));
```

### 2. 事件总线

```javascript
class EventBus {
  constructor() {
    if (EventBus.instance) {
      return EventBus.instance;
    }
    
    this.events = {};
    EventBus.instance = this;
  }
  
  static getInstance() {
    if (!EventBus.instance) {
      EventBus.instance = new EventBus();
    }
    return EventBus.instance;
  }
  
  on(event, callback) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(callback);
  }
  
  off(event, callback) {
    if (this.events[event]) {
      this.events[event] = this.events[event].filter(cb => cb !== callback);
    }
  }
  
  emit(event, data) {
    if (this.events[event]) {
      this.events[event].forEach(callback => callback(data));
    }
  }
  
  once(event, callback) {
    const onceCallback = (data) => {
      callback(data);
      this.off(event, onceCallback);
    };
    this.on(event, onceCallback);
  }
}

// 使用
const eventBus = EventBus.getInstance();

// 订阅事件
eventBus.on('user:login', (user) => {
  console.log('用户登录:', user.name);
});

eventBus.once('app:ready', () => {
  console.log('应用已准备就绪');
});

// 触发事件
eventBus.emit('user:login', { id: 1, name: 'Alice' });
eventBus.emit('app:ready'); // 只执行一次
eventBus.emit('app:ready'); // 不会执行
```

### 3. 缓存管理

```javascript
class CacheManager {
  constructor() {
    if (CacheManager.instance) {
      return CacheManager.instance;
    }
    
    this.cache = new Map();
    this.ttl = new Map();
    
    // 定期清理过期缓存
    this.cleanupInterval = setInterval(() => {
      this.cleanup();
    }, 60000);
    
    CacheManager.instance = this;
  }
  
  static getInstance() {
    if (!CacheManager.instance) {
      CacheManager.instance = new CacheManager();
    }
    return CacheManager.instance;
  }
  
  set(key, value, ttl = 300000) {
    this.cache.set(key, value);
    this.ttl.set(key, Date.now() + ttl);
  }
  
  get(key) {
    if (!this.cache.has(key)) {
      return null;
    }
    
    const expiry = this.ttl.get(key);
    if (Date.now() > expiry) {
      this.delete(key);
      return null;
    }
    
    return this.cache.get(key);
  }
  
  delete(key) {
    this.cache.delete(key);
    this.ttl.delete(key);
  }
  
  clear() {
    this.cache.clear();
    this.ttl.clear();
  }
  
  cleanup() {
    const now = Date.now();
    for (const [key, expiry] of this.ttl) {
      if (now > expiry) {
        this.delete(key);
      }
    }
  }
  
  destroy() {
    this.clear();
    clearInterval(this.cleanupInterval);
  }
}

// 使用
const cache = CacheManager.getInstance();

cache.set('user:1', { name: 'Alice' }, 5000);
console.log(cache.get('user:1')); // { name: 'Alice' }

setTimeout(() => {
  console.log(cache.get('user:1')); // null (已过期)
}, 6000);
```

### 4. 日志管理

```javascript
class LogManager {
  constructor() {
    if (LogManager.instance) {
      return LogManager.instance;
    }
    
    this.logs = [];
    this.maxLogs = 1000;
    this.level = 'INFO';
    
    LogManager.instance = this;
  }
  
  static getInstance() {
    if (!LogManager.instance) {
      LogManager.instance = new LogManager();
    }
    return LogManager.instance;
  }
  
  setLevel(level) {
    this.level = level;
  }
  
  log(level, message, meta = {}) {
    const timestamp = new Date().toISOString();
    const logEntry = { timestamp, level, message, meta };
    
    this.logs.push(logEntry);
    
    if (this.logs.length > this.maxLogs) {
      this.logs.shift();
    }
    
    if (this.shouldLog(level)) {
      console.log(`[${timestamp}] ${level}: ${message}`, meta);
    }
  }
  
  shouldLog(level) {
    const levels = ['DEBUG', 'INFO', 'WARN', 'ERROR'];
    return levels.indexOf(level) >= levels.indexOf(this.level);
  }
  
  debug(message, meta) {
    this.log('DEBUG', message, meta);
  }
  
  info(message, meta) {
    this.log('INFO', message, meta);
  }
  
  warn(message, meta) {
    this.log('WARN', message, meta);
  }
  
  error(message, meta) {
    this.log('ERROR', message, meta);
  }
  
  getLogs(level = null) {
    if (level) {
      return this.logs.filter(log => log.level === level);
    }
    return this.logs;
  }
  
  clear() {
    this.logs = [];
  }
}

// 使用
const logger = LogManager.getInstance();

logger.setLevel('DEBUG');
logger.debug('调试信息', { userId: 1 });
logger.info('用户登录', { userId: 1 });
logger.warn('警告信息', { attempts: 3 });
logger.error('错误信息', { error: 'Timeout' });

console.log(logger.getLogs('ERROR'));
```

### 5. 状态管理

```javascript
class StateManager {
  constructor() {
    if (StateManager.instance) {
      return StateManager.instance;
    }
    
    this.state = {};
    this.listeners = new Set();
    
    StateManager.instance = this;
  }
  
  static getInstance() {
    if (!StateManager.instance) {
      StateManager.instance = new StateManager();
    }
    return StateManager.instance;
  }
  
  setState(newState) {
    const prevState = { ...this.state };
    this.state = { ...this.state, ...newState };
    
    this.notify(prevState, this.state);
  }
  
  getState() {
    return { ...this.state };
  }
  
  subscribe(listener) {
    this.listeners.add(listener);
    
    // 返回取消订阅函数
    return () => {
      this.listeners.delete(listener);
    };
  }
  
  notify(prevState, currentState) {
    this.listeners.forEach(listener => {
      listener(prevState, currentState);
    });
  }
  
  reset() {
    const prevState = { ...this.state };
    this.state = {};
    this.notify(prevState, this.state);
  }
}

// 使用
const store = StateManager.getInstance();

// 订阅状态变化
const unsubscribe = store.subscribe((prev, current) => {
  console.log('状态变化:', prev, current);
});

// 更新状态
store.setState({ user: { name: 'Alice' } });
store.setState({ user: { name: 'Alice', age: 25 } });

// 取消订阅
unsubscribe();
```

## 高级应用

### 1. 单例工厂

```javascript
class SingletonFactory {
  static instances = new Map();
  
  static getInstance(Class, ...args) {
    const className = Class.name;
    
    if (!this.instances.has(className)) {
      const instance = new Class(...args);
      this.instances.set(className, instance);
    }
    
    return this.instances.get(className);
  }
  
  static clearInstance(Class) {
    const className = Class.name;
    this.instances.delete(className);
  }
  
  static clearAll() {
    this.instances.clear();
  }
}

// 使用
class Database {
  constructor(config) {
    this.config = config;
    this.connected = false;
  }
  
  connect() {
    this.connected = true;
    console.log('数据库已连接');
  }
}

const db1 = SingletonFactory.getInstance(Database, { host: 'localhost' });
const db2 = SingletonFactory.getInstance(Database, { host: 'localhost' });

console.log(db1 === db2); // true
db1.connect();
```

### 2. 懒加载单例

```javascript
class LazySingleton {
  static getInstance() {
    if (!LazySingleton.instance) {
      LazySingleton.instance = new Promise((resolve) => {
        // 模拟异步初始化
        setTimeout(() => {
          const instance = {
            data: [],
            initialized: true,
            add(item) {
              this.data.push(item);
            },
            get() {
              return this.data;
            }
          };
          resolve(instance);
        }, 100);
      });
    }
    return LazySingleton.instance;
  }
}

// 使用
LazySingleton.getInstance().then(instance => {
  console.log('实例已创建:', instance.initialized);
  instance.add('item1');
  console.log(instance.get());
});
```

### 3. 单例装饰器

```javascript
function Singleton(target) {
  target.getInstance = function() {
    if (!target.instance) {
      target.instance = new target();
    }
    return target.instance;
  };
  
  // 防止直接实例化
  const originalConstructor = target;
  target = function(...args) {
    if (target.instance) {
      return target.instance;
    }
    return originalConstructor.apply(this, args);
  };
  target.prototype = originalConstructor.prototype;
  
  return target;
}

// 使用
@Singleton
class APIClient {
  constructor() {
    this.baseUrl = 'https://api.example.com';
  }
  
  async request(endpoint) {
    const response = await fetch(`${this.baseUrl}${endpoint}`);
    return response.json();
  }
}

// 使用
const client1 = new APIClient();
const client2 = APIClient.getInstance();

console.log(client1 === client2); // true
```

## 注意事项

### 1. 测试困难

```javascript
// 单例模式可能导致测试困难
// 解决方案: 提供重置方法

class TestableSingleton {
  constructor() {
    if (TestableSingleton.instance) {
      return TestableSingleton.instance;
    }
    
    this.data = [];
    TestableSingleton.instance = this;
  }
  
  static reset() {
    TestableSingleton.instance = null;
  }
}

// 测试中
beforeEach(() => {
  TestableSingleton.reset();
});

test('test singleton', () => {
  const instance = new TestableSingleton();
  // 测试代码
});
```

### 2. 隐式依赖

```javascript
// 避免隐式依赖,改为显式传递
class UserService {
  constructor(config) {
    this.config = config;
  }
  
  // 而不是在内部直接获取单例
  // constructor() {
  //   this.config = ConfigManager.getInstance();
  // }
}

// 更好的方式
const config = ConfigManager.getInstance();
const userService = new UserService(config);
```

### 3. 全局状态污染

```javascript
// 谨慎使用单例管理状态
// 考虑使用有限的作用域

class ScopedSingleton {
  static instances = new Map();
  
  static getInstance(scope, Class) {
    if (!this.instances.has(scope)) {
      this.instances.set(scope, new Class());
    }
    return this.instances.get(scope);
  }
}

// 使用
const userCache = ScopedSingleton.getInstance('user', Cache);
const productCache = ScopedSingleton.getInstance('product', Cache);
```

### 4. 多实例需求

```javascript
// 考虑使用对象池而不是单例
class ObjectPool {
  static instances = new Map();
  
  static acquire(Class) {
    const className = Class.name;
    
    if (!this.instances.has(className)) {
      this.instances.set(className, []);
    }
    
    const pool = this.instances.get(className);
    const instance = pool.pop() || new Class();
    
    return instance;
  }
  
  static release(Class, instance) {
    const className = Class.name;
    
    if (!this.instances.has(className)) {
      this.instances.set(className, []);
    }
    
    const pool = this.instances.get(className);
    pool.push(instance);
  }
}
```

## 最佳实践

### 1. 明确使用场景

```javascript
// 适合使用单例的场景
const suitableScenarios = [
  '全局配置管理',
  '事件总线',
  '缓存服务',
  '日志管理',
  'API 客户端',
  '数据库连接池'
];

// 不适合使用单例的场景
const unsuitableScenarios = [
  '业务数据存储',
  '临时状态管理',
  '需要多个实例的场景',
  '需要隔离的环境'
];
```

### 2. 提供清理方法

```javascript
class CleanableSingleton {
  constructor() {
    if (CleanableSingleton.instance) {
      return CleanableSingleton.instance;
    }
    
    this.data = [];
    CleanableSingleton.instance = this;
  }
  
  static destroy() {
    if (CleanableSingleton.instance) {
      CleanableSingleton.instance.data = null;
      CleanableSingleton.instance = null;
    }
  }
}

// 应用关闭时清理
window.addEventListener('beforeunload', () => {
  CleanableSingleton.destroy();
});
```

### 3. 线程安全考虑

```javascript
class SafeSingleton {
  constructor() {
    if (SafeSingleton.instance) {
      return SafeSingleton.instance;
    }
    
    // 添加锁机制
    if (SafeSingleton.lock) {
      return SafeSingleton.instance;
    }
    
    SafeSingleton.lock = true;
    
    try {
      if (!SafeSingleton.instance) {
        SafeSingleton.instance = this;
      }
    } finally {
      SafeSingleton.lock = false;
    }
  }
}
```

### 4. 使用模块化

```javascript
// 使用 ES6 模块实现单例
// singleton.js
export const singleton = {
  data: [],
  add(item) {
    this.data.push(item);
  },
  get() {
    return this.data;
  }
};

// 其他文件
import { singleton } from './singleton';
singleton.add('item');
```

## 总结

单例模式的核心要点:

1. **唯一实例**: 确保类只有一个实例存在
2. **多种实现**: 静态属性、闭包、模块模式等
3. **实际应用**: 配置管理、事件总线、缓存、日志、状态管理
4. **高级用法**: 单例工厂、懒加载、装饰器模式
5. **注意事项**: 测试困难、隐式依赖、全局状态、多实例需求
6. **最佳实践**: 明确场景、提供清理、线程安全、模块化
7. **适度使用**: 在合适的场景使用,避免滥用

单例模式是简单但强大的设计模式,合理使用可以简化代码结构,提高资源利用率。