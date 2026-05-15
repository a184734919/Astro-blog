---
title: React TypeScript
published: 2023-10-23
description: 'TS 项目配置的详细介绍和学习笔记'
image: ''
tags: ["React","TypeScript"]
category: 'React'
draft: false
lang: 'zh-CN'
---

## 概述

React 与 TypeScript 的结合为前端开发带来了强大的类型安全保障。TypeScript 能够在开发阶段捕获潜在的错误，提供更好的代码提示，并改善代码的可维护性。本文将详细介绍如何在 React 项目中配置和使用 TypeScript，包括项目设置、组件类型定义、Hooks 类型、泛型组件等核心内容。

## 核心概念

### TypeScript 在 React 中的优势

1. **类型安全**：在编译时捕获类型错误，避免运行时问题
2. **更好的 IDE 支持**：智能代码补全、参数提示、重构功能
3. **自文档化**：类型定义充当代码文档
4. **重构安全性**：类型系统确保重构不会破坏代码
5. **团队协作**：统一类型定义减少沟通成本

### 基本类型系统

TypeScript 提供了丰富的类型系统：
- 基础类型：string、number、boolean、null、undefined
- 对象类型：interface、type、class
- 数组类型：Array<T>、T[]
- 函数类型：参数类型、返回类型
- 高级类型：联合类型、交叉类型、泛型

### React 类型系统

React 提供了专门的类型定义：
- 组件 Props 类型
- 组件 Ref 类型
- Hooks 类型
- 事件类型
- 表单类型

## 基本用法

### 项目初始化配置

```bash
# 创建新的 TypeScript React 项目
npm create vite@latest my-app -- --template react-ts

# 或者在现有项目中添加 TypeScript
npm install --save-dev typescript @types/react @types/react-dom @types/node
```

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,

    /* Bundler mode */
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",

    /* Linting */
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true,

    /* Path mapping */
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"],
      "@/utils/*": ["./src/utils/*"],
      "@/types/*": ["./src/types/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### 基础组件类型定义

```tsx
// 基础组件示例
interface ButtonProps {
  children: React.ReactNode;
  onClick?: () => void;
  disabled?: boolean;
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'small' | 'medium' | 'large';
}

function Button({ children, onClick, disabled, variant = 'primary', size = 'medium' }: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant} btn-${size}`}
    >
      {children}
    </button>
  );
}

// 使用组件
function App() {
  return (
    <div>
      <Button onClick={() => console.log('clicked')}>
        Click me
      </Button>
      <Button variant="secondary" size="large">
        Large Button
      </Button>
    </div>
  );
}

// 可选 props 和默认值
interface CardProps {
  title: string;
  content: string;
  imageUrl?: string;
  showActions?: boolean;
}

function Card({ title, content, imageUrl, showActions = true }: CardProps) {
  return (
    <div className="card">
      {imageUrl && <img src={imageUrl} alt={title} />}
      <h3>{title}</h3>
      <p>{content}</p>
      {showActions && (
        <div className="card-actions">
          <Button>Edit</Button>
          <Button variant="danger">Delete</Button>
        </div>
      )}
    </div>
  );
}
```

### 复杂组件类型定义

```tsx
// 用户卡片组件
interface User {
  id: number;
  name: string;
  email: string;
  avatar?: string;
  role: 'admin' | 'user' | 'moderator';
  createdAt: string;
}

interface UserCardProps {
  user: User;
  onEdit?: (userId: number) => void;
  onDelete?: (userId: number) => void;
  showActions?: boolean;
  customClass?: string;
}

function UserCard({ user, onEdit, onDelete, showActions = true, customClass }: UserCardProps) {
  const handleEdit = () => {
    onEdit?.(user.id);
  };

  const handleDelete = () => {
    if (window.confirm(`Are you sure you want to delete ${user.name}?`)) {
      onDelete?.(user.id);
    }
  };

  return (
    <div className={`user-card ${customClass || ''}`}>
      <div className="user-avatar">
        {user.avatar ? (
          <img src={user.avatar} alt={user.name} />
        ) : (
          <div className="avatar-placeholder">{user.name.charAt(0)}</div>
        )}
      </div>

      <div className="user-info">
        <h4>{user.name}</h4>
        <p>{user.email}</p>
        <span className={`role-badge role-${user.role}`}>{user.role}</span>
      </div>

      {showActions && (
        <div className="user-actions">
          <Button onClick={handleEdit} size="small">Edit</Button>
          <Button onClick={handleDelete} variant="danger" size="small">Delete</Button>
        </div>
      )}
    </div>
  );
}

// 列表组件
interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => React.ReactNode;
  keyExtractor: (item: T) => string | number;
  emptyMessage?: string;
  loading?: boolean;
}

function List<T>({ items, renderItem, keyExtractor, emptyMessage = 'No items', loading }: ListProps<T>) {
  if (loading) {
    return <div className="loading">Loading...</div>;
  }

  if (items.length === 0) {
    return <div className="empty">{emptyMessage}</div>;
  }

  return (
    <ul className="list">
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  );
}

// 使用泛型组件
function UserList() {
  const users: User[] = [
    {
      id: 1,
      name: 'John Doe',
      email: 'john@example.com',
      role: 'admin',
      createdAt: '2024-01-01'
    },
    {
      id: 2,
      name: 'Jane Smith',
      email: 'jane@example.com',
      role: 'user',
      createdAt: '2024-01-02'
    }
  ];

  return (
    <List
      items={users}
      keyExtractor={(user) => user.id}
      renderItem={(user) => (
        <UserCard
          user={user}
          onEdit={(id) => console.log('Edit user:', id)}
          onDelete={(id) => console.log('Delete user:', id)}
        />
      )}
    />
  );
}
```

### Hooks 类型定义

```tsx
// useState 类型定义
function Counter() {
  const [count, setCount] = useState<number>(0);
  const [user, setUser] = useState<User | null>(null);
  const [items, setItems] = useState<string[]>([]);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

// useEffect 类型定义
function DataFetcher() {
  const [data, setData] = useState<User[]>([]);
  const [loading, setLoading] = useState<boolean>(true);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch('/api/users');
        const users: User[] = await response.json();
        setData(users);
      } catch (error) {
        console.error('Failed to fetch users:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, []); // 空依赖数组

  if (loading) return <div>Loading...</div>;

  return (
    <ul>
      {data.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// useRef 类型定义
function TextInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  const focusInput = () => {
    inputRef.current?.focus();
  };

  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus Input</button>
    </div>
  );
}

// 自定义 Hook 类型定义
function useFetch<T>(url: string): { data: T[]; loading: boolean; error: string | null } {
  const [data, setData] = useState<T[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        const response = await fetch(url);
        const result: T[] = await response.json();
        setData(result);
        setError(null);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'An error occurred');
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return { data, loading, error };
}

// 使用自定义 Hook
function UsersPage() {
  const { data: users, loading, error } = useFetch<User>('/api/users');

  if (loading) return <div>Loading users...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <h2>Users</h2>
      <ul>
        {users.map(user => (
          <li key={user.id}>{user.name} ({user.email})</li>
        ))}
      </ul>
    </div>
  );
}

// useCallback 类型定义
function ParentComponent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback((id: number) => {
    console.log('Item clicked:', id);
  }, []); // 空依赖数组表示函数引用稳定

  const handleIncrement = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);

  return (
    <div>
      <button onClick={handleIncrement}>Count: {count}</button>
      <ChildComponent onClick={handleClick} />
    </div>
  );
}

interface ChildComponentProps {
  onClick: (id: number) => void;
}

function ChildComponent({ onClick }: ChildComponentProps) {
  return (
    <div>
      <button onClick={() => onClick(1)}>Item 1</button>
      <button onClick={() => onClick(2)}>Item 2</button>
    </div>
  );
}

// useMemo 类型定义
function DataProcessor({ items }: { items: Item[] }) {
  const statistics = useMemo<{
    total: number;
    average: number;
    max: number;
    min: number;
  }>(() => {
    if (items.length === 0) {
      return { total: 0, average: 0, max: 0, min: 0 };
    }

    const values = items.map(item => item.value);
    const total = values.reduce((sum, value) => sum + value, 0);
    const average = total / values.length;
    const max = Math.max(...values);
    const min = Math.min(...values);

    return { total, average, max, min };
  }, [items]);

  return (
    <div>
      <p>Total: {statistics.total}</p>
      <p>Average: {statistics.average.toFixed(2)}</p>
      <p>Max: {statistics.max}</p>
      <p>Min: {statistics.min}</p>
    </div>
  );
}
```

### 事件类型定义

```tsx
// 事件处理器类型定义
interface FormProps {
  onSubmit: (data: { username: string; email: string }) => void;
}

function LoginForm({ onSubmit }: FormProps) {
  const [username, setUsername] = useState('');
  const [email, setEmail] = useState('');

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    onSubmit({ username, email });
  };

  const handleUsernameChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setUsername(e.target.value);
  };

  const handleEmailChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setEmail(e.target.value);
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="username">Username:</label>
        <input
          id="username"
          type="text"
          value={username}
          onChange={handleUsernameChange}
        />
      </div>
      <div>
        <label htmlFor="email">Email:</label>
        <input
          id="email"
          type="email"
          value={email}
          onChange={handleEmailChange}
        />
      </div>
      <button type="submit">Submit</button>
    </form>
  );
}

// 键盘事件
function SearchBox() {
  const [query, setQuery] = useState('');

  const handleKeyDown = (e: React.KeyboardEvent<HTMLInputElement>) => {
    if (e.key === 'Enter') {
      console.log('Search:', query);
    }
  };

  return (
    <input
      type="text"
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      onKeyDown={handleKeyDown}
      placeholder="Search..."
    />
  );
}

// 鼠标事件
function DraggableBox() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  const [isDragging, setIsDragging] = useState(false);

  const handleMouseDown = (e: React.MouseEvent<HTMLDivElement>) => {
    setIsDragging(true);
  };

  const handleMouseMove = (e: React.MouseEvent<HTMLDivElement>) => {
    if (isDragging) {
      setPosition({
        x: e.clientX,
        y: e.clientY
      });
    }
  };

  const handleMouseUp = () => {
    setIsDragging(false);
  };

  return (
    <div
      onMouseDown={handleMouseDown}
      onMouseMove={handleMouseMove}
      onMouseUp={handleMouseUp}
      style={{
        position: 'absolute',
        left: position.x,
        top: position.y,
        width: '100px',
        height: '100px',
        background: isDragging ? 'blue' : 'gray',
        cursor: 'move'
      }}
    >
      Drag me
    </div>
  );
}
```

## 实际应用

### 完整的 CRUD 应用类型定义

```tsx
// types/user.ts
export interface User {
  id: number;
  name: string;
  email: string;
  avatar?: string;
  role: UserRole;
  status: UserStatus;
  createdAt: string;
  updatedAt: string;
}

export type UserRole = 'admin' | 'user' | 'moderator';
export type UserStatus = 'active' | 'inactive' | 'suspended';

export interface CreateUserDto {
  name: string;
  email: string;
  role: UserRole;
}

export interface UpdateUserDto {
  name?: string;
  email?: string;
  avatar?: string;
  role?: UserRole;
  status?: UserStatus;
}

export interface UserListParams {
  page?: number;
  limit?: number;
  search?: string;
  role?: UserRole;
  status?: UserStatus;
}

export interface PaginatedResponse<T> {
  data: T[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
}

// services/userService.ts
import { User, CreateUserDto, UpdateUserDto, UserListParams, PaginatedResponse } from '../types/user';

class UserService {
  private baseUrl = '/api/users';

  async getUsers(params: UserListParams = {}): Promise<PaginatedResponse<User>> {
    const queryParams = new URLSearchParams();
    Object.entries(params).forEach(([key, value]) => {
      if (value !== undefined) {
        queryParams.append(key, String(value));
      }
    });

    const response = await fetch(`${this.baseUrl}?${queryParams.toString()}`);
    if (!response.ok) {
      throw new Error('Failed to fetch users');
    }
    return response.json();
  }

  async getUserById(id: number): Promise<User> {
    const response = await fetch(`${this.baseUrl}/${id}`);
    if (!response.ok) {
      throw new Error('Failed to fetch user');
    }
    return response.json();
  }

  async createUser(data: CreateUserDto): Promise<User> {
    const response = await fetch(this.baseUrl, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });

    if (!response.ok) {
      throw new Error('Failed to create user');
    }
    return response.json();
  }

  async updateUser(id: number, data: UpdateUserDto): Promise<User> {
    const response = await fetch(`${this.baseUrl}/${id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });

    if (!response.ok) {
      throw new Error('Failed to update user');
    }
    return response.json();
  }

  async deleteUser(id: number): Promise<void> {
    const response = await fetch(`${this.baseUrl}/${id}`, {
      method: 'DELETE'
    });

    if (!response.ok) {
      throw new Error('Failed to delete user');
    }
  }
}

export const userService = new UserService();

// hooks/useUsers.ts
import { useState, useEffect } from 'react';
import { User, UserListParams } from '../types/user';
import { userService } from '../services/userService';

export function useUsers(params: UserListParams = {}) {
  const [users, setUsers] = useState<User[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [total, setTotal] = useState(0);

  useEffect(() => {
    const fetchUsers = async () => {
      try {
        setLoading(true);
        setError(null);
        const response = await userService.getUsers(params);
        setUsers(response.data);
        setTotal(response.total);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Failed to fetch users');
      } finally {
        setLoading(false);
      }
    };

    fetchUsers();
  }, [JSON.stringify(params)]);

  return { users, loading, error, total };
}

export function useUser(id: number) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        setError(null);
        const userData = await userService.getUserById(id);
        setUser(userData);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Failed to fetch user');
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [id]);

  return { user, loading, error };
}

// components/UserTable.tsx
import React, { useState } from 'react';
import { User, UserRole, UserStatus } from '../types/user';
import { useUsers } from '../hooks/useUsers';

interface UserTableProps {
  onEdit?: (user: User) => void;
  onDelete?: (userId: number) => void;
}

function UserTable({ onEdit, onDelete }: UserTableProps) {
  const [page, setPage] = useState(1);
  const [search, setSearch] = useState('');
  const [roleFilter, setRoleFilter] = useState<UserRole | 'all'>('all');
  const { users, loading, error, total } = useUsers({
    page,
    limit: 10,
    search,
    role: roleFilter === 'all' ? undefined : roleFilter
  });

  const handleSearch = (e: React.ChangeEvent<HTMLInputElement>) => {
    setSearch(e.target.value);
    setPage(1);
  };

  const handleRoleFilter = (e: React.ChangeEvent<HTMLSelectElement>) => {
    setRoleFilter(e.target.value as UserRole | 'all');
    setPage(1);
  };

  if (loading) return <div>Loading users...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div className="user-table-container">
      <div className="table-controls">
        <input
          type="text"
          value={search}
          onChange={handleSearch}
          placeholder="Search users..."
          className="search-input"
        />

        <select value={roleFilter} onChange={handleRoleFilter} className="role-filter">
          <option value="all">All Roles</option>
          <option value="admin">Admin</option>
          <option value="user">User</option>
          <option value="moderator">Moderator</option>
        </select>
      </div>

      <table className="user-table">
        <thead>
          <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Email</th>
            <th>Role</th>
            <th>Status</th>
            <th>Created At</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          {users.map(user => (
            <tr key={user.id}>
              <td>{user.id}</td>
              <td>
                <div className="user-cell">
                  {user.avatar && (
                    <img src={user.avatar} alt={user.name} className="avatar-small" />
                  )}
                  <span>{user.name}</span>
                </div>
              </td>
              <td>{user.email}</td>
              <td>
                <span className={`role-badge role-${user.role}`}>
                  {user.role}
                </span>
              </td>
              <td>
                <span className={`status-badge status-${user.status}`}>
                  {user.status}
                </span>
              </td>
              <td>{new Date(user.createdAt).toLocaleDateString()}</td>
              <td>
                <button onClick={() => onEdit?.(user)}>Edit</button>
                <button onClick={() => onDelete?.(user.id)}>Delete</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>

      <div className="pagination">
        <button
          onClick={() => setPage(p => Math.max(1, p - 1))}
          disabled={page === 1}
        >
          Previous
        </button>
        <span>Page {page}</span>
        <button
          onClick={() => setPage(p => p + 1)}
          disabled={users.length === 0}
        >
          Next
        </button>
      </div>
    </div>
  );
}

export default UserTable;
```

### 高级类型技巧

```tsx
// 条件类型
type NonNullable<T> = T extends null | undefined ? never : T;

// 映射类型
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Partial<T> = {
  [P in keyof T]?: T[P];
};

// 模板字面量类型
type EventName<T extends string> = `on${Capitalize<T>}`;

type ButtonEvents = EventName<'click' | 'hover' | 'focus'>;
// 结果: 'onClick' | 'onHover' | 'onFocus'

// 索引访问类型
interface User {
  id: number;
  name: string;
  email: string;
}

type UserName = User['name']; // string
type UserKeys = keyof User; // 'id' | 'name' | 'email'

// 工具类型
type RequiredKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? never : K;
}[keyof T];

type OptionalKeys<T> = {
  [K in keyof T]-?: {} extends Pick<T, K> ? K : never;
}[keyof T];

// 联合类型工具
type UnionToIntersection<U> = (U extends any ? (k: U) => void : never) extends (k: infer I) => void ? I : never;

// React 高级类型
type PropsWithChildren<P> = P & { children?: React.ReactNode };

interface ButtonProps {
  variant: 'primary' | 'secondary';
  onClick: () => void;
}

type ButtonWithChildren = PropsWithChildren<ButtonProps>;

function Button({ children, variant, onClick }: ButtonWithChildren) {
  return (
    <button className={`btn-${variant}`} onClick={onClick}>
      {children}
    </button>
  );
}

// 类型守卫
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value &&
    'email' in value
  );
}

function processValue(value: unknown) {
  if (isUser(value)) {
    console.log(`Processing user: ${value.name}`);
  } else {
    console.log('Value is not a user');
  }
}

// 类型推断
function createUser<T extends object>(initial: T): T {
  return initial;
}

const user = createUser({
  name: 'John',
  age: 30,
  email: 'john@example.com'
});

// 类型: { name: string; age: number; email: string; }
```

### 类型安全的 Redux 集成

```tsx
// store/index.ts
import { configureStore, createSlice, PayloadAction } from '@reduxjs/toolkit';

// 类型定义
interface CounterState {
  value: number;
}

interface UserState {
  currentUser: User | null;
  users: User[];
  loading: boolean;
  error: string | null;
}

interface RootState {
  counter: CounterState;
  user: UserState;
}

// Counter Slice
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 } as CounterState,
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action: PayloadAction<number>) => {
      state.value += action.payload;
    }
  }
});

// User Slice
const userSlice = createSlice({
  name: 'user',
  initialState: {
    currentUser: null,
    users: [],
    loading: false,
    error: null
  } as UserState,
  reducers: {
    setUser: (state, action: PayloadAction<User>) => {
      state.currentUser = action.payload;
    },
    setUsers: (state, action: PayloadAction<User[]>) => {
      state.users = action.payload;
    },
    setLoading: (state, action: PayloadAction<boolean>) => {
      state.loading = action.payload;
    },
    setError: (state, action: PayloadAction<string | null>) => {
      state.error = action.payload;
    }
  }
});

// Store 配置
const store = configureStore({
  reducer: {
    counter: counterSlice.reducer,
    user: userSlice.reducer
  }
});

// 类型导出
export type AppDispatch = typeof store.dispatch;
export type RootState = ReturnType<typeof store.getState>;

// Hooks
import { useDispatch, useSelector } from 'react-redux';
import type { TypedUseSelectorHook } from 'react-redux';

export const useAppDispatch: () => AppDispatch = useDispatch;
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;

// Actions
export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export const { setUser, setUsers, setLoading, setError } = userSlice.actions;

export default store;

// 使用示例
function Counter() {
  const count = useAppSelector(state => state.counter.value);
  const dispatch = useAppDispatch();

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => dispatch(increment())}>Increment</button>
      <button onClick={() => dispatch(decrement())}>Decrement</button>
    </div>
  );
}

function UserProfile() {
  const user = useAppSelector(state => state.user.currentUser);
  const dispatch = useAppDispatch();

  useEffect(() => {
    // 获取用户数据
    const fetchUser = async () => {
      dispatch(setLoading(true));
      try {
        const response = await fetch('/api/user');
        const userData: User = await response.json();
        dispatch(setUser(userData));
      } catch (error) {
        dispatch(setError('Failed to fetch user'));
      } finally {
        dispatch(setLoading(false));
      }
    };

    fetchUser();
  }, [dispatch]);

  if (!user) return <div>Loading...</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

## 注意事项

### 避免 any 类型

```tsx
// 问题：过度使用 any
function processData(data: any) {
  return data.map((item: any) => item.value);
}

// 解决方案：使用具体类型
interface DataItem {
  id: number;
  value: string;
}

function processData(data: DataItem[]): string[] {
  return data.map(item => item.value);
}

// 或使用泛型
function processData<T extends { value: string }>(data: T[]): string[] {
  return data.map(item => item.value);
}
```

### 类型推断 vs 显式类型

```tsx
// 依赖类型推断（简洁）
const count = useState(0);
const users = useState<User[]>([]);

// 显式类型定义（清晰）
const [count, setCount] = useState<number>(0);
const [users, setUsers] = useState<User[]>([]);

// 复杂类型需要显式定义
const [config, setConfig] = useState<{
  theme: 'light' | 'dark';
  language: string;
  notifications: boolean;
}>({
  theme: 'light',
  language: 'en',
  notifications: true
});
```

### 正确处理 null 和 undefined

```tsx
// 问题：忽略 null 检查
function getUserName(user: User | null) {
  return user.name; // 可能为 null 的错误
}

// 解决方案1：类型守卫
function getUserName(user: User | null) {
  if (!user) return 'Unknown';
  return user.name;
}

// 解决方案2：可选链
function getUserName(user: User | null) {
  return user?.name ?? 'Unknown';
}

// 解决方案3：空值合并
function getUserName(user: User | null) {
  return user?.name || 'Unknown';
}

// 解决方案4：非空断言（谨慎使用）
function getUserName(user: User | null) {
  return user!.name; // 确保不为 null 时使用
}
```

### 类型断言的合理使用

```tsx
// 问题：过度使用类型断言
const data = response as User; // 危险，可能运行时错误

// 解决方案：类型守卫
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'name' in value &&
    'email' in value
  );
}

if (isUser(response)) {
  console.log(response.name); // 类型安全
}

// 泛型约束
function processUser<T extends User>(user: T): T {
  return user;
}
```

## 总结

React TypeScript 的结合为现代前端开发提供了强大的类型安全保障和开发体验提升：

### 核心优势

1. **类型安全**：在编译时捕获错误，减少运行时问题
2. **更好的 IDE 支持**：智能代码补全、重构、导航功能
3. **自文档化**：类型定义充当活的文档
4. **团队协作**：统一的类型定义减少沟通成本
5. **重构安全**：类型系统确保重构不会破坏代码

### 最佳实践

1. **类型定义**：创建专门的类型文件，避免重复定义
2. **避免 any**：尽量减少 any 的使用，使用具体类型或泛型
3. **工具类型**：充分利用 TypeScript 的工具类型简化代码
4. **类型守卫**：使用类型守卫确保运行时类型安全
5. **逐步迁移**：在现有项目中逐步引入 TypeScript

### 配置建议

1. **启用严格模式**：在 tsconfig.json 中设置 `"strict": true`
2. **路径别名**：配置路径别名简化导入路径
3. **ESLint 集成**：结合 ESLint 提供代码质量检查
4. **Prettier 集成**：统一代码格式化风格

通过合理应用 TypeScript，可以构建出更加健壮、可维护的 React 应用。类型系统的投入会在项目规模增长时带来显著的回报，特别是在团队协作和长期维护方面。