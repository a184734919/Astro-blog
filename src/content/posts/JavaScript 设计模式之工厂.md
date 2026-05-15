---
title: JavaScript 设计模式之工厂
published: 2022-08-18
description: '工厂模式的实现和应用的详细介绍和学习笔记'
image: ''
tags: ["JS","设计模式"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

工厂模式（Factory Pattern）是一种创建型设计模式，它提供了一种创建对象的最佳方式。在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。

## 核心概念

工厂模式主要解决接口选择的问题，核心思想是将对象的创建和使用分离。通过工厂类，我们可以根据传入的参数动态创建不同类型的对象，而不需要知道具体的创建细节。

工厂模式的三种形式：
- **简单工厂模式**（Simple Factory）
- **工厂方法模式**（Factory Method）
- **抽象工厂模式**（Abstract Factory）

## 基本用法

### 简单工厂模式

```javascript
// 简单工厂模式示例
class Car {
  constructor(type) {
    this.type = type;
  }
  
  drive() {
    console.log(`驾驶 ${this.type} 汽车`);
  }
}

class Bike {
  constructor(type) {
    this.type = type;
  }
  
  ride() {
    console.log(`骑行 ${this.type} 自行车`);
  }
}

// 简单工厂
class VehicleFactory {
  static createVehicle(type) {
    switch(type) {
      case 'car':
        return new Car('小汽车');
      case 'bike':
        return new Bike('自行车');
      default:
        throw new Error('未知车辆类型');
    }
  }
}

// 使用
const car = VehicleFactory.createVehicle('car');
car.drive(); // 驾驶 小汽车 汽车

const bike = VehicleFactory.createVehicle('bike');
bike.ride(); // 骑行 自行车 自行车
```

### 工厂方法模式

```javascript
// 工厂方法模式示例
class LoggerFactory {
  createLogger() {
    throw new Error('子类必须实现 createLogger 方法');
  }
}

class ConsoleLoggerFactory extends LoggerFactory {
  createLogger() {
    return new ConsoleLogger();
  }
}

class FileLoggerFactory extends LoggerFactory {
  createLogger() {
    return new FileLogger();
  }
}

class ConsoleLogger {
  log(message) {
    console.log(`[Console] ${message}`);
  }
}

class FileLogger {
  log(message) {
    // 模拟写入文件
    console.log(`[File] ${message}`);
  }
}

// 使用
const consoleFactory = new ConsoleLoggerFactory();
const consoleLogger = consoleFactory.createLogger();
consoleLogger.log('这是一条日志'); // [Console] 这是一条日志

const fileFactory = new FileLoggerFactory();
const fileLogger = fileFactory.createLogger();
fileLogger.log('写入文件的日志'); // [File] 写入文件的日志
```

### 抽象工厂模式

```javascript
// 抽象工厂模式示例
class Button {
  render() {
    throw new Error('子类必须实现 render 方法');
  }
}

class Input {
  render() {
    throw new Error('子类必须实现 render 方法');
  }
}

class LightThemeFactory {
  createButton() {
    return new LightButton();
  }
  
  createInput() {
    return new LightInput();
  }
}

class DarkThemeFactory {
  createButton() {
    return new DarkButton();
  }
  
  createInput() {
    return new DarkInput();
  }
}

class LightButton extends Button {
  render() {
    return '<button class="light">按钮</button>';
  }
}

class DarkButton extends Button {
  render() {
    return '<button class="dark">按钮</button>';
  }
}

class LightInput extends Input {
  render() {
    return '<input class="light" />';
  }
}

class DarkInput extends Input {
  render() {
    return '<input class="dark" />';
  }
}

// 使用
const lightFactory = new LightThemeFactory();
const lightButton = lightFactory.createButton();
const lightInput = lightFactory.createInput();
console.log(lightButton.render()); // <button class="light">按钮</button>
console.log(lightInput.render()); // <input class="light" />

const darkFactory = new DarkThemeFactory();
const darkButton = darkFactory.createButton();
const darkInput = darkFactory.createInput();
console.log(darkButton.render()); // <button class="dark">按钮</button>
console.log(darkInput.render()); // <input class="dark" />
```

## 实际应用

### 1. Vue 组件创建

```javascript
// Vue 中使用工厂模式创建组件
const componentFactory = {
  create(type, props = {}) {
    const components = {
      'alert': Vue.extend(AlertComponent),
      'modal': Vue.extend(ModalComponent),
      'toast': Vue.extend(ToastComponent)
    };
    
    const Component = components[type];
    if (!Component) {
      throw new Error(`未知的组件类型: ${type}`);
    }
    
    return new Component({
      propsData: props
    });
  }
};

// 使用
const alert = componentFactory.create('alert', { message: '提示信息' });
```

### 2. XMLHttpRequest 工厂

```javascript
class AjaxFactory {
  static createRequest() {
    if (window.XMLHttpRequest) {
      return new XMLHttpRequest();
    } else if (window.ActiveXObject) {
      // 兼容旧版 IE
      return new ActiveXObject('Microsoft.XMLHTTP');
    }
    throw new Error('浏览器不支持 Ajax');
  }
}

// 使用
const xhr = AjaxFactory.createRequest();
xhr.open('GET', '/api/data', true);
xhr.send();
```

### 3. 图表组件工厂

```javascript
class ChartFactory {
  static createChart(type, container, data) {
    const chartTypes = {
      'line': LineChart,
      'bar': BarChart,
      'pie': PieChart
    };
    
    const ChartClass = chartTypes[type];
    if (!ChartClass) {
      throw new Error(`不支持的图表类型: ${type}`);
    }
    
    return new ChartClass(container, data);
  }
}

// 使用
const lineChart = ChartFactory.createChart('line', '#chart', data);
const barChart = ChartFactory.createChart('bar', '#chart2', data2);
```

## 注意事项

1. **避免过度使用**：只有在需要根据条件动态创建对象时才使用工厂模式，简单的对象创建可以直接使用构造函数或类
2. **保持工厂类单一职责**：每个工厂类应该只负责创建一组相关的对象
3. **考虑扩展性**：设计时要考虑未来可能添加的新类型，使用抽象类或接口定义规范
4. **性能影响**：工厂模式会增加一层抽象，对性能有轻微影响，但在大多数场景下可以忽略
5. **遵循开闭原则**：新增产品类型时应该扩展工厂，而不是修改已有代码

## 优缺点

### 优点
- 解耦对象的创建和使用，降低耦合度
- 符合开闭原则，易于扩展
- 代码复用性好，统一管理对象创建逻辑
- 隐藏创建细节，提高安全性

### 缺点
- 增加了系统的复杂度
- 需要引入抽象层，理解成本增加
- 简单场景下使用可能显得过度设计

## 总结

工厂模式是一种非常实用的创建型设计模式，它通过将对象的创建和使用分离，提高了代码的灵活性和可维护性。在实际开发中，应根据具体场景选择合适的工厂模式类型：

- **简单工厂**：创建对象种类少、逻辑简单时使用
- **工厂方法**：需要支持扩展、遵循开闭原则时使用
- **抽象工厂**：需要创建多个相关对象族时使用

合理使用工厂模式可以大大提升代码质量和可维护性。