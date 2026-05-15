---
title: React Hooks 入门
published: 2024-08-21
description: 'React Hooks 详解：从概念到实践，掌握现代React开发的核心技能'
image: ''
tags: ["React","Hooks"]
category: '前端框架'
draft: false
lang: 'zh-CN'
---

## 概述

React Hooks 是 React 16.8 引入的革命性特性，它改变了我们编写 React 组件的方式。Hooks 允许我们在不编写 class 组件的情况下使用 state 和其他 React 特性，这使得代码更加简洁、易于理解和维护。

### 为什么需要 Hooks

在 Hooks 出现之前，React 开发者面临几个主要问题：

1. **组件逻辑复用困难**：在组件之间复用状态逻辑很复杂，需要使用高阶组件（HOC）或 render props，这会导致"包装地狱"
2. **复杂组件难以理解**：class 组件中的生命周期方法经常包含不相关的逻辑，使得组件难以拆分
3. **class 组件的学习曲线**：this 指向、事件绑定、构造函数等概念对初学者来说比较复杂

Hooks 解决了这些问题，让函数组件拥有状态和生命周期，同时保持了代码的简洁性。

## 核心概念

### 什么是 Hooks

Hooks 是一些可以让你在函数组件里"钩入" React state 及生命周期等特性的函数。所有的 Hooks 都以 `use` 开头，这是一个约定，帮助你在代码中快速识别它们。

### Hooks 的两个基本原则

1. **只在函数最外层调用 Hook**：不要在循环、条件判断或者子函数中调用 Hook
2. **只在 React 函数组件或自定义 Hook 中调用 Hook**：不要在普通的 JavaScript 函数中调用 Hook

### 内置 Hooks 分类

- **基础 Hooks**：useState, useEffect, useContext
- **额外的 Hooks**：useReducer, useCallback, useMemo, useRef, useImperativeHandle, useLayoutEffect, useDebugValue, useTransition, useDeferredValue, useId, useSyncExternalStore

## 基本用法

### useState Hook

useState 是最基本的 Hook，用于在函数组件中添加状态：

```jsx
import { useState } from 'react';

function Counter() {
  // 声明一个叫 count 的状态变量，初始值为 0
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>当前计数: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        增加
      </button>
      <button onClick={() => setCount(0)}>
        重置
      </button>
    </div>
  );
}
```

### useEffect Hook

useEffect 用于处理副作用，如数据获取、订阅、DOM 操作等：

```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // 副作用：获取用户数据
    setLoading(true);
    fetch(`/api/users/${userId}`)
      .then(response => response.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      });
    
    // 清理函数（可选）
    return () => {
      console.log('组件卸载或 userId 变化时执行');
    };
  }, [userId]); // 依赖数组：只有 userId 变化时才重新执行

  if (loading) return <div>加载中...</div>;
  return <div>用户名: {user?.name}</div>;
}
```

### useContext Hook

useContext 用于在组件树中传递数据，避免 prop drilling：

```jsx
import { createContext, useContext, useState } from 'react';

// 创建 Context
const ThemeContext = createContext();

// 提供者组件
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 消费者组件
function ThemeButton() {
  const { theme, setTheme } = useContext(ThemeContext);
  
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      切换主题: {theme}
    </button>
  );
}
```

## 实际应用

### 自定义 Hook

自定义 Hook 是复用组件逻辑的强大工具：

```jsx
// 自定义 Hook：处理表单输入
function useForm(initialValues) {
  const [values, setValues] = useState(initialValues);
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues(prev => ({
      ...prev,
      [name]: value
    }));
  };
  
  const reset = () => {
    setValues(initialValues);
  };
  
  return [values, handleChange, reset];
}

// 使用自定义 Hook
function LoginForm() {
  const [form, handleChange, reset] = useForm({
    username: '',
    password: ''
  });
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('提交表单:', form);
    reset();
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        name="username"
        value={form.username}
        onChange={handleChange}
        placeholder="用户名"
      />
      <input
        name="password"
        type="password"
        value={form.password}
        onChange={handleChange}
        placeholder="密码"
      />
      <button type="submit">登录</button>
      <button type="button" onClick={reset}>重置</button>
    </form>
  );
}
```

### 数据获取 Hook

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const abortController = new AbortController();

    async function fetchData() {
      try {
        setLoading(true);
        setError(null);
        const response = await fetch(url, {
          signal: abortController.signal
        });
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const result = await response.json();
        setData(result);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchData();
    
    return () => {
      abortController.abort();
    };
  }, [url]);

  return { data, loading, error };
}

// 使用
function UserList() {
  const { data: users, loading, error } = useFetch('/api/users');
  
  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误: {error}</div>;
  
  return (
    <ul>
      {users?.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

## 注意事项

### 1. 遵循 Hooks 规则

**错误示例：**
```jsx
// ❌ 错误：在条件语句中使用 Hook
function BadComponent() {
  if (someCondition) {
    const [state, setState] = useState(0); // 错误！
  }
  return <div>...</div>;
}

// ❌ 错误：在循环中使用 Hook
function BadComponent() {
  const items = [1, 2, 3];
  items.forEach(item => {
    const [state, setState] = useState(item); // 错误！
  });
  return <div>...</div>;
}
```

**正确示例：**
```jsx
// ✅ 正确：在函数顶层使用 Hook
function GoodComponent({ someCondition }) {
  const [state, setState] = useState(0);
  
  if (someCondition) {
    // 在条件语句中使用 state，而不是创建 Hook
    return <div>{state}</div>;
  }
  return <div>{state}</div>;
}
```

### 2. 依赖数组的正确使用

```jsx
// ❌ 错误：忘记依赖项
function BadComponent({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, []); // 缺少 userId 依赖
  
  return <div>{user?.name}</div>;
}

// ✅ 正确：包含所有依赖项
function GoodComponent({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]); // 包含 userId 依赖
  
  return <div>{user?.name}</div>;
}
```

### 3. 闭包陷阱

```jsx
// ❌ 闭包陷阱
function Timer() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      console.log(count); // 总是打印 0
      setCount(count + 1);
    }, 1000);
    
    return () => clearInterval(interval);
  }, []); // 依赖数组为空，count 永远是初始值
  
  return <div>{count}</div>;
}

// ✅ 解决方案1：使用函数式更新
function TimerFixed() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setCount(prev => prev + 1); // 使用函数式更新
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);
  
  return <div>{count}</div>;
}

// ✅ 解决方案2：添加正确的依赖
function TimerFixed2() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      console.log(count);
      setCount(count + 1);
    }, 1000);
    
    return () => clearInterval(interval);
  }, [count]); // 依赖 count
  
  return <div>{count}</div>;
}
```

### 4. 性能优化

```jsx
// ❌ 每次渲染都创建新函数
function ParentComponent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <Child onClick={() => setCount(count + 1)} />
      <button onClick={() => setCount(count + 1)}>增加</button>
    </div>
  );
}

// ✅ 使用 useCallback 优化
import { useCallback } from 'react';

function OptimizedParentComponent() {
  const [count, setCount] = useState(0);
  
  const increment = useCallback(() => {
    setCount(prev => prev + 1);
  }, []); // 空依赖数组，函数不会重新创建
  
  return (
    <div>
      <Child onClick={increment} />
      <button onClick={increment}>增加</button>
    </div>
  );
}
```

## 总结

React Hooks 彻底改变了我们编写 React 应用的方式：

### 核心优势

1. **代码更简洁**：函数组件比 class 组件更简洁易懂
2. **逻辑复用**：自定义 Hook 提供了强大的代码复用能力
3. **减少样板代码**：不再需要构造函数、绑定方法等
4. **更好的类型推断**：配合 TypeScript 提供更好的类型支持

### 学习路径

1. **掌握基础 Hooks**：useState, useEffect, useContext
2. **理解性能优化**：useCallback, useMemo, React.memo
3. **学习高级特性**：useReducer, useRef, useLayoutEffect
4. **实践自定义 Hook**：封装可复用的业务逻辑

### 最佳实践

1. **始终遵循 Hooks 规则**：只在顶层调用，只在 React 函数中调用
2. **合理使用依赖数组**：确保包含所有外部依赖
3. **注意闭包陷阱**：理解闭包的影响，使用适当的更新策略
4. **不要过早优化**：性能优化（useCallback, useMemo）只在必要时使用
5. **编写可读的代码**：自定义 Hook 应该有清晰的名称和职责

Hooks 使 React 开发更加现代化和高效。掌握 Hooks 是成为优秀 React 开发者的必备技能，它为函数组件提供了强大的能力，同时保持了代码的简洁性和可维护性。