---
title: React 状态管理
published: 2024-10-31
description: '状态管理方案对比的详细介绍和学习笔记'
image: ''
tags: ["React","状态"]
category: '前端框架'
draft: false
lang: 'zh-CN'
---

## 概述

React 状态管理是前端开发中的核心概念，合理的状态管理能够使应用的数据流清晰、组件间通信高效、代码易于维护。本文将详细介绍 React 中各种状态管理方案，从基础的 useState 到高级的全局状态管理库，帮助开发者根据项目需求选择合适的状态管理策略。

## 核心概念

### 状态的定义

状态（State）是组件的数据模型，它决定了组件的渲染输出。状态具有以下特点：
- 是可变的：通过 `setState` 等方法更新
- 是私有的：只能在组件内部访问和修改
- 驱动渲染：状态变化会触发组件重新渲染

### 状态管理的层次

1. **组件内部状态**：使用 `useState` 管理组件私有的状态
2. **组件间状态共享**：通过 props 传递或 Context API
3. **全局状态管理**：使用 Redux、Zustand 等状态管理库
4. **服务端状态**：使用 React Query、SWR 等库管理服务端数据

### 状态管理原则

- **单一数据源**：每个状态应该有一个明确的来源
- **不可变性**：状态更新应该是不可变的，避免直接修改
- **最小化状态**：只存储必要的数据，衍生数据通过计算获得
- **就近原则**：状态应该放在最接近使用它的组件中

## 基本用法

### useState - 组件内部状态

`useState` 是最基础的状态管理 Hook：

```jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
      <button onClick={() => setCount(count - 1)}>
        Decrement
      </button>
      <button onClick={() => setCount(0)}>
        Reset
      </button>
    </div>
  );
}

// 复杂对象状态
function UserProfile() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });

  const updateField = (field, value) => {
    setUser(prev => ({
      ...prev,
      [field]: value
    }));
  };

  return (
    <div>
      <input
        type="text"
        value={user.name}
        onChange={(e) => updateField('name', e.target.value)}
        placeholder="Name"
      />
      <input
        type="email"
        value={user.email}
        onChange={(e) => updateField('email', e.target.value)}
        placeholder="Email"
      />
      <input
        type="number"
        value={user.age}
        onChange={(e) => updateField('age', parseInt(e.target.value) || 0)}
        placeholder="Age"
      />
      <pre>{JSON.stringify(user, null, 2)}</pre>
    </div>
  );
}

// 函数式初始化
function ExpensiveComponent() {
  const [data, setData] = useState(() => {
    // 只在组件首次渲染时执行一次
    console.log('Initializing expensive data...');
    return Array.from({ length: 1000 }, (_, i) => ({
      id: i,
      value: Math.random()
    }));
  });

  return <div>Data items: {data.length}</div>;
}
```

### useReducer - 复杂状态逻辑

当状态逻辑复杂时，使用 `useReducer` 更合适：

```jsx
import React, { useReducer } from 'react';

// 定义 action 类型
const ACTION_TYPES = {
  INCREMENT: 'INCREMENT',
  DECREMENT: 'DECREMENT',
  SET_VALUE: 'SET_VALUE',
  RESET: 'RESET'
};

// 定义 reducer
function counterReducer(state, action) {
  switch (action.type) {
    case ACTION_TYPES.INCREMENT:
      return { count: state.count + 1 };
    case ACTION_TYPES.DECREMENT:
      return { count: state.count - 1 };
    case ACTION_TYPES.SET_VALUE:
      return { count: action.payload };
    case ACTION_TYPES.RESET:
      return { count: 0 };
    default:
      return state;
  }
}

function CounterWithReducer() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: ACTION_TYPES.INCREMENT })}>
        Increment
      </button>
      <button onClick={() => dispatch({ type: ACTION_TYPES.DECREMENT })}>
        Decrement
      </button>
      <button onClick={() => dispatch({ type: ACTION_TYPES.SET_VALUE, payload: 10 })}>
        Set to 10
      </button>
      <button onClick={() => dispatch({ type: ACTION_TYPES.RESET })}>
        Reset
      </button>
    </div>
  );
}

// 复杂状态管理示例
function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: Date.now(),
            text: action.payload.text,
            completed: false
          }
        ]
      };

    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload.id
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };

    case 'DELETE_TODO':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload.id)
      };

    case 'SET_FILTER':
      return {
        ...state,
        filter: action.payload.filter
      };

    default:
      return state;
  }
}

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, {
    todos: [],
    filter: 'all' // all, active, completed
  });

  const [inputValue, setInputValue] = useState('');

  const handleAddTodo = (e) => {
    e.preventDefault();
    if (inputValue.trim()) {
      dispatch({ type: 'ADD_TODO', payload: { text: inputValue } });
      setInputValue('');
    }
  };

  const filteredTodos = state.todos.filter(todo => {
    if (state.filter === 'active') return !todo.completed;
    if (state.filter === 'completed') return todo.completed;
    return true;
  });

  return (
    <div>
      <form onSubmit={handleAddTodo}>
        <input
          type="text"
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          placeholder="Add a new todo"
        />
        <button type="submit">Add</button>
      </form>

      <div>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: { filter: 'all' } })}>
          All
        </button>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: { filter: 'active' } })}>
          Active
        </button>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: { filter: 'completed' } })}>
          Completed
        </button>
      </div>

      <ul>
        {filteredTodos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => dispatch({ type: 'TOGGLE_TODO', payload: { id: todo.id } })}
            />
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
              {todo.text}
            </span>
            <button onClick={() => dispatch({ type: 'DELETE_TODO', payload: { id: todo.id } })}>
              Delete
            </button>
          </li>
        ))}
      </ul>

      <p>Total: {state.todos.length} | Active: {state.todos.filter(t => !t.completed).length}</p>
    </div>
  );
}
```

### useContext - 跨组件状态共享

```jsx
import React, { useState, useContext, createContext } from 'react';

// 创建 Context
const ThemeContext = createContext();
const UserContext = createContext();

// Context Provider 组件
function AppProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const [user, setUser] = useState(null);

  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };

  const login = (userData) => {
    setUser(userData);
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      <UserContext.Provider value={{ user, login, logout }}>
        {children}
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}

// 自定义 Hook 简化 Context 使用
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

function useUser() {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within UserProvider');
  }
  return context;
}

// 使用 Context 的组件
function Header() {
  const { theme, toggleTheme } = useTheme();
  const { user, logout } = useUser();

  return (
    <header style={{
      background: theme === 'light' ? '#fff' : '#333',
      color: theme === 'light' ? '#333' : '#fff',
      padding: '1rem',
      display: 'flex',
      justifyContent: 'space-between',
      alignItems: 'center'
    }}>
      <h1>My App</h1>
      <div>
        <button onClick={toggleTheme}>
          Toggle Theme
        </button>
        {user ? (
          <>
            <span>Welcome, {user.name}</span>
            <button onClick={logout}>Logout</button>
          </>
        ) : (
          <button onClick={() => login({ name: 'John' })}>Login</button>
        )}
      </div>
    </header>
  );
}

function Dashboard() {
  const { theme } = useTheme();
  const { user } = useUser();

  if (!user) {
    return <p>Please login to view dashboard</p>;
  }

  return (
    <div style={{
      background: theme === 'light' ? '#f5f5f5' : '#222',
      color: theme === 'light' ? '#333' : '#fff',
      padding: '2rem'
    }}>
      <h2>Dashboard</h2>
      <p>Hello, {user.name}!</p>
    </div>
  );
}

// 应用组件
function App() {
  return (
    <AppProvider>
      <Header />
      <Dashboard />
    </AppProvider>
  );
}
```

## 实际应用

### Zustand - 轻量级状态管理

Zustand 是一个轻量级、简单的状态管理库：

```jsx
import { create } from 'zustand';

// 创建 store
const useStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
}));

// 组件中使用
function Counter() {
  const { count, increment, decrement, reset } = useStore();

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Increment</button>
      <button onClick={decrement}>Decrement</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

// 选择性订阅，避免不必要的重渲染
function CountDisplay() {
  const count = useStore((state) => state.count);
  return <p>Count: {count}</p>;
}

function IncrementButton() {
  const increment = useStore((state) => state.increment);
  return <button onClick={increment}>Increment</button>;
}

// 复杂 store 示例
const useTodoStore = create((set) => ({
  todos: [],
  filter: 'all',

  addTodo: (text) =>
    set((state) => ({
      todos: [
        ...state.todos,
        { id: Date.now(), text, completed: false }
      ]
    })),

  toggleTodo: (id) =>
    set((state) => ({
      todos: state.todos.map((todo) =>
        todo.id === id
          ? { ...todo, completed: !todo.completed }
          : todo
      )
    })),

  deleteTodo: (id) =>
    set((state) => ({
      todos: state.todos.filter((todo) => todo.id !== id)
    })),

  setFilter: (filter) => set({ filter }),

  // 计算属性
  get filteredTodos() {
    const state = useTodoStore.getState();
    return state.todos.filter((todo) => {
      if (state.filter === 'active') return !todo.completed;
      if (state.filter === 'completed') return todo.completed;
      return true;
    });
  }
}));

// 异步操作
const useAsyncStore = create((set) => ({
  users: [],
  loading: false,
  error: null,

  fetchUsers: async () => {
    set({ loading: true, error: null });
    try {
      const response = await fetch('https://api.example.com/users');
      const users = await response.json();
      set({ users, loading: false });
    } catch (error) {
      set({ error: error.message, loading: false });
    }
  }
}));

// 使用中间件
import { devtools, persist } from 'zustand/middleware';

const usePersistentStore = create(
  devtools(
    persist(
      (set) => ({
        user: null,
        login: (user) => set({ user }),
        logout: () => set({ user: null }),
      }),
      {
        name: 'app-storage', // localStorage key
      }
    )
  )
);
```

### Redux Toolkit - 企业级状态管理

Redux Toolkit 是官方推荐的 Redux 工具集：

```jsx
import { configureStore, createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import { Provider, useDispatch, useSelector } from 'react-redux';

// 创建 async thunk
export const fetchTodos = createAsyncThunk(
  'todos/fetchTodos',
  async () => {
    const response = await fetch('https://jsonplaceholder.typicode.com/todos');
    return response.json();
  }
);

// 创建 slice
const todosSlice = createSlice({
  name: 'todos',
  initialState: {
    items: [],
    loading: false,
    error: null,
  },
  reducers: {
    addTodo: (state, action) => {
      state.items.push({
        id: Date.now(),
        text: action.payload,
        completed: false,
      });
    },
    toggleTodo: (state, action) => {
      const todo = state.items.find(todo => todo.id === action.payload);
      if (todo) {
        todo.completed = !todo.completed;
      }
    },
    deleteTodo: (state, action) => {
      state.items = state.items.filter(todo => todo.id !== action.payload);
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchTodos.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchTodos.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchTodos.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  },
});

export const { addTodo, toggleTodo, deleteTodo } = todosSlice.actions;

// 配置 store
const store = configureStore({
  reducer: {
    todos: todosSlice.reducer,
  },
});

// 组件中使用
function TodoList() {
  const dispatch = useDispatch();
  const { items, loading, error } = useSelector((state) => state.todos);

  useEffect(() => {
    dispatch(fetchTodos());
  }, [dispatch]);

  const handleAddTodo = (text) => {
    dispatch(addTodo(text));
  };

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <AddTodoForm onAdd={handleAddTodo} />
      <ul>
        {items.map((todo) => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => dispatch(toggleTodo(todo.id))}
            />
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
              {todo.text}
            </span>
            <button onClick={() => dispatch(deleteTodo(todo.id))}>
              Delete
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}

// Provider 设置
function App() {
  return (
    <Provider store={store}>
      <TodoList />
    </Provider>
  );
}
```

### React Query - 服务端状态管理

React Query 专注于服务端状态管理：

```jsx
import { QueryClient, QueryClientProvider, useQuery, useMutation, useQueryClient } from 'react-query';

// 创建 Query Client
const queryClient = new QueryClient();

// 获取数据 hook
function useTodos() {
  return useQuery(
    'todos',
    async () => {
      const response = await fetch('https://jsonplaceholder.typicode.com/todos');
      return response.json();
    },
    {
      staleTime: 5 * 60 * 1000, // 5 分钟内数据视为新鲜
      cacheTime: 10 * 60 * 1000, // 10 分钟后清除缓存
      refetchOnWindowFocus: false, // 窗口聚焦时不自动重新获取
    }
  );
}

// 修改数据 hook
function useAddTodo() {
  const queryClient = useQueryClient();

  return useMutation(
    async (newTodo) => {
      const response = await fetch('https://jsonplaceholder.typicode.com/todos', {
        method: 'POST',
        body: JSON.stringify(newTodo),
        headers: {
          'Content-Type': 'application/json',
        },
      });
      return response.json();
    },
    {
      onSuccess: () => {
        // 成功后重新获取数据
        queryClient.invalidateQueries('todos');
      },
    }
  );
}

// 组件中使用
function TodoApp() {
  const { data: todos, isLoading, error } = useTodos();
  const addTodo = useAddTodo();

  const handleAddTodo = (text) => {
    addTodo.mutate({
      title: text,
      completed: false,
      userId: 1,
    });
  };

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error loading todos</div>;

  return (
    <div>
      <AddTodoForm onAdd={handleAddTodo} />
      <ul>
        {todos?.slice(0, 10).map((todo) => (
          <li key={todo.id}>
            <input type="checkbox" checked={todo.completed} readOnly />
            <span>{todo.title}</span>
          </li>
        ))}
      </ul>
    </div>
  );
}

// Provider 设置
function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <TodoApp />
    </QueryClientProvider>
  );
}
```

### Jotai - 原子化状态管理

Jotai 提供了更细粒度的状态管理：

```jsx
import { atom, useAtom, useAtomValue, useSetAtom } from 'jotai';

// 创建原子状态
const countAtom = atom(0);
const nameAtom = atom('John');

// 只读原子
const doubledCountAtom = atom((get) => get(countAtom) * 2);

// 可写原子
const incrementAtom = atom(null, (get, set) => {
  set(countAtom, get(countAtom) + 1);
});

// 异步原子
const userAtom = atom(async (get) => {
  const response = await fetch('https://api.example.com/user');
  return response.json();
});

// 组件中使用
function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const doubledCount = useAtomValue(doubledCountAtom);
  const increment = useSetAtom(incrementAtom);

  return (
    <div>
      <p>Count: {count}</p>
      <p>Doubled: {doubledCount}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={increment}>Increment via Atom</button>
    </div>
  );
}

function UserName() {
  const [name, setName] = useAtom(nameAtom);

  return (
    <div>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Enter your name"
      />
      <p>Hello, {name}!</p>
    </div>
  );
}
```

## 注意事项

### 状态粒度设计

```jsx
// 糟糕的设计：状态过于分散
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [email, setEmail] = useState('');
const [phone, setPhone] = useState('');

// 良好的设计：相关状态集中
const [userProfile, setUserProfile] = useState({
  firstName: '',
  lastName: '',
  email: '',
  phone: ''
});

// 糟糕的设计：状态过于集中
const [appState, setAppState] = useState({
  user: null,
  theme: 'light',
  notifications: [],
  cart: [],
  preferences: {}
});

// 良好的设计：按职责拆分
const [user, setUser] = useState(null);
const [theme, setTheme] = useState('light');
const [notifications, setNotifications] = useState([]);
```

### 避免直接修改状态

```jsx
// 错误：直接修改数组
const [items, setItems] = useState([1, 2, 3]);
items.push(4); // 错误！
setItems(items); // 错误！

// 正确：创建新数组
setItems([...items, 4]);

// 错误：直接修改对象
const [user, setUser] = useState({ name: 'John', age: 30 });
user.age = 31; // 错误！
setUser(user); // 错误！

// 正确：创建新对象
setUser({ ...user, age: 31 });

// 复杂嵌套对象的正确更新
const [data, setData] = useState({
  users: [
    { id: 1, name: 'John', posts: [{ id: 1, title: 'Hello' }] }
  ]
});

// 正确更新嵌套属性
setData(prev => ({
  ...prev,
  users: prev.users.map(user =>
    user.id === 1
      ? { ...user, posts: [...user.posts, { id: 2, title: 'World' }] }
      : user
  )
}));
```

### 性能考虑

```jsx
// 问题：大对象作为 Context value 导致所有消费者重渲染
const AppContext = createContext();

function AppProvider({ children }) {
  const [state, setState] = useState({
    user: null,
    theme: 'light',
    notifications: [],
    cart: [],
    // ...很多其他状态
  });

  return (
    <AppContext.Provider value={{ state, setState }}>
      {children}
    </AppContext.Provider>
  );
}

// 解决方案：拆分 Context
const UserContext = createContext();
const ThemeContext = createContext();
const NotificationContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [notifications, setNotifications] = useState([]);

  return (
    <UserContext.Provider value={{ user, setUser }}>
      <ThemeContext.Provider value={{ theme, setTheme }}>
        <NotificationContext.Provider value={{ notifications, setNotifications }}>
          {children}
        </NotificationContext.Provider>
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}
```

### 异步状态处理

```jsx
// 糟糕的异步状态处理
function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    fetchUser()
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, []);

  // 处理竞态条件、清理等变得复杂
}

// 使用自定义 Hook 封装
function useAsyncData(asyncFn, dependencies = []) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    const fetchData = async () => {
      setLoading(true);
      setError(null);

      try {
        const result = await asyncFn();
        if (!cancelled) {
          setData(result);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    };

    fetchData();

    return () => {
      cancelled = true;
    };
  }, dependencies);

  return { data, loading, error };
}

// 使用封装的 Hook
function UserProfile() {
  const { data: user, loading, error } = useAsyncData(fetchUser, []);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return <div>Hello, {user?.name}</div>;
}
```

## 总结

React 状态管理是一个多层次的话题，需要根据项目规模和复杂度选择合适的方案：

### 状态管理方案选择指南

1. **小型项目**（1-5个页面）
   - 使用 `useState` 和 `useContext`
   - 避免引入额外的依赖

2. **中型项目**（5-20个页面）
   - 使用 Zustand 或 Jotai
   - 结合 React Query 处理服务端状态
   - 保持简单和灵活

3. **大型项目**（20+页面，团队开发）
   - 使用 Redux Toolkit
   - 集成 React Query/SWR
   - 建立清晰的状态管理架构

### 最佳实践

1. **状态设计原则**
   - 最小化状态，只存储必要数据
   - 相关状态集中管理
   - 避免状态冗余和重复

2. **性能优化**
   - 合理使用 memoization
   - 拆分 Context 避免不必要的重渲染
   - 选择性订阅状态变化

3. **类型安全**
   - 使用 TypeScript 定义状态类型
   - 为状态创建严格的类型约束
   - 利用类型系统捕获错误

4. **测试友好**
   - 状态管理逻辑与 UI 组件分离
   - 编写可测试的状态管理代码
   - 使用依赖注入便于测试

5. **开发体验**
   - 提供良好的开发工具支持
   - 保持代码的可读性和可维护性
   - 建立团队统一的代码规范

通过合理选择和应用状态管理方案，可以构建出高性能、可维护的 React 应用。记住，没有最好的方案，只有最适合的方案，要根据具体项目需求做出明智的选择。