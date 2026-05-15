---
title: React 自定义 Hook
published: 2023-08-20
description: '提取复用逻辑的详细介绍和学习笔记'
image: ''
tags: ["React","Hooks"]
category: 'React'
draft: false
lang: 'zh-CN'
---

## 概述

React 自定义 Hook 是复用组件逻辑的强大方式。通过将状态逻辑和副作用从组件中提取出来，可以创建可重用的、可测试的逻辑单元。自定义 Hook 命名以 "use" 开头，可以调用其他 Hook，遵循 Hook 的使用规则。本文将详细介绍自定义 Hook 的创建方法、常见模式和最佳实践。

## 核心概念

### 自定义 Hook 的优势

1. **逻辑复用**：在不同组件间共享状态逻辑
2. **关注点分离**：将 UI 和逻辑分离
3. **可测试性**：独立测试业务逻辑
4. **代码组织**：更好地组织复杂的状态管理
5. **团队协作**：提供统一的逻辑接口

### Hook 命名规则

自定义 Hook 必须以 "use" 开头，让 React 识别其为 Hook：
- `useFetch` - 数据获取
- `useLocalStorage` - 本地存储
- `useDebounce` - 防抖
- `useToggle` - 切换状态
- `useForm` - 表单管理

### Hook 使用规则

1. 只在 React 函数组件或自定义 Hook 中调用 Hook
2. 只在顶层调用 Hook，不在循环、条件或嵌套函数中调用
3. 自定义 Hook 可以调用其他 Hook

## 基本用法

### 基础自定义 Hook

```jsx
// 简单的状态管理 Hook
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => setValue(prev => !prev), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);

  return { value, toggle, setTrue, setFalse, setValue };
}

// 使用示例
function Modal() {
  const { value: isOpen, toggle, setFalse: closeModal } = useToggle(false);

  return (
    <div>
      <button onClick={toggle}>Open Modal</button>
      {isOpen && (
        <div className="modal">
          <div className="modal-content">
            <h2>Modal Title</h2>
            <p>Modal content goes here.</p>
            <button onClick={closeModal}>Close</button>
          </div>
        </div>
      )}
    </div>
  );
}

// 数组操作 Hook
function useArray(initialValue = []) {
  const [array, setArray] = useState(initialValue);

  const push = useCallback((element) => {
    setArray(prev => [...prev, element]);
  }, []);

  const filter = useCallback((callback) => {
    setArray(prev => prev.filter(callback));
  }, []);

  const update = useCallback((index, newElement) => {
    setArray(prev => [
      ...prev.slice(0, index),
      newElement,
      ...prev.slice(index + 1)
    ]);
  }, []);

  const remove = useCallback((index) => {
    setArray(prev => [
      ...prev.slice(0, index),
      ...prev.slice(index + 1)
    ]);
  }, []);

  const clear = useCallback(() => setArray([]), []);

  return { array, set: setArray, push, filter, update, remove, clear };
}

// 使用数组 Hook
function TodoList() {
  const { array: todos, push: addTodo, remove: removeTodo } = useArray([
    { id: 1, text: 'Learn React Hooks' }
  ]);

  const [inputValue, setInputValue] = useState('');

  const handleAdd = () => {
    if (inputValue.trim()) {
      addTodo({ id: Date.now(), text: inputValue });
      setInputValue('');
    }
  };

  return (
    <div>
      <input
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        placeholder="Add a new todo"
      />
      <button onClick={handleAdd}>Add</button>

      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            {todo.text}
            <button onClick={() => removeTodo(todos.indexOf(todo))}>
              Remove
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### 数据获取 Hook

```jsx
// 基础的数据获取 Hook
function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const controller = new AbortController();

    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);

        const response = await fetch(url, {
          ...options,
          signal: controller.signal
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
    };

    fetchData();

    return () => controller.abort();
  }, [url, JSON.stringify(options)]);

  return { data, loading, error };
}

// 带缓存的数据获取 Hook
function useFetchWithCache(url, options = {}, cacheTime = 5 * 60 * 1000) {
  const cacheRef = useRef(new Map());

  const fetchData = useCallback(async () => {
    const cacheKey = url + JSON.stringify(options);
    const cached = cacheRef.current.get(cacheKey);

    // 检查缓存是否有效
    if (cached && Date.now() - cached.timestamp < cacheTime) {
      return cached.data;
    }

    const response = await fetch(url, options);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();

    // 更新缓存
    cacheRef.current.set(cacheKey, {
      data,
      timestamp: Date.now()
    });

    return data;
  }, [url, JSON.stringify(options), cacheTime]);

  return useAsync(fetchData, [url, options]);
}

// 异步操作 Hook
function useAsync(asyncFunction, immediate = true) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(immediate);
  const [error, setError] = useState(null);

  const execute = useCallback(async (...args) => {
    setLoading(true);
    setError(null);

    try {
      const result = await asyncFunction(...args);
      setData(result);
      return result;
    } catch (err) {
      setError(err);
      throw err;
    } finally {
      setLoading(false);
    }
  }, [asyncFunction]);

  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);

  return { data, loading, error, execute };
}

// 使用示例
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(`/api/users/${userId}`);

  if (loading) return <div>Loading user...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

### 表单管理 Hook

```jsx
// 表单验证 Hook
function useFormValidation(initialState, validate) {
  const [values, setValues] = useState(initialState);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});

  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));

    // 实时验证
    if (touched[name]) {
      setErrors(prev => ({
        ...prev,
        [name]: validate({ ...values, [name]: value })[name]
      }));
    }
  };

  const handleBlur = (e) => {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));

    // 失焦时验证
    setErrors(validate(values));
  };

  const handleSubmit = (callback) => (e) => {
    e.preventDefault();

    // 验证所有字段
    const validationErrors = validate(values);
    setErrors(validationErrors);

    // 如果没有错误，提交表单
    if (Object.keys(validationErrors).length === 0) {
      callback(values);
    }
  };

  const resetForm = () => {
    setValues(initialState);
    setErrors({});
    setTouched({});
  };

  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    handleSubmit,
    resetForm,
    isValid: Object.keys(errors).length === 0
  };
}

// 登录表单示例
function LoginForm() {
  const { values, errors, handleChange, handleBlur, handleSubmit, isValid } = useFormValidation(
    {
      email: '',
      password: ''
    },
    (values) => {
      const errors = {};

      if (!values.email) {
        errors.email = 'Email is required';
      } else if (!/\S+@\S+\.\S+/.test(values.email)) {
        errors.email = 'Email is invalid';
      }

      if (!values.password) {
        errors.password = 'Password is required';
      } else if (values.password.length < 6) {
        errors.password = 'Password must be at least 6 characters';
      }

      return errors;
    }
  );

  const handleLogin = (formData) => {
    console.log('Login with:', formData);
    // 实际的登录逻辑
  };

  return (
    <form onSubmit={handleSubmit(handleLogin)}>
      <div>
        <label>Email:</label>
        <input
          type="email"
          name="email"
          value={values.email}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>

      <div>
        <label>Password:</label>
        <input
          type="password"
          name="password"
          value={values.password}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {errors.password && <span className="error">{errors.password}</span>}
      </div>

      <button type="submit" disabled={!isValid}>
        Login
      </button>
    </form>
  );
}

// 通用表单 Hook
function useForm(initialValues, onSubmit) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [submitting, setSubmitting] = useState(false);

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setValues(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    setSubmitting(true);

    try {
      await onSubmit(values);
    } catch (error) {
      setErrors(error);
    } finally {
      setSubmitting(false);
    }
  };

  const reset = () => {
    setValues(initialValues);
    setErrors({});
  };

  return {
    values,
    errors,
    submitting,
    handleChange,
    handleSubmit,
    reset
  };
}
```

### 本地存储 Hook

```jsx
// LocalStorage Hook
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading localStorage key "${key}":`, error);
      return initialValue;
    }
  });

  const setValue = useCallback((value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);

      if (valueToStore === undefined) {
        window.localStorage.removeItem(key);
      } else {
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
      }
    } catch (error) {
      console.error(`Error setting localStorage key "${key}":`, error);
    }
  }, [key, storedValue]);

  const removeValue = useCallback(() => {
    setStoredValue(initialValue);
    window.localStorage.removeItem(key);
  }, [key, initialValue]);

  return [storedValue, setValue, removeValue];
}

// 使用示例
function ThemeToggle() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  return (
    <div className={`app ${theme}`}>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} theme
      </button>
    </div>
  );
}

// SessionStorage Hook
function useSessionStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.sessionStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(`Error reading sessionStorage key "${key}":`, error);
      return initialValue;
    }
  });

  const setValue = useCallback((value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);

      if (valueToStore === undefined) {
        window.sessionStorage.removeItem(key);
      } else {
        window.sessionStorage.setItem(key, JSON.stringify(valueToStore));
      }
    } catch (error) {
      console.error(`Error setting sessionStorage key "${key}":`, error);
    }
  }, [key, storedValue]);

  return [storedValue, setValue];
}
```

## 实际应用

### 防抖和节流 Hook

```jsx
// 防抖 Hook
function useDebounce(callback, delay) {
  const timerRef = useRef(null);

  const debouncedCallback = useCallback((...args) => {
    if (timerRef.current) {
      clearTimeout(timerRef.current);
    }

    timerRef.current = setTimeout(() => {
      callback(...args);
    }, delay);
  }, [callback, delay]);

  useEffect(() => {
    return () => {
      if (timerRef.current) {
        clearTimeout(timerRef.current);
      }
    };
  }, [debouncedCallback]);

  return debouncedCallback;
}

// 节流 Hook
function useThrottle(callback, delay) {
  const lastRunRef = useRef(Date.now());
  const timerRef = useRef(null);

  const throttledCallback = useCallback((...args) => {
    const now = Date.now();

    if (now - lastRunRef.current >= delay) {
      callback(...args);
      lastRunRef.current = now;
    } else {
      if (timerRef.current) {
        clearTimeout(timerRef.current);
      }

      timerRef.current = setTimeout(() => {
        callback(...args);
        lastRunRef.current = Date.now();
      }, delay - (now - lastRunRef.current));
    }
  }, [callback, delay]);

  return throttledCallback;
}

// 防抖值 Hook
function useDebounceValue(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// 使用示例
function SearchInput() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounceValue(query, 300);
  const [results, setResults] = useState([]);

  // 搜索逻辑只在防抖后的值变化时执行
  useEffect(() => {
    if (debouncedQuery) {
      performSearch(debouncedQuery).then(setResults);
    } else {
      setResults([]);
    }
  }, [debouncedQuery]);

  const handleScroll = useThrottle(() => {
    console.log('Scroll event throttled');
  }, 200);

  return (
    <div onScroll={handleScroll}>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <ul>
        {results.map(result => (
          <li key={result.id}>{result.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 媒体查询 Hook

```jsx
// 响应式 Hook
function useMediaQuery(query) {
  const [matches, setMatches] = useState(() => {
    if (typeof window !== 'undefined') {
      return window.matchMedia(query).matches;
    }
    return false;
  });

  useEffect(() => {
    const mediaQuery = window.matchMedia(query);
    const handler = (event) => setMatches(event.matches);

    // 现代浏览器
    mediaQuery.addEventListener('change', handler);
    return () => mediaQuery.removeEventListener('change', handler);
  }, [query]);

  return matches;
}

// 预定义的媒体查询 Hook
function useBreakpoints() {
  const isMobile = useMediaQuery('(max-width: 640px)');
  const isTablet = useMediaQuery('(min-width: 641px) and (max-width: 1024px)');
  const isDesktop = useMediaQuery('(min-width: 1025px)');

  return { isMobile, isTablet, isDesktop };
}

// 使用示例
function ResponsiveComponent() {
  const { isMobile, isTablet, isDesktop } = useBreakpoints();

  return (
    <div className={`responsive ${isMobile ? 'mobile' : isTablet ? 'tablet' : 'desktop'}`}>
      <h1>Responsive Layout</h1>
      {isMobile && <p>Mobile view</p>}
      {isTablet && <p>Tablet view</p>}
      {isDesktop && <p>Desktop view</p>}
    </div>
  );
}

// 窗口大小 Hook
function useWindowSize() {
  const [windowSize, setWindowSize] = useState({
    width: typeof window !== 'undefined' ? window.innerWidth : 0,
    height: typeof window !== 'undefined' ? window.innerHeight : 0
  });

  useEffect(() => {
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return windowSize;
}
```

### 权限和认证 Hook

```jsx
// 认证 Hook
function useAuth() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [isAuthenticated, setIsAuthenticated] = useState(false);

  useEffect(() => {
    checkAuth();
  }, []);

  const checkAuth = async () => {
    try {
      const token = localStorage.getItem('token');
      if (token) {
        const response = await fetch('/api/auth/me', {
          headers: { Authorization: `Bearer ${token}` }
        });

        if (response.ok) {
          const userData = await response.json();
          setUser(userData);
          setIsAuthenticated(true);
        } else {
          logout();
        }
      }
    } catch (error) {
      console.error('Auth check failed:', error);
    } finally {
      setLoading(false);
    }
  };

  const login = async (credentials) => {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credentials)
    });

    if (response.ok) {
      const { token, user: userData } = await response.json();
      localStorage.setItem('token', token);
      setUser(userData);
      setIsAuthenticated(true);
      return true;
    }

    return false;
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
    setIsAuthenticated(false);
  };

  return { user, loading, isAuthenticated, login, logout, checkAuth };
}

// 权限 Hook
function usePermissions() {
  const { user } = useAuth();

  const hasPermission = useCallback((permission) => {
    if (!user) return false;
    return user.permissions?.includes(permission) || false;
  }, [user]);

  const hasRole = useCallback((role) => {
    if (!user) return false;
    return user.role === role || user.roles?.includes(role);
  }, [user]);

  const hasAnyRole = useCallback((roles) => {
    if (!user) return false;
    return roles.some(role => user.role === role || user.roles?.includes(role));
  }, [user]);

  return { hasPermission, hasRole, hasAnyRole };
}

// 使用示例
function ProtectedRoute({ children, requiredPermission, requiredRole }) {
  const { isAuthenticated } = useAuth();
  const { hasPermission, hasRole } = usePermissions();

  if (!isAuthenticated) {
    return <Navigate to="/login" />;
  }

  if (requiredPermission && !hasPermission(requiredPermission)) {
    return <Navigate to="/unauthorized" />;
  }

  if (requiredRole && !hasRole(requiredRole)) {
    return <Navigate to="/unauthorized" />;
  }

  return children;
}

function AdminPanel() {
  const { hasPermission } = usePermissions();

  return (
    <div>
      <h1>Admin Panel</h1>
      {hasPermission('manage_users') && (
        <button>Manage Users</button>
      )}
      {hasPermission('view_analytics') && (
        <button>View Analytics</button>
      )}
    </div>
  );
}
```

### WebSocket Hook

```jsx
// WebSocket Hook
function useWebSocket(url, options = {}) {
  const [socket, setSocket] = useState(null);
  const [lastMessage, setLastMessage] = useState(null);
  const [connectionStatus, setConnectionStatus] = useState('connecting');
  const [reconnectAttempts, setReconnectAttempts] = useState(0);

  const { reconnect = true, reconnectInterval = 3000, maxReconnectAttempts = 5 } = options;

  useEffect(() => {
    const ws = new WebSocket(url);

    setConnectionStatus('connecting');

    ws.onopen = () => {
      setConnectionStatus('connected');
      setReconnectAttempts(0);
    };

    ws.onmessage = (event) => {
      setLastMessage(JSON.parse(event.data));
    };

    ws.onerror = (error) => {
      setConnectionStatus('error');
      console.error('WebSocket error:', error);
    };

    ws.onclose = () => {
      setConnectionStatus('disconnected');

      if (reconnect && reconnectAttempts < maxReconnectAttempts) {
        setTimeout(() => {
          setReconnectAttempts(prev => prev + 1);
        }, reconnectInterval);
      }
    };

    setSocket(ws);

    return () => {
      ws.close();
    };
  }, [url, reconnect, reconnectInterval, maxReconnectAttempts, reconnectAttempts]);

  const sendMessage = useCallback((message) => {
    if (socket && socket.readyState === WebSocket.OPEN) {
      socket.send(JSON.stringify(message));
    } else {
      console.error('WebSocket is not connected');
    }
  }, [socket]);

  return { socket, lastMessage, connectionStatus, sendMessage };
}

// 使用示例
function ChatComponent() {
  const { lastMessage, connectionStatus, sendMessage } = useWebSocket('ws://localhost:8080');
  const [messageInput, setMessageInput] = useState('');

  const handleSend = () => {
    if (messageInput.trim()) {
      sendMessage({ type: 'chat', text: messageInput });
      setMessageInput('');
    }
  };

  return (
    <div>
      <div>Connection Status: {connectionStatus}</div>
      <div className="messages">
        {lastMessage && <div>{lastMessage.text}</div>}
      </div>
      <input
        value={messageInput}
        onChange={(e) => setMessageInput(e.target.value)}
        placeholder="Type a message..."
      />
      <button onClick={handleSend}>Send</button>
    </div>
  );
}
```

### 拖拽 Hook

```jsx
// 拖拽 Hook
function useDraggable(options = {}) {
  const { onDragStart, onDrag, onDragEnd } = options;
  const [isDragging, setIsDragging] = useState(false);
  const [position, setPosition] = useState({ x: 0, y: 0 });

  const handleMouseDown = (e) => {
    setIsDragging(true);
    setPosition({ x: e.clientX, y: e.clientY });

    if (onDragStart) {
      onDragStart(e);
    }
  };

  const handleMouseMove = useCallback((e) => {
    if (!isDragging) return;

    const newPosition = { x: e.clientX, y: e.clientY };
    setPosition(newPosition);

    if (onDrag) {
      onDrag(e, newPosition);
    }
  }, [isDragging, onDrag]);

  const handleMouseUp = useCallback((e) => {
    if (!isDragging) return;

    setIsDragging(false);

    if (onDragEnd) {
      onDragEnd(e, position);
    }
  }, [isDragging, position, onDragEnd]);

  useEffect(() => {
    if (isDragging) {
      window.addEventListener('mousemove', handleMouseMove);
      window.addEventListener('mouseup', handleMouseUp);

      return () => {
        window.removeEventListener('mousemove', handleMouseMove);
        window.removeEventListener('mouseup', handleMouseUp);
      };
    }
  }, [isDragging, handleMouseMove, handleMouseUp]);

  return {
    isDragging,
    position,
    draggableProps: {
      onMouseDown: handleMouseDown,
      style: {
        cursor: isDragging ? 'grabbing' : 'grab',
        userSelect: 'none'
      }
    }
  };
}

// 使用示例
function DraggableBox() {
  const { isDragging, position, draggableProps } = useDraggable({
    onDragStart: (e) => console.log('Drag started'),
    onDrag: (e, pos) => console.log('Dragging at', pos),
    onDragEnd: (e, finalPos) => console.log('Drag ended at', finalPos)
  });

  return (
    <div
      {...draggableProps}
      style={{
        ...draggableProps.style,
        width: '200px',
        height: '200px',
        background: isDragging ? 'blue' : 'gray',
        position: 'fixed',
        left: position.x - 100,
        top: position.y - 100,
        display: 'flex',
        alignItems: 'center',
        justifyContent: 'center',
        color: 'white',
        fontSize: '18px',
        borderRadius: '8px',
        boxShadow: '0 4px 6px rgba(0, 0, 0, 0.1)'
      }}
    >
      Drag me!
    </div>
  );
}
```

## 注意事项

### Hook 依赖管理

```jsx
// 问题：依赖数组不正确导致无限循环
function useCounter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      setCount(count + 1); // count 在依赖数组中，导致无限循环
    }, 1000);

    return () => clearInterval(timer);
  }, [count]); // 问题：count 变化会重新创建 timer
}

// 解决方案：使用函数式更新
function useCounter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      setCount(prev => prev + 1); // 使用函数式更新
    }, 1000);

    return () => clearInterval(timer);
  }, []); // 空依赖数组

  return count;
}

// 问题：依赖对象引用变化
function useSearch(query, options) {
  useEffect(() => {
    performSearch(query, options);
  }, [query, options]); // options 对象每次都是新引用
}

// 解决方案：使用 useMemo 或 JSON.stringify
function useSearch(query, options) {
  const optionsString = JSON.stringify(options);

  useEffect(() => {
    performSearch(query, options);
  }, [query, optionsString]); // 正确的依赖
}
```

### Hook 性能优化

```jsx
// 问题：每次渲染都创建新函数
function useDataFetcher(url) {
  const [data, setData] = useState(null);

  const fetchData = async () => {
    const response = await fetch(url);
    const result = await response.json();
    setData(result);
  };

  useEffect(() => {
    fetchData();
  }, [fetchData]); // fetchData 每次都是新函数

  return data;
}

// 解决方案：使用 useCallback
function useDataFetcher(url) {
  const [data, setData] = useState(null);

  const fetchData = useCallback(async () => {
    const response = await fetch(url);
    const result = await response.json();
    setData(result);
  }, [url]); // 稳定的依赖

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return data;
}

// 使用 useMemo 缓存计算结果
function useFilteredData(data, filterFn) {
  const filteredData = useMemo(() => {
    return data.filter(filterFn);
  }, [data, filterFn]);

  return filteredData;
}
```

### Hook 测试

```jsx
// Hook 测试工具
function renderHook(hook) {
  const result = { current: null };

  function TestComponent() {
    result.current = hook();
    return null;
  }

  render(<TestComponent />);
  return result;
}

// 测试 useToggle Hook
describe('useToggle', () => {
  it('should initialize with default value', () => {
    const { result } = renderHook(() => useToggle());
    expect(result.current.value).toBe(false);
  });

  it('should toggle value', () => {
    const { result } = renderHook(() => useToggle(true));

    act(() => {
      result.current.toggle();
    });

    expect(result.current.value).toBe(false);
  });

  it('should set value to true', () => {
    const { result } = renderHook(() => useToggle());

    act(() => {
      result.current.setTrue();
    });

    expect(result.current.value).toBe(true);
  });
});

// 测试异步 Hook
describe('useFetch', () => {
  beforeEach(() => {
    global.fetch = jest.fn(() =>
      Promise.resolve({
        ok: true,
        json: () => Promise.resolve({ data: 'test' })
      })
    );
  });

  afterEach(() => {
    jest.restoreAllMocks();
  });

  it('should fetch data successfully', async () => {
    const { result, waitForNextUpdate } = renderHook(() => useFetch('/api/data'));

    expect(result.current.loading).toBe(true);

    await waitForNextUpdate();

    expect(result.current.loading).toBe(false);
    expect(result.current.data).toEqual({ data: 'test' });
    expect(result.current.error).toBeNull();
  });
});
```

## 总结

React 自定义 Hook 是构建可重用、可测试逻辑的强大工具。通过合理使用自定义 Hook，可以显著提升代码质量和开发效率：

### 核心优势

1. **逻辑复用**：在组件间共享复杂的状态管理逻辑
2. **关注点分离**：将 UI 组件与业务逻辑分离
3. **可测试性**：独立测试业务逻辑而不依赖 UI
4. **代码组织**：更好地组织和管理复杂的状态
5. **团队协作**：提供统一的逻辑接口和规范

### 常见模式

1. **状态管理 Hook**：封装复杂的状态逻辑
2. **数据获取 Hook**：统一 API 调用和错误处理
3. **表单管理 Hook**：简化表单验证和提交
4. **性能优化 Hook**：防抖、节流、缓存等
5. **浏览器 API Hook**：封装原生浏览器功能

### 最佳实践

1. **命名规范**：始终以 "use" 开头
2. **单一职责**：每个 Hook 只负责一个功能
3. **正确依赖**：谨慎管理 Hook 的依赖数组
4. **性能考虑**：使用 useCallback、useMemo 优化性能
5. **错误处理**：妥善处理异步操作的错误
6. **清理副作用**：在 useEffect 中正确清理副作用
7. **类型安全**：在 TypeScript 项目中定义类型

### 适用场景

- 多个组件需要共享相同的状态逻辑
- 复杂的状态管理需要封装
- 需要统一处理副作用和清理
- 想要提升代码的可测试性和可维护性

通过掌握自定义 Hook，开发者可以构建更加模块化、可维护的 React 应用。自定义 Hook 体现了 React 的组合思想，是现代 React 开发的重要技能。