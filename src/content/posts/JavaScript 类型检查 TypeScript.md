---
title: JavaScript 类型检查 TypeScript
published: 2022-11-03
description: 'TypeScript 基础类型系统的详细介绍和学习笔记'
image: ''
tags: ["TypeScript"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

TypeScript 是 JavaScript 的超集，添加了静态类型检查功能。它可以在编译阶段捕获类型错误，提高代码质量和开发效率。

## 核心概念

### 类型注解

TypeScript 支持多种基本类型的注解：

```typescript
// 基本类型
let name: string = "张三";
let age: number = 25;
let isStudent: boolean = true;
let nothing: null = null;
let notDefined: undefined = undefined;

// 数组类型
let numbers: number[] = [1, 2, 3];
let strings: string[] = ["a", "b", "c"];

// 元组类型
let person: [string, number] = ["张三", 25];

// 枚举类型
enum Color {
  Red,
  Green,
  Blue
}
let color: Color = Color.Red;
```

### 类型推断

TypeScript 可以根据上下文自动推断变量类型：

```typescript
let count = 10; // 推断为 number 类型
let greeting = "Hello"; // 推断为 string 类型

let values = [1, 2, 3]; // 推断为 number[]
```

### 类型断言

当开发者比 TypeScript 更了解类型时，可以使用类型断言：

```typescript
// 尖括号语法
let someValue: any = "hello world";
let strLength: number = (<string>someValue).length;

// as 语法
let someValue: any = "hello world";
let strLength: number = (someValue as string).length;
```

### 联合类型

变量可以是多种类型之一：

```typescript
let value: string | number;
value = "hello";
value = 42;

function processValue(value: string | number): void {
  if (typeof value === "string") {
    console.log(value.toUpperCase());
  } else {
    console.log(value * 2);
  }
}
```

## 高级类型

### 交叉类型

将多个类型合并为一个类型：

```typescript
type Person = {
  name: string;
};

type Employee = {
  employeeId: number;
};

type EmployeePerson = Person & Employee;

const worker: EmployeePerson = {
  name: "张三",
  employeeId: 12345
};
```

### 字面量类型

使用具体的值作为类型：

```typescript
let direction: "up" | "down" | "left" | "right";
direction = "up"; // ✓
direction = "diagonal"; // ✗ 错误

function setColor(color: "red" | "green" | "blue") {
  console.log(`Setting color to ${color}`);
}
```

### 可选属性和只读属性

```typescript
interface User {
  readonly id: number;
  name: string;
  age?: number; // 可选属性
}

const user: User = {
  id: 1,
  name: "张三"
};

user.age = 25; // ✓
user.id = 2; // ✗ 错误，只读属性不能修改
```

## 类型守卫

### typeof 类型守卫

```typescript
function processValue(value: string | number) {
  if (typeof value === "string") {
    // 在这个分支中，value 被推断为 string 类型
    console.log(value.toUpperCase());
  } else {
    // 在这个分支中，value 被推断为 number 类型
    console.log(value.toFixed(2));
  }
}
```

### instanceof 类型守卫

```typescript
class Dog {
  bark() {
    console.log("汪汪");
  }
}

class Cat {
  meow() {
    console.log("喵喵");
  }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}
```

## 类型别名

为类型创建别名：

```typescript
type ID = string | number;
type User = {
  id: ID;
  name: string;
  email: string;
};

const user: User = {
  id: "123",
  name: "张三",
  email: "zhangsan@example.com"
};
```

## 注意事项

1. **any 类型的慎用**：尽量避免使用 `any` 类型，因为它会绕过类型检查
2. **类型断言要谨慎**：只有确定类型安全时才使用类型断言
3. **合理使用类型注解**：TypeScript 的类型推断能力很强，不需要过度注解
4. **编译时检查**：TypeScript 只在编译时检查类型，运行时不会检查
5. **第三方库类型**：为第三方库安装类型定义包（如 `@types/node`）

## 总结

TypeScript 的类型系统为 JavaScript 带来了静态类型检查，大大提高了代码的可维护性和可靠性。通过合理使用类型注解、类型推断、类型守卫等特性，可以编写更安全、更易于维护的代码。