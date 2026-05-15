---
title: React Router 进阶
published: 2023-09-25
description: '路由守卫和懒加载的详细介绍和学习笔记'
image: ''
tags: ["React","Router"]
category: 'React'
draft: false
lang: 'zh-CN'
---

## 概述

React Router 进阶技术涵盖了路由守卫、懒加载、代码分割、性能优化等高级主题。这些技术能够帮助开发者构建更加安全、高效、用户体验更好的单页应用。本文将深入探讨 React Router 的高级功能和最佳实践。

## 核心概念

### 路由守卫

路由守卫用于保护特定路由的访问权限，实现用户认证、权限控制等功能。常见的守卫场景包括：
- 用户登录验证
- 角色权限检查
- 页面访问条件判断
- 路由跳转前的数据预加载

### 代码分割和懒加载

代码分割是优化应用加载性能的关键技术，通过按需加载路由组件来减少初始加载体积：
- 减少首次加载时间
- 按需加载减少带宽消耗
- 提升用户体验
- 改善 SEO 表现

### 路由元信息

路由元信息提供了额外的路由配置数据，用于：
- 页面标题设置
- 权限控制配置
- 布局组件选择
- SEO 元数据配置

## 基本用法

### 路由守卫实现

```jsx
import { Navigate, useLocation } from 'react-router-dom';

// 认证上下文
const AuthContext = createContext();

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // 检查用户登录状态
    const token = localStorage.getItem('token');
    if (token) {
      // 验证 token 并获取用户信息
      fetchUserProfile(token)
        .then(userData => {
          setUser(userData);
          setLoading(false);
        })
        .catch(() => {
          setLoading(false);
        });
    } else {
      setLoading(false);
    }
  }, []);

  const login = async (credentials) => {
    const response = await loginApi(credentials);
    const { token, user: userData } = response;
    localStorage.setItem('token', token);
    setUser(userData);
  };

  const logout = () => {
    localStorage.removeItem('token');
    setUser(null);
  };

  const value = { user, loading, login, logout };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

// 认证 Hook
function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// 私有路由守卫组件
function PrivateRoute({ children, requiredRole = null }) {
  const { user, loading } = useAuth();
  const location = useLocation();

  if (loading) {
    return <div>Loading...</div>;
  }

  if (!user) {
    // 未登录，重定向到登录页
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  if (requiredRole && !user.roles.includes(requiredRole)) {
    // 权限不足
    return <Navigate to="/unauthorized" replace />;
  }

  return children;
}

// 管理员路由守卫
function AdminRoute({ children }) {
  return <PrivateRoute requiredRole="admin">{children}</PrivateRoute>;
}

// 使用路由守卫
function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          <Route path="/login" element={<LoginPage />} />
          <Route path="/unauthorized" element={<UnauthorizedPage />} />

          <Route
            path="/dashboard"
            element={
              <PrivateRoute>
                <DashboardPage />
              </PrivateRoute>
            }
          />

          <Route
            path="/admin/*"
            element={
              <AdminRoute>
                <AdminLayout />
              </AdminRoute>
            }
          >
            <Route index element={<AdminDashboard />} />
            <Route path="users" element={<UserManagement />} />
            <Route path="settings" element={<AdminSettings />} />
          </Route>

          <Route path="*" element={<Navigate to="/login" replace />} />
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}

// 登录页面
function LoginPage() {
  const { login } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();
  const from = location.state?.from?.pathname || '/dashboard';

  const handleSubmit = async (e) => {
    e.preventDefault();
    const formData = new FormData(e.target);
    const credentials = {
      username: formData.get('username'),
      password: formData.get('password')
    };

    try {
      await login(credentials);
      navigate(from, { replace: true });
    } catch (error) {
      console.error('Login failed:', error);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="username" placeholder="Username" />
      <input name="password" type="password" placeholder="Password" />
      <button type="submit">Login</button>
    </form>
  );
}
```

### 代码分割和懒加载

```jsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route, Navigate } from 'react-router-dom';

// 懒加载组件
const HomePage = lazy(() => import('./pages/HomePage'));
const AboutPage = lazy(() => import('./pages/AboutPage'));
const ContactPage = lazy(() => import('./pages/ContactPage'));
const DashboardPage = lazy(() => import('./pages/DashboardPage'));
const AdminPanel = lazy(() => import('./pages/AdminPanel'));

// 加载状态组件
function LoadingFallback() {
  return (
    <div className="loading-container">
      <div className="spinner"></div>
      <p>Loading...</p>
    </div>
  );
}

// 错误边界组件
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Lazy loading error:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-boundary">
          <h2>Something went wrong</h2>
          <p>{this.state.error?.message}</p>
          <button onClick={() => window.location.reload()}>
            Reload Page
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// 预加载函数
function preloadComponent(componentImport) {
  return componentImport();
}

// 路由配置组件
function App() {
  return (
    <ErrorBoundary>
      <BrowserRouter>
        <Suspense fallback={<LoadingFallback />}>
          <Routes>
            <Route path="/" element={<HomePage />} />
            <Route path="/about" element={<AboutPage />} />
            <Route path="/contact" element={<ContactPage />} />

            {/* 预加载示例 */}
            <Route
              path="/dashboard"
              element={<DashboardPage />}
              loader={async () => {
                // 预加载相关组件
                await preloadComponent(() => import('./pages/DashboardPage'));
                return null;
              }}
            />

            <Route
              path="/admin"
              element={<AdminPanel />}
              loader={async () => {
                // 预加载管理后台相关组件
                await Promise.all([
                  preloadComponent(() => import('./pages/AdminPanel')),
                  preloadComponent(() => import('./pages/UserManagement')),
                  preloadComponent(() => import('./pages/Analytics'))
                ]);
                return null;
              }}
            />

            <Route path="*" element={<Navigate to="/" replace />} />
          </Routes>
        </Suspense>
      </BrowserRouter>
    </ErrorBoundary>
  );
}

// 预加载导航组件
function PreloadLink({ to, children, ...props }) {
  const preload = () => {
    // 根据路由预加载对应组件
    switch (to) {
      case '/dashboard':
        preloadComponent(() => import('./pages/DashboardPage'));
        break;
      case '/admin':
        preloadComponent(() => import('./pages/AdminPanel'));
        break;
      default:
        break;
    }
  };

  return (
    <Link
      to={to}
      onMouseEnter={preload}
      onFocus={preload}
      {...props}
    >
      {children}
    </Link>
  );
}

// 使用预加载链接
function Navigation() {
  return (
    <nav>
      <PreloadLink to="/">Home</PreloadLink>
      <PreloadLink to="/dashboard">Dashboard</PreloadLink>
      <PreloadLink to="/admin">Admin</PreloadLink>
    </nav>
  );
}
```

### 路由数据预加载

```jsx
import { useLoaderData, defer, Await } from 'react-router-dom';

// Loader 函数 - 在路由渲染前加载数据
async function userProfileLoader({ params }) {
  const userId = params.userId;

  try {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
      throw new Response('User not found', { status: 404 });
    }
    return response.json();
  } catch (error) {
    throw new Response('Failed to load user', { status: 500 });
  }
}

// 异步 Loader 函数 - 使用 defer 支持异步加载
async function dashboardLoader() {
  const userPromise = fetch('/api/user').then(res => res.json());
  const statsPromise = fetch('/api/stats').then(res => res.json());

  return defer({
    user: userPromise,
    stats: statsPromise,
  });
}

// 使用 Loader 数据
function UserProfile() {
  const user = useLoaderData();

  return (
    <div>
      <h1>{user.name}</h1>
      <p>Email: {user.email}</p>
      {/* 其他用户信息 */}
    </div>
  );
}

// 使用异步 Loader 数据
function Dashboard() {
  const { user, stats } = useLoaderData();

  return (
    <div>
      <Suspense fallback={<div>Loading user...</div>}>
        <Await resolve={user}>
          {(userData) => (
            <div>
              <h1>Welcome, {userData.name}!</h1>
            </div>
          )}
        </Await>
      </Suspense>

      <Suspense fallback={<div>Loading stats...</div>}>
        <Await resolve={stats}>
          {(statsData) => (
            <div className="stats">
              <div className="stat">
                <h3>Total Users</h3>
                <p>{statsData.totalUsers}</p>
              </div>
              <div className="stat">
                <h3>Revenue</h3>
                <p>${statsData.revenue}</p>
              </div>
            </div>
          )}
        </Await>
      </Suspense>
    </div>
  );
}

// 配置路由和 Loader
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route
          path="/users/:userId"
          element={<UserProfile />}
          loader={userProfileLoader}
        />
        <Route
          path="/dashboard"
          element={<Dashboard />}
          loader={dashboardLoader}
        />
      </Routes>
    </BrowserRouter>
  );
}
```

### 路由元信息配置

```jsx
// 路由配置文件
const routes = [
  {
    path: '/',
    element: <HomeLayout />,
    children: [
      {
        index: true,
        element: <HomePage />,
        meta: {
          title: 'Home',
          description: 'Welcome to our application',
          requiresAuth: false
        }
      },
      {
        path: 'about',
        element: <AboutPage />,
        meta: {
          title: 'About Us',
          description: 'Learn about our company',
          requiresAuth: false
        }
      }
    ]
  },
  {
    path: '/dashboard',
    element: <DashboardLayout />,
    meta: {
      title: 'Dashboard',
      description: 'Your personal dashboard',
      requiresAuth: true,
      roles: ['user', 'admin']
    },
    children: [
      {
        index: true,
        element: <DashboardHome />,
        meta: {
          title: 'Dashboard Home',
          requiresAuth: true
        }
      },
      {
        path: 'settings',
        element: <SettingsPage />,
        meta: {
          title: 'Settings',
          requiresAuth: true
        }
      }
    ]
  },
  {
    path: '/admin',
    element: <AdminLayout />,
    meta: {
      title: 'Admin Panel',
      description: 'Administrative controls',
      requiresAuth: true,
      roles: ['admin']
    },
    children: [
      {
        index: true,
        element: <AdminDashboard />,
        meta: {
          title: 'Admin Dashboard',
          requiresAuth: true,
          roles: ['admin']
        }
      },
      {
        path: 'users',
        element: <UserManagement />,
        meta: {
          title: 'User Management',
          requiresAuth: true,
          roles: ['admin']
        }
      }
    ]
  }
];

// 路由守卫组件
function RouteGuard({ routeMeta, children }) {
  const { user } = useAuth();
  const navigate = useNavigate();

  useEffect(() => {
    // 检查认证要求
    if (routeMeta?.requiresAuth && !user) {
      navigate('/login', { replace: true });
      return;
    }

    // 检查角色权限
    if (routeMeta?.roles && user) {
      const hasRequiredRole = routeMeta.roles.some(role =>
        user.roles.includes(role)
      );

      if (!hasRequiredRole) {
        navigate('/unauthorized', { replace: true });
        return;
      }
    }

    // 设置页面标题
    if (routeMeta?.title) {
      document.title = `${routeMeta.title} - My App`;
    }

    // 设置 meta 标签
    if (routeMeta?.description) {
      updateMetaDescription(routeMeta.description);
    }
  }, [routeMeta, user, navigate]);

  return children;
}

// 更新 meta 描述
function updateMetaDescription(description) {
  const metaTag = document.querySelector('meta[name="description"]');
  if (metaTag) {
    metaTag.setAttribute('content', description);
  } else {
    const newMetaTag = document.createElement('meta');
    newMetaTag.setAttribute('name', 'description');
    newMetaTag.setAttribute('content', description);
    document.head.appendChild(newMetaTag);
  }
}

// 路由渲染组件
function RenderRoutes({ routes }) {
  const location = useLocation();
  const route = findRouteByPath(routes, location.pathname);

  if (!route) {
    return <NotFoundPage />;
  }

  return (
    <RouteGuard routeMeta={route.meta}>
      {route.element}
    </RouteGuard>
  );
}

// 根据路径查找路由
function findRouteByPath(routes, path) {
  for (const route of routes) {
    // 精确匹配
    if (route.path === path) {
      return route;
    }

    // 嵌套路由匹配
    if (route.children) {
      const childRoute = findRouteByPath(route.children, path);
      if (childRoute) {
        return childRoute;
      }
    }

    // 动态路由匹配
    const routePath = route.path.replace(/:[^\s/]+/g, '[^/]+');
    const regex = new RegExp(`^${routePath}$`);
    if (regex.test(path)) {
      return route;
    }
  }

  return null;
}

// 使用路由配置
function App() {
  return (
    <BrowserRouter>
      <Routes>
        {routes.map((route, index) => (
          <Route
            key={index}
            path={route.path}
            element={
              <RouteGuard routeMeta={route.meta}>
                {route.element}
              </RouteGuard>
            }
          >
            {route.children?.map((childRoute, childIndex) => (
              <Route
                key={childIndex}
                path={childRoute.path}
                index={childRoute.index}
                element={
                  <RouteGuard routeMeta={childRoute.meta}>
                    {childRoute.element}
                  </RouteGuard>
                }
              />
            ))}
          </Route>
        ))}
      </Routes>
    </BrowserRouter>
  );
}
```

## 实际应用

### 综合路由管理系统

```jsx
import { BrowserRouter, Routes, Route, Outlet, useLocation } from 'react-router-dom';
import { lazy, Suspense } from 'react';

// 路由常量
const ROUTES = {
  HOME: '/',
  LOGIN: '/login',
  REGISTER: '/register',
  DASHBOARD: '/dashboard',
  PROFILE: '/profile',
  SETTINGS: '/settings',
  ADMIN: '/admin',
  USERS: '/admin/users',
  PRODUCTS: '/admin/products',
  ORDERS: '/admin/orders'
};

// 权限常量
const PERMISSIONS = {
  VIEW_DASHBOARD: 'view_dashboard',
  MANAGE_USERS: 'manage_users',
  MANAGE_PRODUCTS: 'manage_products',
  MANAGE_ORDERS: 'manage_orders'
};

// 懒加载页面组件
const HomePage = lazy(() => import('./pages/Home'));
const LoginPage = lazy(() => import('./pages/auth/Login'));
const RegisterPage = lazy(() => import('./pages/auth/Register'));
const DashboardPage = lazy(() => import('./pages/dashboard/Dashboard'));
const ProfilePage = lazy(() => import('./pages/user/Profile'));
const SettingsPage = lazy(() => import('./pages/user/Settings'));
const AdminLayout = lazy(() => import('./layouts/AdminLayout'));
const AdminDashboard = lazy(() => import('./pages/admin/Dashboard'));
const UserManagement = lazy(() => import('./pages/admin/UserManagement'));
const ProductManagement = lazy(() => import('./pages/admin/ProductManagement'));
const OrderManagement = lazy(() => import('./pages/admin/OrderManagement'));

// 认证服务
class AuthService {
  constructor() {
    this.user = null;
    this.permissions = [];
    this.init();
  }

  async init() {
    const token = localStorage.getItem('token');
    if (token) {
      await this.loadUser();
    }
  }

  async loadUser() {
    try {
      const response = await fetch('/api/auth/me', {
        headers: {
          'Authorization': `Bearer ${localStorage.getItem('token')}`
        }
      });

      if (response.ok) {
        const data = await response.json();
        this.user = data.user;
        this.permissions = data.permissions;
      }
    } catch (error) {
      console.error('Failed to load user:', error);
      this.logout();
    }
  }

  async login(credentials) {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credentials)
    });

    if (response.ok) {
      const data = await response.json();
      localStorage.setItem('token', data.token);
      await this.loadUser();
      return true;
    }

    return false;
  }

  logout() {
    localStorage.removeItem('token');
    this.user = null;
    this.permissions = [];
  }

  isAuthenticated() {
    return this.user !== null;
  }

  hasPermission(permission) {
    return this.permissions.includes(permission);
  }

  hasAnyPermission(permissions) {
    return permissions.some(permission => this.hasPermission(permission));
  }
}

const authService = new AuthService();

// 权限守卫组件
function PermissionGuard({ permissions, requireAll = false, children }) {
  const location = useLocation();
  const navigate = useNavigate();
  const [checking, setChecking] = useState(true);

  useEffect(() => {
    const checkPermission = async () => {
      if (!authService.isAuthenticated()) {
        navigate(ROUTES.LOGIN, {
          state: { from: location },
          replace: true
        });
        setChecking(false);
        return;
      }

      const hasPermission = requireAll
        ? permissions.every(permission => authService.hasPermission(permission))
        : authService.hasAnyPermission(permissions);

      if (!hasPermission) {
        navigate('/unauthorized', { replace: true });
      }

      setChecking(false);
    };

    checkPermission();
  }, [permissions, requireAll, location, navigate]);

  if (checking) {
    return <div>Checking permissions...</div>;
  }

  return children;
}

// 路由配置
const routeConfig = [
  {
    path: ROUTES.HOME,
    element: <HomePage />,
    meta: {
      title: 'Home',
      requiresAuth: false
    }
  },
  {
    path: ROUTES.LOGIN,
    element: <LoginPage />,
    meta: {
      title: 'Login',
      requiresAuth: false
    }
  },
  {
    path: ROUTES.REGISTER,
    element: <RegisterPage />,
    meta: {
      title: 'Register',
      requiresAuth: false
    }
  },
  {
    path: ROUTES.DASHBOARD,
    element: <DashboardPage />,
    meta: {
      title: 'Dashboard',
      requiresAuth: true,
      permissions: [PERMISSIONS.VIEW_DASHBOARD]
    }
  },
  {
    path: ROUTES.PROFILE,
    element: <ProfilePage />,
    meta: {
      title: 'Profile',
      requiresAuth: true
    }
  },
  {
    path: ROUTES.SETTINGS,
    element: <SettingsPage />,
    meta: {
      title: 'Settings',
      requiresAuth: true
    }
  },
  {
    path: ROUTES.ADMIN,
    element: <AdminLayout />,
    meta: {
      title: 'Admin Panel',
      requiresAuth: true
    },
    children: [
      {
        index: true,
        element: <AdminDashboard />,
        meta: {
          title: 'Admin Dashboard',
          requiresAuth: true
        }
      },
      {
        path: 'users',
        element: <UserManagement />,
        meta: {
          title: 'User Management',
          requiresAuth: true,
          permissions: [PERMISSIONS.MANAGE_USERS]
        }
      },
      {
        path: 'products',
        element: <ProductManagement />,
        meta: {
          title: 'Product Management',
          requiresAuth: true,
          permissions: [PERMISSIONS.MANAGE_PRODUCTS]
        }
      },
      {
        path: 'orders',
        element: <OrderManagement />,
        meta: {
          title: 'Order Management',
          requiresAuth: true,
          permissions: [PERMISSIONS.MANAGE_ORDERS]
        }
      }
    ]
  }
];

// 智能路由渲染器
function SmartRouter({ config }) {
  const location = useLocation();

  // 查找匹配的路由
  const matchedRoute = findMatchingRoute(config, location.pathname);

  if (!matchedRoute) {
    return <NotFoundPage />;
  }

  const { element, meta, children } = matchedRoute;

  // 应用路由守卫
  if (meta?.requiresAuth && !authService.isAuthenticated()) {
    return <Navigate to={ROUTES.LOGIN} state={{ from: location }} replace />;
  }

  if (meta?.permissions) {
    const hasPermission = meta.requireAll
      ? meta.permissions.every(permission => authService.hasPermission(permission))
      : authService.hasAnyPermission(meta.permissions);

    if (!hasPermission) {
      return <Navigate to="/unauthorized" replace />;
    }
  }

  // 渲染路由元素
  return (
    <>
      {element}
      {children && (
        <Routes>
          {children.map((child, index) => (
            <Route key={index} path={child.path} element={<SmartRouter config={[child]} />} />
          ))}
        </Routes>
      )}
    </>
  );
}

// 查找匹配的路由
function findMatchingRoute(config, pathname) {
  for (const route of config) {
    // 精确匹配
    if (route.path === pathname) {
      return route;
    }

    // 动态路由匹配
    const routeParts = route.path.split('/');
    const pathParts = pathname.split('/');

    if (routeParts.length === pathParts.length) {
      const isMatch = routeParts.every((part, index) => {
        return part.startsWith(':') || part === pathParts[index];
      });

      if (isMatch) {
        return route;
      }
    }
  }

  return null;
}

// 加载状态组件
function LoadingSpinner() {
  return (
    <div className="loading-spinner">
      <div className="spinner"></div>
      <p>Loading...</p>
    </div>
  );
}

// 主应用组件
function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<LoadingSpinner />}>
        <Routes>
          <Route path="*" element={<SmartRouter config={routeConfig} />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}

// 导出路由配置供其他模块使用
export { ROUTES, PERMISSIONS, authService };
export default App;
```

### 多语言路由支持

```jsx
import { BrowserRouter, Routes, Route, useParams, useLocation } from 'react-router-dom';

// 支持的语言
const SUPPORTED_LANGUAGES = ['en', 'zh', 'ja', 'ko'];

// 语言上下文
const LanguageContext = createContext();

function LanguageProvider({ children }) {
  const { lang } = useParams();
  const location = useLocation();

  // 验证语言参数
  const currentLanguage = SUPPORTED_LANGUAGES.includes(lang) ? lang : 'en';

  // 更新 URL 语言
  const updateLanguage = (newLang) => {
    if (!SUPPORTED_LANGUAGES.includes(newLang)) return;

    const pathParts = location.pathname.split('/');
    pathParts[1] = newLang; // 替换语言部分

    const newPath = pathParts.join('/');
    window.location.href = newPath;
  };

  return (
    <LanguageContext.Provider value={{ language: currentLanguage, updateLanguage }}>
      {children}
    </LanguageContext.Provider>
  );
}

// 使用语言上下文
function useLanguage() {
  const context = useContext(LanguageContext);
  if (!context) {
    throw new Error('useLanguage must be used within LanguageProvider');
  }
  return context;
}

// 多语言路由配置
function App() {
  return (
    <BrowserRouter>
      <Routes>
        {/* 语言路由 */}
        <Route path="/:lang/*" element={<LanguageProvider><AppContent /></LanguageProvider>} />

        {/* 默认重定向到英文 */}
        <Route path="*" element={<Navigate to="/en" replace />} />
      </Routes>
    </BrowserRouter>
  );
}

// 应用内容
function AppContent() {
  const { language } = useLanguage();

  return (
    <div className={`app ${language}`}>
      <LanguageSwitcher />
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="about" element={<AboutPage />} />
        <Route path="contact" element={<ContactPage />} />
        <Route path="products" element={<ProductsPage />} />
      </Routes>
    </div>
  );
}

// 语言切换器
function LanguageSwitcher() {
  const { language, updateLanguage } = useLanguage();

  return (
    <div className="language-switcher">
      {SUPPORTED_LANGUAGES.map(lang => (
        <button
          key={lang}
          onClick={() => updateLanguage(lang)}
          className={language === lang ? 'active' : ''}
        >
          {lang.toUpperCase()}
        </button>
      ))}
    </div>
  );
}

// 多语言页面组件
function HomePage() {
  const { language } = useLanguage();

  const translations = {
    en: {
      title: 'Welcome to Our Application',
      description: 'This is a multi-language application.',
      buttonText: 'Get Started'
    },
    zh: {
      title: '欢迎来到我们的应用',
      description: '这是一个多语言应用程序。',
      buttonText: '开始使用'
    },
    ja: {
      title: '私たちのアプリケーションへようこそ',
      description: 'これは多言語アプリケーションです。',
      buttonText: '始める'
    },
    ko: {
      title: '우리 애플리케이션에 오신 것을 환영합니다',
      description: '이것은 다국어 애플리케이션입니다.',
      buttonText: '시작하기'
    }
  };

  const t = translations[language];

  return (
    <div className="home-page">
      <h1>{t.title}</h1>
      <p>{t.description}</p>
      <button>{t.buttonText}</button>
    </div>
  );
}
```

## 注意事项

### 路由守卫性能

```jsx
// 问题：每次路由变化都重新检查权限
function ProtectedRoute({ children, requiredPermissions }) {
  const { user, permissions } = useAuth();
  const location = useLocation();

  useEffect(() => {
    // 每次渲染都检查权限
    if (!hasRequiredPermissions(permissions, requiredPermissions)) {
      navigate('/unauthorized');
    }
  }, [permissions, requiredPermissions, location]);

  return children;
}

// 优化：使用 useMemo 缓存权限检查结果
function ProtectedRoute({ children, requiredPermissions }) {
  const { user, permissions } = useAuth();
  const navigate = useNavigate();

  const hasPermission = useMemo(() => {
    return hasRequiredPermissions(permissions, requiredPermissions);
  }, [permissions, requiredPermissions]);

  useEffect(() => {
    if (!hasPermission) {
      navigate('/unauthorized');
    }
  }, [hasPermission, navigate]);

  if (!hasPermission) {
    return null;
  }

  return children;
}
```

### 懒加载错误处理

```jsx
// 问题：懒加载失败没有错误处理
const HeavyComponent = lazy(() => import('./HeavyComponent'));

// 解决方案：添加错误边界和重试逻辑
class LazyLoadErrorBoundary extends React.Component {
  state = { hasError: false, error: null, retryCount: 0 };

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Lazy load error:', error, errorInfo);
  }

  handleRetry = () => {
    this.setState({ hasError: false, error: null, retryCount: this.state.retryCount + 1 });
    window.location.reload();
  };

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-fallback">
          <h3>Failed to load component</h3>
          <p>{this.state.error?.message}</p>
          {this.state.retryCount < 3 ? (
            <button onClick={this.handleRetry}>Retry</button>
          ) : (
            <p>Max retry attempts reached. Please refresh the page.</p>
          )}
        </div>
      );
    }

    return this.props.children;
  }
}

// 使用错误边界
function App() {
  return (
    <BrowserRouter>
      <LazyLoadErrorBoundary>
        <Suspense fallback={<div>Loading...</div>}>
          <Routes>
            <Route path="/heavy" element={<HeavyComponent />} />
          </Routes>
        </Suspense>
      </LazyLoadErrorBoundary>
    </BrowserRouter>
  );
}
```

### 路由参数验证

```jsx
// 验证路由参数
function useValidatedParams(schema) {
  const params = useParams();
  const [error, setError] = useState(null);

  useEffect(() => {
    try {
      schema.validate(params, { abortEarly: false });
      setError(null);
    } catch (validationError) {
      setError(validationError);
    }
  }, [params, schema]);

  return { params, error };
}

// 使用参数验证
function UserProfile() {
  const { params, error } = useValidatedParams(
    yup.object().shape({
      userId: yup.string().required().matches(/^\d+$/, 'User ID must be a number')
    })
  );

  if (error) {
    return <Navigate to="/404" replace />;
  }

  // 安全地使用参数
  const userId = parseInt(params.userId);
  // ... 组件逻辑
}
```

## 总结

React Router 进阶技术为构建复杂、高性能的单页应用提供了强大的工具：

### 关键技术点

1. **路由守卫**：实现认证、授权等安全控制
2. **代码分割**：通过懒加载优化应用性能
3. **数据预加载**：使用 Loader 提升用户体验
4. **路由元信息**：提供额外的路由配置和上下文
5. **多语言支持**：构建国际化的路由系统

### 最佳实践

1. **安全性优先**：始终实施适当的认证和授权
2. **性能优化**：合理使用代码分割和懒加载
3. **用户体验**：提供良好的加载状态和错误处理
4. **可维护性**：保持路由配置清晰和模块化
5. **类型安全**：在 TypeScript 项目中严格定义类型

通过掌握这些进阶技术，开发者可以构建出功能丰富、性能优异、安全可靠的企业级 React 应用。React Router 的强大功能为现代前端开发提供了坚实的基础。