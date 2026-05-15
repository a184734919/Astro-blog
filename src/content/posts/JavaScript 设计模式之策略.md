---
title: JavaScript 设计模式之策略
published: 2022-09-07
description: '策略模式的实现和应用的详细介绍和学习笔记'
image: ''
tags: ["设计模式"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

策略模式(Strategy Pattern)是一种行为型设计模式，它定义了一系列算法，并将每个算法封装起来，使它们可以相互替换。策略模式让算法独立于使用它的客户端而变化。

在 JavaScript 中，策略模式可以用来解决多个条件分支的问题，将不同的业务逻辑封装成独立的策略类或函数，提高代码的可维护性和扩展性。

## 核心概念

策略模式主要包含以下几个核心概念：

1. **策略接口(Strategy Interface)**：定义所有策略的公共接口
2. **具体策略(Concrete Strategy)**：实现了策略接口的具体算法
3. **上下文(Context)**：使用策略的对象，维护对策略的引用

**策略模式的优点：**
- 算法可以自由切换
- 避免使用多重条件判断
- 扩展性良好，符合开闭原则
- 提高代码的可维护性

**策略模式的缺点：**
- 策略类数量会增加
- 客户端必须知道所有策略类
- 策略模式需要维护额外的策略对象

## 基本用法

### 1. 简单的策略模式实现

```javascript
// 定义策略对象
const strategies = {
  // 加法策略
  add: function(a, b) {
    return a + b;
  },
  // 减法策略
  subtract: function(a, b) {
    return a - b;
  },
  // 乘法策略
  multiply: function(a, b) {
    return a * b;
  },
  // 除法策略
  divide: function(a, b) {
    if (b === 0) {
      throw new Error('除数不能为0');
    }
    return a / b;
  }
};

// 上下文类
function Calculator() {
  this.strategy = null;
}

Calculator.prototype.setStrategy = function(strategy) {
  this.strategy = strategy;
};

Calculator.prototype.execute = function(a, b) {
  if (!this.strategy) {
    throw new Error('未设置策略');
  }
  return this.strategy(a, b);
};

// 使用示例
const calculator = new Calculator();
calculator.setStrategy(strategies.add);
console.log(calculator.execute(10, 5)); // 输出: 15

calculator.setStrategy(strategies.subtract);
console.log(calculator.execute(10, 5)); // 输出: 5
```

### 2. 使用 ES6 类实现策略模式

```javascript
// 定义策略接口
class Strategy {
  execute(a, b) {
    throw new Error('必须实现 execute 方法');
  }
}

// 具体策略：加法
class AddStrategy extends Strategy {
  execute(a, b) {
    return a + b;
  }
}

// 具体策略：减法
class SubtractStrategy extends Strategy {
  execute(a, b) {
    return a - b;
  }
}

// 具体策略：乘法
class MultiplyStrategy extends Strategy {
  execute(a, b) {
    return a * b;
  }
}

// 上下文类
class Calculator {
  constructor(strategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy) {
    this.strategy = strategy;
  }

  execute(a, b) {
    return this.strategy.execute(a, b);
  }
}

// 使用示例
const calculator = new Calculator(new AddStrategy());
console.log(calculator.execute(10, 5)); // 输出: 15

calculator.setStrategy(new MultiplyStrategy());
console.log(calculator.execute(10, 5)); // 输出: 50
```

### 3. 表单验证策略模式

```javascript
// 定义验证策略
const validationStrategies = {
  // 必填验证
  required: (value) => {
    if (!value || value.trim() === '') {
      return '此项不能为空';
    }
    return '';
  },
  // 邮箱验证
  email: (value) => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(value)) {
      return '请输入有效的邮箱地址';
    }
    return '';
  },
  // 手机号验证
  phone: (value) => {
    const phoneRegex = /^1[3-9]\d{9}$/;
    if (!phoneRegex.test(value)) {
      return '请输入有效的手机号';
    }
    return '';
  },
  // 最小长度验证
  minLength: (value, min) => {
    if (value.length < min) {
      return `长度不能小于 ${min} 个字符`;
    }
    return '';
  },
  // 最大长度验证
  maxLength: (value, max) => {
    if (value.length > max) {
      return `长度不能超过 ${max} 个字符`;
    }
    return '';
  }
};

// 表单验证器
class Validator {
  constructor() {
    this.rules = [];
  }

  // 添加验证规则
  addRule(field, strategy, ...args) {
    this.rules.push({ field, strategy, args });
    return this;
  }

  // 执行验证
  validate(data) {
    const errors = {};
    let isValid = true;

    for (const rule of this.rules) {
      const value = data[rule.field];
      const strategy = validationStrategies[rule.strategy];

      if (strategy) {
        const error = strategy(value, ...rule.args);
        if (error) {
          errors[rule.field] = error;
          isValid = false;
        }
      }
    }

    return { isValid, errors };
  }
}

// 使用示例
const validator = new Validator();
validator
  .addRule('username', 'required')
  .addRule('username', 'minLength', 3)
  .addRule('username', 'maxLength', 20)
  .addRule('email', 'required')
  .addRule('email', 'email')
  .addRule('phone', 'phone');

const formData = {
  username: 'test',
  email: 'invalid-email',
  phone: '12345678901'
};

const result = validator.validate(formData);
console.log(result);
// 输出: {
//   isValid: false,
//   errors: { email: '请输入有效的邮箱地址', phone: '请输入有效的手机号' }
// }
```

## 实际应用

### 1. 支付方式策略

```javascript
// 支付策略
const paymentStrategies = {
  // 支付宝支付
  alipay: {
    pay(amount, orderInfo) {
      console.log(`使用支付宝支付 ${amount} 元`);
      console.log('订单信息:', orderInfo);
      return Promise.resolve({
        success: true,
        paymentId: 'ALIPAY_' + Date.now()
      });
    }
  },
  // 微信支付
  wechat: {
    pay(amount, orderInfo) {
      console.log(`使用微信支付 ${amount} 元`);
      console.log('订单信息:', orderInfo);
      return Promise.resolve({
        success: true,
        paymentId: 'WECHAT_' + Date.now()
      });
    }
  },
  // 银行卡支付
  bankCard: {
    pay(amount, orderInfo) {
      console.log(`使用银行卡支付 ${amount} 元`);
      console.log('订单信息:', orderInfo);
      return Promise.resolve({
        success: true,
        paymentId: 'BANK_' + Date.now()
      });
    }
  }
};

// 支付上下文
class PaymentContext {
  constructor(strategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy) {
    this.strategy = strategy;
  }

  pay(amount, orderInfo) {
    return this.strategy.pay(amount, orderInfo);
  }
}

// 使用示例
async function processPayment() {
  const payment = new PaymentContext(paymentStrategies.alipay);
  const result1 = await payment.pay(100, { orderId: 'ORDER_001' });
  console.log('支付结果:', result1);

  payment.setStrategy(paymentStrategies.wechat);
  const result2 = await payment.pay(200, { orderId: 'ORDER_002' });
  console.log('支付结果:', result2);
}

processPayment();
```

### 2. 动画策略模式

```javascript
// 动画策略
const animationStrategies = {
  // 渐入动画
  fadeIn: function(element, duration = 500) {
    element.style.opacity = '0';
    element.style.transition = `opacity ${duration}ms`;
    return new Promise((resolve) => {
      setTimeout(() => {
        element.style.opacity = '1';
        setTimeout(resolve, duration);
      }, 10);
    });
  },
  // 滑入动画
  slideIn: function(element, duration = 500, direction = 'left') {
    const startTransform = direction === 'left' ? 'translateX(-100%)' : 'translateY(-100%)';
    element.style.transform = startTransform;
    element.style.transition = `transform ${duration}ms`;
    return new Promise((resolve) => {
      setTimeout(() => {
        element.style.transform = 'translateX(0)';
        setTimeout(resolve, duration);
      }, 10);
    });
  },
  // 缩放动画
  scale: function(element, duration = 500) {
    element.style.transform = 'scale(0)';
    element.style.transition = `transform ${duration}ms`;
    return new Promise((resolve) => {
      setTimeout(() => {
        element.style.transform = 'scale(1)';
        setTimeout(resolve, duration);
      }, 10);
    });
  }
};

// 动画执行器
class Animator {
  constructor(strategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy) {
    this.strategy = strategy;
  }

  animate(element, ...args) {
    return this.strategy(element, ...args);
  }
}

// 使用示例
const animator = new Animator(animationStrategies.fadeIn);
const box = document.createElement('div');
box.style.width = '100px';
box.style.height = '100px';
box.style.background = 'red';
document.body.appendChild(box);

animator.animate(box, 1000).then(() => {
  console.log('动画完成');
});
```

### 3. 缓存策略模式

```javascript
// 缓存策略
const cacheStrategies = {
  // 内存缓存
  memory: {
    cache: new Map(),
    get(key) {
      return this.cache.get(key);
    },
    set(key, value, ttl = 60000) {
      this.cache.set(key, {
        value,
        expire: Date.now() + ttl
      });
    },
    has(key) {
      const item = this.cache.get(key);
      if (!item) return false;
      if (Date.now() > item.expire) {
        this.cache.delete(key);
        return false;
      }
      return true;
    },
    clear() {
      this.cache.clear();
    }
  },
  // LocalStorage 缓存
  localStorage: {
    get(key) {
      const item = localStorage.getItem(key);
      if (!item) return null;
      const parsed = JSON.parse(item);
      if (Date.now() > parsed.expire) {
        localStorage.removeItem(key);
        return null;
      }
      return parsed.value;
    },
    set(key, value, ttl = 60000) {
      const item = {
        value,
        expire: Date.now() + ttl
      };
      localStorage.setItem(key, JSON.stringify(item));
    },
    has(key) {
      return this.get(key) !== null;
    },
    clear() {
      localStorage.clear();
    }
  }
};

// 缓存管理器
class CacheManager {
  constructor(strategy = 'memory') {
    this.setStrategy(strategy);
  }

  setStrategy(strategy) {
    this.strategy = cacheStrategies[strategy] || cacheStrategies.memory;
  }

  get(key) {
    return this.strategy.get(key);
  }

  set(key, value, ttl) {
    this.strategy.set(key, value, ttl);
  }

  has(key) {
    return this.strategy.has(key);
  }

  clear() {
    this.strategy.clear();
  }
}

// 使用示例
const cache = new CacheManager('memory');

// 设置缓存
cache.set('user:1', { name: '张三', age: 25 }, 60000);

// 获取缓存
const user = cache.get('user:1');
console.log(user); // { name: '张三', age: 25 }

// 检查缓存是否存在
console.log(cache.has('user:1')); // true

// 切换缓存策略
cache.setStrategy('localStorage');
cache.set('config', { theme: 'dark' }, 86400000);
```

## 注意事项

1. **策略的选择**：策略模式适用于有多种算法可以互换的场景，不要为了使用模式而使用模式
2. **策略的粒度**：策略粒度应该适中，过于细分会增加复杂性，过于粗分会失去灵活性
3. **性能考虑**：频繁切换策略可能带来性能开销，需要根据实际场景评估
4. **错误处理**：每个策略都应该有完善的错误处理机制
5. **策略管理**：当策略数量较多时，考虑使用策略工厂模式来管理策略对象
6. **内存管理**：策略对象如果持有状态，需要注意内存泄漏问题
7. **文档完善**：为每个策略提供清晰的文档和使用示例

## 总结

策略模式是一种灵活且强大的设计模式，它通过封装算法族，使它们可以相互替换，让算法独立于使用它的客户端而变化。

**策略模式的最佳实践：**
- 使用策略模式消除大量的 if-else 或 switch-case 语句
- 策略应该是无状态的，避免在策略中保存状态信息
- 合理组织策略的命名和结构，提高代码可读性
- 结合工厂模式创建策略对象，降低客户端复杂度
- 为策略提供默认实现，简化使用方式

通过合理使用策略模式，我们可以编写出更加灵活、可维护和可扩展的代码。在实际项目中，表单验证、支付方式、缓存策略等场景都是策略模式的典型应用案例。