---
title: React useContext
published: 2023-08-04
description: '深入理解 React useContext Hook：跨组件状态管理和数据传递'
image: ''
tags: ["React","Hooks"]
category: 'React'
draft: false
lang: 'zh-CN'
---

## 概述

useContext 是 React 中用于在组件树中共享数据的 Hook。它提供了一种无需手动传递 props 就能在组件层级中传递数据的方法，有效解决了"prop drilling"（属性穿透）问题。Context API 是 React 官方推荐的全局状态管理方案之一，特别适合跨多层级组件的数据共享场景。

### 为什么需要 useContext

在没有 Context API 的情况下，数据需要在组件树中逐层传递：

```jsx
// ❌ Prop drilling 问题
function App() {
  const user = { name: 'John', role: 'admin' };
  return <Header user={user} />;
}

function Header({ user }) {
  return <Navigation user={user} />;
}

function Navigation({ user }) {
  return <UserProfile user={user} />;
}

function UserProfile({ user }) {
  return <div>{user.name}</div>;
}
```

使用 useContext 可以避免这种逐层传递：

```jsx
// ✅ 使用 Context API
const UserContext = createContext();

function App() {
  const user = { name: 'John', role: 'admin' };
  return (
    <UserContext.Provider value={user}>
      <Header />
    </UserContext.Provider>
  );
}

function Header() {
  return <Navigation />;
}

function Navigation() {
  return <UserProfile />;
}

function UserProfile() {
  const user = useContext(UserContext);
  return <div>{user.name}</div>;
}
```

## 核心概念

### Context API 的组成部分

1. **createContext**：创建 Context 对象
2. **Provider**：向组件树提供数据
3. **Consumer**：消费 Context 数据（或使用 useContext Hook）
4. **useContext**：在函数组件中消费 Context 数据

### Context 的基本结构

```jsx
// 1. 创建 Context
const MyContext = createContext(defaultValue);

// 2. 提供 Context 数据
<MyContext.Provider value={/* 数据 */}>
  {/* 子组件 */}
</MyContext.Provider>

// 3. 消费 Context 数据
const value = useContext(MyContext);
```

### 默认值的作用

当组件没有被 Provider 包裹时，会使用默认值：

```jsx
const ThemeContext = createContext('light'); // 默认值为 'light'

function Button() {
  const theme = useContext(ThemeContext);
  // 如果没有 Provider，theme 将是 'light'
  return <button className={theme}>Click me</button>;
}
```

## 基本用法

### 创建和使用 Context

```jsx
import { createContext, useContext } from 'react';

// 1. 创建 Context
const ThemeContext = createContext();

// 2. 创建 Provider 组件
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  const value = {
    theme,
    toggleTheme
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. 在组件中使用 Context
function ThemeButton() {
  const { theme, toggleTheme } = useContext(ThemeContext);
  
  return (
    <button
      onClick={toggleTheme}
      style={{
        background: theme === 'light' ? '#fff' : '#333',
        color: theme === 'light' ? '#333' : '#fff'
      }}
    >
      当前主题: {theme}
    </button>
  );
}

// 4. 在应用中使用
function App() {
  return (
    <ThemeProvider>
      <ThemeButton />
    </ThemeProvider>
  );
}
```

### 多个 Context 组合

```jsx
const ThemeContext = createContext();
const UserContext = createContext();

function App() {
  const [theme, setTheme] = useState('light');
  const [user, setUser] = useState({ name: 'John', role: 'user' });
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <UserContext.Provider value={{ user, setUser }}>
        <Dashboard />
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}

function Dashboard() {
  const { theme } = useContext(ThemeContext);
  const { user } = useContext(UserContext);
  
  return (
    <div className={theme}>
      <h1>欢迎, {user.name}</h1>
      <Settings />
    </div>
  );
}

function Settings() {
  const { theme, setTheme } = useContext(ThemeContext);
  const { user, setUser } = useContext(UserContext);
  
  return (
    <div>
      <p>当前主题: {theme}</p>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        切换主题
      </button>
      
      <p>用户角色: {user.role}</p>
      <button onClick={() => setUser({ ...user, role: 'admin' })}>
        升级为管理员
      </button>
    </div>
  );
}
```

### 自定义 Context Hook

```jsx
// 创建自定义 Hook 封装 Context 逻辑
function useTheme() {
  const context = useContext(ThemeContext);
  
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  
  return context;
}

// 使用自定义 Hook
function Component() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button onClick={toggleTheme}>
      主题: {theme}
    </button>
  );
}
```

## 实际应用

### 主题管理系统

```jsx
import { createContext, useContext, useState, useCallback } from 'react';

const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState({
    mode: 'light', // 'light' | 'dark'
    primary: '#3b82f6',
    secondary: '#6b7280',
    background: '#ffffff',
    text: '#1f2937'
  });
  
  const setThemeMode = useCallback((mode) => {
    setTheme(prev => ({
      ...prev,
      mode,
      background: mode === 'light' ? '#ffffff' : '#1f2937',
      text: mode === 'light' ? '#1f2937' : '#ffffff'
    }));
  }, []);
  
  const setColors = useCallback((colors) => {
    setTheme(prev => ({ ...prev, ...colors }));
  }, []);
  
  const resetTheme = useCallback(() => {
    setTheme({
      mode: 'light',
      primary: '#3b82f6',
      secondary: '#6b7280',
      background: '#ffffff',
      text: '#1f2937'
    });
  }, []);
  
  const value = {
    theme,
    setThemeMode,
    setColors,
    resetTheme
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// 使用示例
function ThemeControls() {
  const { theme, setThemeMode, setColors, resetTheme } = useTheme();
  
  return (
    <div style={{ 
      background: theme.background, 
      color: theme.text,
      padding: '20px'
    }}>
      <h2>主题控制</h2>
      
      <div>
        <h3>主题模式</h3>
        <button onClick={() => setThemeMode('light')}>浅色</button>
        <button onClick={() => setThemeMode('dark')}>深色</button>
      </div>
      
      <div>
        <h3>颜色设置</h3>
        <label>
          主色调:
          <input
            type="color"
            value={theme.primary}
            onChange={(e) => setColors({ primary: e.target.value })}
          />
        </label>
        
        <label>
          次要色调:
          <input
            type="color"
            value={theme.secondary}
            onChange={(e) => setColors({ secondary: e.target.value })}
          />
        </label>
      </div>
      
      <button onClick={resetTheme}>重置主题</button>
    </div>
  );
}

function App() {
  return (
    <ThemeProvider>
      <ThemeControls />
    </ThemeProvider>
  );
}
```

### 用户认证系统

```jsx
const AuthContext = createContext();

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const login = useCallback(async (credentials) => {
    try {
      setLoading(true);
      setError(null);
      
      // 模拟 API 调用
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(credentials)
      });
      
      const data = await response.json();
      
      if (!response.ok) {
        throw new Error(data.message || '登录失败');
      }
      
      setUser(data.user);
      localStorage.setItem('token', data.token);
      
      return data.user;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  }, []);
  
  const logout = useCallback(() => {
    setUser(null);
    localStorage.removeItem('token');
  }, []);
  
  const register = useCallback(async (userData) => {
    try {
      setLoading(true);
      setError(null);
      
      const response = await fetch('/api/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData)
      });
      
      const data = await response.json();
      
      if (!response.ok) {
        throw new Error(data.message || '注册失败');
      }
      
      setUser(data.user);
      localStorage.setItem('token', data.token);
      
      return data.user;
    } catch (err) {
      setError(err.message);
      throw err;
    } finally {
      setLoading(false);
    }
  }, []);
  
  const checkAuth = useCallback(async () => {
    const token = localStorage.getItem('token');
    if (!token) return;
    
    try {
      setLoading(true);
      const response = await fetch('/api/me', {
        headers: { 'Authorization': `Bearer ${token}` }
      });
      
      if (response.ok) {
        const data = await response.json();
        setUser(data.user);
      } else {
        logout();
      }
    } catch (err) {
      logout();
    } finally {
      setLoading(false);
    }
  }, [logout]);
  
  // 组件挂载时检查认证状态
  useEffect(() => {
    checkAuth();
  }, [checkAuth]);
  
  const value = {
    user,
    loading,
    error,
    login,
    logout,
    register,
    checkAuth,
    isAuthenticated: !!user
  };
  
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// 使用示例
function LoginForm() {
  const { login, loading, error } = useAuth();
  const [formData, setFormData] = useState({
    username: '',
    password: ''
  });
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await login(formData);
    } catch (err) {
      console.error('登录失败:', err);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={formData.username}
        onChange={(e) => setFormData({ ...formData, username: e.target.value })}
        placeholder="用户名"
      />
      <input
        type="password"
        value={formData.password}
        onChange={(e) => setFormData({ ...formData, password: e.target.value })}
        placeholder="密码"
      />
      <button type="submit" disabled={loading}>
        {loading ? '登录中...' : '登录'}
      </button>
      {error && <div style={{ color: 'red' }}>{error}</div>}
    </form>
  );
}

function ProtectedRoute({ children }) {
  const { isAuthenticated, loading } = useAuth();
  
  if (loading) {
    return <div>加载中...</div>;
  }
  
  if (!isAuthenticated) {
    return <LoginForm />;
  }
  
  return children;
}

function Dashboard() {
  const { user, logout } = useAuth();
  
  return (
    <div>
      <h1>欢迎, {user?.name}</h1>
      <p>角色: {user?.role}</p>
      <button onClick={logout}>退出登录</button>
    </div>
  );
}

function App() {
  return (
    <AuthProvider>
      <ProtectedRoute>
        <Dashboard />
      </ProtectedRoute>
    </AuthProvider>
  );
}
```

### 多语言支持

```jsx
const LanguageContext = createContext();

const translations = {
  en: {
    welcome: 'Welcome',
    login: 'Login',
    logout: 'Logout',
    dashboard: 'Dashboard',
    settings: 'Settings'
  },
  zh: {
    welcome: '欢迎',
    login: '登录',
    logout: '退出',
    dashboard: '仪表盘',
    settings: '设置'
  },
  ja: {
    welcome: 'ようこそ',
    login: 'ログイン',
    logout: 'ログアウト',
    dashboard: 'ダッシュボード',
    settings: '設定'
  }
};

function LanguageProvider({ children }) {
  const [language, setLanguage] = useState('en');
  
  const t = useCallback((key) => {
    return translations[language][key] || key;
  }, [language]);
  
  const changeLanguage = useCallback((lang) => {
    setLanguage(lang);
    localStorage.setItem('language', lang);
  }, []);
  
  // 从本地存储读取语言设置
  useEffect(() => {
    const savedLanguage = localStorage.getItem('language');
    if (savedLanguage && translations[savedLanguage]) {
      setLanguage(savedLanguage);
    }
  }, []);
  
  const value = {
    language,
    t,
    changeLanguage,
    availableLanguages: Object.keys(translations)
  };
  
  return (
    <LanguageContext.Provider value={value}>
      {children}
    </LanguageContext.Provider>
  );
}

function useLanguage() {
  const context = useContext(LanguageContext);
  if (!context) {
    throw new Error('useLanguage must be used within LanguageProvider');
  }
  return context;
}

// 使用示例
function LanguageSelector() {
  const { language, changeLanguage, availableLanguages } = useLanguage();
  
  return (
    <select value={language} onChange={(e) => changeLanguage(e.target.value)}>
      {availableLanguages.map(lang => (
        <option key={lang} value={lang}>{lang.toUpperCase()}</option>
      ))}
    </select>
  );
}

function WelcomeMessage() {
  const { t } = useLanguage();
  
  return <h1>{t('welcome')}</h1>;
}

function App() {
  return (
    <LanguageProvider>
      <LanguageSelector />
      <WelcomeMessage />
    </LanguageProvider>
  );
}
```

### 全局通知系统

```jsx
const NotificationContext = createContext();

function NotificationProvider({ children }) {
  const [notifications, setNotifications] = useState([]);
  
  const addNotification = useCallback((notification) => {
    const id = Date.now();
    setNotifications(prev => [
      ...prev,
      { id, ...notification }
    ]);
    
    // 自动移除通知
    if (notification.duration !== false) {
      setTimeout(() => {
        removeNotification(id);
      }, notification.duration || 3000);
    }
    
    return id;
  }, []);
  
  const removeNotification = useCallback((id) => {
    setNotifications(prev => prev.filter(n => n.id !== id));
  }, []);
  
  const clearAll = useCallback(() => {
    setNotifications([]);
  }, []);
  
  const showSuccess = useCallback((message, options = {}) => {
    return addNotification({
      type: 'success',
      message,
      ...options
    });
  }, [addNotification]);
  
  const showError = useCallback((message, options = {}) => {
    return addNotification({
      type: 'error',
      message,
      ...options
    });
  }, [addNotification]);
  
  const showWarning = useCallback((message, options = {}) => {
    return addNotification({
      type: 'warning',
      message,
      ...options
    });
  }, [addNotification]);
  
  const showInfo = useCallback((message, options = {}) => {
    return addNotification({
      type: 'info',
      message,
      ...options
    });
  }, [addNotification]);
  
  const value = {
    notifications,
    addNotification,
    removeNotification,
    clearAll,
    showSuccess,
    showError,
    showWarning,
    showInfo
  };
  
  return (
    <NotificationContext.Provider value={value}>
      {children}
      <NotificationContainer />
    </NotificationContext.Provider>
  );
}

function useNotification() {
  const context = useContext(NotificationContext);
  if (!context) {
    throw new Error('useNotification must be used within NotificationProvider');
  }
  return context;
}

function NotificationContainer() {
  const { notifications, removeNotification } = useNotification();
  
  const notificationStyles = {
    success: { backgroundColor: '#10b981', color: 'white' },
    error: { backgroundColor: '#ef4444', color: 'white' },
    warning: { backgroundColor: '#f59e0b', color: 'white' },
    info: { backgroundColor: '#3b82f6', color: 'white' }
  };
  
  return (
    <div style={{
      position: 'fixed',
      top: '20px',
      right: '20px',
      zIndex: 9999
    }}>
      {notifications.map(notification => (
        <div
          key={notification.id}
          style={{
            ...notificationStyles[notification.type],
            padding: '12px 20px',
            marginBottom: '10px',
            borderRadius: '8px',
            boxShadow: '0 2px 8px rgba(0,0,0,0.1)',
            display: 'flex',
            alignItems: 'center',
            justifyContent: 'space-between',
            minWidth: '300px'
          }}
        >
          <span>{notification.message}</span>
          <button
            onClick={() => removeNotification(notification.id)}
            style={{
              background: 'none',
              border: 'none',
              color: 'white',
              fontSize: '16px',
              cursor: 'pointer',
              marginLeft: '10px'
            }}
          >
            ×
          </button>
        </div>
      ))}
    </div>
  );
}

// 使用示例
function NotificationDemo() {
  const { showSuccess, showError, showWarning, showInfo } = useNotification();
  
  return (
    <div>
      <button onClick={() => showSuccess('操作成功！')}>
        成功通知
      </button>
      <button onClick={() => showError('操作失败！')}>
        错误通知
      </button>
      <button onClick={() => showWarning('请注意！')}>
        警告通知
      </button>
      <button onClick={() => showInfo('提示信息')}>
        信息通知
      </button>
    </div>
  );
}

function App() {
  return (
    <NotificationProvider>
      <NotificationDemo />
    </NotificationProvider>
  );
}
```

## 注意事项

### 1. Context 的性能影响

```jsx
// ❌ 性能问题：每次渲染都创建新的 value 对象
function BadProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  return (
    <MyContext.Provider value={{ user, theme, setUser, setTheme }}>
      {children}
    </MyContext.Provider>
  );
}

// ✅ 优化方案：使用 useMemo 缓存 value
function GoodProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  const value = useMemo(() => ({
    user,
    theme,
    setUser,
    setTheme
  }), [user, theme]);
  
  return (
    <MyContext.Provider value={value}>
      {children}
    </MyContext.Provider>
  );
}
```

### 2. 分离 Context 避免不必要的重渲染

```jsx
// ❌ 所有数据都在一个 Context 中
const CombinedContext = createContext();

// ✅ 分离 Context，减少重渲染
const UserContext = createContext();
const ThemeContext = createContext();

function App() {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <ThemeContext.Provider value={{ theme, setTheme }}>
        <Dashboard />
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}
```

### 3. 默认值的处理

```jsx
// ❌ 没有默认值，可能导致错误
const MyContext = createContext();

function Component() {
  const value = useContext(MyContext);
  // 如果没有 Provider，value 是 undefined
  return <div>{value}</div>;
}

// ✅ 提供合理的默认值
const MyContext = createContext({
  user: null,
  theme: 'light'
});

function Component() {
  const value = useContext(MyContext);
  // 即便没有 Provider，也有默认值
  return <div>{value.theme}</div>;
}
```

### 4. 错误边界和默认值

```jsx
// 创建一个带错误检查的自定义 Hook
function useMyContext() {
  const context = useContext(MyContext);
  
  if (context === undefined) {
    throw new Error('useMyContext must be used within MyProvider');
  }
  
  return context;
}

// 或者提供默认值
function useMyContextWithDefault() {
  const context = useContext(MyContext);
  return context ?? defaultValue;
}
```

### 5. 避免在 Context 中存储过多数据

```jsx
// ❌ 存储过多不必要的数据
const AppContext = createContext();

function AppProvider({ children }) {
  const [users, setUsers] = useState([]); // 大量数据
  const [posts, setPosts] = useState([]);  // 大量数据
  const [comments, setComments] = useState([]); // 大量数据
  
  return (
    <AppContext.Provider value={{ users, posts, comments, setUsers, setPosts, setComments }}>
      {children}
    </AppContext.Provider>
  );
}

// ✅ 只存储必要的数据
const AppContext = createContext();

function AppProvider({ children }) {
  const [currentUser, setCurrentUser] = useState(null); // 只存储当前用户
  const [theme, setTheme] = useState('light'); // 只存储主题
  
  // 大数据单独处理
  const { users } = useUsers(); // 使用自定义 Hook 获取用户数据
  const { posts } = usePosts(); // 使用自定义 Hook 获取文章数据
  
  return (
    <AppContext.Provider value={{ currentUser, setCurrentUser, theme, setTheme }}>
      {children}
    </AppContext.Provider>
  );
}
```

## 总结

useContext 是 React 中强大的状态管理工具：

### 核心要点

1. **避免 prop drilling**：无需逐层传递 props
2. **跨组件共享数据**：在组件树中任何位置访问数据
3. **全局状态管理**：适合应用级的全局状态
4. **组合使用**：可以组合多个 Context 满足不同需求

### 使用场景

1. **主题管理**：全局主题切换
2. **用户认证**：登录状态和用户信息
3. **多语言支持**：国际化处理
4. **通知系统**：全局通知管理
5. **配置管理**：应用配置和设置

### 最佳实践

1. **合理分离**：按功能分离不同的 Context
2. **性能优化**：使用 useMemo 缓存 Context value
3. **自定义 Hooks**：封装 Context 逻辑提高可复用性
4. **错误处理**：提供默认值或错误检查
5. **数据控制**：避免在 Context 中存储过多数据

### Context vs 其他状态管理方案

**适用 Context 的场景：**
- 跨多个层级的少量数据
- 相对静态的配置信息
- 主题、语言等全局设置

**适用 Redux/Zustand 的场景：**
- 复杂的状态逻辑
- 频繁的状态更新
- 需要时间旅行调试
- 大规模应用的状态管理

useContext 是 React 生态系统中重要的一环，它与 React 生态系统完美集成，为开发者提供了简洁而强大的状态管理方案。理解其工作原理和适用场景，能够帮助你构建更清晰、更易维护的 React 应用。