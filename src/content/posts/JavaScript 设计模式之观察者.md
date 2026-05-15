---
title: JavaScript 设计模式之观察者
published: 2022-08-25
description: '观察者模式的实现和应用的详细介绍和学习笔记'
image: ''
tags: ["设计模式"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

观察者模式（Observer Pattern）是一种行为型设计模式，它定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

观察者模式又被称为发布-订阅（Publish-Subscribe）模式的简化版，但它们在实现上有一些区别。观察者模式中，观察者（Observer）和被观察者（Subject）直接交互，而发布-订阅模式通过一个调度中心来实现解耦。

## 核心概念

观察者模式的核心概念包括：
- **Subject（被观察者/主题）**：维护一组观察者，提供添加、删除和通知观察者的方法
- **Observer（观察者）**：定义一个更新接口，以便在得到主题通知时更新自己
- **ConcreteSubject（具体被观察者）**：将有关状态存入具体观察者对象，在状态改变时通知观察者
- **ConcreteObserver（具体观察者）**：维护一个指向具体主题对象的引用，实现更新接口

## 基本用法

### 基础观察者模式实现

```javascript
// 观察者类
class Observer {
  constructor(name) {
    this.name = name;
  }
  
  update(message) {
    console.log(`${this.name} 收到消息: ${message}`);
  }
}

// 被观察者类
class Subject {
  constructor() {
    this.observers = [];
  }
  
  // 添加观察者
  attach(observer) {
    this.observers.push(observer);
    console.log(`${observer.name} 已被添加到观察列表`);
  }
  
  // 移除观察者
  detach(observer) {
    const index = this.observers.indexOf(observer);
    if (index > -1) {
      this.observers.splice(index, 1);
      console.log(`${observer.name} 已从观察列表中移除`);
    }
  }
  
  // 通知所有观察者
  notify(message) {
    console.log('发送消息给所有观察者...');
    this.observers.forEach(observer => {
      observer.update(message);
    });
  }
}

// 使用示例
const subject = new Subject();

const observer1 = new Observer('观察者A');
const observer2 = new Observer('观察者B');
const observer3 = new Observer('观察者C');

subject.attach(observer1);
subject.attach(observer2);
subject.attach(observer3);

subject.notify('系统更新已完成');
// 输出:
// 观察者A 收到消息: 系统更新已完成
// 观察者B 收到消息: 系统更新已完成
// 观察者C 收到消息: 系统更新已完成

subject.detach(observer2);
subject.notify('新的通知');
// 输出:
// 观察者A 收到消息: 新的通知
// 观察者C 收到消息: 新的通知
```

### 带状态的观察者模式

```javascript
class NewsAgency {
  constructor() {
    this.subscribers = [];
    this.news = '';
  }
  
  subscribe(subscriber) {
    this.subscribers.push(subscriber);
  }
  
  unsubscribe(subscriber) {
    this.subscribers = this.subscribers.filter(sub => sub !== subscriber);
  }
  
  publish(news) {
    this.news = news;
    this.notifyAll();
  }
  
  notifyAll() {
    this.subscribers.forEach(subscriber => subscriber.update(this.news));
  }
}

class NewsChannel {
  constructor(name) {
    this.name = name;
    this.currentNews = '';
  }
  
  update(news) {
    this.currentNews = news;
    console.log(`${this.name} 收到新闻: ${news}`);
  }
}

// 使用
const agency = new NewsAgency();

const channel1 = new NewsChannel('新闻频道 A');
const channel2 = new NewsChannel('新闻频道 B');

agency.subscribe(channel1);
agency.subscribe(channel2);

agency.publish('今日头条：重大科技突破！');
// 输出:
// 新闻频道 A 收到新闻: 今日头条：重大科技突破！
// 新闻频道 B 收到新闻: 今日头条：重大科技突破！
```

## 实际应用

### 1. Vue 的响应式系统

Vue 2 的响应式系统就是基于观察者模式实现的：

```javascript
// 简化的 Vue 响应式系统
class Dep {
  constructor() {
    this.subscribers = [];
  }
  
  depend() {
    if (activeSub) {
      this.subscribers.push(activeSub);
    }
  }
  
  notify() {
    this.subscribers.forEach(sub => sub.update());
  }
}

class Watcher {
  constructor(vm, expOrFn, callback) {
    this.vm = vm;
    this.expOrFn = expOrFn;
    this.callback = callback;
    this.value = this.get();
  }
  
  get() {
    activeSub = this;
    const value = this.vm[this.expOrFn];
    activeSub = null;
    return value;
  }
  
  update() {
    const newValue = this.get();
    if (newValue !== this.value) {
      const oldValue = this.value;
      this.value = newValue;
      this.callback.call(this.vm, newValue, oldValue);
    }
  }
}

// 使用
let activeSub = null;

const data = {
  message: 'Hello Vue'
};

const dep = new Dep();
let message = data.message;

Object.defineProperty(data, 'message', {
  get() {
    dep.depend();
    return message;
  },
  set(newValue) {
    if (newValue !== message) {
      message = newValue;
      dep.notify();
    }
  }
});

// 创建观察者
const watcher = new Watcher(data, 'message', (newVal, oldVal) => {
  console.log(`数据变化: ${oldVal} -> ${newVal}`);
});

data.message = 'Hello World';
// 输出: 数据变化: Hello Vue -> Hello World
```

### 2. DOM 事件监听

DOM 事件监听是观察者模式的经典应用：

```javascript
// DOM 元素作为被观察者
const button = document.getElementById('submit-btn');

// 添加观察者（事件监听器）
button.addEventListener('click', function() {
  console.log('处理按钮点击');
});

button.addEventListener('click', function() {
  console.log('记录点击日志');
});

button.addEventListener('click', function() {
  console.log('发送统计数据');
});

// 触发事件（通知所有观察者）
button.click();
```

### 3. 网页通知系统

```javascript
class NotificationSystem {
  constructor() {
    this.listeners = [];
  }
  
  addListener(listener) {
    this.listeners.push(listener);
  }
  
  removeListener(listener) {
    this.listeners = this.listeners.filter(l => l !== listener);
  }
  
  notify(notification) {
    this.listeners.forEach(listener => listener.onNotification(notification));
  }
}

class DesktopNotification {
  onNotification(notification) {
    if (Notification.permission === 'granted') {
      new Notification(notification.title, {
        body: notification.message,
        icon: notification.icon
      });
    }
  }
}

class ToastNotification {
  onNotification(notification) {
    // 显示 Toast 提示
    const toast = document.createElement('div');
    toast.className = 'toast';
    toast.textContent = notification.message;
    document.body.appendChild(toast);
    
    setTimeout(() => {
      toast.remove();
    }, 3000);
  }
}

// 使用
const notificationSystem = new NotificationSystem();
notificationSystem.addListener(new DesktopNotification());
notificationSystem.addListener(new ToastNotification());

notificationSystem.notify({
  title: '新消息',
  message: '您收到了一条新消息',
  icon: '/icon.png'
});
```

## 注意事项

1. **避免循环引用**：观察者和被观察者之间要小心处理引用关系，防止内存泄漏
2. **异步通知**：在复杂场景中，考虑使用异步方式通知观察者，避免阻塞主线程
3. **错误处理**：观察者的 update 方法应该有错误处理机制，避免一个观察者出错影响其他观察者
4. **通知顺序**：明确观察者的执行顺序，必要时可以添加优先级机制
5. **内存管理**：及时移除不再需要的观察者，防止内存泄漏
6. **性能考虑**：观察者数量过多时，通知机制可能成为性能瓶颈

## 观察者模式 vs 发布-订阅模式

### 观察者模式
- 观察者和被观察者直接交互
- 耦合度相对较高
- 实现简单，适合小规模应用

### 发布-订阅模式
- 通过调度中心（事件总线）实现解耦
- 耦合度低，灵活性强
- 支持异步通信
- 适合复杂的大规模应用

## 优缺点

### 优点
- 支持松耦合，主题和观察者之间是抽象耦合
- 支持广播通信，一个消息可以传递给多个订阅者
- 符合开闭原则，可以随时增加观察者
- 抽象耦合，主题和观察者都可以独立扩展

### 缺点
- 观察者数量过多时，通知会消耗较多性能
- 如果观察者和被观察者之间有循环依赖，可能导致循环调用
- 观察者不知道被观察者是如何变化的，只知道状态改变了
- 异步操作会增加系统的复杂性

## 总结

观察者模式是一种非常实用的设计模式，在前端开发中有着广泛的应用。从 DOM 事件监听到 Vue 的响应式系统，观察者模式都发挥着重要作用。

合理使用观察者模式可以：
- 降低对象间的耦合度
- 提高代码的可维护性和可扩展性
- 实现灵活的事件驱动架构

在实际项目中，要根据场景选择合适的实现方式，小规模应用可以使用简单的观察者模式，复杂场景可以考虑使用发布-订阅模式或其他成熟的事件库。