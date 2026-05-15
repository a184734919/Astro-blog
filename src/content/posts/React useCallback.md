---
title: React useCallback
published: 2023-08-08
description: '深入理解 React useCallback Hook：性能优化和回调函数缓存'
image: ''
tags: ["React","Hooks"]
category: 'React'
draft: false
lang: 'zh-CN'
---

## 概述

useCallback 是 React 提供的性能优化 Hook，用于缓存函数引用。它返回一个 memoized（记忆化）的回调函数，该函数只在依赖项发生变化时才会更新。这对于避免不必要的子组件重渲染、优化性能非常重要。

### 为什么需要 useCallback

在 React 组件中，每次渲染都会创建新的函数实例，这可能导致以下问题：

1. **子组件不必要的重渲染**：传递给子组件的函数 prop 每次都是新的引用
2. **依赖该函数的副作用频繁执行**：useEffect 的依赖数组包含该函数时
3. **性能浪费**：在大型组件树中，不必要的重渲染会影响性能

useCallback 通过缓存函数引用解决了这些问题。

## 核心概念

### 函数引用问题

```jsx
// ❌ 问题：每次渲染都创建新的函数
function ParentComponent() {
  const [count, setCount] = useState(0);
  
  // 每次渲染都会创建新的 handleClick 函数
  const handleClick = () => {
    setCount(count + 1);
  };
  
  return (
    <div>
      <ChildComponent onClick={handleClick} />
      <button onClick={handleClick}>Count: {count}</button>
    </div>
  );
}

// ChildComponent 会因为 onClick prop 变化而重渲染
const ChildComponent = React.memo(({ onClick }) => {
  console.log('ChildComponent 渲染');
  return <button onClick={onClick}>子组件</button>;
});
```

### useCallback 的工作原理

useCallback 返回一个稳定的函数引用，只有在依赖项变化时才会创建新的函数：

```jsx
// ✅ 解决方案：使用 useCallback 缓存函数
function OptimizedParentComponent() {
  const [count, setCount] = useState(0);
  
  // 只有 count 变化时才会创建新的函数
  const handleClick = useCallback(() => {
    setCount(count + 1);
  }, [count]);
  
  return (
    <div>
      <ChildComponent onClick={handleClick} />
      <button onClick={handleClick}>Count: {count}</button>
    </div>
  );
}
```

### useCallback 签名

```jsx
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b]
);
```

- **第一个参数**：要缓存的函数
- **第二个参数**：依赖数组，函数在这些值变化时重新创建

## 基本用法

### 基本语法

```jsx
import { useCallback, useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  // 使用 useCallback 缓存函数
  const increment = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);
  
  const decrement = useCallback(() => {
    setCount(prev => prev - 1);
  }, []);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>增加</button>
      <button onClick={decrement}>减少</button>
    </div>
  );
}
```

### 带参数的回调

```jsx
function ItemList({ items }) {
  const [selectedId, setSelectedId] = useState(null);
  
  // 带参数的回调
  const handleSelect = useCallback((id) => {
    setSelectedId(id);
  }, []);
  
  const handleDelete = useCallback((id) => {
    console.log('删除项目:', id);
  }, []);
  
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>
          <span>{item.name}</span>
          <button onClick={() => handleSelect(item.id)}>
            选择
          </button>
          <button onClick={() => handleDelete(item.id)}>
            删除
          </button>
        </li>
      ))}
    </ul>
  );
}
```

### 函数式更新

```jsx
function OptimizedCounter() {
  const [count, setCount] = useState(0);
  
  // 使用函数式更新，避免依赖 count
  const increment = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);
  
  const multiply = useCallback((factor) => {
    setCount(prev => prev * factor);
  }, []);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>增加</button>
      <button onClick={() => multiply(2)}>加倍</button>
    </div>
  );
}
```

## 实际应用

### 防止子组件不必要的重渲染

```jsx
import { useState, useCallback, memo } from 'react';

// 使用 React.memo 优化子组件
const ExpensiveChild = memo(({ data, onUpdate }) => {
  console.log('ExpensiveChild 渲染');
  // 模拟昂贵操作
  const result = data.reduce((sum, item) => sum + item.value, 0);
  
  return (
    <div>
      <p>总和: {result}</p>
      <button onClick={onUpdate}>更新数据</button>
    </div>
  );
});

function ParentComponent() {
  const [count, setCount] = useState(0);
  const [data] = useState([
    { id: 1, value: 10 },
    { id: 2, value: 20 },
    { id: 3, value: 30 }
  ]);
  
  // 缓存回调函数，避免子组件不必要的重渲染
  const handleUpdate = useCallback(() => {
    console.log('更新数据');
  }, []);
  
  return (
    <div>
      <h2>父组件</h2>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        增加 Count（不影响子组件）
      </button>
      
      <ExpensiveChild data={data} onUpdate={handleUpdate} />
    </div>
  );
}
```

### 优化事件处理函数

```jsx
function EventHandlers() {
  const [isOnline, setIsOnline] = useState(false);
  const [userName, setUserName] = useState('');
  
  // 鼠标事件处理器
  const handleMouseEnter = useCallback(() => {
    console.log('鼠标进入');
  }, []);
  
  const handleMouseLeave = useCallback(() => {
    console.log('鼠标离开');
  }, []);
  
  // 表单事件处理器
  const handleInputChange = useCallback((e) => {
    setUserName(e.target.value);
  }, []);
  
  const handleSubmit = useCallback((e) => {
    e.preventDefault();
    console.log('提交表单:', userName);
  }, [userName]);
  
  return (
    <form onSubmit={handleSubmit}>
      <div
        onMouseEnter={handleMouseEnter}
        onMouseLeave={handleMouseLeave}
      >
        <input
          value={userName}
          onChange={handleInputChange}
          placeholder="用户名"
        />
        <button type="submit">提交</button>
      </div>
    </form>
  );
}
```

### 列表操作优化

```jsx
function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: '学习 React', completed: false },
    { id: 2, text: '掌握 Hooks', completed: false },
    { id: 3, text: '实践项目', completed: false }
  ]);
  const [filter, setFilter] = useState('all');
  
  // 添加待办事项
  const addTodo = useCallback((text) => {
    setTodos(prev => [
      ...prev,
      {
        id: Date.now(),
        text,
        completed: false
      }
    ]);
  }, []);
  
  // 删除待办事项
  const removeTodo = useCallback((id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  }, []);
  
  // 切换完成状态
  const toggleTodo = useCallback((id) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  }, []);
  
  // 过滤待办事项
  const filteredTodos = todos.filter(todo => {
    if (filter === 'completed') return todo.completed;
    if (filter === 'active') return !todo.completed;
    return true;
  });
  
  return (
    <div>
      <div>
        <button onClick={() => setFilter('all')}>全部</button>
        <button onClick={() => setFilter('active')}>未完成</button>
        <button onClick={() => setFilter('completed')}>已完成</button>
      </div>
      
      <ul>
        {filteredTodos.map(todo => (
          <TodoItem
            key={todo.id}
            todo={todo}
            onToggle={toggleTodo}
            onRemove={removeTodo}
          />
        ))}
      </ul>
      
      <AddTodoForm onAdd={addTodo} />
    </div>
  );
}

// 优化的待办事项组件
const TodoItem = memo(({ todo, onToggle, onRemove }) => {
  return (
    <li>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      <span style={{
        textDecoration: todo.completed ? 'line-through' : 'none'
      }}>
        {todo.text}
      </span>
      <button onClick={() => onRemove(todo.id)}>删除</button>
    </li>
  );
});

// 添加待办事项表单
function AddTodoForm({ onAdd }) {
  const [text, setText] = useState('');
  
  const handleSubmit = useCallback((e) => {
    e.preventDefault();
    if (text.trim()) {
      onAdd(text);
      setText('');
    }
  }, [text, onAdd]);
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={text}
        onChange={(e) => setText(e.target.value)}
        placeholder="添加待办事项"
      />
      <button type="submit">添加</button>
    </form>
  );
}
```

### 异步操作

```jsx
function AsyncOperations() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  // 获取用户列表
  const fetchUsers = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const response = await fetch('/api/users');
      const data = await response.json();
      setUsers(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, []);
  
  // 创建用户
  const createUser = useCallback(async (userData) => {
    try {
      setLoading(true);
      const response = await fetch('/api/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData)
      });
      const data = await response.json();
      setUsers(prev => [...prev, data]);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, []);
  
  // 删除用户
  const deleteUser = useCallback(async (userId) => {
    try {
      setLoading(true);
      await fetch(`/api/users/${userId}`, {
        method: 'DELETE'
      });
      setUsers(prev => prev.filter(user => user.id !== userId));
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }, []);
  
  return (
    <div>
      <button onClick={fetchUsers}>刷新用户列表</button>
      
      {loading && <div>加载中...</div>}
      {error && <div>错误: {error}</div>}
      
      <ul>
        {users.map(user => (
          <li key={user.id}>
            {user.name}
            <button onClick={() => deleteUser(user.id)}>删除</button>
          </li>
        ))}
      </ul>
      
      <CreateUserForm onCreate={createUser} />
    </div>
  );
}
```

## 注意事项

### 1. 不要过度使用 useCallback

```jsx
// ❌ 过度使用：简单的函数不需要缓存
function OverOptimizedComponent() {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    setCount(count + 1);
  }, [count]);
  
  return <button onClick={handleClick}>{count}</button>;
}

// ✅ 合理使用：只有当函数被传递给子组件时才使用
function SimpleComponent() {
  const [count, setCount] = useState(0);
  
  const handleClick = () => {
    setCount(count + 1);
  };
  
  return <button onClick={handleClick}>{count}</button>;
}
```

### 2. 依赖数组的正确使用

```jsx
// ❌ 错误：依赖数组不完整
function IncompleteDeps({ multiplier }) {
  const [count, setCount] = useState(0);
  
  const multiplyAndSet = useCallback(() => {
    setCount(count * multiplier); // 缺少 multiplier 依赖
  }, [count]);
  
  return <button onClick={multiplyAndSet}>乘法</button>;
}

// ✅ 正确：包含所有依赖
function CompleteDeps({ multiplier }) {
  const [count, setCount] = useState(0);
  
  const multiplyAndSet = useCallback(() => {
    setCount(count * multiplier);
  }, [count, multiplier]);
  
  return <button onClick={multiplyAndSet}>乘法</button>;
}

// ✅ 更好：使用函数式更新减少依赖
function BetterDeps({ multiplier }) {
  const [count, setCount] = useState(0);
  
  const multiplyAndSet = useCallback(() => {
    setCount(prev => prev * multiplier);
  }, [multiplier]);
  
  return <button onClick={multiplyAndSet}>乘法</button>;
}
```

### 3. useCallback 与 React.memo 配合使用

```jsx
// ❌ 错误：只有 useCallback 没有 React.memo
function WithoutReactMemo() {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    console.log('点击');
  }, []);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>增加</button>
      <ChildComponent onClick={handleClick} />
    </div>
  );
}

// ✅ 正确：useCallback 配合 React.memo
function WithReactMemo() {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    console.log('点击');
  }, []);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>增加</button>
      <MemoizedChildComponent onClick={handleClick} />
    </div>
  );
}

const MemoizedChildComponent = memo(({ onClick }) => {
  console.log('子组件渲染');
  return <button onClick={onClick}>子组件按钮</button>;
});
```

### 4. 函数式更新的优势

```jsx
// ❌ 依赖前一个状态，需要包含在依赖数组中
function WithStateDependence() {
  const [count, setCount] = useState(0);
  
  const increment = useCallback(() => {
    setCount(count + 1); // 依赖 count
  }, [count]);
  
  return <button onClick={increment}>{count}</button>;
}

// ✅ 使用函数式更新，避免依赖
function WithFunctionalUpdate() {
  const [count, setCount] = useState(0);
  
  const increment = useCallback(() => {
    setCount(prev => prev + 1); // 不依赖 count
  }, []);
  
  return <button onClick={increment}>{count}</button>;
}
```

### 5. 性能测量

```jsx
function PerformanceMeasurement() {
  const [count, setCount] = useState(0);
  const [data] = useState(largeDataSet);
  
  // 不使用 useCallback
  const handleClick1 = () => {
    console.log('处理数据');
    processData(data);
  };
  
  // 使用 useCallback
  const handleClick2 = useCallback(() => {
    console.log('处理数据');
    processData(data);
  }, [data]);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>增加: {count}</button>
      <ChildComponent onClick1={handleClick1} onClick2={handleClick2} />
    </div>
  );
}

// 使用 React DevTools Profiler 测量性能
// 观察不同实现下的重渲染情况
```

## 总结

useCallback 是 React 性能优化的重要工具，但需要正确使用：

### 核心要点

1. **函数缓存**：useCallback 缓存函数引用，只在依赖变化时更新
2. **性能优化**：主要配合 React.memo 使用，避免子组件不必要的重渲染
3. **依赖管理**：正确声明依赖数组，避免闭包陷阱
4. **理性使用**：不是所有函数都需要 useCallback，避免过度优化

### 使用场景

1. **传递给子组件的回调**：特别是配合 React.memo 使用时
2. **useEffect 的依赖**：当函数作为 useEffect 依赖时
3. **事件处理器**：复杂组件中的多个事件处理器
4. **异步操作**：API 调用、数据操作等异步函数

### 最佳实践

1. **按需使用**：只在真正需要性能优化时使用 useCallback
2. **合理依赖**：确保依赖数组包含所有外部依赖
3. **函数式更新**：优先使用函数式更新减少依赖
4. **配合 React.memo**：与 React.memo 配合使用效果最佳
5. **性能测量**：使用 DevTools Profiler 测量实际性能提升

useCallback 是一个强大的性能优化工具，但要记住"过早优化是万恶之源"。在确保代码正确性的基础上，针对性能瓶颈进行有针对性的优化。理解其工作原理和适用场景，才能发挥其真正的价值。