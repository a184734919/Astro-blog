---
title: TypeScript 装饰器详解
published: 2022-11-22
description: '装饰器的使用场景的详细介绍和学习笔记'
image: ''
tags: ["TypeScript","装饰器"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

TypeScript 装饰器（Decorators）是一种特殊的声明,可以附加到类、方法、属性、参数和访问器上,用于修改或扩展它们的行为。装饰器本质上是一个函数,它接收目标对象作为参数,并可以对其进行包装或增强。

装饰器是 TypeScript 的实验性功能,需要在 `tsconfig.json` 中启用:

```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

## 核心概念

装饰器的工作原理依赖于 JavaScript 的元编程能力。当装饰器应用到一个元素时:

1. **装饰器函数被调用**,传入目标对象和相关信息
2. **装饰器可以修改或替换目标对象**
3. **装饰器返回的新对象将替换原对象**(如果返回值存在)

装饰器执行顺序:
- **类装饰器**: 从下往上
- **成员装饰器**: 从内向外

## 基本用法

### 1. 类装饰器

类装饰器接收类构造函数作为参数:

```typescript
function Logger<T extends { new (...args: any[]): {} }>(
  constructor: T
) {
  return class extends constructor {
    constructor(...args: any[]) {
      console.log(`Creating instance of ${constructor.name}`);
      super(...args);
    }
  };
}

@Logger
class UserService {
  constructor(private name: string) {}

  greet() {
    return `Hello, ${this.name}!`;
  }
}

const user = new UserService('Alice');
// 输出: Creating instance of UserService
```

### 2. 方法装饰器

方法装饰器可以修改方法的行为:

```typescript
function LogMethod(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with args:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`${propertyKey} returned:`, result);
    return result;
  };
}

class Calculator {
  @LogMethod
  add(a: number, b: number) {
    return a + b;
  }
}

const calc = new Calculator();
calc.add(5, 3);
// 输出: Calling add with args: [5, 3]
// 输出: add returned: 8
```

### 3. 属性装饰器

```typescript
function DefaultValue(value: any) {
  return function (target: any, propertyKey: string) {
    let _value = value;

    Object.defineProperty(target, propertyKey, {
      get() {
        return _value;
      },
      set(newValue) {
        console.log(`Setting ${propertyKey} to ${newValue}`);
        _value = newValue;
      },
      enumerable: true,
      configurable: true,
    });
  };
}

class User {
  @DefaultValue('Guest')
  name: string;
}

const user = new User();
console.log(user.name); // Guest
user.name = 'Alice'; // Setting name to Alice
```

### 4. 参数装饰器

```typescript
function Required(target: any, propertyKey: string, parameterIndex: number) {
  const existingRequiredParameters =
    Reflect.getMetadata('required', target, propertyKey) || [];
  existingRequiredParameters.push(parameterIndex);
  Reflect.defineMetadata(
    'required',
    existingRequiredParameters,
    target,
    propertyKey
  );
}

class FormValidator {
  validate(@Required name: string, @Required email: string) {
    console.log(`Validating ${name} (${email})`);
  }
}
```

### 5. 访问器装饰器

```typescript
function ValidateRange(min: number, max: number) {
  return function (
    target: any,
    propertyKey: string,
    descriptor: PropertyDescriptor
  ) {
    const originalSetter = descriptor.set;

    descriptor.set = function (this: any, value: number) {
      if (value < min || value > max) {
        throw new Error(`${propertyKey} must be between ${min} and ${max}`);
      }
      originalSetter?.call(this, value);
    };
  };
}

class Temperature {
  private _celsius: number = 0;

  @ValidateRange(-273.15, 1000)
  set celsius(value: number) {
    this._celsius = value;
  }

  get celsius(): number {
    return this._celsius;
  }
}
```

## 实际应用

### 1. 依赖注入

```typescript
const ServiceRegistry = new Map<string, any>();

function Injectable(token: string) {
  return function (constructor: Function) {
    ServiceRegistry.set(token, new constructor());
  };
}

function Inject(token: string) {
  return function (target: any, propertyKey: string) {
    Object.defineProperty(target, propertyKey, {
      get() {
        return ServiceRegistry.get(token);
      },
    });
  };
}

@Injectable('database')
class DatabaseService {
  query() {
    return 'Query result';
  }
}

@Injectable('logger')
class LoggerService {
  log(message: string) {
    console.log(message);
  }
}

class UserService {
  @Inject('database')
  database!: DatabaseService;

  @Inject('logger')
  logger!: LoggerService;

  getUser() {
    const data = this.database.query();
    this.logger.log(data);
    return data;
  }
}

const userService = new UserService();
userService.getUser();
```

### 2. 性能监控

```typescript
function MeasurePerformance(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    const start = performance.now();
    const result = originalMethod.apply(this, args);
    const end = performance.now();
    console.log(`${propertyKey} took ${end - start}ms`);
    return result;
  };
}

class DataProcessor {
  @MeasurePerformance
  processLargeData() {
    // 模拟耗时操作
    let sum = 0;
    for (let i = 0; i < 1000000; i++) {
      sum += i;
    }
    return sum;
  }
}

const processor = new DataProcessor();
processor.processLargeData();
```

### 3. 权限验证

```typescript
function RequireRole(role: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = function (...args: any[]) {
      const currentUser = (this as any).currentUser;
      if (!currentUser || currentUser.role !== role) {
        throw new Error(`Access denied. Required role: ${role}`);
      }
      return originalMethod.apply(this, args);
    };
  };
}

class AdminPanel {
  currentUser = { role: 'admin' };

  @RequireRole('admin')
  deleteUser(userId: string) {
    console.log(`Deleting user ${userId}`);
  }

  @RequireRole('moderator')
  banUser(userId: string) {
    console.log(`Banning user ${userId}`);
  }
}

const panel = new AdminPanel();
panel.deleteUser('123'); // 成功
// panel.banUser('456'); // 抛出错误: Access denied
```

### 4. 防抖和节流

```typescript
function Debounce(delay: number) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    let timeoutId: NodeJS.Timeout | null = null;
    const originalMethod = descriptor.value;

    descriptor.value = function (...args: any[]) {
      if (timeoutId) {
        clearTimeout(timeoutId);
      }
      timeoutId = setTimeout(() => {
        originalMethod.apply(this, args);
        timeoutId = null;
      }, delay);
    };
  };
}

function Throttle(delay: number) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    let lastCall = 0;
    const originalMethod = descriptor.value;

    descriptor.value = function (...args: any[]) {
      const now = Date.now();
      if (now - lastCall >= delay) {
        originalMethod.apply(this, args);
        lastCall = now;
      }
    };
  };
}

class SearchComponent {
  @Debounce(300)
  search(query: string) {
    console.log(`Searching for: ${query}`);
  }

  @Throttle(1000)
  trackEvent(eventName: string) {
    console.log(`Tracking event: ${eventName}`);
  }
}
```

## 注意事项

1. **装饰器是实验性功能**,标准仍在发展中,可能在未来版本中发生变化

2. **执行顺序很重要**:
   - 类装饰器: 从下往上
   - 成员装饰器: 先参数,再访问器,然后是方法和属性

3. **属性装饰器不能修改属性值**,只能添加元数据

4. **装饰器工厂**支持参数传递,更灵活:
   ```typescript
   function FactoryDecorator(options: { name: string }) {
     return function (target: any, propertyKey: string) {
       console.log(`${options.name} applied to ${propertyKey}`);
     };
   }
   ```

5. **性能考虑**: 装饰器会在类定义时立即执行,避免在装饰器中进行复杂计算

6. **反射元数据**: 使用 `reflect-metadata` 库可以存储和读取装饰器元数据:
   ```bash
   npm install reflect-metadata
   ```
   ```typescript
   import 'reflect-metadata';

   @Reflect.metadata('key', 'value')
   class MyClass {
     @Reflect.metadata('methodKey', 'methodValue')
     method() {}
   }
   ```

## 总结

TypeScript 装饰器是一种强大的元编程工具,可以帮助我们:

- **减少重复代码**: 通过装饰器实现横切关注点
- **增强可读性**: 声明式的方式更直观
- **提高可维护性**: 逻辑集中管理

合理使用装饰器可以让代码更加简洁、优雅,但也要注意避免过度使用导致代码难以理解。在实际项目中,装饰器常用于 AOP(面向切面编程)、依赖注入、日志记录、性能监控等场景。