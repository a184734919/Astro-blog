---
title: React Router 基础
published: 2023-09-23
description: 'React Router 6 使用的详细介绍和学习笔记'
image: ''
tags: ["React","Router"]
category: 'React'
draft: false
lang: 'zh-CN'
---

## 概述

React Router 是 React 应用中最流行的路由解决方案，它为单页应用（SPA）提供了声明式的路由配置、动态路由、嵌套路由等强大功能。React Router v6 带来了重大的更新和改进，使路由管理更加简洁和强大。本文将详细介绍 React Router 6 的基础概念和核心功能。

## 核心概念

### 路由的基本原理

React Router 基于浏览器的 History API，通过监听 URL 变化来动态渲染不同的组件，而不需要页面刷新。它提供了以下核心功能：

- **声明式路由**：使用 JSX 语法配置路由规则
- **动态路由匹配**：支持路由参数和嵌套路由
- **程序化导航**：通过编程方式控制导航
- **路由守卫**：保护特定路由的访问权限
- **代码分割**：支持懒加载优化性能

### React Router 6 主要变化

与 v5 相比，v6 的主要变化包括：
- 简化的组件 API（`Routes` 替代 `Switch`）
- 支持相对路由
- 嵌套路由更加自然
- 新的 `useNavigate` Hook 替代 `useHistory`
- 更好的 TypeScript 支持
- 移除了 `withRouter` 高阶组件

## 基本用法

### 基础路由配置

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// 简单的页面组件
function Home() {
  return <h1>Home Page</h1>;
}

function About() {
  return <h1>About Page</h1>;
}

function Contact() {
  return <h1>Contact Page</h1>;
}

function NotFound() {
  return <h1>404 - Page Not Found</h1>;
}

// 路由配置
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}
```

### 导航组件

```jsx
import { Link, NavLink, useNavigate } from 'react-router-dom';

function Navigation() {
  const navigate = useNavigate();

  const handleGoHome = () => {
    navigate('/');
  };

  const handleGoBack = () => {
    navigate(-1);
  };

  return (
    <nav>
      {/* Link 组件 - 基础导航 */}
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>
      <Link to="/contact">Contact</Link>

      {/* NavLink 组件 - 带激活状态的导航 */}
      <NavLink
        to="/about"
        className={({ isActive }) => isActive ? 'active' : ''}
      >
        About
      </NavLink>

      {/* 程序化导航 */}
      <button onClick={handleGoHome}>Go Home</button>
      <button onClick={handleGoBack}>Go Back</button>

      {/* 带状态的导航 */}
      <Link
        to="/profile"
        state={{ from: 'navigation' }}
      >
        Go to Profile
      </Link>
    </nav>
  );
}

// 使用 Navigation 组件
function App() {
  return (
    <BrowserRouter>
      <Navigation />
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/contact" element={<Contact />} />
      </Routes>
    </BrowserRouter>
  );
}
```

### 路由参数

```jsx
import { useParams, useSearchParams } from 'react-router-dom';

// URL 参数
function UserProfile() {
  const { userId } = useParams();

  return (
    <div>
      <h2>User Profile</h2>
      <p>User ID: {userId}</p>
    </div>
  );
}

// 查询参数
function ProductList() {
  const [searchParams, setSearchParams] = useSearchParams();

  const category = searchParams.get('category');
  const page = searchParams.get('page') || '1';

  const handleCategoryChange = (newCategory) => {
    setSearchParams({ category: newCategory, page: '1' });
  };

  const handlePageChange = (newPage) => {
    setSearchParams({ category, page: newPage });
  };

  return (
    <div>
      <h2>Products</h2>
      <p>Category: {category}</p>
      <p>Page: {page}</p>

      <select
        value={category || ''}
        onChange={(e) => handleCategoryChange(e.target.value)}
      >
        <option value="">All Categories</option>
        <option value="electronics">Electronics</option>
        <option value="clothing">Clothing</option>
        <option value="books">Books</option>
      </select>

      <button onClick={() => handlePageChange(parseInt(page) + 1)}>
        Next Page
      </button>
    </div>
  );
}

// 路由配置
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/users/:userId" element={<UserProfile />} />
        <Route path="/products" element={<ProductList />} />
      </Routes>
    </BrowserRouter>
  );
}
```

### 嵌套路由

```jsx
import { Outlet } from 'react-router-dom';

// 父路由组件
function Dashboard() {
  return (
    <div>
      <aside>
        <nav>
          <Link to="overview">Overview</Link>
          <Link to="settings">Settings</Link>
          <Link to="analytics">Analytics</Link>
        </nav>
      </aside>
      <main>
        {/* 子路由将在这里渲染 */}
        <Outlet />
      </main>
    </div>
  );
}

// 子路由组件
function DashboardOverview() {
  return <h2>Dashboard Overview</h2>;
}

function DashboardSettings() {
  return <h2>Dashboard Settings</h2>;
}

function DashboardAnalytics() {
  return <h2>Dashboard Analytics</h2>;
}

// 嵌套路由配置
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/dashboard" element={<Dashboard />}>
          <Route path="overview" element={<DashboardOverview />} />
          <Route path="settings" element={<DashboardSettings />} />
          <Route path="analytics" element={<DashboardAnalytics />} />
          {/* 默认子路由 */}
          <Route index element={<DashboardOverview />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

### 路由重定向

```jsx
import { Navigate } from 'react-router-dom';

// 基础重定向
function RedirectToHome() {
  return <Navigate to="/" replace />;
}

// 条件重定向
function ProtectedRoute({ children }) {
  const isAuthenticated = false; // 实际应用中从认证状态获取

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  return children;
}

// 带状态的重定向
function LoginRedirect() {
  return (
    <Navigate
      to="/login"
      state={{ from: '/protected' }}
      replace
    />
  );
}

// 在路由配置中使用
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/old-route" element={<Navigate to="/new-route" replace />} />

        <Route path="/protected" element={
          <ProtectedRoute>
            <ProtectedComponent />
          </ProtectedRoute>
        } />

        <Route path="/login" element={<LoginPage />} />
      </Routes>
    </BrowserRouter>
  );
}
```

## 实际应用

### 完整的应用结构

```jsx
import { BrowserRouter, Routes, Route, Outlet, Link, useLocation } from 'react-router-dom';

// 布局组件
function MainLayout() {
  const location = useLocation();

  return (
    <div className="app">
      <Header />
      <main className="content">
        <Outlet />
      </main>
      <Footer />
    </div>
  );
}

function Header() {
  return (
    <header>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/products">Products</Link>
        <Link to="/about">About</Link>
        <Link to="/contact">Contact</Link>
      </nav>
    </header>
  );
}

function Footer() {
  return (
    <footer>
      <p>© 2024 My App. All rights reserved.</p>
    </footer>
  );
}

// 页面组件
function HomePage() {
  return (
    <div>
      <h1>Welcome to My App</h1>
      <p>This is the home page content.</p>
    </div>
  );
}

function ProductsPage() {
  const products = [
    { id: 1, name: 'Product 1', price: 99.99 },
    { id: 2, name: 'Product 2', price: 149.99 },
    { id: 3, name: 'Product 3', price: 199.99 },
  ];

  return (
    <div>
      <h1>Products</h1>
      <div className="products-grid">
        {products.map(product => (
          <div key={product.id} className="product-card">
            <h3>{product.name}</h3>
            <p>${product.price}</p>
            <Link to={`/products/${product.id}`}>View Details</Link>
          </div>
        ))}
      </div>
    </div>
  );
}

function ProductDetail() {
  const { productId } = useParams();

  // 模拟获取产品详情
  const product = {
    id: productId,
    name: `Product ${productId}`,
    description: 'This is a detailed description of the product.',
    price: 99.99,
    features: ['Feature 1', 'Feature 2', 'Feature 3']
  };

  return (
    <div>
      <Link to="/products">← Back to Products</Link>
      <h1>{product.name}</h1>
      <p className="price">${product.price}</p>
      <p className="description">{product.description}</p>

      <h3>Features</h3>
      <ul>
        {product.features.map((feature, index) => (
          <li key={index}>{feature}</li>
        ))}
      </ul>
    </div>
  );
}

function AboutPage() {
  return (
    <div>
      <h1>About Us</h1>
      <p>We are a company dedicated to providing great products and services.</p>
    </div>
  );
}

function ContactPage() {
  return (
    <div>
      <h1>Contact Us</h1>
      <form>
        <div>
          <label>Name:</label>
          <input type="text" name="name" />
        </div>
        <div>
          <label>Email:</label>
          <input type="email" name="email" />
        </div>
        <div>
          <label>Message:</label>
          <textarea name="message" rows="5"></textarea>
        </div>
        <button type="submit">Send Message</button>
      </form>
    </div>
  );
}

function NotFoundPage() {
  return (
    <div>
      <h1>404 - Page Not Found</h1>
      <p>The page you are looking for doesn't exist.</p>
      <Link to="/">Go to Home Page</Link>
    </div>
  );
}

// 主应用组件
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<MainLayout />}>
          <Route index element={<HomePage />} />
          <Route path="products">
            <Route index element={<ProductsPage />} />
            <Route path=":productId" element={<ProductDetail />} />
          </Route>
          <Route path="about" element={<AboutPage />} />
          <Route path="contact" element={<ContactPage />} />
          <Route path="*" element={<NotFoundPage />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

### 管理后台路由结构

```jsx
import { BrowserRouter, Routes, Route, Outlet, Link, NavLink } from 'react-router-dom';

// 管理后台布局
function AdminLayout() {
  return (
    <div className="admin-layout">
      <AdminSidebar />
      <div className="admin-content">
        <AdminHeader />
        <main>
          <Outlet />
        </main>
      </div>
    </div>
  );
}

function AdminSidebar() {
  return (
    <aside className="admin-sidebar">
      <div className="logo">Admin Panel</div>
      <nav>
        <NavLink to="/admin">
          <DashboardIcon /> Dashboard
        </NavLink>
        <NavLink to="/admin/users">
          <UsersIcon /> Users
        </NavLink>
        <NavLink to="/admin/products">
          <ProductsIcon /> Products
        </NavLink>
        <NavLink to="/admin/orders">
          <OrdersIcon /> Orders
        </NavLink>
        <NavLink to="/admin/settings">
          <SettingsIcon /> Settings
        </NavLink>
      </nav>
    </aside>
  );
}

function AdminHeader() {
  return (
    <header className="admin-header">
      <h1>Admin Dashboard</h1>
      <div className="user-menu">
        <span>Admin User</span>
        <Link to="/">Exit Admin</Link>
      </div>
    </header>
  );
}

// 管理页面组件
function AdminDashboard() {
  return (
    <div>
      <h2>Dashboard Overview</h2>
      <div className="stats-grid">
        <div className="stat-card">
          <h3>Total Users</h3>
          <p className="stat-number">1,234</p>
        </div>
        <div className="stat-card">
          <h3>Total Orders</h3>
          <p className="stat-number">567</p>
        </div>
        <div className="stat-card">
          <h3>Revenue</h3>
          <p className="stat-number">$12,345</p>
        </div>
      </div>
    </div>
  );
}

function AdminUsers() {
  const [searchParams] = useSearchParams();
  const page = parseInt(searchParams.get('page')) || 1;
  const search = searchParams.get('search') || '';

  const users = [
    { id: 1, name: 'John Doe', email: 'john@example.com', role: 'Admin' },
    { id: 2, name: 'Jane Smith', email: 'jane@example.com', role: 'User' },
  ];

  return (
    <div>
      <div className="page-header">
        <h2>Users Management</h2>
        <button>Add New User</button>
      </div>

      <div className="filters">
        <input
          type="text"
          placeholder="Search users..."
          defaultValue={search}
        />
        <select>
          <option value="">All Roles</option>
          <option value="admin">Admin</option>
          <option value="user">User</option>
        </select>
      </div>

      <table className="data-table">
        <thead>
          <tr>
            <th>ID</th>
            <th>Name</th>
            <th>Email</th>
            <th>Role</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          {users.map(user => (
            <tr key={user.id}>
              <td>{user.id}</td>
              <td>{user.name}</td>
              <td>{user.email}</td>
              <td>{user.role}</td>
              <td>
                <Link to={`/admin/users/${user.id}`}>Edit</Link>
                <button>Delete</button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>

      <div className="pagination">
        <button disabled={page === 1}>Previous</button>
        <span>Page {page}</span>
        <button>Next</button>
      </div>
    </div>
  );
}

function AdminUserDetail() {
  const { userId } = useParams();

  return (
    <div>
      <div className="page-header">
        <Link to="/admin/users">← Back to Users</Link>
        <h2>Edit User {userId}</h2>
      </div>

      <form>
        <div>
          <label>Name:</label>
          <input type="text" name="name" />
        </div>
        <div>
          <label>Email:</label>
          <input type="email" name="email" />
        </div>
        <div>
          <label>Role:</label>
          <select name="role">
            <option value="user">User</option>
            <option value="admin">Admin</option>
          </select>
        </div>
        <button type="submit">Save Changes</button>
      </form>
    </div>
  );
}

// 管理路由配置
function AdminApp() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/admin/*" element={<AdminLayout />}>
          <Route index element={<AdminDashboard />} />
          <Route path="users">
            <Route index element={<AdminUsers />} />
            <Route path=":userId" element={<AdminUserDetail />} />
          </Route>
          <Route path="products" element={<div>Products Management</div>} />
          <Route path="orders" element={<div>Orders Management</div>} />
          <Route path="settings" element={<div>Settings</div>} />
        </Route>
        <Route path="/" element={<div>Please login to access admin panel</div>} />
      </Routes>
    </BrowserRouter>
  );
}
```

### 博客应用路由

```jsx
import { BrowserRouter, Routes, Route, Outlet, Link } from 'react-router-dom';

// 博客布局
function BlogLayout() {
  return (
    <div className="blog-layout">
      <BlogHeader />
      <div className="blog-content">
        <BlogSidebar />
        <main>
          <Outlet />
        </main>
      </div>
      <BlogFooter />
    </div>
  );
}

function BlogHeader() {
  return (
    <header className="blog-header">
      <h1>My Tech Blog</h1>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/categories">Categories</Link>
        <Link to="/about">About</Link>
      </nav>
    </header>
  );
}

function BlogSidebar() {
  const categories = ['React', 'JavaScript', 'CSS', 'Node.js'];

  return (
    <aside className="blog-sidebar">
      <div className="widget">
        <h3>Categories</h3>
        <ul>
          {categories.map(category => (
            <li key={category}>
              <Link to={`/categories/${category.toLowerCase()}`}>
                {category}
              </Link>
            </li>
          ))}
        </ul>
      </div>

      <div className="widget">
        <h3>Recent Posts</h3>
        <ul>
          <li><Link to="/posts/1">React Hooks Guide</Link></li>
          <li><Link to="/posts/2">JavaScript ES6 Features</Link></li>
          <li><Link to="/posts/3">CSS Grid Layout</Link></li>
        </ul>
      </div>
    </aside>
  );
}

// 博客页面组件
function BlogHome() {
  const posts = [
    { id: 1, title: 'React Hooks Guide', excerpt: 'Learn how to use React Hooks...', date: '2024-01-15' },
    { id: 2, title: 'JavaScript ES6 Features', excerpt: 'Explore modern JavaScript...', date: '2024-01-10' },
    { id: 3, title: 'CSS Grid Layout', excerpt: 'Master CSS Grid for layouts...', date: '2024-01-05' },
  ];

  return (
    <div>
      <h1>Latest Posts</h1>
      <div className="posts-list">
        {posts.map(post => (
          <article key={post.id} className="post-card">
            <Link to={`/posts/${post.id}`}>
              <h2>{post.title}</h2>
            </Link>
            <p className="post-meta">{post.date}</p>
            <p className="post-excerpt">{post.excerpt}</p>
            <Link to={`/posts/${post.id}`}>Read more →</Link>
          </article>
        ))}
      </div>
    </div>
  );
}

function BlogPost() {
  const { postId } = useParams();

  // 模拟文章数据
  const post = {
    id: postId,
    title: `Post ${postId}`,
    content: 'This is the full content of the blog post...',
    author: 'John Doe',
    date: '2024-01-15',
    tags: ['React', 'JavaScript']
  };

  return (
    <article className="blog-post">
      <h1>{post.title}</h1>
      <div className="post-meta">
        <span>{post.author}</span>
        <span>{post.date}</span>
        <div className="tags">
          {post.tags.map(tag => (
            <Link key={tag} to={`/tags/${tag}`}>{tag}</Link>
          ))}
        </div>
      </div>
      <div className="post-content">
        <p>{post.content}</p>
      </div>
      <div className="post-navigation">
        <Link to="/">← Back to Posts</Link>
      </div>
    </article>
  );
}

function CategoryPosts() {
  const { categoryName } = useParams();

  return (
    <div>
      <h1>Category: {categoryName}</h1>
      <p>Posts in {categoryName} category</p>
    </div>
  );
}

// 博客路由配置
function BlogApp() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<BlogLayout />}>
          <Route index element={<BlogHome />} />
          <Route path="posts">
            <Route path=":postId" element={<BlogPost />} />
          </Route>
          <Route path="categories">
            <Route path=":categoryName" element={<CategoryPosts />} />
          </Route>
          <Route path="tags">
            <Route path=":tagName" element={<div>Tag Posts</div>} />
          </Route>
          <Route path="about" element={<div>About Page</div>} />
          <Route path="*" element={<div>404 - Page Not Found</div>} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

## 注意事项

### 路由匹配顺序

```jsx
// 错误示例：顺序不当
<Routes>
  <Route path="/users/:userId" element={<UserProfile />} />
  <Route path="/users/new" element={<NewUser />} /> {/* 永远不会匹配 */}
</Routes>

// 正确示例：具体路由在前
<Routes>
  <Route path="/users/new" element={<NewUser />} />
  <Route path="/users/:userId" element={<UserProfile />} />
</Routes>
```

### 相对路径使用

```jsx
function UsersPage() {
  return (
    <div>
      <h1>Users</h1>

      {/* 相对路径 - 相对于当前路由 */}
      <Link to="new">Add New User</Link>

      {/* 绝对路径 - 从根路径开始 */}
      <Link to="/admin/users/new">Add New User (Absolute)</Link>

      {/* 嵌套路由 */}
      <Routes>
        <Route path="new" element={<NewUserForm />} />
        <Route path=":userId" element={<UserProfile />} />
      </Routes>
    </div>
  );
}
```

### 避免重复渲染

```jsx
// 问题：Layout 组件在路由变化时重新渲染
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/users" element={<UsersLayout />}>
          <Route index element={<UsersList />} />
          <Route path=":userId" element={<UserDetail />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}

// 解决方案：使用 React.memo 或提取常量
const UsersLayout = React.memo(function UsersLayout() {
  return (
    <div>
      <nav>Users Navigation</nav>
      <Outlet />
    </div>
  );
});
```

### 路由参数类型安全

```jsx
// TypeScript 类型定义
interface UserParams {
  userId: string;
}

function UserProfile() {
  const { userId } = useParams<UserParams>();

  // 验证参数
  useEffect(() => {
    if (!userId || isNaN(parseInt(userId))) {
      // 处理无效参数
      navigate('/users');
    }
  }, [userId, navigate]);

  // 使用参数
  const numericId = parseInt(userId!);

  return <div>User ID: {numericId}</div>;
}
```

## 总结

React Router 6 提供了强大而灵活的路由解决方案，掌握其基础概念和用法对于构建现代 React 应用至关重要：

### 核心要点

1. **路由配置**：使用 `BrowserRouter`、`Routes` 和 `Route` 组件配置路由
2. **导航方式**：使用 `Link`、`NavLink` 组件和 `useNavigate` Hook
3. **参数处理**：使用 `useParams` 和 `useSearchParams` 处理路由参数
4. **嵌套路由**：使用 `Outlet` 组件实现嵌套路由布局
5. **路由保护**：使用 `Navigate` 组件实现路由重定向和权限控制

### 最佳实践

1. **合理设计路由结构**：保持路由层次清晰，避免过深的嵌套
2. **使用嵌套路由**：利用嵌套路由简化布局组件
3. **处理 404 情况**：始终提供 fallback 路由处理未匹配的路径
4. **考虑用户体验**：使用 NavLink 提供导航状态反馈
5. **类型安全**：在 TypeScript 项目中正确定义路由参数类型

通过合理应用这些概念和技术，可以构建出结构清晰、用户体验良好的单页应用路由系统。React Router 6 的改进使得路由管理变得更加直观和强大，值得在项目中广泛应用。