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

## 实际应用场景

### 表单验证

```typescript
interface FormField {
  value: string;
  error?: string;
  required: boolean;
  minLength?: number;
  maxLength?: number;
  pattern?: RegExp;
}

interface FormData {
  [fieldName: string]: FormField;
}

class FormValidator {
  private data: FormData;

  constructor(data: FormData) {
    this.data = data;
  }

  validate(): boolean {
    let isValid = true;

    for (const [fieldName, field] of Object.entries(this.data)) {
      field.error = '';

      // 必填验证
      if (field.required && !field.value.trim()) {
        field.error = `${fieldName} is required`;
        isValid = false;
        continue;
      }

      // 最小长度验证
      if (field.minLength && field.value.length < field.minLength) {
        field.error = `${fieldName} must be at least ${field.minLength} characters`;
        isValid = false;
      }

      // 最大长度验证
      if (field.maxLength && field.value.length > field.maxLength) {
        field.error = `${fieldName} must be at most ${field.maxLength} characters`;
        isValid = false;
      }

      // 模式验证
      if (field.pattern && !field.pattern.test(field.value)) {
        field.error = `${fieldName} has invalid format`;
        isValid = false;
      }
    }

    return isValid;
  }

  getErrors(): { [fieldName: string]: string } {
    const errors: { [fieldName: string]: string } = {};
    for (const [fieldName, field] of Object.entries(this.data)) {
      if (field.error) {
        errors[fieldName] = field.error;
      }
    }
    return errors;
  }

  getFieldErrors(fieldName: string): string | undefined {
    return this.data[fieldName]?.error;
  }
}

// 使用示例
const userForm = new FormValidator({
  name: {
    value: '张三',
    required: true,
    minLength: 2,
    maxLength: 50
  },
  email: {
    value: 'zhangsan@example.com',
    required: true,
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  },
  password: {
    value: 'password123',
    required: true,
    minLength: 8,
    maxLength: 100
  }
});

if (userForm.validate()) {
  console.log('Form is valid!');
} else {
  console.log('Form errors:', userForm.getErrors());
}
```

### API 客户端类型安全

```typescript
interface ApiConfig {
  baseURL: string;
  timeout?: number;
  headers?: Record<string, string>;
}

class ApiClient {
  private config: ApiConfig;

  constructor(config: ApiConfig) {
    this.config = {
      timeout: 5000,
      headers: {
        'Content-Type': 'application/json'
      },
      ...config
    };
  }

  async get<T>(url: string, params?: Record<string, any>): Promise<T> {
    const queryString = params ? this.buildQueryString(params) : '';
    const response = await fetch(`${this.config.baseURL}${url}${queryString}`, {
      method: 'GET',
      headers: this.config.headers
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    return response.json();
  }

  async post<T>(url: string, data: any): Promise<T> {
    const response = await fetch(`${this.config.baseURL}${url}`, {
      method: 'POST',
      headers: this.config.headers,
      body: JSON.stringify(data)
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    return response.json();
  }

  async put<T>(url: string, data: any): Promise<T> {
    const response = await fetch(`${this.config.baseURL}${url}`, {
      method: 'PUT',
      headers: this.config.headers,
      body: JSON.stringify(data)
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    return response.json();
  }

  async delete<T>(url: string): Promise<T> {
    const response = await fetch(`${this.config.baseURL}${url}`, {
      method: 'DELETE',
      headers: this.config.headers
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    return response.json();
  }

  private buildQueryString(params: Record<string, any>): string {
    const searchParams = new URLSearchParams();
    for (const [key, value] of Object.entries(params)) {
      if (value !== undefined && value !== null) {
        searchParams.append(key, String(value));
      }
    }
    return searchParams.toString() ? `?${searchParams.toString()}` : '';
  }
}

// 定义 API 类型
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: string;
}

interface Post {
  id: number;
  title: string;
  content: string;
  authorId: number;
  createdAt: string;
}

interface CreateUserRequest {
  name: string;
  email: string;
  password: string;
}

interface UpdateUserRequest {
  name?: string;
  email?: string;
}

// 使用示例
const api = new ApiClient({
  baseURL: 'https://api.example.com'
});

async function exampleUsage() {
  try {
    // 获取用户列表
    const users: User[] = await api.get<User[]>('/users');

    // 获取单个用户
    const user: User = await api.get<User>('/users/1');

    // 创建用户
    const newUser: User = await api.post<User, CreateUserRequest>('/users', {
      name: '张三',
      email: 'zhangsan@example.com',
      password: 'password123'
    });

    // 更新用户
    const updatedUser: User = await api.put<User, UpdateUserRequest>('/users/1', {
      name: '李四'
    });

    // 删除用户
    await api.delete<void>('/users/1');

  } catch (error) {
    console.error('API error:', error);
  }
}
```

### 状态管理类型安全

```typescript
type Action<T = any> = {
  type: string;
  payload?: T;
};

interface State {
  user: User | null;
  posts: Post[];
  loading: boolean;
  error: string | null;
}

type StateActions =
  | Action<{ user: User }>
  | Action<{ posts: Post[] }>
  | Action<{ loading: boolean }>
  | Action<{ error: string }>;

function reducer(state: State, action: StateActions): State {
  switch (action.type) {
    case 'SET_USER':
      return { ...state, user: action.payload!.user };
    case 'SET_POSTS':
      return { ...state, posts: action.payload!.posts };
    case 'SET_LOADING':
      return { ...state, loading: action.payload!.loading };
    case 'SET_ERROR':
      return { ...state, error: action.payload!.error };
    default:
      return state;
  }
}

class Store<TState, TActions extends Action> {
  private state: TState;
  private listeners: Array<(state: TState) => void> = [];
  private reducer: (state: TState, action: TActions) => TState;

  constructor(initialState: TState, reducer: (state: TState, action: TActions) => TState) {
    this.state = initialState;
    this.reducer = reducer;
  }

  getState(): TState {
    return this.state;
  }

  dispatch(action: TActions): void {
    this.state = this.reducer(this.state, action);
    this.notifyListeners();
  }

  subscribe(listener: (state: TState) => void): () => void {
    this.listeners.push(listener);
    return () => {
      this.listeners = this.listeners.filter(l => l !== listener);
    };
  }

  private notifyListeners(): void {
    this.listeners.forEach(listener => listener(this.state));
  }
}

// 使用示例
const initialState: State = {
  user: null,
  posts: [],
  loading: false,
  error: null
};

const store = new Store<State, StateActions>(initialState, reducer);

// 订阅状态变化
const unsubscribe = store.subscribe((state) => {
  console.log('State changed:', state);
});

// 派发 action
store.dispatch({ type: 'SET_LOADING', payload: { loading: true } });
store.dispatch({ type: 'SET_USER', payload: { user: { id: 1, name: '张三', email: 'zhangsan@example.com', createdAt: new Date().toISOString() } } });
store.dispatch({ type: 'SET_LOADING', payload: { loading: false } });

// 取消订阅
unsubscribe();
```

## 高级类型技巧

### 类型守卫函数

```typescript
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function isNumber(value: unknown): value is number {
  return typeof value === 'number';
}

function isBoolean(value: unknown): value is boolean {
  return typeof value === 'boolean';
}

function isArray<T>(value: unknown, guard: (item: unknown) => item is T): value is T[] {
  return Array.isArray(value) && value.every(guard);
}

function isObject(value: unknown): value is Record<string, unknown> {
  return typeof value === 'object' && value !== null && !Array.isArray(value);
}

function validateUserData(data: unknown): data is User {
  if (!isObject(data)) return false;

  return (
    isNumber(data.id) &&
    isString(data.name) &&
    isString(data.email) &&
    isString(data.createdAt)
  );
}

// 使用示例
function processUserData(data: unknown) {
  if (validateUserData(data)) {
    console.log(`User: ${data.name} (${data.email})`);
  } else {
    console.log('Invalid user data');
  }
}
```

### 严格模式配置

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true
  }
}
```

### 类型断言 vs 类型守卫

```typescript
// 类型断言 - 危险，绕过类型检查
function dangerousFunction(value: unknown) {
  const user = value as User;
  console.log(user.name); // 如果 value 不是 User，运行时会出错
}

// 类型守卫 - 安全，进行运行时检查
function safeFunction(value: unknown) {
  if (validateUserData(value)) {
    console.log(value.name); // TypeScript 知道 value 是 User
  }
}
```

## 最佳实践

### 1. 优先使用类型推断

```typescript
// 好的做法 - 使用类型推断
const user = {
  id: 1,
  name: '张三',
  email: 'zhangsan@example.com'
}; // TypeScript 推断为 { id: number; name: string; email: string; }

// 不好的做法 - 过度注解
const user: {
  id: number;
  name: string;
  email: string;
} = {
  id: 1,
  name: '张三',
  email: 'zhangsan@example.com'
};
```

### 2. 函数参数和返回值类型

```typescript
// 好的做法 - 明确指定函数类型
function calculateTotal(price: number, quantity: number): number {
  return price * quantity;
}

// 回调函数类型
function processData(
  data: string[],
  callback: (result: string) => void
): void {
  const result = data.join(', ');
  callback(result);
}
```

### 3. 使用 unknown 而不是 any

```typescript
// 好的做法 - 使用 unknown
function parseJSON(json: string): unknown {
  return JSON.parse(json);
}

const data = parseJSON('{"name": "张三"}');
if (isObject(data) && isString(data.name)) {
  console.log(data.name.toUpperCase());
}

// 不好的做法 - 使用 any
function parseJSONUnsafe(json: string): any {
  return JSON.parse(json);
}

const dataUnsafe = parseJSONUnsafe('{"name": "张三"}');
console.log(dataUnsafe.name.toUpperCase()); // 可能运行时出错
```

## 常见问题解决

### 1. 如何处理动态属性？

```typescript
interface DynamicObject {
  [key: string]: string | number;
}

const obj: DynamicObject = {
  name: '张三',
  age: 25,
  city: '北京'
};

// 使用 Record 工具类型
type StringMap = Record<string, string>;
const stringMap: StringMap = {
  name: '张三',
  email: 'zhangsan@example.com'
};
```

### 2. 如何处理可选链？

```typescript
interface User {
  profile?: {
    address?: {
      city?: string;
    };
  };
}

function getUserCity(user: User): string {
  // 使用可选链操作符
  return user.profile?.address?.city ?? 'Unknown';
}

// 类型安全的使用方式
function safeGetCity(user: User): string | null {
  if (user.profile?.address?.city) {
    return user.profile.address.city;
  }
  return null;
}
```

### 3. 如何处理函数重载？

```typescript
function processInput(input: string): string;
function processInput(input: number): number;
function processInput(input: string | number): string | number {
  if (typeof input === 'string') {
    return input.toUpperCase();
  } else {
    return input * 2;
  }
}

const result1 = processInput('hello'); // string
const result2 = processInput(42);      // number
```

## 工具和配置

### 1. ESLint TypeScript 配置

```javascript
// .eslintrc.js
module.exports = {
  parser: '@typescript-eslint/parser',
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended'
  ],
  rules: {
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/explicit-function-return-type': 'off',
    '@typescript-eslint/no-unused-vars': 'error'
  }
};
```

### 2. Prettier TypeScript 配置

```javascript
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 80
}
```

### 3. VS Code 配置

```json
{
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.suggest.autoImports": true,
  "typescript.suggest.completeFunctionCalls": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

## 总结

TypeScript 的类型系统为 JavaScript 带来了静态类型检查，大大提高了代码的可维护性和可靠性。通过合理使用类型注解、类型推断、类型守卫等特性，可以编写更安全、更易于维护的代码。

关键要点：
- **类型安全**：在编译时捕获错误，减少运行时问题
- **代码提示**：IDE 智能提示，提高开发效率
- **重构信心**：类型系统让重构更加安全
- **文档作用**：类型即文档，提高代码可读性

最佳实践：
- 启用严格模式，充分利用类型系统
- 优先使用类型推断，避免过度注解
- 使用 unknown 而不是 any，保持类型安全
- 编写类型守卫函数，进行运行时验证
- 合理配置工具链，提高开发体验

TypeScript 的类型系统是前端开发的重要工具，掌握它能够让你编写出更加健壮、可维护的代码。记住，类型系统的目的是帮助开发者写出更好的代码，而不是增加开发的负担。