---
title: TypeScript 接口与类型
published: 2022-11-09
description: '接口定义和高级类型的详细介绍和学习笔记'
image: ''
tags: ["TypeScript","类型"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

TypeScript 的接口（Interface）和类型（Type）是定义对象形状和类型的两种重要方式。它们各有特点，适用于不同的场景。

## 接口（Interface）

### 基本接口定义

```typescript
interface User {
  id: number;
  name: string;
  age: number;
}

function createUser(user: User) {
  console.log(`Creating user: ${user.name}`);
}

const user: User = {
  id: 1,
  name: "张三",
  age: 25
};
```

### 可选属性

```typescript
interface User {
  id: number;
  name: string;
  age?: number; // 可选属性
  email?: string; // 可选属性
}

const user: User = {
  id: 1,
  name: "张三"
  // age 和 email 可以不提供
};
```

### 只读属性

```typescript
interface User {
  readonly id: number; // 只读属性
  name: string;
  age: number;
}

const user: User = {
  id: 1,
  name: "张三",
  age: 25
};

user.name = "李四"; // ✓
user.id = 2; // ✗ 错误，无法修改只读属性
```

### 接口继承

```typescript
interface Animal {
  name: string;
  age: number;
}

interface Dog extends Animal {
  breed: string;
  bark(): void;
}

const dog: Dog = {
  name: "旺财",
  age: 3,
  breed: "金毛",
  bark() {
    console.log("汪汪");
  }
};
```

### 多重继承

```typescript
interface CanFly {
  fly(): void;
}

interface CanSwim {
  swim(): void;
}

interface Duck extends CanFly, CanSwim {
  quack(): void;
}

const duck: Duck = {
  fly() {
    console.log("飞起来了");
  },
  swim() {
    console.log("游起来了");
  },
  quack() {
    console.log("嘎嘎");
  }
};
```

## 类型别名（Type Alias）

### 基本类型别名

```typescript
type ID = string | number;
type Age = number;
type Name = string;

const userId: ID = "123";
const userAge: Age = 25;
const userName: Name = "张三";
```

### 对象类型别名

```typescript
type User = {
  id: number;
  name: string;
  age: number;
};

const user: User = {
  id: 1,
  name: "张三",
  age: 25
};
```

### 联合类型

```typescript
type Success = {
  success: true;
  data: any;
};

type Failure = {
  success: false;
  error: string;
};

type Result = Success | Failure;

function handleResult(result: Result) {
  if (result.success) {
    console.log(result.data);
  } else {
    console.error(result.error);
  }
}
```

### 交叉类型

```typescript
type Person = {
  name: string;
  age: number;
};

type Employee = {
  employeeId: number;
  department: string;
};

type EmployeePerson = Person & Employee;

const worker: EmployeePerson = {
  name: "张三",
  age: 30,
  employeeId: 12345,
  department: "技术部"
};
```

## 高级类型

### 字面量类型

```typescript
type Direction = "up" | "down" | "left" | "right";
type Status = "success" | "error" | "loading";

function move(direction: Direction) {
  console.log(`Moving ${direction}`);
}

function handleStatus(status: Status) {
  switch (status) {
    case "success":
      console.log("操作成功");
      break;
    case "error":
      console.log("操作失败");
      break;
    case "loading":
      console.log("加载中");
      break;
  }
}
```

### 映射类型

```typescript
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Partial<T> = {
  [P in keyof T]?: T[P];
};

interface User {
  id: number;
  name: string;
  age: number;
}

type ReadonlyUser = Readonly<User>;
type PartialUser = Partial<User>;

// ReadonlyUser 所有属性都是只读的
// PartialUser 所有属性都是可选的
```

### 条件类型

```typescript
type NonNullable<T> = T extends null | undefined ? never : T;

type Result = NonNullable<string | null | undefined>; // string
```

### 索引类型

```typescript
interface User {
  id: number;
  name: string;
  age: number;
}

type UserKeys = keyof User; // "id" | "name" | "age"
type UserType = User["age"]; // number

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user: User = {
  id: 1,
  name: "张三",
  age: 25
};

const age = getProperty(user, "age"); // number
const name = getProperty(user, "name"); // string
```

## 接口 vs 类型别名

### 使用接口的场景

- 需要定义对象的形状
- 需要实现接口继承
- 需要定义类的契约
- 传统的面向对象编程风格

```typescript
interface Animal {
  name: string;
}

interface Dog extends Animal {
  bark(): void;
}

class MyDog implements Dog {
  name: string;
  bark() {
    console.log("汪汪");
  }
}
```

### 使用类型别名的场景

- 需要定义联合类型
- 需要定义交叉类型
- 需要定义字面量类型
- 需要使用高级类型特性

```typescript
type ID = string | number;
type Coordinate = [number, number];
type Status = "success" | "error" | "loading";
```

## 类型保护

### 自定义类型保护

```typescript
interface User {
  type: "user";
  name: string;
  email: string;
}

interface Admin {
  type: "admin";
  name: string;
  permissions: string[];
}

function isUser(user: User | Admin): user is User {
  return user.type === "user";
}

function handleUser(user: User | Admin) {
  if (isUser(user)) {
    console.log(user.email);
  } else {
    console.log(user.permissions);
  }
}
```

## 注意事项

1. **接口与类型的选择**：根据使用场景选择合适的类型定义方式
2. **避免过度嵌套**：复杂的类型定义会影响代码可读性
3. **合理使用类型推断**：TypeScript 能够自动推断很多类型，不需要过度注解
4. **命名规范**：接口名使用 PascalCase，类型别名使用 PascalCase
5. **版本兼容性**：类型别名比接口更灵活，在复杂场景下优先考虑

## 总结

TypeScript 的接口和类型别名都是定义类型的重要工具。接口更适合传统的面向对象编程和继承场景，而类型别名更适合联合类型、交叉类型等高级类型特性。在实际开发中，根据具体需求选择合适的类型定义方式，可以大大提高代码的类型安全性和可维护性。