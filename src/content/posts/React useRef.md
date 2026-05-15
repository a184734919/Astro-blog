---
title: React useRef
published: 2023-08-12
description: '深入理解 React useRef Hook：DOM 访问和可变引用存储'
image: ''
tags: ["React","Hooks"]
category: 'React'
draft: false
lang: 'zh-CN'
---

## 概述

useRef 是 React 中一个特殊的 Hook，它返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数。返回的 ref 对象在组件的整个生命周期内保持不变。useRef 主要用于两个目的：访问 DOM 元素和在组件渲染之间保存可变值。

### useRef 的独特之处

与其他 Hook 不同，useRef 返回的对象在组件的整个生命周期内保持稳定，即使组件重新渲染，ref 对象本身也不会改变。这使得它成为存储不需要触发重新渲染的数据的理想选择。

## 核心概念

### useRef 的基本特征

1. **可变性**：`.current` 属性是可变的，可以随时更新
2. **持久性**：在组件的整个生命周期内保持不变
3. **不触发重渲染**：修改 `.current` 不会触发组件重新渲染
4. **同步更新**：`.current` 的变化是同步的

### useRef 签名

```jsx
const refContainer = useRef(initialValue);
```

- **initialValue**：ref 的初始值
- **返回值**：包含 `.current` 属性的对象

### 工作原理

```jsx
function RefExample() {
  const myRef = useRef(0);
  
  console.log('渲染时的 ref:', myRef.current);
  
  const updateRef = () => {
    myRef.current += 1;
    console.log('更新后的 ref:', myRef.current);
    // 不会触发组件重新渲染
  };
  
  return (
    <div>
      <button onClick={updateRef}>更新 Ref</button>
      <p>Ref 值: {myRef.current}</p>
    </div>
  );
}
```

## 基本用法

### DOM 元素访问

```jsx
function FocusInput() {
  const inputRef = useRef(null);
  
  const handleFocus = () => {
    // 访问 DOM 元素
    inputRef.current.focus();
  };
  
  return (
    <div>
      <input ref={inputRef} placeholder="点击按钮聚焦" />
      <button onClick={handleFocus}>聚焦输入框</button>
    </div>
  );
}
```

### 存储可变值

```jsx
function Timer() {
  const [count, setCount] = useState(0);
  const timerRef = useRef(null);
  
  const startTimer = () => {
    if (timerRef.current) return; // 防止重复启动
    
    timerRef.current = setInterval(() => {
      setCount(prev => prev + 1);
    }, 1000);
  };
  
  const stopTimer = () => {
    if (timerRef.current) {
      clearInterval(timerRef.current);
      timerRef.current = null;
    }
  };
  
  const resetTimer = () => {
    stopTimer();
    setCount(0);
  };
  
  // 组件卸载时清理
  useEffect(() => {
    return () => {
      if (timerRef.current) {
        clearInterval(timerRef.current);
      }
    };
  }, []);
  
  return (
    <div>
      <p>计时器: {count} 秒</p>
      <button onClick={startTimer}>开始</button>
      <button onClick={stopTimer}>停止</button>
      <button onClick={resetTimer}>重置</button>
    </div>
  );
}
```

### 记录渲染次数

```jsx
function RenderCounter() {
  const renderCount = useRef(0);
  
  renderCount.current += 1;
  
  return (
    <div>
      <p>组件渲染了 {renderCount.current} 次</p>
    </div>
  );
}
```

### 存储上一次的值

```jsx
function PreviousValue() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();
  
  // 在渲染后更新 ref
  useEffect(() => {
    prevCountRef.current = count;
  });
  
  const prevCount = prevCountRef.current;
  
  return (
    <div>
      <p>当前值: {count}</p>
      <p>上一次的值: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>增加</button>
    </div>
  );
}
```

## 实际应用

### 表单焦点管理

```jsx
function FormFocusManager() {
  const nameInputRef = useRef(null);
  const emailInputRef = useRef(null);
  const passwordInputRef = useRef(null);
  
  const focusNext = (currentRef, nextRef) => {
    if (currentRef.current && nextRef.current) {
      nextRef.current.focus();
    }
  };
  
  return (
    <form>
      <input
        ref={nameInputRef}
        placeholder="姓名"
        onKeyDown={(e) => {
          if (e.key === 'Enter') {
            e.preventDefault();
            focusNext(nameInputRef, emailInputRef);
          }
        }}
      />
      <input
        ref={emailInputRef}
        placeholder="邮箱"
        onKeyDown={(e) => {
          if (e.key === 'Enter') {
            e.preventDefault();
            focusNext(emailInputRef, passwordInputRef);
          }
        }}
      />
      <input
        ref={passwordInputRef}
        type="password"
        placeholder="密码"
      />
      <button type="submit">提交</button>
    </form>
  );
}
```

### 滚动到指定位置

```jsx
function ScrollToSection() {
  const sectionRef = useRef(null);
  
  const scrollToSection = () => {
    if (sectionRef.current) {
      sectionRef.current.scrollIntoView({
        behavior: 'smooth',
        block: 'start'
      });
    }
  };
  
  return (
    <div>
      <button onClick={scrollToSection}>滚动到章节</button>
      
      <div style={{ height: '1000px' }}>中间内容</div>
      
      <div ref={sectionRef} style={{ background: 'lightblue', padding: '20px' }}>
        <h2>目标章节</h2>
        <p>这里是滚动目标位置</p>
      </div>
    </div>
  );
}
```

### 动画控制

```jsx
function AnimationController() {
  const boxRef = useRef(null);
  const animationRef = useRef(null);
  const isAnimatingRef = useRef(false);
  
  const startAnimation = () => {
    if (isAnimatingRef.current) return;
    
    isAnimatingRef.current = true;
    const box = boxRef.current;
    let position = 0;
    const speed = 2;
    
    const animate = () => {
      position += speed;
      if (position > 300) {
        position = 0;
      }
      
      box.style.transform = `translateX(${position}px)`;
      animationRef.current = requestAnimationFrame(animate);
    };
    
    animationRef.current = requestAnimationFrame(animate);
  };
  
  const stopAnimation = () => {
    if (animationRef.current) {
      cancelAnimationFrame(animationRef.current);
      animationRef.current = null;
      isAnimatingRef.current = false;
    }
  };
  
  // 组件卸载时清理
  useEffect(() => {
    return () => {
      if (animationRef.current) {
        cancelAnimationFrame(animationRef.current);
      }
    };
  }, []);
  
  return (
    <div>
      <div
        ref={boxRef}
        style={{
          width: '50px',
          height: '50px',
          background: 'red',
          transition: 'none'
        }}
      />
      <button onClick={startAnimation}>开始动画</button>
      <button onClick={stopAnimation}>停止动画</button>
    </div>
  );
}
```

### 防抖和节流

```jsx
function useDebouncedCallback(callback, delay) {
  const timeoutRef = useRef(null);
  
  return useCallback((...args) => {
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }
    
    timeoutRef.current = setTimeout(() => {
      callback(...args);
    }, delay);
  }, [callback, delay]);
}

function DebouncedSearch() {
  const [searchTerm, setSearchTerm] = useState('');
  const [results, setResults] = useState([]);
  
  const debouncedSearch = useDebouncedCallback((term) => {
    // 模拟搜索 API 调用
    console.log('搜索:', term);
    // 这里可以调用实际的搜索 API
  }, 500);
  
  const handleChange = (e) => {
    const value = e.target.value;
    setSearchTerm(value);
    debouncedSearch(value);
  };
  
  return (
    <div>
      <input
        value={searchTerm}
        onChange={handleChange}
        placeholder="搜索..."
      />
      <div>
        {results.map(result => (
          <div key={result.id}>{result.name}</div>
        ))}
      </div>
    </div>
  );
}
```

### Canvas 绘图

```jsx
function CanvasDrawing() {
  const canvasRef = useRef(null);
  const isDrawingRef = useRef(false);
  const lastXRef = useRef(0);
  const lastYRef = useRef(0);
  
  const startDrawing = (e) => {
    isDrawingRef.current = true;
    [lastXRef.current, lastYRef.current] = [e.nativeEvent.offsetX, e.nativeEvent.offsetY];
  };
  
  const draw = (e) => {
    if (!isDrawingRef.current) return;
    
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    
    ctx.beginPath();
    ctx.moveTo(lastXRef.current, lastYRef.current);
    ctx.lineTo(e.nativeEvent.offsetX, e.nativeEvent.offsetY);
    ctx.stroke();
    
    [lastXRef.current, lastYRef.current] = [e.nativeEvent.offsetX, e.nativeEvent.offsetY];
  };
  
  const stopDrawing = () => {
    isDrawingRef.current = false;
  };
  
  const clearCanvas = () => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext('2d');
    ctx.clearRect(0, 0, canvas.width, canvas.height);
  };
  
  return (
    <div>
      <canvas
        ref={canvasRef}
        width={400}
        height={300}
        style={{ border: '1px solid black' }}
        onMouseDown={startDrawing}
        onMouseMove={draw}
        onMouseUp={stopDrawing}
        onMouseLeave={stopDrawing}
      />
      <button onClick={clearCanvas}>清除画布</button>
    </div>
  );
}
```

### WebSocket 连接管理

```jsx
function useWebSocket(url) {
  const socketRef = useRef(null);
  const messageQueueRef = useRef([]);
  
  const connect = useCallback(() => {
    if (socketRef.current?.readyState === WebSocket.OPEN) {
      return;
    }
    
    const socket = new WebSocket(url);
    
    socket.onopen = () => {
      console.log('WebSocket 连接已建立');
      // 发送队列中的消息
      messageQueueRef.current.forEach(message => {
        socket.send(JSON.stringify(message));
      });
      messageQueueRef.current = [];
    };
    
    socket.onmessage = (event) => {
      const data = JSON.parse(event.data);
      console.log('收到消息:', data);
    };
    
    socket.onerror = (error) => {
      console.error('WebSocket 错误:', error);
    };
    
    socket.onclose = () => {
      console.log('WebSocket 连接已关闭');
      socketRef.current = null;
    };
    
    socketRef.current = socket;
  }, [url]);
  
  const sendMessage = useCallback((message) => {
    if (socketRef.current?.readyState === WebSocket.OPEN) {
      socketRef.current.send(JSON.stringify(message));
    } else {
      // 连接未建立，将消息加入队列
      messageQueueRef.current.push(message);
    }
  }, []);
  
  const disconnect = useCallback(() => {
    if (socketRef.current) {
      socketRef.current.close();
      socketRef.current = null;
    }
  }, []);
  
  // 组件卸载时断开连接
  useEffect(() => {
    return () => {
      disconnect();
    };
  }, [disconnect]);
  
  return { connect, sendMessage, disconnect };
}

function ChatRoom({ roomId }) {
  const ws = useWebSocket(`wss://chat.example.com/rooms/${roomId}`);
  
  useEffect(() => {
    ws.connect();
  }, [ws]);
  
  const sendMessage = (text) => {
    ws.sendMessage({
      type: 'message',
      roomId,
      text,
      timestamp: Date.now()
    });
  };
  
  return (
    <div>
      <button onClick={() => sendMessage('Hello')}>发送消息</button>
      <button onClick={ws.disconnect}>断开连接</button>
    </div>
  );
}
```

## 注意事项

### 1. useRef 不触发重渲染

```jsx
function NoReRender() {
  const [count, setCount] = useState(0);
  const refCount = useRef(0);
  
  const updateState = () => {
    setCount(count + 1); // 触发重渲染
  };
  
  const updateRef = () => {
    refCount.current += 1; // 不触发重渲染
    console.log('Ref value:', refCount.current);
  };
  
  return (
    <div>
      <p>State: {count}</p>
      <p>Ref: {refCount.current}</p>
      <button onClick={updateState}>更新 State</button>
      <button onClick={updateRef}>更新 Ref</button>
    </div>
  );
}
```

### 2. 初始化时机

```jsx
function InitTiming() {
  const ref1 = useRef(console.log('初始化时执行')); // 每次渲染都执行
  const ref2 = useRef(() => console.log('函数不会执行')); // 不会执行
  
  console.log('渲染时执行');
  
  return <div>Ref 示例</div>;
}
```

### 3. ref.current 的更新时机

```jsx
function RefUpdateTiming() {
  const [count, setCount] = useState(0);
  const ref = useRef(0);
  
  const handleClick = () => {
    ref.current = count + 1;
    console.log('同步更新后:', ref.current); // 立即更新
    setCount(count + 1);
  };
  
  console.log('渲染时 ref:', ref.current);
  
  return (
    <div>
      <button onClick={handleClick}>更新</button>
      <p>Count: {count}</p>
    </div>
  );
}
```

### 4. 不要在渲染中读取 ref.current

```jsx
// ❌ 错误：在渲染中读取 ref 可能导致不一致
function BadReadRef() {
  const ref = useRef(0);
  
  const value = ref.current; // 在渲染中读取
  
  return <div>{value}</div>;
}

// ✅ 正确：在事件处理器或副作用中读取
function GoodReadRef() {
  const ref = useRef(0);
  
  const handleClick = () => {
    const value = ref.current; // 在事件处理器中读取
    console.log(value);
  };
  
  return <button onClick={handleClick}>读取 Ref</button>;
}
```

### 5. 清理副作用

```jsx
function CleanupEffects() {
  const intervalRef = useRef(null);
  const animationRef = useRef(null);
  
  const startTimers = () => {
    intervalRef.current = setInterval(() => {
      console.log('定时器');
    }, 1000);
    
    const animate = () => {
      console.log('动画帧');
      animationRef.current = requestAnimationFrame(animate);
    };
    
    animationRef.current = requestAnimationFrame(animate);
  };
  
  const stopTimers = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
    
    if (animationRef.current) {
      cancelAnimationFrame(animationRef.current);
      animationRef.current = null;
    }
  };
  
  // 组件卸载时清理
  useEffect(() => {
    return () => {
      stopTimers();
    };
  }, []);
  
  return (
    <div>
      <button onClick={startTimers}>启动</button>
      <button onClick={stopTimers}>停止</button>
    </div>
  );
}
```

## 总结

useRef 是 React 中一个强大但经常被忽视的 Hook：

### 核心要点

1. **可变引用**：返回的对象在组件生命周期内保持不变
2. **不触发重渲染**：修改 `.current` 不会触发组件重新渲染
3. **DOM 访问**：可以直接访问 DOM 元素
4. **数据持久化**：在渲染之间保存可变数据

### 使用场景

1. **DOM 操作**：访问和操作 DOM 元素
2. **定时器管理**：存储定时器 ID，便于清理
3. **动画控制**：管理动画状态和帧
4. **数据缓存**：存储不需要触发重渲染的数据
5. **副作用管理**：在副作用中存储和访问数据

### 最佳实践

1. **明确用途**：清楚区分 DOM ref 和数据 ref 的使用场景
2. **及时清理**：在组件卸载时清理副作用
3. **避免渲染读取**：不要在渲染中读取 ref.current
4. **合理使用**：不需要触发重渲染时使用 ref，否则使用 state
5. **类型安全**：配合 TypeScript 提供类型定义

useRef 提供了一种在 React 函数组件中处理可变数据的强大方式。理解它的工作原理和适用场景，能够帮助你编写更高效、更灵活的 React 组件。它是连接 React 声明式编程和传统命令式编程的重要桥梁。