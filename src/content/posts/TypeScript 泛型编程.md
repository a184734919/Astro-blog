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

## 注意事项

1. **合理使用泛型**：只有在需要类型灵活性和复用性时才使用泛型
2. **命名规范**：泛型参数通常使用单个大写字母（T、U、V）或描述性名称
3. **泛型约束**：使用约束来限制泛型参数的范围，提高类型安全性
4. **可读性考虑**：过度使用泛型会降低代码可读性，需要在灵活性和可读性之间平衡
5. **性能影响**：泛型在编译后会被擦除，不会影响运行时性能

## 总结

TypeScript 的泛型编程是一个强大的工具，它能够让我们编写更加灵活、类型安全的代码。通过泛型函数、泛型接口、泛型类以及高级泛型特性，我们可以构建出高度复用的代码库。合理使用泛型可以大大提高代码质量和开发效率，但也要注意不要过度使用，保持代码的可读性和可维护性。