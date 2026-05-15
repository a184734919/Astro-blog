---
title: React 事件处理
published: 2024-12-15
description: 'React 事件处理机制的详细介绍和学习笔记'
image: ''
tags: ["React","事件"]
category: 'React'
draft: false
lang: 'zh-CN'
---

## 概述

React 事件处理是构建交互式应用的核心机制。与原生 DOM 事件相比，React 事件系统使用合成事件（Synthetic Event），提供了跨浏览器的一致性和更好的性能。本文将深入探讨 React 事件处理的各个方面，从基础概念到高级应用。

## 核心概念

### 合成事件

React 使用合成事件系统来处理事件，它的特点包括：

- **跨浏览器一致性**：React 将不同浏览器的事件差异标准化
- **事件委托**：事件在根节点统一处理，提高性能
- **自动绑定**：事件处理函数默认绑定组件实例
- **性能优化**：使用事件池减少内存分配

### 事件处理机制

React 事件处理遵循以下流程：

1. **事件触发**：用户交互触发原生事件
2. **事件包装**：React 包装原生事件为合成事件
3. **事件委托**：事件冒泡到 React 根节点
4. **处理函数调用**：调用对应的事件处理函数
5. **事件对象重用**：事件对象被回收到事件池

## 基本用法

### 1. 事件绑定

#### 函数组件

```jsx
import { useState } from 'react';

function EventExample() {
  const [count, setCount] = useState(0);

  // 使用箭头函数
  const handleClick = () => {
    setCount(count + 1);
  };

  return (
    <button onClick={handleClick}>
      点击次数: {count}
    </button>
  );
}
```

#### 类组件

```jsx
import { Component } from 'react';

class EventExample extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    
    // 绑定 this
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        点击次数: {this.state.count}
      </button>
    );
  }
}
```

### 2. 事件处理器传参

```jsx
function EventExample() {
  const [users, setUsers] = useState([
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' },
    { id: 3, name: 'Charlie' }
  ]);

  // 使用箭头函数传参
  const handleUserClick = (userId) => {
    console.log('用户 ID:', userId);
  };

  return (
    <div>
      {users.map(user => (
        <button 
          key={user.id}
          onClick={() => handleUserClick(user.id)}
        >
          {user.name}
        </button>
      ))}
    </div>
  );
}
```

### 3. 事件对象访问

```jsx
function EventExample() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseMove = (event) => {
    setPosition({
      x: event.clientX,
      y: event.clientY
    });
  };

  return (
    <div 
      onMouseMove={handleMouseMove}
      style={{ height: '200px', border: '1px solid #ccc' }}
    >
      <p>鼠标位置: x={position.x}, y={position.y}</p>
    </div>
  );
}
```

### 4. 常见事件类型

```jsx
function CommonEvents() {
  return (
    <div>
      {/* 鼠标事件 */}
      <button
        onClick={() => console.log('点击')}
        onMouseEnter={() => console.log('鼠标进入')}
        onMouseLeave={() => console.log('鼠标离开')}
        onMouseDown={() => console.log('鼠标按下')}
        onMouseUp={() => console.log('鼠标松开')}
      >
        鼠标事件
      </button>

      {/* 键盘事件 */}
      <input
        type="text"
        onKeyDown={(e) => console.log('按下:', e.key)}
        onKeyUp={(e) => console.log('松开:', e.key)}
        onKeyPress={(e) => console.log('按键:', e.key)}
      />

      {/* 表单事件 */}
      <input
        type="text"
        onChange={(e) => console.log('值改变:', e.target.value)}
        onFocus={() => console.log('获得焦点')}
        onBlur={() => console.log('失去焦点')}
        onSubmit={() => console.log('表单提交')}
      />

      {/* 其他事件 */}
      <img
        src="image.jpg"
        onLoad={() => console.log('图片加载完成')}
        onError={() => console.log('图片加载失败')}
      />
    </div>
  );
}
```

## 表单处理

### 1. 受控组件

```jsx
function ControlledForm() {
  const [formData, setFormData] = useState({
    username: '',
    email: '',
    password: '',
    age: 18,
    subscribed: false
  });

  const handleChange = (event) => {
    const { name, value, type, checked } = event.target;
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  const handleSubmit = (event) => {
    event.preventDefault();
    console.log('表单数据:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        name="username"
        value={formData.username}
        onChange={handleChange}
        placeholder="用户名"
      />

      <input
        type="email"
        name="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="邮箱"
      />

      <input
        type="password"
        name="password"
        value={formData.password}
        onChange={handleChange}
        placeholder="密码"
      />

      <input
        type="number"
        name="age"
        value={formData.age}
        onChange={handleChange}
      />

      <label>
        <input
          type="checkbox"
          name="subscribed"
          checked={formData.subscribed}
          onChange={handleChange}
        />
        订阅邮件
      </label>

      <button type="submit">提交</button>
    </form>
  );
}
```

### 2. 非受控组件

```jsx
import { useRef } from 'react';

function UncontrolledForm() {
  const usernameRef = useRef(null);
  const emailRef = useRef(null);

  const handleSubmit = (event) => {
    event.preventDefault();
    const formData = {
      username: usernameRef.current.value,
      email: emailRef.current.value
    };
    console.log('表单数据:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        ref={usernameRef}
        type="text"
        placeholder="用户名"
        defaultValue=""
      />

      <input
        ref={emailRef}
        type="email"
        placeholder="邮箱"
        defaultValue=""
      />

      <button type="submit">提交</button>
    </form>
  );
}
```

## 高级应用

### 1. 事件委托

```jsx
function EventDelegation() {
  const [logs, setLogs] = useState([]);

  const handleClick = (event) => {
    const { target } = event;
    
    if (target.matches('.action-button')) {
      const action = target.dataset.action;
      setLogs(prev => [...prev, `执行操作: ${action}`]);
    }
  };

  return (
    <div onClick={handleClick}>
      <button className="action-button" data-action="save">
        保存
      </button>
      <button className="action-button" data-action="delete">
        删除
      </button>
      <button className="action-button" data-action="edit">
        编辑
      </button>

      <div>
        {logs.map((log, index) => (
          <p key={index}>{log}</p>
        ))}
      </div>
    </div>
  );
}
```

### 2. 防抖和节流

```jsx
import { useState, useCallback } from 'react';

// 防抖函数
function debounce(func, wait) {
  let timeout;
  return function executedFunction(...args) {
    const later = () => {
      clearTimeout(timeout);
      func(...args);
    };
    clearTimeout(timeout);
    timeout = setTimeout(later, wait);
  };
}

// 节流函数
function throttle(func, limit) {
  let inThrottle;
  return function executedFunction(...args) {
    if (!inThrottle) {
      func(...args);
      inThrottle = true;
      setTimeout(() => inThrottle = false, limit);
    }
  };
}

function DebounceThrottleExample() {
  const [inputValue, setInputValue] = useState('');
  const [searchResults, setSearchResults] = useState([]);

  // 防抖搜索
  const handleSearch = useCallback(
    debounce((value) => {
      console.log('搜索:', value);
      // 模拟搜索 API 调用
      setSearchResults([`结果1: ${value}`, `结果2: ${value}`]);
    }, 500),
    []
  );

  // 节流滚动
  const handleScroll = useCallback(
    throttle((event) => {
      console.log('滚动位置:', event.target.scrollTop);
    }, 200),
    []
  );

  const handleChange = (event) => {
    const value = event.target.value;
    setInputValue(value);
    handleSearch(value);
  };

  return (
    <div>
      <input
        type="text"
        value={inputValue}
        onChange={handleChange}
        placeholder="搜索（防抖）"
      />

      <div 
        style={{ height: '200px', overflow: 'auto', border: '1px solid #ccc' }}
        onScroll={handleScroll}
      >
        <div style={{ height: '500px' }}>
          滚动此区域查看节流效果
        </div>
      </div>

      <div>
        <h4>搜索结果:</h4>
        {searchResults.map((result, index) => (
          <p key={index}>{result}</p>
        ))}
      </div>
    </div>
  );
}
```

### 3. 自定义 Hook 事件处理

```jsx
import { useEffect, useRef, useState } from 'react';

// 监听窗口大小变化的 Hook
function useWindowSize() {
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

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}

// 监听键盘事件的 Hook
function useKeyPress(targetKey) {
  const [keyPressed, setKeyPressed] = useState(false);

  useEffect(() => {
    const handleKeyDown = ({ key }) => {
      if (key === targetKey) {
        setKeyPressed(true);
      }
    };

    const handleKeyUp = ({ key }) => {
      if (key === targetKey) {
        setKeyPressed(false);
      }
    };

    window.addEventListener('keydown', handleKeyDown);
    window.addEventListener('keyup', handleKeyUp);

    return () => {
      window.removeEventListener('keydown', handleKeyDown);
      window.removeEventListener('keyup', handleKeyUp);
    };
  }, [targetKey]);

  return keyPressed;
}

// 监听点击外部事件的 Hook
function useClickOutside(ref, handler) {
  useEffect(() => {
    const handleClickOutside = (event) => {
      if (ref.current && !ref.current.contains(event.target)) {
        handler();
      }
    };

    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, [ref, handler]);
}

// 使用示例
function CustomHooksExample() {
  const windowSize = useWindowSize();
  const enterPressed = useKeyPress('Enter');
  const dropdownRef = useRef(null);
  const [isOpen, setIsOpen] = useState(false);

  useClickOutside(dropdownRef, () => setIsOpen(false));

  return (
    <div>
      <p>窗口大小: {windowSize.width} x {windowSize.height}</p>
      <p>Enter 键状态: {enterPressed ? '按下' : '未按下'}</p>

      <div ref={dropdownRef}>
        <button onClick={() => setIsOpen(!isOpen)}>
          {isOpen ? '关闭下拉菜单' : '打开下拉菜单'}
        </button>
        {isOpen && (
          <div style={{ border: '1px solid #ccc', padding: '10px' }}>
            <p>点击外部关闭菜单</p>
          </div>
        )}
      </div>
    </div>
  );
}
```

### 4. 事件池优化

```jsx
function EventPoolExample() {
  const [logs, setLogs] = useState([]);

  // 批量更新状态
  const handleBatchClick = () => {
    React.startTransition(() => {
      setLogs(prev => [...prev, '日志1']);
      setLogs(prev => [...prev, '日志2']);
      setLogs(prev => [...prev, '日志3']);
    });
  };

  // 事件对象重用
  const handleEventReuse = (event) => {
    // 在 React 17+ 中，事件对象不再被复用
    console.log('事件类型:', event.type);
    console.log('事件目标:', event.target);
    
    // 可以安全地异步访问事件属性
    setTimeout(() => {
      console.log('异步访问:', event.type);
    }, 100);
  };

  return (
    <div>
      <button onClick={handleBatchClick}>
        批量更新
      </button>

      <button onClick={handleEventReuse}>
        事件对象重用
      </button>

      <div>
        {logs.map((log, index) => (
          <p key={index}>{log}</p>
        ))}
      </div>
    </div>
  );
}
```

## 注意事项

### 1. 性能优化

```jsx
// ❌ 不好的做法：每次渲染都创建新函数
function BadExample() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      点击
    </button>
  );
}

// ✅ 好的做法：使用 useCallback
import { useCallback } from 'react';

function GoodExample() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    setCount(prev => prev + 1);
  }, []);

  return (
    <button onClick={handleClick}>
      点击
    </button>
  );
}
```

### 2. 事件清理

```jsx
function EventCleanup() {
  const [isActive, setIsActive] = useState(false);

  useEffect(() => {
    const handleKeyDown = (event) => {
      if (event.key === 'Escape') {
        setIsActive(false);
      }
    };

    if (isActive) {
      window.addEventListener('keydown', handleKeyDown);
    }

    return () => {
      window.removeEventListener('keydown', handleKeyDown);
    };
  }, [isActive]);

  return (
    <button onClick={() => setIsActive(true)}>
      {isActive ? '按 ESC 关闭' : '打开'}
    </button>
  );
}
```

### 3. 阻止默认行为和事件冒泡

```jsx
function EventPropagation() {
  const [logs, setLogs] = useState([]);

  const handleParentClick = (event) => {
    event.stopPropagation(); // 阻止事件冒泡
    setLogs(prev => [...prev, '父元素点击']);
  };

  const handleChildClick = (event) => {
    event.stopPropagation(); // 阻止事件冒泡到父元素
    setLogs(prev => [...prev, '子元素点击']);
  };

  const handleFormSubmit = (event) => {
    event.preventDefault(); // 阻止表单默认提交
    setLogs(prev => [...prev, '表单提交']);
  };

  return (
    <div>
      <div onClick={handleParentClick} style={{ padding: '20px', border: '2px solid blue' }}>
        <div onClick={handleChildClick} style={{ padding: '20px', border: '2px solid red' }}>
          点击子元素
        </div>
      </div>

      <form onSubmit={handleFormSubmit}>
        <input type="text" />
        <button type="submit">提交</button>
      </form>

      <div>
        {logs.map((log, index) => (
          <p key={index}>{log}</p>
        ))}
      </div>
    </div>
  );
}
```

### 4. 无障碍性（Accessibility）

```jsx
function AccessibleButton() {
  const handleClick = () => {
    console.log('按钮被点击');
  };

  return (
    // ✅ 好的做法：使用键盘可访问
    <button onClick={handleClick}>
      点击我
    </button>

    // ✅ 对于非按钮元素，添加键盘支持
    <div
      role="button"
      tabIndex={0}
      onClick={handleClick}
      onKeyDown={(event) => {
        if (event.key === 'Enter' || event.key === ' ') {
          event.preventDefault();
          handleClick();
        }
      }}
      style={{ cursor: 'pointer' }}
    >
      点击我
    </div>
  );
}
```

## 最佳实践

### 1. 命名规范

```jsx
// 使用描述性的函数名
const handleUserClick = () => { };
const handleSubmit = () => { };
const handleInputChange = () => { };
const handleMouseMove = () => { };
```

### 2. 类型安全（TypeScript）

```typescript
import { useState, ChangeEvent, FormEvent } from 'react';

interface FormData {
  username: string;
  email: string;
}

function TypedForm() {
  const [formData, setFormData] = useState<FormData>({
    username: '',
    email: ''
  });

  const handleChange = (event: ChangeEvent<HTMLInputElement>) => {
    const { name, value } = event.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
  };

  const handleSubmit = (event: FormEvent<HTMLFormElement>) => {
    event.preventDefault();
    console.log('表单数据:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        name="username"
        value={formData.username}
        onChange={handleChange}
      />
      <button type="submit">提交</button>
    </form>
  );
}
```

### 3. 错误处理

```jsx
function ErrorHandlingExample() {
  const [error, setError] = useState(null);

  const handleClick = async () => {
    try {
      // 可能抛出错误的操作
      const result = await fetchData();
      console.log('成功:', result);
    } catch (err) {
      console.error('错误:', err);
      setError(err.message);
    }
  };

  const handleErrorClick = (error) => {
    console.error('发生错误:', error);
    // 可以显示错误消息或上报错误
  };

  return (
    <div>
      <button onClick={handleClick}>
        点击执行操作
      </button>

      {error && (
        <div style={{ color: 'red' }}>
          错误: {error}
        </div>
      )}
    </div>
  );
}
```

## 总结

React 事件处理是构建交互式应用的核心技能。通过本文的学习，你已经掌握了：

**核心概念**:
- 合成事件系统的工作原理
- 事件委托机制
- 事件对象的重用和性能优化

**基本用法**:
- 函数组件和类组件的事件绑定
- 事件处理器传参
- 常见事件类型的使用
- 受控和非受控组件

**高级应用**:
- 事件委托的实现
- 防抖和节流的使用
- 自定义 Hook 封装事件逻辑
- 事件池优化

**最佳实践**:
- 使用 useCallback 优化性能
- 正确的事件清理
- 阻止默认行为和事件冒泡
- 无障碍性考虑
- TypeScript 类型安全

**注意事项**:
- 避免在事件处理器中直接修改状态
- 合理使用防抖和节流
- 确保事件监听器的正确清理
- 考虑键盘访问和无障碍性

通过深入理解 React 事件处理机制，你将能够构建出更加高效、可维护的用户界面。持续实践和探索，你将掌握 React 事件处理的各种技巧和最佳实践。