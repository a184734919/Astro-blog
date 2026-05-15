---
title: JavaScript 设计模式之发布订阅
published: 2022-08-31
description: '发布订阅模式的实现和应用的详细介绍和学习笔记'
image: ''
tags: ["设计模式"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

发布-订阅模式（Publish-Subscribe Pattern）是一种消息传递模式，发送者（发布者）不会将消息直接发送给特定的接收者（订阅者），而是将发布的消息分为不同的类别，无需了解哪些订阅者可能存在。同样的，订阅者可以表达对一个或多个类别的兴趣，只接收感兴趣的消息，无需了解哪些发布者存在。

发布-订阅模式与观察者模式很相似，但有一个关键区别：发布-订阅模式引入了**事件通道（Event Channel）**或**调度中心（Event Bus）**作为中介，实现了发布者和订阅者的完全解耦。

## 核心概念

发布-订阅模式的核心组成部分：
- **Publisher（发布者）**：产生消息或事件，但不直接发送给订阅者
- **Subscriber（订阅者）**：订阅特定的事件，在事件触发时接收通知
- **Event Channel（事件通道/调度中心）**：维护订阅关系，将发布者的消息路由给相应的订阅者
- **Event（事件）**：发布者产生的消息，订阅者订阅的目标

## 基本用法

### 基础发布-订阅模式实现

```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  // 订阅事件
  on(eventName, callback) {
    if (!this.events[eventName]) {
      this.events[eventName] = [];
    }
    this.events[eventName].push(callback);
    console.log(`已订阅事件: ${eventName}`);
  }
  
  // 取消订阅
  off(eventName, callback) {
    if (!this.events[eventName]) return;
    
    this.events[eventName] = this.events[eventName].filter(cb => cb !== callback);
    console.log(`已取消订阅事件: ${eventName}`);
  }
  
  // 触发事件
  emit(eventName, ...args) {
    if (!this.events[eventName]) {
      console.log(`事件 ${eventName} 没有订阅者`);
      return;
    }
    
    console.log(`触发事件: ${eventName}`);
    this.events[eventName].forEach(callback => {
      callback(...args);
    });
  }
  
  // 只订阅一次
  once(eventName, callback) {
    const onceCallback = (...args) => {
      callback(...args);
      this.off(eventName, onceCallback);
    };
    this.on(eventName, onceCallback);
  }
}

// 使用示例
const emitter = new EventEmitter();

// 订阅事件
emitter.on('user-login', (username) => {
  console.log(`欢迎 ${username} 登录系统`);
});

emitter.on('user-login', (username) => {
  console.log(`记录用户 ${username} 的登录时间`);
});

emitter.once('first-login', (username) => {
  console.log(`${username} 是首次登录！`);
});

// 触发事件
emitter.emit('user-login', '张三');
emitter.emit('user-login', '李四');
emitter.emit('first-login', '王五');
emitter.emit('first-login', '赵六'); // 不会触发，因为 once 只执行一次
```

### 支持命名空间的 EventEmitter

```javascript
class NamespacedEventEmitter {
  constructor() {
    this.events = {};
  }
  
  // 解析命名空间
  parseEventName(eventName) {
    const [namespace, ...nameParts] = eventName.split(':');
    return { namespace, name: nameParts.join(':') };
  }
  
  on(eventName, callback) {
    const { namespace, name } = this.parseEventName(eventName);
    
    if (!this.events[name]) {
      this.events[name] = [];
    }
    
    this.events[name].push({
      callback,
      namespace
    });
  }
  
  off(eventName, callback) {
    const { namespace, name } = this.parseEventName(eventName);
    
    if (!this.events[name]) return;
    
    if (callback) {
      this.events[name] = this.events[name].filter(
        handler => !(handler.callback === callback && handler.namespace === namespace)
      );
    } else if (namespace) {
      this.events[name] = this.events[name].filter(
        handler => handler.namespace !== namespace
      );
    }
  }
  
  emit(eventName, ...args) {
    const { name } = this.parseEventName(eventName);
    
    if (!this.events[name]) return;
    
    this.events[name].forEach(handler => {
      handler.callback(...args);
    });
  }
  
  // 取消某个命名空间的所有订阅
  offNamespace(namespace) {
    Object.keys(this.events).forEach(eventName => {
      this.events[eventName] = this.events[eventName].filter(
        handler => handler.namespace !== namespace
      );
    });
  }
}

// 使用
const emitter = new NamespacedEventEmitter();

emitter.on('moduleA:user-click', () => console.log('模块A 处理点击'));
emitter.on('moduleB:user-click', () => console.log('模块B 处理点击'));
emitter.on('moduleA:user-scroll', () => console.log('模块A 处理滚动'));

emitter.emit('user-click'); // 两个模块都会处理

// 取消模块A的所有订阅
emitter.offNamespace('moduleA');
emitter.emit('user-click'); // 只有模块B会处理
```

## 实际应用

### 1. 组件间通信

```javascript
// 全局事件总线
const eventBus = new EventEmitter();

// 组件 A（父组件）
class ParentComponent {
  constructor() {
    this.data = [];
    
    // 订阅子组件的更新事件
    eventBus.on('child:updated', (newData) => {
      this.data.push(newData);
      this.render();
    });
  }
  
  render() {
    console.log('父组件数据:', this.data);
  }
}

// 组件 B（子组件）
class ChildComponent {
  constructor(id) {
    this.id = id;
  }
  
  handleClick() {
    const newData = { id: this.id, value: Math.random() };
    // 发布更新事件
    eventBus.emit('child:updated', newData);
  }
}

// 使用
const parent = new ParentComponent();
const child1 = new ChildComponent(1);
const child2 = new ChildComponent(2);

child1.handleClick();
child2.handleClick();
```

### 2. 异步任务管理

```javascript
class TaskManager {
  constructor() {
    this.eventBus = new EventEmitter();
    this.tasks = new Map();
  }
  
  addTask(id, taskFn) {
    this.eventBus.on(`task:${id}:complete`, (result) => {
      console.log(`任务 ${id} 完成:`, result);
    });
    
    this.eventBus.on(`task:${id}:error`, (error) => {
      console.error(`任务 ${id} 失败:`, error);
    });
    
    // 执行任务
    taskFn()
      .then(result => {
        this.eventBus.emit(`task:${id}:complete`, result);
      })
      .catch(error => {
        this.eventBus.emit(`task:${id}:error`, error);
      });
  }
  
  cancelTask(id) {
    // 清理事件监听
    this.eventBus.off(`task:${id}:complete`);
    this.eventBus.off(`task:${id}:error`);
  }
}

// 使用
const taskManager = new TaskManager();

taskManager.addTask('fetch-user', () => {
  return fetch('/api/user').then(res => res.json());
});

taskManager.addTask('fetch-posts', () => {
  return fetch('/api/posts').then(res => res.json());
});
```

### 3. 状态管理（简化版 Redux）

```javascript
class Store {
  constructor(reducer, initialState) {
    this.reducer = reducer;
    this.state = initialState;
    this.eventBus = new EventEmitter();
  }
  
  getState() {
    return this.state;
  }
  
  dispatch(action) {
    console.log('派发动作:', action);
    this.state = this.reducer(this.state, action);
    this.eventBus.emit('state-change', this.state);
  }
  
  subscribe(listener) {
    this.eventBus.on('state-change', listener);
    
    // 返回取消订阅函数
    return () => {
      this.eventBus.off('state-change', listener);
    };
  }
}

// 使用示例
const initialState = {
  count: 0,
  user: null
};

function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    case 'SET_USER':
      return { ...state, user: action.payload };
    default:
      return state;
  }
}

const store = new Store(reducer, initialState);

// 订阅状态变化
const unsubscribe = store.subscribe((state) => {
  console.log('状态更新:', state);
  // 触发 UI 更新
  updateUI(state);
});

function updateUI(state) {
  console.log('更新 UI:', state);
}

// 派发动作
store.dispatch({ type: 'INCREMENT' });
store.dispatch({ type: 'INCREMENT' });
store.dispatch({ type: 'SET_USER', payload: { name: '张三' } });

// 取消订阅
unsubscribe();
```

### 4. WebSocket 消息处理

```javascript
class WebSocketManager {
  constructor(url) {
    this.url = url;
    this.eventBus = new EventEmitter();
    this.ws = null;
  }
  
  connect() {
    this.ws = new WebSocket(this.url);
    
    this.ws.onopen = () => {
      this.eventBus.emit('connected');
    };
    
    this.ws.onmessage = (event) => {
      const message = JSON.parse(event.data);
      this.eventBus.emit(`message:${message.type}`, message.data);
    };
    
    this.ws.onclose = () => {
      this.eventBus.emit('disconnected');
    };
    
    this.ws.onerror = (error) => {
      this.eventBus.emit('error', error);
    };
  }
  
  send(type, data) {
    if (this.ws && this.ws.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify({ type, data }));
    }
  }
  
  on(eventType, callback) {
    this.eventBus.on(eventType, callback);
  }
  
  off(eventType, callback) {
    this.eventBus.off(eventType, callback);
  }
}

// 使用
const wsManager = new WebSocketManager('ws://localhost:8080');

wsManager.on('connected', () => {
  console.log('WebSocket 已连接');
});

wsManager.on('message:chat', (message) => {
  console.log('收到聊天消息:', message);
});

wsManager.on('message:notification', (notification) => {
  console.log('收到通知:', notification);
});

wsManager.connect();
```

## 注意事项

1. **内存泄漏**：订阅后记得取消订阅，特别是在组件卸载时
2. **事件命名**：使用清晰、有层次结构的事件命名规范
3. **错误处理**：在回调函数中添加错误处理，避免影响其他订阅者
4. **事件泛滥**：避免过度使用发布-订阅模式，保持事件通道的简洁
5. **性能优化**：高频事件时考虑使用节流或防抖
6. **调试困难**：发布者和订阅者解耦使得调试变得困难，建议添加日志记录

## 观察者模式 vs 发布-订阅模式

### 对比表

| 特性 | 观察者模式 | 发布-订阅模式 |
|------|-----------|--------------|
| 耦合度 | 中等，观察者直接订阅主题 | 低，通过事件通道解耦 |
| 通信方式 | 直接通信 | 通过中介通信 |
| 灵活性 | 较低 | 较高 |
| 复杂度 | 简单 | 较复杂 |
| 异步支持 | 通常同步 | 支持异步 |
| 应用场景 | 简单的一对多关系 | 复杂的多对多关系 |

## 优缺点

### 优点
- 完全解耦发布者和订阅者
- 支持灵活的订阅和取消订阅
- 易于扩展和维护
- 支持异步通信
- 适合分布式系统

### 缺点
- 系统复杂度增加
- 消息传递可靠性需要自己保证
- 调试困难，难以追踪消息流向
- 可能产生消息积压
- 需要额外的事件管理机制

## 总结

发布-订阅模式是一种强大且灵活的消息传递模式，通过引入事件通道作为中介，实现了发布者和订阅者的完全解耦。这种模式在前端开发中有着广泛的应用，从简单的组件间通信到复杂的状态管理系统。

适用场景：
- 需要完全解耦的组件间通信
- 异步任务处理和状态管理
- 实时数据更新和推送
- 插件系统或模块化架构
- WebSocket 等长连接的消息处理

在实际项目中，可以结合使用观察者模式和发布-订阅模式，根据具体场景选择最合适的方案。简单场景使用观察者模式，复杂场景使用发布-订阅模式，以达到最佳的架构设计。