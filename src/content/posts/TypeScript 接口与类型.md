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

### 其他类型保护方式

```typescript
// in 操作符类型保护
interface Bird {
  type: "bird";
  fly(): void;
  layEggs(): void;
}

interface Fish {
  type: "fish";
  swim(): void;
  layEggs(): void;
}

function moveInWater(animal: Bird | Fish) {
  if ("swim" in animal) {
    animal.swim();
  } else {
    console.log("This animal cannot swim");
  }
}

// typeof 类型保护
function processValue(value: string | number | null) {
  if (typeof value === "string") {
    return value.toUpperCase();
  } else if (typeof value === "number") {
    return value.toFixed(2);
  } else {
    return "null value";
  }
}

// instanceof 类型保护
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

## 实际应用场景

### 数据验证接口

```typescript
interface ValidationResult {
  isValid: boolean;
  errors: string[];
}

interface UserValidation {
  name: {
    minLength: number;
    maxLength: number;
    pattern?: RegExp;
  };
  email: {
    pattern: RegExp;
  };
  age: {
    min: number;
    max: number;
  };
}

class UserValidator {
  private rules: UserValidation;

  constructor(rules: UserValidation) {
    this.rules = rules;
  }

  validate(userData: { name: string; email: string; age: number }): ValidationResult {
    const errors: string[] = [];

    // 验证姓名
    if (userData.name.length < this.rules.name.minLength) {
      errors.push(`Name must be at least ${this.rules.name.minLength} characters`);
    }
    if (userData.name.length > this.rules.name.maxLength) {
      errors.push(`Name must be at most ${this.rules.name.maxLength} characters`);
    }
    if (this.rules.name.pattern && !this.rules.name.pattern.test(userData.name)) {
      errors.push("Name contains invalid characters");
    }

    // 验证邮箱
    if (!this.rules.email.pattern.test(userData.email)) {
      errors.push("Invalid email format");
    }

    // 验证年龄
    if (userData.age < this.rules.age.min) {
      errors.push(`Age must be at least ${this.rules.age.min}`);
    }
    if (userData.age > this.rules.age.max) {
      errors.push(`Age must be at most ${this.rules.age.max}`);
    }

    return {
      isValid: errors.length === 0,
      errors
    };
  }
}

// 使用示例
const validator = new UserValidator({
  name: {
    minLength: 2,
    maxLength: 50,
    pattern: /^[a-zA-Z\u4e00-\u9fa5\s]+$/
  },
  email: {
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  },
  age: {
    min: 18,
    max: 120
  }
});

const result = validator.validate({
  name: "张三",
  email: "zhangsan@example.com",
  age: 25
});
```

### API 响应类型定义

```typescript
// 基础响应类型
interface BaseResponse {
  success: boolean;
  message: string;
  timestamp: number;
}

// 成功响应
interface SuccessResponse<T> extends BaseResponse {
  success: true;
  data: T;
  error?: never;
}

// 错误响应
interface ErrorResponse extends BaseResponse {
  success: false;
  data?: never;
  error: {
    code: string;
    details: string;
  };
}

// 联合响应类型
type ApiResponse<T> = SuccessResponse<T> | ErrorResponse;

// 具体的业务类型
interface User {
  id: number;
  name: string;
  email: string;
  avatar?: string;
}

interface Post {
  id: number;
  title: string;
  content: string;
  authorId: number;
  createdAt: string;
}

// 类型安全的API调用
async function fetchUser(id: number): Promise<ApiResponse<User>> {
  try {
    const response = await fetch(`/api/users/${id}`);
    const data = await response.json();

    if (response.ok) {
      return {
        success: true,
        message: "User fetched successfully",
        timestamp: Date.now(),
        data
      };
    } else {
      return {
        success: false,
        message: "Failed to fetch user",
        timestamp: Date.now(),
        error: {
          code: "USER_NOT_FOUND",
          details: data.message || "User not found"
        }
      };
    }
  } catch (error) {
    return {
      success: false,
      message: "Network error",
      timestamp: Date.now(),
      error: {
        code: "NETWORK_ERROR",
        details: error instanceof Error ? error.message : "Unknown error"
      }
    };
  }
}

// 响应处理函数
function handleApiResponse<T>(response: ApiResponse<T>, onSuccess: (data: T) => void) {
  if (response.success) {
    onSuccess(response.data);
  } else {
    console.error(`Error ${response.error.code}: ${response.error.details}`);
  }
}

// 使用示例
async function main() {
  const userResponse = await fetchUser(1);

  handleApiResponse(userResponse, (user) => {
    console.log(`User: ${user.name} (${user.email})`);
  });
}
```

### 状态管理类型定义

```typescript
// 状态接口
interface LoadingState {
  status: "loading";
  data: null;
  error: null;
}

interface SuccessState<T> {
  status: "success";
  data: T;
  error: null;
}

interface ErrorState {
  status: "error";
  data: null;
  error: Error;
}

type AsyncState<T> = LoadingState | SuccessState<T> | ErrorState;

// 状态管理类
class AsyncStateManager<T> {
  private state: AsyncState<T>;

  constructor() {
    this.state = { status: "loading", data: null, error: null };
  }

  setLoading(): void {
    this.state = { status: "loading", data: null, error: null };
  }

  setSuccess(data: T): void {
    this.state = { status: "success", data, error: null };
  }

  setError(error: Error): void {
    this.state = { status: "error", data: null, error };
  }

  getState(): AsyncState<T> {
    return this.state;
  }

  isLoading(): boolean {
    return this.state.status === "loading";
  }

  isSuccess(): boolean {
    return this.state.status === "success";
  }

  isError(): boolean {
    return this.state.status === "error";
  }

  getData(): T | null {
    return this.state.status === "success" ? this.state.data : null;
  }

  getError(): Error | null {
    return this.state.status === "error" ? this.state.error : null;
  }

  // 渲染方法
  render(
    loadingRenderer: () => React.ReactNode,
    successRenderer: (data: T) => React.ReactNode,
    errorRenderer: (error: Error) => React.ReactNode
  ): React.ReactNode {
    switch (this.state.status) {
      case "loading":
        return loadingRenderer();
      case "success":
        return successRenderer(this.state.data);
      case "error":
        return errorRenderer(this.state.error);
    }
  }
}

// 使用示例
const userManager = new AsyncStateManager<User>();

async function loadUser(id: number) {
  userManager.setLoading();

  try {
    const userResponse = await fetchUser(id);

    handleApiResponse(userResponse, (user) => {
      userManager.setSuccess(user);
    });
  } catch (error) {
    userManager.setError(error instanceof Error ? error : new Error("Unknown error"));
  }
}

// 在 React 组件中使用
function UserComponent({ userId }: { userId: number }) {
  const state = userManager.getState();

  return state.render(
    () => <div>Loading...</div>,
    (user) => <div>{user.name}</div>,
    (error) => <div>Error: {error.message}</div>
  );
}
```

## 高级技巧

### 模板字面量类型

```typescript
type EventName<T extends string> = `on${Capitalize<T>}`;

type ButtonEvents = EventName<"click" | "hover" | "focus">;
// "onClick" | "onHover" | "onFocus"

// 动态属性名
type ObjectWithDynamicKeys<T extends string> = {
  [K in T]: `prefix_${K}`;
};

type Result = ObjectWithDynamicKeys<"name" | "age">;
// { name: "prefix_name"; age: "prefix_age" }
```

### 递归类型

```typescript
// JSON 类型
type Json =
  | string
  | number
  | boolean
  | null
  | Json[]
  | { [key: string]: Json };

// 树形结构
type TreeNode<T> = {
  value: T;
  children?: TreeNode<T>[];
};

// 树操作工具
class TreeUtils {
  static depthFirstSearch<T>(
    tree: TreeNode<T>,
    predicate: (value: T) => boolean
  ): T | null {
    if (predicate(tree.value)) {
      return tree.value;
    }

    if (tree.children) {
      for (const child of tree.children) {
        const result = this.depthFirstSearch(child, predicate);
        if (result) return result;
      }
    }

    return null;
  }

  static breadthFirstSearch<T>(
    tree: TreeNode<T>,
    predicate: (value: T) => boolean
  ): T | null {
    const queue: TreeNode<T>[] = [tree];

    while (queue.length > 0) {
      const node = queue.shift()!;
      if (predicate(node.value)) {
        return node.value;
      }

      if (node.children) {
        queue.push(...node.children);
      }
    }

    return null;
  }

  static flatten<T>(tree: TreeNode<T>): T[] {
    const result: T[] = [];

    function traverse(node: TreeNode<T>) {
      result.push(node.value);
      if (node.children) {
        node.children.forEach(traverse);
      }
    }

    traverse(tree);
    return result;
  }
}

// 使用示例
const fileTree: TreeNode<string> = {
  value: "root",
  children: [
    {
      value: "src",
      children: [
        { value: "index.ts" },
        { value: "app.ts" },
        {
          value: "components",
          children: [
            { value: "Button.tsx" },
            { value: "Input.tsx" }
          ]
        }
      ]
    },
    {
      value: "package.json"
    }
  ]
};

const allFiles = TreeUtils.flatten(fileTree);
// ["root", "src", "index.ts", "app.ts", "components", "Button.tsx", "Input.tsx", "package.json"]
```

## 最佳实践

### 1. 接口设计原则

```typescript
// 好的接口设计：简洁、专注、单一职责
interface UserRepository {
  findById(id: number): Promise<User | null>;
  findByEmail(email: string): Promise<User | null>;
  create(user: CreateUserDto): Promise<User>;
  update(id: number, updates: UpdateUserDto): Promise<User>;
  delete(id: number): Promise<void>;
}

// 不好的设计：接口过于庞大
interface UserService {
  // 用户管理
  findUser(id: number): Promise<User>;
  createUser(user: any): Promise<User>;

  // 认证相关
  login(email: string, password: string): Promise<string>;
  logout(token: string): Promise<void>;
  validateToken(token: string): Promise<boolean>;

  // 邮件发送
  sendWelcomeEmail(user: User): Promise<void>;
  sendPasswordResetEmail(email: string): Promise<void>;

  // 文件上传
  uploadAvatar(userId: number, file: File): Promise<string>;
  deleteAvatar(userId: number): Promise<void>;
}
```

### 2. 类型命名约定

```typescript
// 接口使用 PascalCase
interface UserService {}

interface UserRepository {}

interface HttpResponse<T> {}

// 类型别名使用 PascalCase
type UserId = number;
type UserEmail = string;
type UserRole = "admin" | "user" | "guest";

// 枚举使用 PascalCase
enum UserRole {
  Admin = "admin",
  User = "user",
  Guest = "guest"
}

// 泛型参数使用单个大写字母或描述性名称
interface Repository<T> {}
interface Mapper<TSource, TDestination> {}
interface EventHandler<TEvent, TResult> {}
```

### 3. 避免类型重复

```typescript
// 不好的做法：重复定义相同的类型
interface UserCreateForm {
  name: string;
  email: string;
  age: number;
}

interface UserUpdateForm {
  name: string;
  email: string;
  age: number;
}

interface UserResponse {
  name: string;
  email: string;
  age: number;
}

// 好的做法：复用类型
interface UserBase {
  name: string;
  email: string;
  age: number;
}

interface UserCreateForm extends UserBase {
  password: string;
}

interface UserUpdateForm extends Partial<UserBase> {}

interface UserResponse extends UserBase {
  id: number;
  createdAt: Date;
}
```

## 注意事项

1. **接口与类型的选择**：根据使用场景选择合适的类型定义方式
2. **避免过度嵌套**：复杂的类型定义会影响代码可读性
3. **合理使用类型推断**：TypeScript 能够自动推断很多类型，不需要过度注解
4. **命名规范**：接口名使用 PascalCase，类型别名使用 PascalCase
5. **版本兼容性**：类型别名比接口更灵活，在复杂场景下优先考虑
6. **避免循环引用**：类型之间的相互引用会导致编译错误
7. **使用工具类型**：善用 TypeScript 提供的工具类型简化类型定义

## 常见问题解决

### 1. 如何定义动态属性的对象类型？

```typescript
// 方法1：索引签名
interface DynamicObject {
  [key: string]: any;
}

// 方法2：Record 工具类型
type DynamicObject2 = Record<string, any>;

// 方法3：键值对映射
type UserPermissions = Record<string, boolean>;

const permissions: UserPermissions = {
  canRead: true,
  canWrite: false,
  canDelete: false
};
```

### 2. 如何处理可选链和空值合并？

```typescript
interface User {
  profile?: {
    address?: {
      city?: string;
      country?: string;
    };
  };
}

function getUserCity(user: User): string {
  // 可选链和空值合并
  return user.profile?.address?.city ?? "Unknown";
}

// 类型守卫
function hasAddress(user: User): user is User & {
  profile: { address: { city: string } }
} {
  return !!user.profile?.address?.city;
}

if (hasAddress(user)) {
  // user.profile.address.city 在这里确定存在
  console.log(user.profile.address.city.toUpperCase());
}
```

### 3. 如何处理函数重载？

```typescript
// 函数重载
function processInput(input: string): string;
function processInput(input: number): number;
function processInput(input: string | number): string | number {
  if (typeof input === "string") {
    return input.toUpperCase();
  } else {
    return input * 2;
  }
}

// 使用重载
const result1 = processInput("hello"); // string
const result2 = processInput(42);      // number

// 接口中的方法重载
interface Calculator {
  add(a: number, b: number): number;
  add(a: string, b: string): string;
  add(a: any, b: any): any;
}

class SimpleCalculator implements Calculator {
  add(a: number, b: number): number;
  add(a: string, b: string): string;
  add(a: any, b: any): any {
    if (typeof a === "number" && typeof b === "number") {
      return a + b;
    } else if (typeof a === "string" && typeof b === "string") {
      return a + b;
    }
    throw new Error("Invalid types");
  }
}
```

## 总结

TypeScript 的接口和类型别名都是定义类型的重要工具。接口更适合传统的面向对象编程和继承场景，而类型别名更适合联合类型、交叉类型等高级类型特性。在实际开发中，根据具体需求选择合适的类型定义方式，可以大大提高代码的类型安全性和可维护性。

通过合理使用接口继承、类型别名、高级类型和工具类型，我们可以构建出类型安全、可维护的代码库。记住选择合适的类型定义方式、遵循命名规范、避免过度复杂化，这些都是写出高质量 TypeScript 代码的关键。