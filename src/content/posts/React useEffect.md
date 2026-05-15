---
title: React useEffect
published: 2023-08-02
description: '副作用 Hook 使用的详细介绍和学习笔记'
image: ''
tags: ["React","Hooks"]
category: 'React'
draft: false
lang: 'zh-CN'
---

## React Hooks 入门

Hooks 是 React 16.8 引入的新特性，让你在不编写 class 的情况下使用 state 和其他 React 特性。

## useState

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        增加
      </button>
    </div>
  );
}

// 使用函数更新
setCount(prev => prev + 1);

// 对象状态
const [user, setUser] = useState({ name: '', age: 0 });
setUser(prev => ({ ...prev, name: 'new name' }));
```

## useEffect

```jsx
import { useEffect, useState } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  // 组件挂载时执行
  useEffect(() => {
    fetchUser(userId).then(data => setUser(data));
  }, [userId]); // 依赖数组
  
  // 清理函数
  useEffect(() => {
    const subscription = subscribe();
    return () => {
      subscription.unsubscribe();
    };
  }, []);
  
  return <div>{user?.name}</div>;
}
```

## 自定义 Hook

```jsx
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

// 使用
const { width, height } = useWindowSize();
```

## 总结

Hooks 让函数组件拥有了状态和生命周期，是 React 开发的现代化方式。