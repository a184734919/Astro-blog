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

### 5. 数据验证

```typescript
function Validate(validationFn: (value: any) => boolean, errorMessage: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalSetter = descriptor.set;

    descriptor.set = function (this: any, value: any) {
      if (!validationFn(value)) {
        throw new Error(errorMessage);
      }
      originalSetter?.call(this, value);
    };
  };
}

class UserProfile {
  private _email: string = '';
  private _age: number = 0;

  @Validate(
    (value) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
    'Invalid email format'
  )
  set email(value: string) {
    this._email = value;
  }

  get email(): string {
    return this._email;
  }

  @Validate(
    (value) => value >= 0 && value <= 150,
    'Age must be between 0 and 150'
  )
  set age(value: number) {
    this._age = value;
  }

  get age(): number {
    return this._age;
  }
}

const profile = new UserProfile();
profile.email = 'valid@example.com'; // 成功
// profile.email = 'invalid'; // 抛出错误: Invalid email format
```

### 6. 缓存装饰器

```typescript
function Cache(ttl: number = 60000) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    const cache = new Map<string, { value: any; expiresAt: number }>();

    descriptor.value = function (...args: any[]) {
      const cacheKey = `${propertyKey}:${JSON.stringify(args)}`;
      const cached = cache.get(cacheKey);

      if (cached && Date.now() < cached.expiresAt) {
        console.log(`Cache hit for ${propertyKey}`);
        return cached.value;
      }

      console.log(`Cache miss for ${propertyKey}`);
      const result = originalMethod.apply(this, args);
      cache.set(cacheKey, {
        value: result,
        expiresAt: Date.now() + ttl
      });
      return result;
    };
  };
}

class ApiService {
  @Cache(5000) // 5秒缓存
  async fetchUser(id: number): Promise<any> {
    console.log(`Fetching user ${id} from API`);
    // 模拟 API 调用
    await new Promise(resolve => setTimeout(resolve, 100));
    return { id, name: `User ${id}` };
  }
}

const api = new ApiService();
api.fetchUser(1); // 调用 API
api.fetchUser(1); // 从缓存读取
```

### 7. 重试装饰器

```typescript
function Retry(maxAttempts: number = 3, delay: number = 1000) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = async function (...args: any[]) {
      let lastError: Error | null = null;

      for (let attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
          return await originalMethod.apply(this, args);
        } catch (error) {
          lastError = error as Error;
          console.warn(`Attempt ${attempt} failed for ${propertyKey}:`, error);

          if (attempt < maxAttempts) {
            await new Promise(resolve => setTimeout(resolve, delay));
          }
        }
      }

      throw lastError;
    };
  };
}

class ExternalService {
  @Retry(3, 1000)
  async callExternalApi(): Promise<string> {
    // 模拟可能失败的外部 API 调用
    if (Math.random() > 0.7) {
      throw new Error('API call failed');
    }
    return 'Success';
  }
}

const service = new ExternalService();
service.callExternalApi(); // 可能会重试多次
```

### 8. 限流装饰器

```typescript
function RateLimit(maxCalls: number, timeWindow: number) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    const calls: number[] = [];

    descriptor.value = function (...args: any[]) {
      const now = Date.now();
      // 清理过期的调用记录
      const validCalls = calls.filter(callTime => now - callTime < timeWindow);
      calls.length = 0;
      calls.push(...validCalls);

      if (calls.length >= maxCalls) {
        throw new Error(`Rate limit exceeded for ${propertyKey}`);
      }

      calls.push(now);
      return originalMethod.apply(this, args);
    };
  };
}

class ApiClient {
  @RateLimit(5, 60000) // 每分钟最多5次调用
  async expensiveOperation(): Promise<string> {
    console.log('Performing expensive operation');
    return 'Operation completed';
  }
}

const client = new ApiClient();
// 调用超过5次会抛出错误
```

## 高级装饰器模式

### 1. 装饰器组合

```typescript
function LogExecution(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  descriptor.value = function (...args: any[]) {
    console.log(`Executing ${propertyKey}`);
    return originalMethod.apply(this, args);
  };
}

function MeasureTime(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  descriptor.value = function (...args: any[]) {
    const start = Date.now();
    const result = originalMethod.apply(this, args);
    const end = Date.now();
    console.log(`${propertyKey} took ${end - start}ms`);
    return result;
  };
}

class DataManager {
  @LogExecution
  @MeasureTime
  @Cache(1000)
  async fetchData(id: string): Promise<any> {
    // 模拟数据获取
    await new Promise(resolve => setTimeout(resolve, 100));
    return { id, data: `Data for ${id}` };
  }
}
```

### 2. 装饰器工厂

```typescript
function ApiResponse(statusCode: number, message: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    descriptor.value = async function (...args: any[]) {
      try {
        const data = await originalMethod.apply(this, args);
        return {
          success: true,
          statusCode,
          message,
          data
        };
      } catch (error) {
        return {
          success: false,
          statusCode: 500,
          message: 'Internal server error',
          error: error instanceof Error ? error.message : 'Unknown error'
        };
      }
    };
  };
}

class UserController {
  @ApiResponse(200, 'User retrieved successfully')
  async getUser(id: string): Promise<any> {
    // 业务逻辑
    return { id, name: 'John Doe' };
  }
}
```

### 3. 元数据存储

```typescript
import 'reflect-metadata';

const ROUTE_METADATA_KEY = Symbol('route');

interface RouteMetadata {
  path: string;
  method: 'GET' | 'POST' | 'PUT' | 'DELETE';
}

function Get(path: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const metadata: RouteMetadata = { path, method: 'GET' };
    Reflect.defineMetadata(ROUTE_METADATA_KEY, metadata, target, propertyKey);
  };
}

function Post(path: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const metadata: RouteMetadata = { path, method: 'POST' };
    Reflect.defineMetadata(ROUTE_METADATA_KEY, metadata, target, propertyKey);
  };
}

class UserController {
  @Get('/users/:id')
  getUser(id: string): any {
    return { id, name: 'John Doe' };
  }

  @Post('/users')
  createUser(userData: any): any {
    return { id: '123', ...userData };
  }
}

// 路由注册
function registerRoutes(controller: any) {
  const prototype = controller.prototype;
  const methodNames = Object.getOwnPropertyNames(prototype);

  methodNames.forEach(methodName => {
    const metadata = Reflect.getMetadata(ROUTE_METADATA_KEY, prototype, methodName);
    if (metadata) {
      console.log(`Registering ${metadata.method} ${metadata.path} -> ${methodName}`);
      // 实际的路由注册逻辑
    }
  });
}

registerRoutes(UserController);
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

7. **兼容性考虑**: 装饰器在最终输出中会被移除,只影响编译时

8. **调试困难**: 装饰器增加了抽象层,可能使调试变得更困难

## 最佳实践

### 1. 装饰器职责单一

```typescript
// 好的做法：每个装饰器只做一件事
@LogExecution
@ValidateInput
@Cache(1000)
async getUser(id: string): Promise<User> {
  // 业务逻辑
}

// 不好的做法：一个装饰器做太多事情
@ComplexDecorator({
  log: true,
  validate: true,
  cache: true,
  retry: true
})
async getUser(id: string): Promise<User> {
  // 业务逻辑
}
```

### 2. 装饰器可组合性

```typescript
// 设计可组合的装饰器
function withAuth(requiredRole: string) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    // 认证逻辑
  };
}

function withValidation(schema: any) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    // 验证逻辑
  };
}

// 使用时可以自由组合
class UserController {
  @withAuth('admin')
  @withValidation(userSchema)
  createUser(userData: any): any {
    // 业务逻辑
  }
}
```

### 3. 错误处理

```typescript
function SafeExecute(defaultValue?: any) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    const originalMethod = descriptor.value;
    descriptor.value = async function (...args: any[]) {
      try {
        return await originalMethod.apply(this, args);
      } catch (error) {
        console.error(`Error in ${propertyKey}:`, error);
        return defaultValue;
      }
    };
  };
}
```

## 常见问题解决

### 1. 装饰器不生效？

确保在 `tsconfig.json` 中启用了装饰器：
```json
{
  "compilerOptions": {
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  }
}
```

### 2. 如何在装饰器中访问实例属性？

```typescript
function AccessInstance(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  descriptor.value = function (...args: any[]) {
    console.log('Instance:', this);
    console.log('Instance property:', (this as any).someProperty);
    return originalMethod.apply(this, args);
  };
}
```

### 3. 如何处理异步方法？

```typescript
function AsyncDecorator(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  descriptor.value = async function (...args: any[]) {
    console.log(`Before async ${propertyKey}`);
    const result = await originalMethod.apply(this, args);
    console.log(`After async ${propertyKey}`);
    return result;
  };
}
```

## 总结

TypeScript 装饰器是一种强大的元编程工具,可以帮助我们:

- **减少重复代码**: 通过装饰器实现横切关注点
- **增强可读性**: 声明式的方式更直观
- **提高可维护性**: 逻辑集中管理
- **实现AOP**: 面向切面编程的理想选择

合理使用装饰器可以让代码更加简洁、优雅,但也要注意避免过度使用导致代码难以理解。在实际项目中,装饰器常用于 AOP(面向切面编程)、依赖注入、日志记录、性能监控、缓存、权限验证等场景。

记住装饰器的核心价值：
- **声明式编程**: 用声明的方式描述行为
- **横切关注点**: 将通用逻辑从业务代码中分离
- **代码复用**: 一次定义,多处使用
- **元编程**: 在运行时修改程序行为

通过合理使用装饰器,我们可以编写更加清晰、可维护的代码,但也要注意不要过度使用,保持代码的简单性和可理解性。