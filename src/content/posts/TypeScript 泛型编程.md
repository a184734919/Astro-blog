---
title: TypeScript 泛型编程
published: 2022-11-16
description: '泛型的概念和应用的详细介绍和学习笔记'
image: ''
tags: ["TypeScript"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

泛型（Generics）是 TypeScript 中一种强大的类型系统特性，它允许我们在定义函数、接口和类时使用类型参数，从而提高代码的复用性和类型安全性。

## 泛型函数

### 基本泛型函数

```typescript
function identity<T>(arg: T): T {
  return arg;
}

// 使用方式1：显式指定类型
const result1 = identity<string>("hello");
const result2 = identity<number>(123);

// 使用方式2：类型推断
const result3 = identity("hello"); // 自动推断为 string
const result4 = identity(123);     // 自动推断为 number
```

### 多个类型参数

```typescript
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const pair1 = pair<string, number>("hello", 42);
const pair2 = pair<boolean, string>(true, "world");
```

### 泛型约束

```typescript
interface Lengthwise {
  length: number;
}

function getLength<T extends Lengthwise>(arg: T): number {
  return arg.length;
}

const length1 = getLength("hello");      // 5
const length2 = getLength([1, 2, 3]);    // 3
const length3 = getLength({ length: 10 }); // 10
// getLength(123); // ✗ 错误，数字没有 length 属性
```

## 泛型接口

### 基本泛型接口

```typescript
interface Box<T> {
  value: T;
}

const stringBox: Box<string> = { value: "hello" };
const numberBox: Box<number> = { value: 42 };

function processBox<T>(box: Box<T>): T {
  return box.value;
}
```

### 复杂泛型接口

```typescript
interface Response<T> {
  success: boolean;
  data?: T;
  error?: string;
}

function createSuccessResponse<T>(data: T): Response<T> {
  return {
    success: true,
    data
  };
}

function createErrorResponse<T>(error: string): Response<T> {
  return {
    success: false,
    error
  };
}

const successResponse = createSuccessResponse({ name: "张三" });
const errorResponse = createErrorResponse<string>("服务器错误");
```

## 泛型类

### 基本泛型类

```typescript
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }

  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }

  isEmpty(): boolean {
    return this.items.length === 0;
  }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
console.log(numberStack.pop()); // 2

const stringStack = new Stack<string>();
stringStack.push("hello");
stringStack.push("world");
console.log(stringStack.pop()); // "world"
```

### 泛型约束类

```typescript
interface HasId {
  id: number;
}

class Repository<T extends HasId> {
  private items: T[] = [];

  add(item: T): void {
    this.items.push(item);
  }

  findById(id: number): T | undefined {
    return this.items.find(item => item.id === id);
  }

  update(id: number, updates: Partial<T>): boolean {
    const item = this.findById(id);
    if (item) {
      Object.assign(item, updates);
      return true;
    }
    return false;
  }
}

interface User {
  id: number;
  name: string;
  email: string;
}

const userRepository = new Repository<User>();
userRepository.add({ id: 1, name: "张三", email: "zhangsan@example.com" });
userRepository.update(1, { name: "李四" });
```

## 高级泛型

### 条件类型

```typescript
type NonNullable<T> = T extends null | undefined ? never : T;

type Result1 = NonNullable<string | null>;    // string
type Result2 = NonNullable<number | undefined>; // number

// 更复杂的条件类型
type ExtractArrayType<T> = T extends (infer U)[] ? U : never;

type Numbers = ExtractArrayType<number[]>;    // number
type Strings = ExtractArrayType<string[]>;    // string
type NotArray = ExtractArrayType<number>;     // never
```

### 映射类型

```typescript
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Partial<T> = {
  [P in keyof T]?: T[P];
};

type Required<T> = {
  [P in keyof T]-?: T[P];
};

interface User {
  id: number;
  name: string;
  age: number;
}

type ReadonlyUser = Readonly<User>;  // 所有属性只读
type PartialUser = Partial<User>;    // 所有属性可选
type RequiredUser = Required<Partial<User>>; // 所有属性必需
```

### 递归类型

```typescript
type TreeNode<T> = {
  value: T;
  left?: TreeNode<T>;
  right?: TreeNode<T>;
};

function createTree<T>(value: T, left?: TreeNode<T>, right?: TreeNode<T>): TreeNode<T> {
  return { value, left, right };
}

const tree = createTree(
  1,
  createTree(2, createTree(4), createTree(5)),
  createTree(3, createTree(6), createTree(7))
);
```

## 泛型工具类型

### 常用工具类型

```typescript
// Partial<T> - 将所有属性设为可选
interface User {
  id: number;
  name: string;
  age: number;
}

type PartialUser = Partial<User>;

// Required<T> - 将所有属性设为必需
type RequiredUser = Required<Partial<User>>;

// Readonly<T> - 将所有属性设为只读
type ReadonlyUser = Readonly<User>;

// Pick<T, K> - 选择特定属性
type UserBasicInfo = Pick<User, "id" | "name">;

// Omit<T, K> - 排除特定属性
type UserWithoutId = Omit<User, "id">;

// Record<K, T> - 创建对象类型
type UserMap = Record<string, User>;

// Exclude<T, U> - 排除某些类型
type NumberOrString = number | string;
type OnlyString = Exclude<NumberOrString, number>;

// Extract<T, U> - 提取某些类型
type OnlyNumber = Extract<NumberOrString, number>;
```

## 实际应用场景

### API 响应类型

```typescript
interface ApiResponse<T> {
  status: number;
  message: string;
  data: T;
}

interface User {
  id: number;
  name: string;
  email: string;
}

interface Post {
  id: number;
  title: string;
  content: string;
}

// 使用泛型定义不同类型的 API 响应
type UserResponse = ApiResponse<User>;
type PostResponse = ApiResponse<Post>;

async function fetchUser(id: number): Promise<UserResponse> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

async function fetchPost(id: number): Promise<PostResponse> {
  const response = await fetch(`/api/posts/${id}`);
  return response.json();
}
```

### 数据存储容器

```typescript
class Container<T> {
  private data: T;

  constructor(data: T) {
    this.data = data;
  }

  get(): T {
    return this.data;
  }

  set(value: T): void {
    this.data = value;
  }

  transform<U>(fn: (value: T) => U): Container<U> {
    return new Container(fn(this.data));
  }
}

// 链式调用示例
const container = new Container(10)
  .transform(x => x * 2)
  .transform(x => x.toString())
  .transform(x => x.toUpperCase());

console.log(container.get()); // "20"
```

### 事件总线实现

```typescript
type EventHandler<T = any> = (payload: T) => void;

class EventBus {
  private events: Map<string, Set<EventHandler>> = new Map();

  // 注册事件监听器
  on<T = any>(eventName: string, handler: EventHandler<T>): void {
    if (!this.events.has(eventName)) {
      this.events.set(eventName, new Set());
    }
    this.events.get(eventName)!.add(handler);
  }

  // 移除事件监听器
  off<T = any>(eventName: string, handler: EventHandler<T>): void {
    const handlers = this.events.get(eventName);
    if (handlers) {
      handlers.delete(handler);
      if (handlers.size === 0) {
        this.events.delete(eventName);
      }
    }
  }

  // 触发事件
  emit<T = any>(eventName: string, payload: T): void {
    const handlers = this.events.get(eventName);
    if (handlers) {
      handlers.forEach(handler => {
        try {
          handler(payload);
        } catch (error) {
          console.error(`Error in event handler for ${eventName}:`, error);
        }
      });
    }
  }

  // 一次性事件监听器
  once<T = any>(eventName: string, handler: EventHandler<T>): void {
    const onceHandler: EventHandler<T> = (payload) => {
      handler(payload);
      this.off(eventName, onceHandler);
    };
    this.on(eventName, onceHandler);
  }

  // 清除所有事件监听器
  clear(): void {
    this.events.clear();
  }

  // 清除特定事件的所有监听器
  clearEvent(eventName: string): void {
    this.events.delete(eventName);
  }
}

// 使用示例
const bus = new EventBus();

// 定义事件类型
interface UserCreatedEvent {
  userId: number;
  userName: string;
  timestamp: number;
}

interface OrderPlacedEvent {
  orderId: string;
  userId: number;
  amount: number;
  items: string[];
}

// 注册事件监听器
bus.on<UserCreatedEvent>("user:created", (event) => {
  console.log(`User created: ${event.userName} (ID: ${event.userId})`);
  // 发送欢迎邮件
  sendWelcomeEmail(event.userId);
});

bus.on<OrderPlacedEvent>("order:placed", (event) => {
  console.log(`Order placed: ${event.orderId} by user ${event.userId}`);
  // 处理订单
  processOrder(event);
});

// 触发事件
bus.emit<UserCreatedEvent>("user:created", {
  userId: 123,
  userName: "张三",
  timestamp: Date.now()
});

bus.emit<OrderPlacedEvent>("order:placed", {
  orderId: "ORD-2023-001",
  userId: 123,
  amount: 99.99,
  items: ["商品1", "商品2"]
});
```

### 通用缓存类

```typescript
interface CacheEntry<T> {
  value: T;
  expiresAt: number;
}

class Cache<T> {
  private cache: Map<string, CacheEntry<T>> = new Map();
  private defaultTTL: number;

  constructor(defaultTTL: number = 60000) { // 默认60秒
    this.defaultTTL = defaultTTL;
  }

  // 设置缓存
  set(key: string, value: T, ttl?: number): void {
    const expiresAt = Date.now() + (ttl ?? this.defaultTTL);
    this.cache.set(key, { value, expiresAt });
  }

  // 获取缓存
  get(key: string): T | undefined {
    const entry = this.cache.get(key);
    if (!entry) return undefined;

    // 检查是否过期
    if (Date.now() > entry.expiresAt) {
      this.cache.delete(key);
      return undefined;
    }

    return entry.value;
  }

  // 检查缓存是否存在且未过期
  has(key: string): boolean {
    const entry = this.cache.get(key);
    if (!entry) return false;

    if (Date.now() > entry.expiresAt) {
      this.cache.delete(key);
      return false;
    }

    return true;
  }

  // 删除缓存
  delete(key: string): boolean {
    return this.cache.delete(key);
  }

  // 清空所有缓存
  clear(): void {
    this.cache.clear();
  }

  // 获取或设置（缓存穿透保护）
  async getOrSet(
    key: string,
    factory: () => Promise<T>,
    ttl?: number
  ): Promise<T> {
    const cached = this.get(key);
    if (cached !== undefined) {
      return cached;
    }

    const value = await factory();
    this.set(key, value, ttl);
    return value;
  }

  // 清理过期缓存
  cleanup(): number {
    const now = Date.now();
    let cleaned = 0;

    for (const [key, entry] of this.cache.entries()) {
      if (now > entry.expiresAt) {
        this.cache.delete(key);
        cleaned++;
      }
    }

    return cleaned;
  }

  // 获取缓存统计信息
  getStats(): { size: number; keys: string[] } {
    return {
      size: this.cache.size,
      keys: Array.from(this.cache.keys())
    };
  }
}

// 使用示例
const userCache = new Cache<User>(300000); // 5分钟过期

async function getUserWithCache(userId: number): Promise<User> {
  const cacheKey = `user:${userId}`;

  return await userCache.getOrSet(cacheKey, async () => {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  });
}

// 定期清理过期缓存
setInterval(() => {
  const cleaned = userCache.cleanup();
  if (cleaned > 0) {
    console.log(`Cleaned ${cleaned} expired cache entries`);
  }
}, 60000); // 每分钟清理一次
```

### 状态机实现

```typescript
type StateHandler<TState, TEvent> = (
  currentState: TState,
  event: TEvent
) => TState;

class StateMachine<TState extends string, TEvent extends string> {
  private currentState: TState;
  private transitions: Map<TState, Map<TEvent, TState>> = new Map();
  private handlers: Map<string, StateHandler<TState, TEvent>> = new Map();
  private onEnterHandlers: Map<TState, (() => void)[]> = new Map();
  private onExitHandlers: Map<TState, (() => void)[]> = new Map();

  constructor(initialState: TState) {
    this.currentState = initialState;
  }

  // 添加状态转换
  addTransition(from: TState, event: TEvent, to: TState): this {
    if (!this.transitions.has(from)) {
      this.transitions.set(from, new Map());
    }
    this.transitions.get(from)!.set(event, to);
    return this;
  }

  // 添加状态处理器
  addHandler(
    from: TState,
    event: TEvent,
    handler: StateHandler<TState, TEvent>
  ): this {
    const key = `${from}:${event}`;
    this.handlers.set(key, handler);
    return this;
  }

  // 添加进入状态处理器
  onEnter(state: TState, handler: () => void): this {
    if (!this.onEnterHandlers.has(state)) {
      this.onEnterHandlers.set(state, []);
    }
    this.onEnterHandlers.get(state)!.push(handler);
    return this;
  }

  // 添加退出状态处理器
  onExit(state: TState, handler: () => void): this {
    if (!this.onExitHandlers.has(state)) {
      this.onExitHandlers.set(state, []);
    }
    this.onExitHandlers.get(state)!.push(handler);
    return this;
  }

  // 触发事件
  dispatch(event: TEvent): boolean {
    const transitions = this.transitions.get(this.currentState);
    if (!transitions) {
      console.warn(`No transitions defined for state: ${this.currentState}`);
      return false;
    }

    const nextState = transitions.get(event);
    if (!nextState) {
      console.warn(`No transition defined for event: ${event} in state: ${this.currentState}`);
      return false;
    }

    // 执行退出处理器
    const exitHandlers = this.onExitHandlers.get(this.currentState);
    if (exitHandlers) {
      exitHandlers.forEach(handler => handler());
    }

    // 执行状态处理器
    const key = `${this.currentState}:${event}`;
    const handler = this.handlers.get(key);
    const actualNextState = handler ? handler(this.currentState, event) : nextState;

    // 更新状态
    this.currentState = actualNextState;

    // 执行进入处理器
    const enterHandlers = this.onEnterHandlers.get(this.currentState);
    if (enterHandlers) {
      enterHandlers.forEach(handler => handler());
    }

    return true;
  }

  // 获取当前状态
  getState(): TState {
    return this.currentState;
  }

  // 重置状态
  reset(state: TState): void {
    this.currentState = state;
  }
}

// 使用示例 - 订单状态机
type OrderState = "created" | "paid" | "shipped" | "delivered" | "cancelled";
type OrderEvent = "pay" | "ship" | "deliver" | "cancel";

const orderStateMachine = new StateMachine<OrderState, OrderEvent>("created");

// 定义状态转换
orderStateMachine
  .addTransition("created", "pay", "paid")
  .addTransition("created", "cancel", "cancelled")
  .addTransition("paid", "ship", "shipped")
  .addTransition("paid", "cancel", "cancelled")
  .addTransition("shipped", "deliver", "delivered")
  .addTransition("shipped", "cancel", "cancelled");

// 添加状态处理器
orderStateMachine
  .onEnter("paid", () => {
    console.log("Order paid, preparing for shipment");
    sendPaymentConfirmation();
  })
  .onEnter("shipped", () => {
    console.log("Order shipped");
    sendShippingNotification();
  })
  .onEnter("delivered", () => {
    console.log("Order delivered");
    requestDeliveryConfirmation();
  })
  .onEnter("cancelled", () => {
    console.log("Order cancelled");
    processRefund();
  });

// 使用状态机
console.log(orderStateMachine.getState()); // "created"
orderStateMachine.dispatch("pay");
console.log(orderStateMachine.getState()); // "paid"
orderStateMachine.dispatch("ship");
console.log(orderStateMachine.getState()); // "shipped"
orderStateMachine.dispatch("deliver");
console.log(orderStateMachine.getState()); // "delivered"
```

## 高级泛型技巧

### 泛型约束工厂

```typescript
// 创建具有特定属性的对象
function createWithId<T extends object>(
  factory: (id: string) => T
): (id: string) => T & { id: string } {
  return (id: string) => ({
    id,
    ...factory(id)
  });
}

const createUser = createWithId((id: string) => ({
  name: `User-${id}`,
  email: `user-${id}@example.com`
}));

const user = createUser("123");
console.log(user.id); // "123"
console.log(user.name); // "User-123"
```

### 深度只读类型

```typescript
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? T[P] extends Function
      ? T[P]
      : DeepReadonly<T[P]>
    : T[P];
};

interface NestedObject {
  user: {
    id: number;
    profile: {
      name: string;
      address: {
        street: string;
        city: string;
      };
    };
  };
}

type ReadonlyNested = DeepReadonly<NestedObject>;

const obj: ReadonlyNested = {
  user: {
    id: 1,
    profile: {
      name: "张三",
      address: {
        street: "长安街",
        city: "北京"
      }
    }
  }
};

// obj.user.profile.address.city = "上海"; // 编译错误
```

### 部分只读类型

```typescript
type PartialReadonly<T, K extends keyof T> = T & Readonly<Pick<T, K>>;

interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

type UserWithImmutableId = PartialReadonly<User, "id">;

const user: UserWithImmutableId = {
  id: 1,
  name: "张三",
  email: "zhangsan@example.com",
  password: "password123"
};

user.name = "李四"; // 可以修改
// user.id = 2; // 编译错误
```

## 性能优化技巧

### 避免不必要的泛型实例化

```typescript
// 不好的做法：每次调用都创建新的泛型类型
function badGeneric<T>(value: T): T {
  return value;
}

// 好的做法：使用具体类型或约束泛型
function goodGeneric<T extends string>(value: T): T {
  return value;
}

// 更好的做法：如果可能，使用具体类型
function bestGeneric(value: string): string {
  return value;
}
```

### 泛型缓存

```typescript
class GenericCache {
  private static cache = new Map<any, any>();

  static get<T>(key: string, factory: () => T): T {
    if (!this.cache.has(key)) {
      this.cache.set(key, factory());
    }
    return this.cache.get(key) as T;
  }
}

// 使用示例
const users = GenericCache.get<User[]>("users", () => []);
const posts = GenericCache.get<Post[]>("posts", () => []);
```

## 最佳实践

### 1. 合理命名泛型参数

```typescript
// 好的命名
interface Repository<TEntity, TId> {
  findById(id: TId): Promise<TEntity | null>;
  save(entity: TEntity): Promise<void>;
}

// 不好的命名
interface Repository<T, U> {
  findById(id: U): Promise<T | null>;
  save(entity: T): Promise<void>;
}
```

### 2. 提供合理的默认值

```typescript
// 有默认值，使用更方便
interface Container<T = any> {
  value: T;
}

const container1: Container<string> = { value: "hello" };
const container2: Container = { value: 123 }; // 推断为 Container<number>
```

### 3. 使用泛型约束提高类型安全性

```typescript
// 有约束，更安全
function process<T extends { id: number }>(item: T): void {
  console.log(`Processing item ${item.id}`);
}

// 无约束，可能不安全
function processUnsafe<T>(item: T): void {
  // console.log(item.id); // 编译错误
}
```

## 注意事项

1. **合理使用泛型**：只有在需要类型灵活性和复用性时才使用泛型
2. **命名规范**：泛型参数通常使用单个大写字母（T、U、V）或描述性名称
3. **泛型约束**：使用约束来限制泛型参数的范围，提高类型安全性
4. **可读性考虑**：过度使用泛型会降低代码可读性，需要在灵活性和可读性之间平衡
5. **性能影响**：泛型在编译后会被擦除，不会影响运行时性能
6. **避免过度泛型**：如果只有一两种使用场景，可能不需要泛型
7. **文档化泛型**：为泛型参数添加清晰的注释说明其用途和约束

## 常见问题解决

### 1. 如何处理泛型默认类型？

```typescript
interface Box<T = string> {
  value: T;
}

const box1: Box<number> = { value: 123 };
const box2: Box = { value: "hello" }; // 使用默认类型 string
```

### 2. 如何处理泛型推断失败？

```typescript
// 当类型推断失败时，显式指定类型
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

// 类型推断成功
const pair1 = pair("hello", 42);

// 类型推断可能失败时，显式指定
const pair2: [string, number] = pair("hello", 42);
const pair3 = pair<string, number>("hello", 42);
```

### 3. 如何处理泛型函数重载？

```typescript
function createArray<T>(length: number, value: T): T[];
function createArray<T>(length: number, factory: () => T): T[];
function createArray<T>(
  length: number,
  valueOrFactory: T | (() => T)
): T[] {
  const array: T[] = [];
  for (let i = 0; i < length; i++) {
    array.push(
      typeof valueOrFactory === "function"
        ? (valueOrFactory as () => T)()
        : valueOrFactory
    );
  }
  return array;
}

// 使用重载
const numbers = createArray(5, 0); // number[]
const strings = createArray(3, () => "hello"); // string[]
```

## 总结

TypeScript 的泛型编程是一个强大的工具，它能够让我们编写更加灵活、类型安全的代码。通过泛型函数、泛型接口、泛型类以及高级泛型特性，我们可以构建出高度复用的代码库。合理使用泛型可以大大提高代码质量和开发效率，但也要注意不要过度使用，保持代码的可读性和可维护性。

泛型的真正价值在于：
- **类型安全**：在编译时捕获类型错误
- **代码复用**：编写一次，多种类型使用
- **灵活性**：适应不同的使用场景
- **可维护性**：减少重复代码，提高代码质量

记住，泛型的目的是提高代码质量和开发效率，而不是为了使用而使用。在实际项目中，要根据具体需求合理使用泛型特性。