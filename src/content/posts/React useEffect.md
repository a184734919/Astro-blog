---
title: React useEffect
published: 2023-08-02
description: '深入掌握 React useEffect Hook：副作用处理的完整指南'
image: ''
tags: ["React","Hooks"]
category: 'React'
draft: false
lang: 'zh-CN'
---

## 概述

useEffect 是 React 中处理副作用的官方 Hook。它让你能够在函数组件中执行副作用操作，如数据获取、订阅、手动修改 DOM、设置定时器等。useEffect 结合了 React class 组件中 componentDidMount、componentDidUpdate 和 componentWillUnmount 的功能，是 React Hooks 中最复杂但也最重要的 Hook 之一。

### 什么是副作用

在 React 中，"副作用"（Side Effect）指的是任何在渲染之外发生的事情，包括：

- **数据获取**：从 API 获取数据
- **订阅**：WebSocket 连接、事件监听器
- **DOM 操作**：手动修改 DOM 元素
- **定时器**：setTimeout、setInterval
- **本地存储**：localStorage、sessionStorage 操作
- **路由操作**：程序化导航

### useEffect 的优势

1. **统一的生命周期**：用单一 API 替代多个生命周期方法
2. **关注点分离**：将相关的副作用逻辑组织在一起
3. **自动清理**：内置清理机制，防止内存泄漏
4. **依赖声明**：明确声明副作用依赖的值，提高代码可维护性

## 核心概念

### useEffect 的签名

```jsx
useEffect(effectFunction, [dependencies]);
```

- **effectFunction**：包含副作用逻辑的函数
- **dependencies**：依赖数组（可选），控制副作用何时执行

### 执行时机

useEffect 的执行时机与其他代码不同：

1. **默认行为**：在浏览器完成渲染之后执行（不阻塞渲染）
2. **同步执行**：使用 useLayoutEffect 在浏览器渲染之前执行
3. **首次渲染**：组件首次挂载时执行一次
4. **依赖变化**：依赖数组中的值发生变化时执行

### 清理机制

useEffect 可以返回一个清理函数，在组件卸载或下次副作用执行前调用：

```jsx
useEffect(() => {
  // 设置副作用
  const subscription = subscribe();
  
  // 返回清理函数
  return () => {
    subscription.unsubscribe();
  };
}, [dependencies]);
```

## 基本用法

### 无依赖的副作用

在每次渲染后都执行副作用：

```jsx
function Logger() {
  useEffect(() => {
    console.log('组件渲染完成');
    // 这个副作用会在每次渲染后执行
  });
  
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>增加</button>
    </div>
  );
}
```

### 仅在挂载时执行

使用空依赖数组，副作用只在组件首次挂载时执行一次：

```jsx
function DataFetcher() {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    console.log('组件挂载，开始获取数据');
    fetchData('/api/data').then(setData);
  }, []); // 空依赖数组
  
  if (!data) return <div>加载中...</div>;
  return <div>{data}</div>;
}
```

### 基于依赖的副作用

只在依赖项变化时执行副作用：

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    console.log(`用户 ID 变化: ${userId}`);
    fetchUser(userId).then(setUser);
  }, [userId]); // 只有 userId 变化时才执行
  
  return user ? <div>{user.name}</div> : <div>加载中...</div>;
}
```

### 带清理的副作用

```jsx
function WindowSizeTracker() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    // 添加事件监听器
    window.addEventListener('resize', handleResize);
    console.log('添加 resize 监听器');
    
    // 返回清理函数
    return () => {
      window.removeEventListener('resize', handleResize);
      console.log('移除 resize 监听器');
    };
  }, []); // 只在挂载时设置，卸载时清理
  
  return (
    <div>
      窗口大小: {size.width} x {size.height}
    </div>
  );
}
```

## 实际应用

### 数据获取

```jsx
function UserList() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const fetchUsers = async () => {
      try {
        setLoading(true);
        setError(null);
        const response = await fetch('/api/users');
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data = await response.json();
        setUsers(data);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };
    
    fetchUsers();
  }, []); // 只在挂载时获取一次
  
  if (loading) return <div>加载中...</div>;
  if (error) return <div>错误: {error}</div>;
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### 可取消的数据获取

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    let cancelled = false;
    
    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        
        // 检查是否已取消
        if (!cancelled) {
          setUser(data);
        }
      } catch (err) {
        if (!cancelled) {
          console.error('获取用户失败:', err);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    };
    
    fetchUser();
    
    // 清理函数
    return () => {
      cancelled = true;
    };
  }, [userId]);
  
  if (loading) return <div>加载中...</div>;
  return user ? <div>{user.name}</div> : <div>用户不存在</div>;
}
```

### WebSocket 连接

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  const [inputValue, setInputValue] = useState('');
  
  useEffect(() => {
    const ws = new WebSocket(`wss://chat.example.com/rooms/${roomId}`);
    
    ws.onopen = () => {
      console.log('WebSocket 连接已建立');
    };
    
    ws.onmessage = (event) => {
      const message = JSON.parse(event.data);
      setMessages(prev => [...prev, message]);
    };
    
    ws.onerror = (error) => {
      console.error('WebSocket 错误:', error);
    };
    
    ws.onclose = () => {
      console.log('WebSocket 连接已关闭');
    };
    
    // 清理函数
    return () => {
      ws.close();
    };
  }, [roomId]);
  
  const sendMessage = () => {
    const ws = new WebSocket(`wss://chat.example.com/rooms/${roomId}`);
    ws.send(JSON.stringify({ text: inputValue }));
    setInputValue('');
  };
  
  return (
    <div>
      <div>
        {messages.map((msg, index) => (
          <div key={index}>{msg.text}</div>
        ))}
      </div>
      <input
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        placeholder="输入消息"
      />
      <button onClick={sendMessage}>发送</button>
    </div>
  );
}
```

### 定时器

```jsx
function Countdown({ seconds }) {
  const [timeLeft, setTimeLeft] = useState(seconds);
  
  useEffect(() => {
    const timer = setInterval(() => {
      setTimeLeft(prev => {
        if (prev <= 1) {
          clearInterval(timer);
          return 0;
        }
        return prev - 1;
      });
    }, 1000);
    
    // 清理函数
    return () => clearInterval(timer);
  }, [seconds]);
  
  return <div>剩余时间: {timeLeft} 秒</div>;
}
```

### 本地存储同步

```jsx
function LocalStorageInput({ key, defaultValue }) {
  const [value, setValue] = useState(() => {
    // 从本地存储读取初始值
    const saved = localStorage.getItem(key);
    return saved !== null ? saved : defaultValue;
  });
  
  // 当 value 变化时，同步到本地存储
  useEffect(() => {
    localStorage.setItem(key, value);
  }, [key, value]);
  
  return (
    <input
      value={value}
      onChange={(e) => setValue(e.target.value)}
      placeholder={defaultValue}
    />
  );
}

// 使用
function App() {
  return (
    <div>
      <LocalStorageInput key="username" defaultValue="用户名" />
      <LocalStorageInput key="email" defaultValue="邮箱" />
    </div>
  );
}
```

### 文档标题更新

```jsx
function useDocumentTitle(title) {
  useEffect(() => {
    const previousTitle = document.title;
    document.title = title;
    
    return () => {
      document.title = previousTitle;
    };
  }, [title]);
}

function Page({ title }) {
  useDocumentTitle(title);
  
  return <div>{title}</div>;
}
```

### 事件监听器管理

```jsx
function KeyboardShortcuts() {
  useEffect(() => {
    const handleKeyDown = (event) => {
      if (event.ctrlKey && event.key === 's') {
        event.preventDefault();
        console.log('保存文件');
        // 触发保存逻辑
      }
    };
    
    window.addEventListener('keydown', handleKeyDown);
    
    return () => {
      window.removeEventListener('keydown', handleKeyDown);
    };
  }, []);
  
  return <div>按 Ctrl+S 保存</div>;
}
```

## 注意事项

### 1. 依赖数组的正确使用

```jsx
// ❌ 错误：缺少依赖项
function BadComponent({ userId }) {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetchData(userId).then(setData);
  }, []); // 缺少 userId 依赖
  
  return <div>{data}</div>;
}

// ✅ 正确：包含所有依赖项
function GoodComponent({ userId }) {
  const [data, setData] = useState(null);
  
  useEffect(() => {
    fetchData(userId).then(setData);
  }, [userId]); // 包含 userId 依赖
  
  return <div>{data}</div>;
}

// ✅ 使用 ESLint 插件自动检测
// 安装：npm install eslint-plugin-react-hooks
```

### 2. 避免无限循环

```jsx
// ❌ 错误：会导致无限循环
function InfiniteLoop() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    setCount(count + 1);
  }, [count]); // count 变化触发 useEffect，useEffect 又更新 count
  
  return <div>{count}</div>;
}

// ✅ 正确：根据条件更新状态
function ConditionalUpdate() {
  const [count, setCount] = useState(0);
  const [shouldUpdate, setShouldUpdate] = useState(true);
  
  useEffect(() => {
    if (shouldUpdate && count < 10) {
      setCount(count + 1);
      setShouldUpdate(false);
    }
  }, [count, shouldUpdate]);
  
  return <div>{count}</div>;
}
```

### 3. 闭包陷阱

```jsx
// ❌ 闭包陷阱
function Counter() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count); // 总是打印 0
      setCount(count + 1);
    }, 1000);
    
    return () => clearInterval(timer);
  }, []); // 依赖数组为空
  
  return <div>{count}</div>;
}

// ✅ 解决方案1：使用函数式更新
function CounterFixed() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      setCount(prev => prev + 1); // 使用函数式更新
    }, 1000);
    
    return () => clearInterval(timer);
  }, []);
  
  return <div>{count}</div>;
}

// ✅ 解决方案2：添加正确的依赖
function CounterFixed2() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count);
      setCount(count + 1);
    }, 1000);
    
    return () => clearInterval(timer);
  }, [count]); // 依赖 count
  
  return <div>{count}</div>;
}
```

### 4. useEffect vs useLayoutEffect

```jsx
// useEffect：在浏览器绘制之后执行（不阻塞绘制）
function UseEffectExample() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    // 不会阻塞绘制，用户会看到中间状态
    console.log('useEffect 执行');
  });
  
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}

// useLayoutEffect：在浏览器绘制之前执行（阻塞绘制）
import { useLayoutEffect } from 'react';

function UseLayoutEffectExample() {
  const [count, setCount] = useState(0);
  
  useLayoutEffect(() => {
    // 阻塞绘制，用户不会看到中间状态
    console.log('useLayoutEffect 执行');
  });
  
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}

// 选择建议：
// - 大多数情况使用 useEffect
// - 需要同步读取/修改 DOM 时使用 useLayoutEffect
// - 避免在 useLayoutEffect 中执行耗时操作
```

### 5. 过度使用 useEffect

```jsx
// ❌ 过度使用 useEffect
function OverUsedEffect({ initialCount }) {
  const [count, setCount] = useState(0);
  const [doubled, setDoubled] = useState(0);
  
  useEffect(() => {
    setDoubled(count * 2);
  }, [count]); // 不必要的副作用
  
  useEffect(() => {
    setCount(initialCount);
  }, [initialCount]); // 可以直接使用 useState
  
  return <div>{doubled}</div>;
}

// ✅ 优化后的代码
function OptimizedCode({ initialCount }) {
  const [count, setCount] = useState(initialCount);
  const doubled = count * 2; // 直接计算，不需要 useEffect
  
  return <div>{doubled}</div>;
}
```

## 总结

useEffect 是 React Hooks 中最强大但也最容易出错的 Hook：

### 核心要点

1. **统一生命周期**：替代 componentDidMount、componentDidUpdate 和 componentWillUnmount
2. **依赖管理**：通过依赖数组控制副作用执行时机
3. **自动清理**：返回清理函数，防止内存泄漏
4. **异步执行**：默认在浏览器绘制之后执行，不阻塞渲染

### 最佳实践

1. **正确声明依赖**：依赖数组应包含所有副作用中使用的外部值
2. **合理清理**：为所有需要清理的副作用提供清理函数
3. **避免闭包陷阱**：理解闭包的影响，使用函数式更新或正确声明依赖
4. **性能考虑**：避免不必要的副作用执行，合理使用依赖数组
5. **工具辅助**：使用 ESLint 插件自动检测依赖问题

### 常见模式

1. **数据获取**：在 useEffect 中进行 API 调用
2. **事件监听**：在 useEffect 中添加/移除事件监听器
3. **定时器管理**：使用清理函数管理定时器
4. **本地存储同步**：保持状态与本地存储同步
5. **WebSocket 连接**：管理 WebSocket 连接的生命周期

useEffect 的掌握程度直接影响你的 React 应用质量和用户体验。通过不断的实践和总结，你会更好地理解副作用管理的精髓，写出更健壮的 React 组件。