---
title: React 性能优化
published: 2023-10-09
description: '性能优化技巧的详细介绍和学习笔记'
image: ''
tags: ["React","性能"]
category: 'React'
draft: false
lang: 'zh-CN'
---

## 概述

React 性能优化是提升应用用户体验的关键技术。随着应用规模的增长，不当的状态管理和渲染方式会导致性能问题，表现为页面卡顿、响应延迟等。本文将深入探讨 React 性能优化的核心策略和实践技巧，帮助开发者构建高效、流畅的 React 应用。

## 核心概念

### 虚拟 DOM 和渲染机制

React 使用虚拟 DOM (Virtual DOM) 来最小化真实 DOM 操作，通过 diff 算法高效更新界面。但即使是虚拟 DOM 操作也有性能开销，合理的组件设计和状态管理可以大幅减少不必要的渲染。

### 组件渲染触发

React 组件在以下情况下会重新渲染：
- 父组件重新渲染
- 组件自身的 state 发生变化
- 组件使用的 props 发生变化
- context value 发生变化

### 性能瓶颈识别

常见的性能问题包括：
- 过多的组件重新渲染
- 复杂计算在每次渲染时重复执行
- 大列表渲染导致 DOM 节点过多
- 不必要的 API 调用和数据获取

## 基本用法

### React.memo - 组件记忆化

`React.memo` 可以避免不必要的组件重新渲染：

```jsx
import React, { memo } from 'react';

const ExpensiveComponent = memo(function ExpensiveComponent({ data, onClick }) {
  console.log('ExpensiveComponent rendered');
  return (
    <div>
      {data.map(item => (
        <div key={item.id} onClick={() => onClick(item.id)}>
          {item.name}
        </div>
      ))}
    </div>
  );
});

// 使用示例
function ParentComponent() {
  const [data, setData] = useState([]);
  const [count, setCount] = useState(0);

  const handleClick = (id) => {
    console.log('Item clicked:', id);
  };

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>
      <ExpensiveComponent data={data} onClick={handleClick} />
    </div>
  );
}
```

### useMemo - 缓存计算结果

`useMemo` 用于缓存昂贵的计算结果：

```jsx
import React, { useState, useMemo } from 'react';

function DataProcessor({ items, filterFn }) {
  // 不使用 useMemo - 每次渲染都重新计算
  // const filteredItems = items.filter(filterFn);

  // 使用 useMemo - 只在依赖项变化时重新计算
  const filteredItems = useMemo(() => {
    console.log('Filtering items...');
    return items.filter(filterFn);
  }, [items, filterFn]);

  // 复杂计算示例
  const statistics = useMemo(() => {
    if (filteredItems.length === 0) return null;

    const total = filteredItems.reduce((sum, item) => sum + item.value, 0);
    const average = total / filteredItems.length;
    const max = Math.max(...filteredItems.map(item => item.value));
    const min = Math.min(...filteredItems.map(item => item.value));

    return { total, average, max, min, count: filteredItems.length };
  }, [filteredItems]);

  return (
    <div>
      <h3>Statistics</h3>
      {statistics && (
        <ul>
          <li>Total: {statistics.total}</li>
          <li>Average: {statistics.average.toFixed(2)}</li>
          <li>Max: {statistics.max}</li>
          <li>Min: {statistics.min}</li>
          <li>Count: {statistics.count}</li>
        </ul>
      )}
    </div>
  );
}
```

### useCallback - 缓存函数引用

`useMemo` 用于缓存函数引用，避免子组件因 props 变化而重新渲染：

```jsx
import React, { useState, useCallback } from 'react';

function ParentComponent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([]);

  // 不使用 useCallback - 每次渲染创建新函数
  // const handleClick = (id) => {
  //   console.log('Clicked:', id);
  // };

  // 使用 useCallback - 函数引用保持稳定
  const handleClick = useCallback((id) => {
    console.log('Clicked:', id);
    // 可以访问最新的 state 和 props
    setItems(prevItems =>
      prevItems.filter(item => item.id !== id)
    );
  }, []); // 空依赖数组表示函数永不改变

  const handleAddItem = useCallback(() => {
    setItems(prevItems => [
      ...prevItems,
      { id: Date.now(), value: Math.random() }
    ]);
  }, []);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>
      <button onClick={handleAddItem}>
        Add Item
      </button>
      <ChildComponent items={items} onItemClick={handleClick} />
    </div>
  );
}

const ChildComponent = React.memo(function ChildComponent({ items, onItemClick }) {
  console.log('ChildComponent rendered');
  return (
    <ul>
      {items.map(item => (
        <li key={item.id} onClick={() => onItemClick(item.id)}>
          Item {item.id}
        </li>
      ))}
    </ul>
  );
});
```

### useTransition - 标记非紧急更新

`useTransition` 用于区分紧急和非紧急更新，提升用户体验：

```jsx
import React, { useState, useTransition, Suspense } from 'react';

function SearchComponent() {
  const [isPending, startTransition] = useTransition();
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  const handleChange = (e) => {
    const value = e.target.value;

    // 紧急更新：立即更新输入框
    setQuery(value);

    // 非紧急更新：搜索结果可以延迟
    startTransition(() => {
      performSearch(value).then(data => setResults(data));
    });
  };

  return (
    <div>
      <input
        type="text"
        value={query}
        onChange={handleChange}
        placeholder="Search..."
        style={{ opacity: isPending ? 0.7 : 1 }}
      />
      {isPending && <div className="loading">Searching...</div>}
      <Suspense fallback={<div>Loading results...</div>}>
        <SearchResults results={results} />
      </Suspense>
    </div>
  );
}

// 模拟搜索函数
async function performSearch(query) {
  // 模拟网络延迟
  await new Promise(resolve => setTimeout(resolve, 500));

  // 返回模拟结果
  if (!query) return [];
  return Array.from({ length: 10 }, (_, i) => ({
    id: i,
    text: `${query} result ${i + 1}`
  }));
}
```

### useDeferredValue - 延迟更新值

`useDeferredValue` 用于延迟更新某些值，避免频繁渲染：

```jsx
import React, { useState, useDeferredValue } from 'react';

function AutoComplete() {
  const [input, setInput] = useState('');
  const deferredInput = useDeferredValue(input);

  return (
    <div>
      <input
        type="text"
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="Type to search..."
      />
      <SearchSuggestions query={deferredInput} />
    </div>
  );
}

function SearchSuggestions({ query }) {
  const suggestions = useMemo(() => {
    if (!query) return [];
    // 模拟耗用的搜索计算
    return generateSuggestions(query);
  }, [query]);

  return (
    <ul>
      {suggestions.map((suggestion, index) => (
        <li key={index}>{suggestion}</li>
      ))}
    </ul>
  );
}

function generateSuggestions(query) {
  const words = ['apple', 'application', 'apply', 'apricot', 'banana', 'berry'];
  return words.filter(word => word.toLowerCase().startsWith(query.toLowerCase()));
}
```

## 实际应用

### 列表虚拟化 - 处理大数据列表

对于包含大量数据的长列表，使用虚拟化技术只渲染可见区域的项：

```jsx
import React, { useState, useRef, useEffect } from 'react';

function VirtualList({ items, itemHeight = 50, containerHeight = 400 }) {
  const [scrollTop, setScrollTop] = useState(0);
  const containerRef = useRef(null);

  const visibleCount = Math.ceil(containerHeight / itemHeight);
  const startIndex = Math.max(0, Math.floor(scrollTop / itemHeight) - 5);
  const endIndex = Math.min(items.length, startIndex + visibleCount + 10);
  const visibleItems = items.slice(startIndex, endIndex);

  const offsetY = startIndex * itemHeight;

  const handleScroll = (e) => {
    setScrollTop(e.target.scrollTop);
  };

  return (
    <div
      ref={containerRef}
      onScroll={handleScroll}
      style={{
        height: containerHeight,
        overflow: 'auto',
        border: '1px solid #ccc'
      }}
    >
      <div
        style={{
          height: items.length * itemHeight,
          position: 'relative'
        }}
      >
        {visibleItems.map((item, index) => (
          <div
            key={startIndex + index}
            style={{
              position: 'absolute',
              top: (startIndex + index) * itemHeight,
              left: 0,
              right: 0,
              height: itemHeight,
              display: 'flex',
              alignItems: 'center',
              padding: '0 16px',
              borderBottom: '1px solid #eee'
            }}
          >
            {item.text}
          </div>
        ))}
      </div>
    </div>
  );
}

// 使用示例
function App() {
  const [items] = useState(() =>
    Array.from({ length: 10000 }, (_, i) => ({
      id: i,
      text: `Item ${i + 1}`
    }))
  );

  return <VirtualList items={items} />;
}
```

### 图片懒加载 - 优化资源加载

```jsx
import React, { useState, useRef, useEffect } from 'react';

function LazyImage({ src, alt, placeholder = 'Loading...' }) {
  const [imageSrc, setImageSrc] = useState('');
  const [imageLoaded, setImageLoaded] = useState(false);
  const imgRef = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setImageSrc(src);
          observer.disconnect();
        }
      },
      { threshold: 0.1 }
    );

    if (imgRef.current) {
      observer.observe(imgRef.current);
    }

    return () => observer.disconnect();
  }, [src]);

  const handleLoad = () => {
    setImageLoaded(true);
  };

  return (
    <div
      ref={imgRef}
      style={{
        width: '100%',
        height: '200px',
        background: '#f0f0f0',
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center'
      }}
    >
      {imageSrc ? (
        <img
          src={imageSrc}
          alt={alt}
          onLoad={handleLoad}
          style={{
            width: '100%',
            height: '100%',
            objectFit: 'cover',
            opacity: imageLoaded ? 1 : 0,
            transition: 'opacity 0.3s'
          }}
        />
      ) : (
        <span>{placeholder}</span>
      )}
    </div>
  );
}
```

### 组件懒加载 - 代码分割

```jsx
import React, { lazy, Suspense } from 'react';

// 懒加载组件
const HeavyComponent = lazy(() => import('./HeavyComponent'));
const AnotherComponent = lazy(() => import('./AnotherComponent'));

function App() {
  const [showHeavy, setShowHeavy] = useState(false);
  const [showAnother, setShowAnother] = useState(false);

  return (
    <div>
      <h1>React Lazy Loading Example</h1>
      <button onClick={() => setShowHeavy(!showHeavy)}>
        Toggle Heavy Component
      </button>
      <button onClick={() => setShowAnother(!showAnother)}>
        Toggle Another Component
      </button>

      <Suspense fallback={<div>Loading component...</div>}>
        {showHeavy && <HeavyComponent />}
        {showAnother && <AnotherComponent />}
      </Suspense>
    </div>
  );
}

// 路由级别的懒加载示例
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Contact = lazy(() => import('./pages/Contact'));

function RouterApp() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading page...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/contact" element={<Contact />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

### 防抖和节流 - 优化频繁触发

```jsx
import React, { useState, useCallback } from 'react';

// 防抖 Hook
function useDebounce(callback, delay) {
  const [timeoutId, setTimeoutId] = useState(null);

  const debouncedCallback = useCallback(
    (...args) => {
      if (timeoutId) clearTimeout(timeoutId);
      const newTimeoutId = setTimeout(() => {
        callback(...args);
      }, delay);
      setTimeoutId(newTimeoutId);
    },
    [callback, delay, timeoutId]
  );

  return debouncedCallback;
}

// 节流 Hook
function useThrottle(callback, delay) {
  const [lastRun, setLastRun] = useState(Date.now());
  const [timeoutId, setTimeoutId] = useState(null);

  const throttledCallback = useCallback(
    (...args) => {
      const now = Date.now();
      if (now - lastRun >= delay) {
        callback(...args);
        setLastRun(now);
      } else {
        if (timeoutId) clearTimeout(timeoutId);
        const newTimeoutId = setTimeout(() => {
          callback(...args);
          setLastRun(Date.now());
        }, delay - (now - lastRun));
        setTimeoutId(newTimeoutId);
      }
    },
    [callback, delay, lastRun, timeoutId]
  );

  return throttledCallback;
}

// 使用示例
function SearchComponent() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  const debouncedSearch = useDebounce(async (searchQuery) => {
    if (searchQuery.trim()) {
      const data = await performSearch(searchQuery);
      setResults(data);
    } else {
      setResults([]);
    }
  }, 300);

  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    debouncedSearch(value);
  };

  const throttledScroll = useThrottle(() => {
    console.log('Scroll event throttled');
  }, 200);

  return (
    <div onScroll={throttledScroll}>
      <input
        type="text"
        value={query}
        onChange={handleChange}
        placeholder="Search (debounced)..."
      />
      <ul>
        {results.map((result, index) => (
          <li key={index}>{result}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 状态优化 - 减少不必要的渲染

```jsx
import React, { useState, useCallback, memo } from 'react';

// 拆分状态，避免不相关的更新
function OptimizedForm() {
  const [formData, setFormData] = useState({
    username: '',
    email: '',
    password: ''
  });

  // 使用函数式更新避免依赖
  const updateField = useCallback((field, value) => {
    setFormData(prev => ({
      ...prev,
      [field]: value
    }));
  }, []);

  // 拆分独立的状态
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitError, setSubmitError] = useState(null);

  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsSubmitting(true);
    setSubmitError(null);

    try {
      await submitForm(formData);
      // 处理成功
    } catch (error) {
      setSubmitError(error.message);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <FormField
        label="Username"
        value={formData.username}
        onChange={(value) => updateField('username', value)}
      />
      <FormField
        label="Email"
        value={formData.email}
        onChange={(value) => updateField('email', value)}
      />
      <FormField
        label="Password"
        value={formData.password}
        onChange={(value) => updateField('password', value)}
        type="password"
      />

      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>

      {submitError && <div className="error">{submitError}</div>}
    </form>
  );
}

const FormField = memo(function FormField({ label, value, onChange, type = 'text' }) {
  return (
    <div>
      <label>{label}</label>
      <input
        type={type}
        value={value}
        onChange={(e) => onChange(e.target.value)}
      />
    </div>
  );
});
```

## 注意事项

### 避免过度优化

过早优化是万恶之源：
- 不要盲目使用 `React.memo`、`useMemo`、`useCallback`
- 性能优化本身也有开销
- 先测量性能瓶颈，再有针对性地优化

### 正确使用依赖数组

```jsx
// 错误示例：缺少依赖项
const handleClick = useCallback(() => {
  console.log(count); // count 是过期的闭包值
}, []); // 缺少 count 依赖

// 正确示例：包含所有依赖项
const handleClick = useCallback(() => {
  console.log(count);
}, [count]);

// 使用函数式更新避免依赖
const increment = useCallback(() => {
  setCount(prev => prev + 1);
}, []);
```

### 内存泄漏风险

```jsx
// 错误示例：可能内存泄漏
useEffect(() => {
  const timer = setInterval(() => {
    console.log('Timer tick');
  }, 1000);
  // 缺少清理函数
}, []);

// 正确示例：正确清理副作用
useEffect(() => {
  const timer = setInterval(() => {
    console.log('Timer tick');
  }, 1000);

  return () => clearInterval(timer);
}, []);
```

### Context 性能问题

```jsx
// 错误示例：整个 context 变化导致所有消费者重新渲染
const AppContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');

  return (
    <AppContext.Provider value={{ user, setUser, theme, setTheme }}>
      {children}
    </AppContext.Provider>
  );
}

// 正确示例：拆分 context，减少不必要的渲染
const UserContext = createContext();
const ThemeContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');

  return (
    <UserContext.Provider value={{ user, setUser }}>
      <ThemeContext.Provider value={{ theme, setTheme }}>
        {children}
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}
```

## 总结

React 性能优化是一个系统性的工程，需要从多个维度进行考虑：

1. **组件层面优化**
   - 使用 `React.memo` 避免不必要的重渲染
   - 合理拆分组件，遵循单一职责原则
   - 使用懒加载和代码分割减少初始加载体积

2. **状态和数据优化**
   - 合理设计状态结构，减少状态更新的影响范围
   - 使用 `useMemo` 缓存计算结果
   - 使用 `useCallback` 稳定函数引用

3. **渲染优化**
   - 使用虚拟化处理大数据列表
   - 合理使用 `useTransition` 和 `useDeferredValue`
   - 优化 Context 结构，避免大范围重渲染

4. **资源优化**
   - 图片懒加载和优化
   - API 请求优化（防抖、节流、缓存）
   - 合理使用 CSS 动画和过渡效果

记住，性能优化的核心原则是：**先测量，后优化**。使用 React DevTools Profiler 等工具找出真正的性能瓶颈，然后有针对性地进行优化。同时，保持代码的可读性和可维护性也是非常重要的。

通过合理应用这些优化技术，可以显著提升 React 应用的性能表现，为用户提供更好的使用体验。