---
title: React Redux 入门
published: 2023-09-27
description: 'Redux 基础概念的详细介绍和学习笔记'
image: ''
tags: ["React","Redux"]
category: 'React'
draft: false
lang: 'zh-CN'
---

## 概述

Redux 是一个可预测的状态容器，专门为 JavaScript 应用设计。它提供了集中式存储来管理应用的所有状态，并确保状态更新的可预测性。React Redux 是 Redux 的官方 React 绑定库，提供了无缝的 React 集成。

Redux Toolkit (RTK) 是官方推荐的 Redux 工具集，简化了 Redux 的使用，包含了最佳实践和常用配置，大大降低了 Redux 的学习曲线。

## 核心概念

### 单一数据源

整个应用的 state 存储在一个单一的 store 中：
```javascript
const store = {
  user: {
    isAuthenticated: false,
    profile: null
  },
  todos: [],
  theme: 'light',
  notifications: []
};
```

### State 是只读的

修改 state 的唯一方式是触发 action：
```javascript
// action 是一个普通对象
const action = {
  type: 'ADD_TODO',
  payload: {
    id: 1,
    text: 'Learn Redux',
    completed: false
  }
};
```

### 纯函数修改 State

使用纯函数 reducer 来描述 state 如何变化：
```javascript
function todosReducer(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.payload];
    default:
      return state;
  }
}
```

### 数据流

Redux 遵循单向数据流：
1. 用户交互触发 action
2. Store 分发 action 到所有 reducer
3. Reducer 计算新的 state
4. Store 通知所有订阅者 state 变化
5. UI 组件重新渲染

## 基本用法

### Redux Toolkit 基础设置

```javascript
// store.js
import { configureStore } from '@reduxjs/toolkit';

// 创建 slice
const counterSlice = createSlice({
  name: 'counter',
  initialState: {
    value: 0
  },
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    },
    reset: (state) => {
      state.value = 0;
    }
  }
});

// 导出 actions
export const { increment, decrement, incrementByAmount, reset } = counterSlice.actions;

// 导出 reducer
export default counterSlice.reducer;

// 配置 store
const store = configureStore({
  reducer: {
    counter: counterSlice.reducer
  },
  devTools: process.env.NODE_ENV !== 'production'
});

export default store;
```

### React 组件中使用 Redux

```jsx
// Counter.jsx
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement, incrementByAmount, reset } from './store';

function Counter() {
  // 读取 state
  const count = useSelector((state) => state.counter.value);

  // 获取 dispatch 函数
  const dispatch = useDispatch();

  return (
    <div>
      <h1>Counter: {count}</h1>

      <button onClick={() => dispatch(increment())}>
        Increment
      </button>

      <button onClick={() => dispatch(decrement())}>
        Decrement
      </button>

      <button onClick={() => dispatch(incrementByAmount(5))}>
        Increment by 5
      </button>

      <button onClick={() => dispatch(reset())}>
        Reset
      </button>
    </div>
  );
}

// App.jsx
import { Provider } from 'react-redux';
import store from './store';
import Counter from './Counter';

function App() {
  return (
    <Provider store={store}>
      <Counter />
    </Provider>
  );
}

export default App;
```

### 异步操作处理

```javascript
// asyncSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// 异步 thunk
export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async (_, { rejectWithValue }) => {
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/users');
      if (!response.ok) {
        throw new Error('Failed to fetch users');
      }
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

export const createUser = createAsyncThunk(
  'users/createUser',
  async (userData, { rejectWithValue }) => {
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/users', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(userData)
      });

      if (!response.ok) {
        throw new Error('Failed to create user');
      }

      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

const usersSlice = createSlice({
  name: 'users',
  initialState: {
    data: [],
    loading: false,
    error: null,
    selectedUser: null
  },
  reducers: {
    selectUser: (state, action) => {
      state.selectedUser = state.data.find(user => user.id === action.payload);
    },
    clearSelectedUser: (state) => {
      state.selectedUser = null;
    }
  },
  extraReducers: (builder) => {
    builder
      // fetchUsers
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.loading = false;
        state.data = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      })

      // createUser
      .addCase(createUser.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(createUser.fulfilled, (state, action) => {
        state.loading = false;
        state.data.push(action.payload);
      })
      .addCase(createUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.payload;
      });
  }
});

export const { selectUser, clearSelectedUser } = usersSlice.actions;
export default usersSlice.reducer;
```

### Redux 选择器优化

```javascript
// selectors.js
import { createSelector } from '@reduxjs/toolkit';

// 基础选择器
export const selectUsers = (state) => state.users.data;
export const selectUsersLoading = (state) => state.users.loading;
export const selectUsersError = (state) => state.users.error;
export const selectSelectedUser = (state) => state.users.selectedUser;

// 记忆化选择器
export const selectActiveUsers = createSelector(
  [selectUsers],
  (users) => users.filter(user => user.active)
);

export const selectUserById = createSelector(
  [selectUsers, (state, userId) => userId],
  (users, userId) => users.find(user => user.id === userId)
);

export const selectUsersByDepartment = createSelector(
  [selectUsers, (state, department) => department],
  (users, department) => users.filter(user => user.department === department)
);

export const selectUserStatistics = createSelector(
  [selectUsers],
  (users) => ({
    total: users.length,
    active: users.filter(user => user.active).length,
    inactive: users.filter(user => !user.active).length,
    departments: [...new Set(users.map(user => user.department))].length
  })
);

// 在组件中使用
function UsersList() {
  const users = useSelector(selectUsers);
  const activeUsers = useSelector(selectActiveUsers);
  const statistics = useSelector(selectUserStatistics);

  return (
    <div>
      <h2>User Statistics</h2>
      <ul>
        <li>Total: {statistics.total}</li>
        <li>Active: {statistics.active}</li>
        <li>Inactive: {statistics.inactive}</li>
        <li>Departments: {statistics.departments}</li>
      </ul>

      <h2>Active Users</h2>
      <ul>
        {activeUsers.map(user => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

## 实际应用

### Todo 应用完整实现

```javascript
// todoSlice.js
import { createSlice } from '@reduxjs/toolkit';

const initialState = {
  items: [],
  filter: 'all', // all, active, completed
  searchTerm: ''
};

const todoSlice = createSlice({
  name: 'todos',
  initialState,
  reducers: {
    addTodo: {
      reducer: (state, action) => {
        state.items.unshift(action.payload);
      },
      prepare: (text) => ({
        payload: {
          id: Date.now(),
          text,
          completed: false,
          createdAt: new Date().toISOString()
        }
      })
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
    editTodo: (state, action) => {
      const { id, text } = action.payload;
      const todo = state.items.find(todo => todo.id === id);
      if (todo) {
        todo.text = text;
        todo.updatedAt = new Date().toISOString();
      }
    },
    clearCompleted: (state) => {
      state.items = state.items.filter(todo => !todo.completed);
    },
    setFilter: (state, action) => {
      state.filter = action.payload;
    },
    setSearchTerm: (state, action) => {
      state.searchTerm = action.payload;
    },
    toggleAll: (state) => {
      const allCompleted = state.items.every(todo => todo.completed);
      state.items.forEach(todo => {
        todo.completed = !allCompleted;
      });
    },
    reorderTodos: (state, action) => {
      const { sourceIndex, destinationIndex } = action.payload;
      const [removed] = state.items.splice(sourceIndex, 1);
      state.items.splice(destinationIndex, 0, removed);
    }
  }
});

export const {
  addTodo,
  toggleTodo,
  deleteTodo,
  editTodo,
  clearCompleted,
  setFilter,
  setSearchTerm,
  toggleAll,
  reorderTodos
} = todoSlice.actions;

export default todoSlice.reducer;

// todoSelectors.js
import { createSelector } from '@reduxjs/toolkit';

export const selectTodos = (state) => state.todos.items;
export const selectFilter = (state) => state.todos.filter;
export const selectSearchTerm = (state) => state.todos.searchTerm;

export const selectFilteredTodos = createSelector(
  [selectTodos, selectFilter, selectSearchTerm],
  (todos, filter, searchTerm) => {
    let filtered = todos;

    // 应用搜索过滤
    if (searchTerm) {
      filtered = filtered.filter(todo =>
        todo.text.toLowerCase().includes(searchTerm.toLowerCase())
      );
    }

    // 应用状态过滤
    switch (filter) {
      case 'active':
        return filtered.filter(todo => !todo.completed);
      case 'completed':
        return filtered.filter(todo => todo.completed);
      default:
        return filtered;
    }
  }
);

export const selectTodoStats = createSelector(
  [selectTodos],
  (todos) => ({
    total: todos.length,
    active: todos.filter(todo => !todo.completed).length,
    completed: todos.filter(todo => todo.completed).length
  })
);
```

```jsx
// TodoApp.jsx
import React, { useState } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import {
  addTodo,
  toggleTodo,
  deleteTodo,
  editTodo,
  clearCompleted,
  setFilter,
  setSearchTerm,
  toggleAll
} from './todoSlice';
import { selectFilteredTodos, selectTodoStats } from './todoSelectors';

function TodoApp() {
  const dispatch = useDispatch();
  const [inputValue, setInputValue] = useState('');
  const [editingId, setEditingId] = useState(null);
  const [editingText, setEditingText] = useState('');

  const todos = useSelector(selectFilteredTodos);
  const stats = useSelector(selectTodoStats);
  const filter = useSelector((state) => state.todos.filter);
  const searchTerm = useSelector((state) => state.todos.searchTerm);

  const handleAddTodo = (e) => {
    e.preventDefault();
    if (inputValue.trim()) {
      dispatch(addTodo(inputValue.trim()));
      setInputValue('');
    }
  };

  const handleToggleTodo = (id) => {
    dispatch(toggleTodo(id));
  };

  const handleDeleteTodo = (id) => {
    dispatch(deleteTodo(id));
  };

  const handleStartEdit = (todo) => {
    setEditingId(todo.id);
    setEditingText(todo.text);
  };

  const handleSaveEdit = (id) => {
    if (editingText.trim()) {
      dispatch(editTodo({ id, text: editingText.trim() }));
    }
    setEditingId(null);
    setEditingText('');
  };

  const handleCancelEdit = () => {
    setEditingId(null);
    setEditingText('');
  };

  const handleClearCompleted = () => {
    dispatch(clearCompleted());
  };

  const handleToggleAll = () => {
    dispatch(toggleAll());
  };

  return (
    <div className="todo-app">
      <h1>Todo App</h1>

      {/* 添加 Todo 表单 */}
      <form onSubmit={handleAddTodo} className="todo-form">
        <input
          type="text"
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          placeholder="What needs to be done?"
          className="todo-input"
        />
        <button type="submit" className="add-button">
          Add Todo
        </button>
      </form>

      {/* 搜索和过滤 */}
      <div className="todo-controls">
        <input
          type="text"
          value={searchTerm}
          onChange={(e) => dispatch(setSearchTerm(e.target.value))}
          placeholder="Search todos..."
          className="search-input"
        />

        <div className="filter-buttons">
          {['all', 'active', 'completed'].map(filterType => (
            <button
              key={filterType}
              onClick={() => dispatch(setFilter(filterType))}
              className={filter === filterType ? 'active' : ''}
            >
              {filterType.charAt(0).toUpperCase() + filterType.slice(1)}
            </button>
          ))}
        </div>
      </div>

      {/* Todo 列表 */}
      <ul className="todo-list">
        {todos.map(todo => (
          <li key={todo.id} className={`todo-item ${todo.completed ? 'completed' : ''}`}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => handleToggleTodo(todo.id)}
              className="todo-checkbox"
            />

            {editingId === todo.id ? (
              <div className="edit-form">
                <input
                  type="text"
                  value={editingText}
                  onChange={(e) => setEditingText(e.target.value)}
                  className="edit-input"
                />
                <button onClick={() => handleSaveEdit(todo.id)}>Save</button>
                <button onClick={handleCancelEdit}>Cancel</button>
              </div>
            ) : (
              <span className="todo-text" onDoubleClick={() => handleStartEdit(todo)}>
                {todo.text}
              </span>
            )}

            <div className="todo-actions">
              <button onClick={() => handleStartEdit(todo)}>Edit</button>
              <button onClick={() => handleDeleteTodo(todo.id)}>Delete</button>
            </div>
          </li>
        ))}
      </ul>

      {/* 统计和批量操作 */}
      {todos.length > 0 && (
        <div className="todo-footer">
          <div className="stats">
            <span>Total: {stats.total}</span>
            <span>Active: {stats.active}</span>
            <span>Completed: {stats.completed}</span>
          </div>

          <div className="bulk-actions">
            <button onClick={handleToggleAll}>
              Toggle All
            </button>
            {stats.completed > 0 && (
              <button onClick={handleClearCompleted}>
                Clear Completed
              </button>
            )}
          </div>
        </div>
      )}
    </div>
  );
}

export default TodoApp;
```

### 完整的 Store 配置

```javascript
// store.js
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';
import usersReducer from './usersSlice';
import todosReducer from './todoSlice';
import uiReducer from './uiSlice';

// 配置 store
const store = configureStore({
  reducer: {
    counter: counterReducer,
    users: usersReducer,
    todos: todosReducer,
    ui: uiReducer
  },
  // 中间件配置
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['users/fetchUsers/fulfilled'],
        ignoredPaths: ['users.data']
      }
    }),
  // 开发工具配置
  devTools: process.env.NODE_ENV !== 'production',
  // 预加载状态（SSR）
  preloadedState: {},
  // Redux Enhancer
  enhancers: (defaultEnhancers) =>
    process.env.NODE_ENV === 'production'
      ? defaultEnhancers
      : [...defaultEnhancers]
});

export default store;

// uiSlice.js - UI 状态管理
import { createSlice } from '@reduxjs/toolkit';

const uiSlice = createSlice({
  name: 'ui',
  initialState: {
    theme: 'light',
    sidebarOpen: true,
    notifications: [],
    loading: false,
    modal: null
  },
  reducers: {
    toggleTheme: (state) => {
      state.theme = state.theme === 'light' ? 'dark' : 'light';
    },
    setTheme: (state, action) => {
      state.theme = action.payload;
    },
    toggleSidebar: (state) => {
      state.sidebarOpen = !state.sidebarOpen;
    },
    setSidebarOpen: (state, action) => {
      state.sidebarOpen = action.payload;
    },
    addNotification: (state, action) => {
      state.notifications.unshift({
        id: Date.now(),
        ...action.payload,
        timestamp: new Date().toISOString()
      });
    },
    removeNotification: (state, action) => {
      state.notifications = state.notifications.filter(
        notification => notification.id !== action.payload
      );
    },
    clearNotifications: (state) => {
      state.notifications = [];
    },
    setLoading: (state, action) => {
      state.loading = action.payload;
    },
    openModal: (state, action) => {
      state.modal = {
        type: action.payload.type,
        data: action.payload.data || {}
      };
    },
    closeModal: (state) => {
      state.modal = null;
    }
  }
});

export const {
  toggleTheme,
  setTheme,
  toggleSidebar,
  setSidebarOpen,
  addNotification,
  removeNotification,
  clearNotifications,
  setLoading,
  openModal,
  closeModal
} = uiSlice.actions;

export default uiSlice.reducer;
```

### 自定义 Hooks 封装

```javascript
// hooks.js
import { useSelector, useDispatch } from 'react-redux';
import { useCallback, useEffect } from 'react';

// Counter Hooks
export function useCounter() {
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();

  const increment = useCallback(() => dispatch({ type: 'counter/increment' }), [dispatch]);
  const decrement = useCallback(() => dispatch({ type: 'counter/decrement' }), [dispatch]);

  return { count, increment, decrement };
}

// Users Hooks
export function useUsers() {
  const { data, loading, error } = useSelector((state) => state.users);
  const dispatch = useDispatch();

  const fetchUsers = useCallback(() => {
    dispatch({ type: 'users/fetchUsers' });
  }, [dispatch]);

  const createUser = useCallback((userData) => {
    dispatch({ type: 'users/createUser', payload: userData });
  }, [dispatch]);

  const selectUser = useCallback((userId) => {
    dispatch({ type: 'users/selectUser', payload: userId });
  }, [dispatch]);

  return {
    users: data,
    loading,
    error,
    fetchUsers,
    createUser,
    selectUser
  };
}

// 自动获取数据的 Hook
export function useFetchUsers() {
  const { users, loading, error, fetchUsers } = useUsers();

  useEffect(() => {
    if (users.length === 0 && !loading && !error) {
      fetchUsers();
    }
  }, [users, loading, error, fetchUsers]);

  return { users, loading, error, refetch: fetchUsers };
}

// Todo Hooks
export function useTodos() {
  const todos = useSelector((state) => state.todos.items);
  const filter = useSelector((state) => state.todos.filter);
  const searchTerm = useSelector((state) => state.todos.searchTerm);
  const dispatch = useDispatch();

  const addTodo = useCallback((text) => {
    dispatch({ type: 'todos/addTodo', payload: text });
  }, [dispatch]);

  const toggleTodo = useCallback((id) => {
    dispatch({ type: 'todos/toggleTodo', payload: id });
  }, [dispatch]);

  const deleteTodo = useCallback((id) => {
    dispatch({ type: 'todos/deleteTodo', payload: id });
  }, [dispatch]);

  const setFilter = useCallback((filter) => {
    dispatch({ type: 'todos/setFilter', payload: filter });
  }, [dispatch]);

  const setSearchTerm = useCallback((term) => {
    dispatch({ type: 'todos/setSearchTerm', payload: term });
  }, [dispatch]);

  return {
    todos,
    filter,
    searchTerm,
    addTodo,
    toggleTodo,
    deleteTodo,
    setFilter,
    setSearchTerm
  };
}

// UI Hooks
export function useUI() {
  const { theme, sidebarOpen, notifications, loading, modal } = useSelector((state) => state.ui);
  const dispatch = useDispatch();

  const toggleTheme = useCallback(() => {
    dispatch({ type: 'ui/toggleTheme' });
  }, [dispatch]);

  const setTheme = useCallback((theme) => {
    dispatch({ type: 'ui/setTheme', payload: theme });
  }, [dispatch]);

  const toggleSidebar = useCallback(() => {
    dispatch({ type: 'ui/toggleSidebar' });
  }, [dispatch]);

  const addNotification = useCallback((notification) => {
    dispatch({ type: 'ui/addNotification', payload: notification });
  }, [dispatch]);

  const removeNotification = useCallback((id) => {
    dispatch({ type: 'ui/removeNotification', payload: id });
  }, [dispatch]);

  const openModal = useCallback((type, data = {}) => {
    dispatch({ type: 'ui/openModal', payload: { type, data } });
  }, [dispatch]);

  const closeModal = useCallback(() => {
    dispatch({ type: 'ui/closeModal' });
  }, [dispatch]);

  return {
    theme,
    sidebarOpen,
    notifications,
    loading,
    modal,
    toggleTheme,
    setTheme,
    toggleSidebar,
    addNotification,
    removeNotification,
    openModal,
    closeModal
  };
}
```

## 注意事项

### 避免过度使用 Redux

```javascript
// 问题：将所有状态都放在 Redux 中
const store = {
  // 这些应该放在组件本地状态中
  inputValue: '',
  isModalOpen: false,
  selectedOption: null,
  // 只有真正需要共享的状态才应该放在 Redux 中
  user: null,
  todos: []
};

// 解决方案：合理区分状态
function MyComponent() {
  // 本地状态
  const [inputValue, setInputValue] = useState('');
  const [isModalOpen, setIsModalOpen] = useState(false);

  // Redux 状态
  const user = useSelector(state => state.user);
  const todos = useSelector(state => state.todos);

  // ...
}
```

### 正确处理异步操作

```javascript
// 错误示例：在 reducer 中直接调用 API
function usersReducer(state, action) {
  switch (action.type) {
    case 'FETCH_USERS':
      fetch('/api/users').then(response => {
        // 不能在这里处理响应
      });
      return state;
  }
}

// 正确示例：使用 createAsyncThunk
export const fetchUsers = createAsyncThunk(
  'users/fetchUsers',
  async () => {
    const response = await fetch('/api/users');
    return response.json();
  }
);

const usersSlice = createSlice({
  name: 'users',
  initialState: {
    data: [],
    loading: false,
    error: null
  },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
        state.error = null;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.loading = false;
        state.data = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  }
});
```

### 性能优化

```javascript
// 问题：每次都创建新的选择器函数
function MyComponent() {
  const expensiveValue = useSelector((state) => {
    // 每次渲染都会创建新函数
    return state.items.filter(item => item.active).length;
  });
}

// 解决方案：使用记忆化选择器
import { createSelector } from '@reduxjs/toolkit';

const selectActiveItems = createSelector(
  [(state) => state.items],
  (items) => items.filter(item => item.active)
);

const selectActiveItemsCount = createSelector(
  [selectActiveItems],
  (activeItems) => activeItems.length
);

function MyComponent() {
  const activeItemsCount = useSelector(selectActiveItemsCount);
  // ...
}

// 批量更新减少渲染
import { batch } from 'react-redux';

function handleMultipleUpdates() {
  batch(() => {
    dispatch(action1());
    dispatch(action2());
    dispatch(action3());
  });
}
```

### TypeScript 类型定义

```typescript
// types.ts
export interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user';
}

export interface Todo {
  id: number;
  text: string;
  completed: boolean;
  createdAt: string;
}

export interface RootState {
  counter: CounterState;
  users: UsersState;
  todos: TodosState;
  ui: UIState;
}

// counterSlice.ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface CounterState {
  value: number;
}

const initialState: CounterState = {
  value: 0
};

const counterSlice = createSlice({
  name: 'counter',
  initialState,
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

export const { increment, decrement, incrementByAmount } = counterSlice.actions;
export default counterSlice.reducer;

// hooks.ts
import { useSelector, useDispatch } from 'react-redux';
import type { RootState } from './types';
import type { AppDispatch } from './store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: <T>(selector: (state: RootState) => T) => T = useSelector;

function Counter() {
  const count = useAppSelector((state) => state.counter.value);
  const dispatch = useAppDispatch();

  return <div onClick={() => dispatch(increment())}>{count}</div>;
}
```

## 总结

React Redux 是构建大型应用的强大工具，通过掌握其核心概念和最佳实践，可以构建可维护、可预测的状态管理系统：

### 核心优势

1. **可预测性**：单向数据流和纯函数确保状态更新的可预测性
2. **可调试性**：Redux DevTools 提供强大的调试能力
3. **可维护性**：清晰的状态结构和统一的更新模式
4. **可扩展性**：中间件生态系统丰富，易于扩展功能

### 最佳实践

1. **使用 Redux Toolkit**：简化开发流程，内置最佳实践
2. **合理拆分 Slice**：按功能模块组织状态和逻辑
3. **优化选择器**：使用记忆化选择器提升性能
4. **谨慎使用状态**：只在必要时使用 Redux，避免过度工程化
5. **类型安全**：在 TypeScript 项目中严格定义类型

### 适用场景

- 多个组件需要访问相同状态
- 状态更新逻辑复杂
- 需要时间旅行调试
- 中大型应用开发
- 团队协作项目

通过合理应用 React Redux，可以构建出结构清晰、易于维护、性能优异的现代 React 应用。Redux Toolkit 的出现大大降低了学习成本，让开发者能够专注于业务逻辑而不是配置细节。