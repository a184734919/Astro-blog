---
title: React 生命周期
published: 2023-07-13
description: '类组件生命周期方法的详细介绍和学习笔记'
image: ''
tags: ["React","生命周期"]
category: 'React'
draft: false
lang: 'zh-CN'
---

# React 生命周期

## 概述

React 组件的生命周期指的是组件从创建到销毁的整个过程。理解组件的生命周期对于掌握 React 的工作原理、优化性能以及正确处理副作用至关重要。随着 React Hooks 的引入，生命周期概念在函数组件中得到了重新定义，但理解传统的类组件生命周期仍然有助于深入理解 React 的工作机制。

### 生命周期的三个阶段

**挂载阶段（Mounting）**：组件实例被创建并插入到 DOM 中

**更新阶段（Updating）**：组件的 props 或 state 发生变化，导致组件重新渲染

**卸载阶段（Unmounting）**：组件从 DOM 中移除，组件实例被销毁

**错误处理（Error Handling）**：在渲染过程中、生命周期方法中或子组件构造函数中发生错误时调用

## 核心概念

### 1. 类组件生命周期方法

```jsx
class LifecycleComponent extends React.Component {
  // 1. 挂载阶段

  // 构造函数：在组件挂载前调用，用于初始化 state 和绑定方法
  constructor(props) {
    super(props);
    this.state = {
      count: 0,
      data: null
    };

    // 绑定事件处理器
    this.handleClick = this.handleClick.bind(this);
    this.handleResize = this.handleResize.bind(this);
  }

  // 静态方法：在挂载前调用，返回一个对象来更新 state
  // 可以根据 props 来初始化 state
  static getDerivedStateFromProps(props, state) {
    console.log('getDerivedStateFromProps');
    if (props.userId !== state.userId) {
      return {
        userId: props.userId,
        data: null // 重置数据
      };
    }
    return null;
  }

  // 在组件挂载后立即调用
  // 适合进行 DOM 操作、数据获取、订阅等操作
  componentDidMount() {
    console.log('componentDidMount');

    // 获取数据
    this.fetchData();

    // 添加事件监听
    window.addEventListener('resize', this.handleResize);

    // 设置定时器
    this.timer = setInterval(() => {
      this.setState(prevState => ({ count: prevState.count + 1 }));
    }, 1000);
  }

  // 2. 更新阶段

  // 在每次渲染前调用，返回一个对象来更新 state
  // 应该用于纯函数，避免副作用
  static getDerivedStateFromProps(props, state) {
    console.log('getDerivedStateFromProps (update)');
    return null;
  }

  // 在渲染后调用，返回一个布尔值决定是否更新组件
  // 用于性能优化，避免不必要的重新渲染
  shouldComponentUpdate(nextProps, nextState) {
    console.log('shouldComponentUpdate');
    // 只在特定条件下更新
    return nextProps.id !== this.props.id || nextState.count !== this.state.count;
  }

  // 在渲染后、DOM 更新前调用
  // 返回一个对象，用于更新 DOM（已废弃，推荐使用 getSnapshotBeforeUpdate）
  getSnapshotBeforeUpdate(prevProps, prevState) {
    console.log('getSnapshotBeforeUpdate');
    // 可以在这里获取 DOM 更新前的信息
    if (prevState.list.length < this.state.list.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }

  // 在组件更新后立即调用
  // 适合进行 DOM 操作或网络请求
  componentDidUpdate(prevProps, prevState, snapshot) {
    console.log('componentDidUpdate');

    // 检查特定的 props 或 state 变化
    if (this.props.userId !== prevProps.userId) {
      this.fetchData();
    }

    // 使用 snapshot 恢复滚动位置
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }

  // 3. 卸载阶段

  // 在组件卸载和销毁之前调用
  // 用于清理操作，如取消网络请求、清除定时器、移除事件监听
  componentWillUnmount() {
    console.log('componentWillUnmount');

    // 清除定时器
    clearInterval(this.timer);

    // 移除事件监听
    window.removeEventListener('resize', this.handleResize);

    // 取消网络请求
    if (this.abortController) {
      this.abortController.abort();
    }
  }

  // 4. 错误处理

  // 捕获子组件树中的 JavaScript 错误
  static getDerivedStateFromError(error) {
    console.log('getDerivedStateFromError', error);
    // 更新 state 以显示错误信息
    return { hasError: true, error };
  }

  // 在错误发生后调用，用于记录错误信息
  componentDidCatch(error, errorInfo) {
    console.log('componentDidCatch', error, errorInfo);
    // 可以将错误记录到错误报告服务
    logErrorToService(error, errorInfo);
  }

  // 渲染方法
  render() {
    console.log('render');

    if (this.state.hasError) {
      return <ErrorFallback error={this.state.error} />;
    }

    return (
      <div>
        <h1>Count: {this.state.count}</h1>
        <button onClick={this.handleClick}>Increment</button>
      </div>
    );
  }

  // 自定义方法
  handleClick() {
    this.setState(prevState => ({ count: prevState.count + 1 }));
  }

  handleResize() {
    console.log('Window resized');
  }

  async fetchData() {
    try {
      this.abortController = new AbortController();
      const response = await fetch('/api/data', {
        signal: this.abortController.signal
      });
      const data = await response.json();
      this.setState({ data });
    } catch (error) {
      if (error.name !== 'AbortError') {
        console.error('Fetch error:', error);
      }
    }
  }
}
```

### 2. 旧版生命周期方法（已废弃）

```jsx
class LegacyLifecycleComponent extends React.Component {
  // ❌ 已废弃：在挂载前和更新前调用
  // 容易导致多次调用和副作用，推荐使用 getDerivedStateFromProps 和 componentDidUpdate
  componentWillMount() {
    console.log('componentWillMount (已废弃)');
    // 不应该在这里进行副作用操作
  }

  // ❌ 已废弃：在组件接收到新的 props 时调用
  // 容易导致多次调用，推荐使用 getDerivedStateFromProps 和 componentDidUpdate
  componentWillReceiveProps(nextProps) {
    console.log('componentWillReceiveProps (已废弃)');
    if (nextProps.id !== this.props.id) {
      this.setState({ data: null });
    }
  }

  // ❌ 已废弃：在组件更新前调用
  // 容易导致性能问题，推荐使用 getSnapshotBeforeUpdate
  componentWillUpdate(nextProps, nextState) {
    console.log('componentWillUpdate (已废弃)');
    // 不应该在这里进行副作用操作
  }

  render() {
    return <div>Legacy Component</div>;
  }
}
```

### 3. Hooks 中的生命周期对应

```jsx
import { useState, useEffect, useRef, useLayoutEffect } from 'react';

function HooksLifecycleComponent(props) {
  const [count, setCount] = useState(0);
  const [data, setData] = useState(null);
  const timerRef = useRef(null);
  const prevPropsRef = useRef(props);

  // componentDidMount + componentDidUpdate
  useEffect(() => {
    console.log('useEffect (mount or update)');

    // 相当于 componentDidMount
    if (!data) {
      fetchData();
    }

    // 相当于 componentDidUpdate
    if (prevPropsRef.current.userId !== props.userId) {
      fetchData();
    }

    // 更新 ref 以存储上次的 props
    prevPropsRef.current = props;

    // 清理函数相当于 componentWillUnmount
    return () => {
      console.log('useEffect cleanup (unmount)');
      if (timerRef.current) {
        clearInterval(timerRef.current);
      }
    };
  }, [props.userId]); // 依赖数组

  // 只在挂载时执行（相当于 componentDidMount）
  useEffect(() => {
    console.log('useEffect (mount only)');

    window.addEventListener('resize', handleResize);
    timerRef.current = setInterval(() => {
      setCount(prev => prev + 1);
    }, 1000);

    return () => {
      window.removeEventListener('resize', handleResize);
      clearInterval(timerRef.current);
    };
  }, []); // 空依赖数组

  // 只在特定状态变化时执行（相当于 componentDidUpdate）
  useEffect(() => {
    console.log('useEffect (data changed)');
    if (data) {
      console.log('Data loaded:', data);
    }
  }, [data]); // 只在 data 变化时执行

  // 同步执行（相当于 getSnapshotBeforeUpdate + componentDidUpdate）
  useLayoutEffect(() => {
    console.log('useLayoutEffect');
    // 在 DOM 更新后、浏览器绘制前同步执行
    // 适合需要读取 DOM 布局的场景
  }, []);

  // 自定义 Hook 模拟 shouldComponentUpdate
  function useShouldComponentUpdate(nextProps, nextState) {
    const prevPropsRef = useRef(props);
    const prevStateRef = useRef({ count });

    const shouldUpdate =
      nextProps.id !== prevPropsRef.current.id ||
      nextState.count !== prevStateRef.current.count;

    prevPropsRef.current = nextProps;
    prevStateRef.current = nextState;

    return shouldUpdate;
  }

  const shouldUpdate = useShouldComponentUpdate(props, { count });

  if (!shouldUpdate) {
    return null; // 不更新
  }

  return (
    <div>
      <h1>Count: {count}</h1>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

## 基本用法

### 1. 数据获取和订阅

```jsx
// 类组件方式
class DataFetchingComponent extends React.Component {
  state = {
    data: null,
    loading: true,
    error: null
  };

  abortController = null;

  componentDidMount() {
    this.fetchData();
    this.setupWebSocket();
  }

  componentDidUpdate(prevProps) {
    if (this.props.userId !== prevProps.userId) {
      this.fetchData();
    }
  }

  componentWillUnmount() {
    if (this.abortController) {
      this.abortController.abort();
    }
    this.cleanupWebSocket();
  }

  async fetchData() {
    this.setState({ loading: true, error: null });

    try {
      this.abortController = new AbortController();
      const response = await fetch(`/api/users/${this.props.userId}`, {
        signal: this.abortController.signal
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const data = await response.json();
      this.setState({ data, loading: false });
    } catch (error) {
      if (error.name !== 'AbortError') {
        this.setState({
          error: error.message,
          loading: false
        });
      }
    }
  }

  setupWebSocket() {
    this.ws = new WebSocket('ws://localhost:8080');

    this.ws.onopen = () => {
      console.log('WebSocket connected');
    };

    this.ws.onmessage = (event) => {
      const message = JSON.parse(event.data);
      this.handleWebSocketMessage(message);
    };

    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
  }

  cleanupWebSocket() {
    if (this.ws) {
      this.ws.close();
      this.ws = null;
    }
  }

  handleWebSocketMessage(message) {
    // 处理 WebSocket 消息
    console.log('Received message:', message);
  }

  render() {
    const { data, loading, error } = this.state;

    if (loading) {
      return <div>Loading...</div>;
    }

    if (error) {
      return <div>Error: {error}</div>;
    }

    return (
      <div>
        <h1>User Profile</h1>
        <pre>{JSON.stringify(data, null, 2)}</pre>
      </div>
    );
  }
}

// Hooks 方式
function DataFetchingComponent({ userId }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const wsRef = useRef(null);
  const abortControllerRef = useRef(null);

  // 数据获取
  useEffect(() => {
    const fetchData = async () => {
      setLoading(true);
      setError(null);

      try {
        abortControllerRef.current = new AbortController();
        const response = await fetch(`/api/users/${userId}`, {
          signal: abortControllerRef.current.signal
        });

        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }

        const data = await response.json();
        setData(data);
        setLoading(false);
      } catch (error) {
        if (error.name !== 'AbortError') {
          setError(error.message);
          setLoading(false);
        }
      }
    };

    fetchData();

    return () => {
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, [userId]);

  // WebSocket 连接
  useEffect(() => {
    const ws = new WebSocket('ws://localhost:8080');
    wsRef.current = ws;

    ws.onopen = () => {
      console.log('WebSocket connected');
    };

    ws.onmessage = (event) => {
      const message = JSON.parse(event.data);
      console.log('Received message:', message);
    };

    ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };

    return () => {
      if (wsRef.current) {
        wsRef.current.close();
      }
    };
  }, []);

  if (loading) {
    return <div>Loading...</div>;
  }

  if (error) {
    return <div>Error: {error}</div>;
  }

  return (
    <div>
      <h1>User Profile</h1>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
}
```

### 2. 性能优化

```jsx
// 类组件方式
class OptimizedComponent extends React.Component {
  state = {
    data: [],
    filter: '',
    sortBy: 'name'
  };

  // 优化：避免不必要的重新渲染
  shouldComponentUpdate(nextProps, nextState) {
    // 只有在这些值变化时才更新
    return (
      nextProps.items !== this.props.items ||
      nextState.filter !== this.state.filter ||
      nextState.sortBy !== this.state.sortBy
    );
  }

  // 优化：使用 getDerivedStateFromProps 处理 props 到 state 的同步
  static getDerivedStateFromProps(props, state) {
    // 当 items 变化时重新计算数据
    if (props.items !== state.items) {
      return {
        items: props.items,
        data: processData(props.items, state.filter, state.sortBy)
      };
    }
    return null;
  }

  // 优化：使用 getSnapshotBeforeUpdate 获取 DOM 更新前的信息
  getSnapshotBeforeUpdate(prevProps, prevState) {
    if (prevState.data.length < this.state.data.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }

  // 优化：在 componentDidUpdate 中使用 snapshot
  componentDidUpdate(prevProps, prevState, snapshot) {
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }

  handleFilterChange = (filter) => {
    this.setState({
      filter,
      data: processData(this.props.items, filter, this.state.sortBy)
    });
  };

  handleSortChange = (sortBy) => {
    this.setState({
      sortBy,
      data: processData(this.props.items, this.state.filter, sortBy)
    });
  };

  render() {
    return (
      <div>
        <input
          value={this.state.filter}
          onChange={(e) => this.handleFilterChange(e.target.value)}
          placeholder="Filter items"
        />
        <select
          value={this.state.sortBy}
          onChange={(e) => this.handleSortChange(e.target.value)}
        >
          <option value="name">Sort by Name</option>
          <option value="date">Sort by Date</option>
        </select>
        <ul ref={this.listRef}>
          {this.state.data.map(item => (
            <li key={item.id}>{item.name}</li>
          ))}
        </ul>
      </div>
    );
  }
}

// Hooks 方式
function OptimizedComponent({ items }) {
  const [filter, setFilter] = useState('');
  const [sortBy, setSortBy] = useState('name');
  const listRef = useRef(null);
  const prevDataLengthRef = useRef(0);

  // 优化：使用 useMemo 缓存处理后的数据
  const processedData = useMemo(() => {
    return processData(items, filter, sortBy);
  }, [items, filter, sortBy]);

  // 优化：使用 useCallback 缓存事件处理器
  const handleFilterChange = useCallback((event) => {
    setFilter(event.target.value);
  }, []);

  const handleSortChange = useCallback((event) => {
    setSortBy(event.target.value);
  }, []);

  // 优化：使用 useLayoutEffect 处理滚动位置
  useLayoutEffect(() => {
    if (prevDataLengthRef.current < processedData.length) {
      const list = listRef.current;
      if (list) {
        const scrollOffset = list.scrollHeight - list.scrollTop;
        // 保持滚动位置
      }
    }
    prevDataLengthRef.current = processedData.length;
  }, [processedData.length]);

  return (
    <div>
      <input
        value={filter}
        onChange={handleFilterChange}
        placeholder="Filter items"
      />
      <select value={sortBy} onChange={handleSortChange}>
        <option value="name">Sort by Name</option>
        <option value="date">Sort by Date</option>
      </select>
      <ul ref={listRef}>
        {processedData.map(item => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}

// 辅助函数
function processData(items, filter, sortBy) {
  let result = items;

  // 过滤
  if (filter) {
    result = result.filter(item =>
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }

  // 排序
  result = [...result].sort((a, b) => {
    if (a[sortBy] < b[sortBy]) return -1;
    if (a[sortBy] > b[sortBy]) return 1;
    return 0;
  });

  return result;
}
```

### 3. 动画和 DOM 操作

```jsx
// 类组件方式
class AnimatedComponent extends React.Component {
  state = {
    isVisible: true,
    animationClass: ''
  };

  elementRef = React.createRef();

  componentDidMount() {
    // 组件挂载后开始动画
    setTimeout(() => {
      this.setState({ animationClass: 'fade-in' });
    }, 100);
  }

  componentDidUpdate(prevProps, prevState) {
    // 处理可见性变化
    if (prevState.isVisible !== this.state.isVisible) {
      if (this.state.isVisible) {
        this.startEnterAnimation();
      } else {
        this.startExitAnimation();
      }
    }
  }

  startEnterAnimation() {
    const element = this.elementRef.current;
    if (element) {
      element.style.opacity = '0';
      element.style.transform = 'translateY(-20px)';

      requestAnimationFrame(() => {
        element.style.transition = 'all 0.3s ease-out';
        element.style.opacity = '1';
        element.style.transform = 'translateY(0)';
      });
    }
  }

  startExitAnimation() {
    const element = this.elementRef.current;
    if (element) {
      element.style.transition = 'all 0.3s ease-in';
      element.style.opacity = '0';
      element.style.transform = 'translateY(-20px)';

      setTimeout(() => {
        this.setState({ animationClass: '' });
      }, 300);
    }
  }

  handleToggle = () => {
    this.setState(prevState => ({ isVisible: !prevState.isVisible }));
  };

  render() {
    if (!this.state.isVisible && !this.state.animationClass) {
      return null;
    }

    return (
      <div>
        <button onClick={this.handleToggle}>
          Toggle Visibility
        </button>
        <div
          ref={this.elementRef}
          className={`animated-element ${this.state.animationClass}`}
        >
          Animated Content
        </div>
      </div>
    );
  }
}

// Hooks 方式
function AnimatedComponent() {
  const [isVisible, setIsVisible] = useState(true);
  const [isAnimating, setIsAnimating] = useState(false);
  const elementRef = useRef(null);

  // 进入动画
  useEffect(() => {
    if (isVisible) {
      setIsAnimating(true);
      const element = elementRef.current;

      if (element) {
        element.style.opacity = '0';
        element.style.transform = 'translateY(-20px)';

        requestAnimationFrame(() => {
          element.style.transition = 'all 0.3s ease-out';
          element.style.opacity = '1';
          element.style.transform = 'translateY(0)';

          setTimeout(() => {
            setIsAnimating(false);
          }, 300);
        });
      }
    }
  }, [isVisible]);

  // 退出动画
  const handleToggle = useCallback(() => {
    if (isVisible) {
      const element = elementRef.current;
      if (element) {
        setIsAnimating(true);
        element.style.transition = 'all 0.3s ease-in';
        element.style.opacity = '0';
        element.style.transform = 'translateY(-20px)';

        setTimeout(() => {
          setIsVisible(false);
          setIsAnimating(false);
        }, 300);
      }
    } else {
      setIsVisible(true);
    }
  }, [isVisible]);

  if (!isVisible && !isAnimating) {
    return (
      <div>
        <button onClick={handleToggle}>Show</button>
      </div>
    );
  }

  return (
    <div>
      <button onClick={handleToggle}>
        {isVisible ? 'Hide' : 'Show'}
      </button>
      <div
        ref={elementRef}
        className="animated-element"
      >
        Animated Content
      </div>
    </div>
  );
}
```

## 实际应用

### 1. 无限滚动组件

```jsx
class InfiniteScrollComponent extends React.Component {
  state = {
    items: [],
    loading: false,
    hasMore: true,
    page: 1
  };

  listRef = React.createRef();
  observer = null;
  abortController = null;

  componentDidMount() {
    this.loadItems();
    this.setupIntersectionObserver();
  }

  componentWillUnmount() {
    this.cleanupIntersectionObserver();
    if (this.abortController) {
      this.abortController.abort();
    }
  }

  setupIntersectionObserver() {
    const options = {
      root: null,
      rootMargin: '20px',
      threshold: 0.1
    };

    this.observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting && !this.state.loading && this.state.hasMore) {
          this.loadItems();
        }
      });
    }, options);

    if (this.listRef.current) {
      this.observer.observe(this.listRef.current);
    }
  }

  cleanupIntersectionObserver() {
    if (this.observer) {
      this.observer.disconnect();
      this.observer = null;
    }
  }

  async loadItems() {
    if (this.state.loading) return;

    this.setState({ loading: true });

    try {
      this.abortController = new AbortController();

      const response = await fetch(
        `/api/items?page=${this.state.page}&limit=${this.props.pageSize}`,
        { signal: this.abortController.signal }
      );

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const newItems = await response.json();

      this.setState(prevState => ({
        items: [...prevState.items, ...newItems],
        loading: false,
        hasMore: newItems.length === this.props.pageSize,
        page: prevState.page + 1
      }));
    } catch (error) {
      if (error.name !== 'AbortError') {
        console.error('Error loading items:', error);
        this.setState({ loading: false });
      }
    }
  }

  render() {
    const { items, loading, hasMore } = this.state;

    return (
      <div className="infinite-scroll">
        <div className="item-list">
          {items.map(item => (
            <div key={item.id} className="item">
              <h3>{item.title}</h3>
              <p>{item.description}</p>
            </div>
          ))}
        </div>

        {loading && <div className="loading">Loading...</div>}

        {!hasMore && <div className="end">No more items</div>}

        <div ref={this.listRef} className="sentinel" />
      </div>
    );
  }
}

// Hooks 版本
function InfiniteScrollComponent({ pageSize = 10 }) {
  const [items, setItems] = useState([]);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);
  const [page, setPage] = useState(1);

  const listRef = useRef(null);
  const abortControllerRef = useRef(null);

  // 加载数据
  const loadItems = useCallback(async () => {
    if (loading) return;

    setLoading(true);

    try {
      abortControllerRef.current = new AbortController();

      const response = await fetch(
        `/api/items?page=${page}&limit=${pageSize}`,
        { signal: abortControllerRef.current.signal }
      );

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const newItems = await response.json();

      setItems(prev => [...prev, ...newItems]);
      setHasMore(newItems.length === pageSize);
      setPage(prev => prev + 1);
      setLoading(false);
    } catch (error) {
      if (error.name !== 'AbortError') {
        console.error('Error loading items:', error);
      }
      setLoading(false);
    }
  }, [loading, page, pageSize]);

  // 设置 Intersection Observer
  useEffect(() => {
    const options = {
      root: null,
      rootMargin: '20px',
      threshold: 0.1
    };

    const observer = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting && !loading && hasMore) {
          loadItems();
        }
      });
    }, options);

    if (listRef.current) {
      observer.observe(listRef.current);
    }

    return () => {
      observer.disconnect();
      if (abortControllerRef.current) {
        abortControllerRef.current.abort();
      }
    };
  }, [loading, hasMore, loadItems]);

  return (
    <div className="infinite-scroll">
      <div className="item-list">
        {items.map(item => (
          <div key={item.id} className="item">
            <h3>{item.title}</h3>
            <p>{item.description}</p>
          </div>
        ))}
      </div>

      {loading && <div className="loading">Loading...</div>}

      {!hasMore && <div className="end">No more items</div>}

      <div ref={listRef} className="sentinel" />
    </div>
  );
}
```

### 2. 实时数据同步组件

```jsx
class RealTimeSyncComponent extends React.Component {
  state = {
    data: [],
    connected: false,
    error: null,
    lastUpdate: null
  };

  ws = null;
  reconnectAttempts = 0;
  maxReconnectAttempts = 5;
  reconnectTimeout = null;

  componentDidMount() {
    this.connectWebSocket();
    this.startPolling();
  }

  componentWillUnmount() {
    this.disconnectWebSocket();
    this.stopPolling();
    this.clearReconnectTimeout();
  }

  connectWebSocket() {
    try {
      this.ws = new WebSocket(this.props.wsUrl);

      this.ws.onopen = () => {
        console.log('WebSocket connected');
        this.setState({
          connected: true,
          error: null
        });
        this.reconnectAttempts = 0;
      };

      this.ws.onmessage = (event) => {
        const message = JSON.parse(event.data);
        this.handleMessage(message);
      };

      this.ws.onerror = (error) => {
        console.error('WebSocket error:', error);
        this.setState({ error: 'Connection error' });
      };

      this.ws.onclose = () => {
        console.log('WebSocket disconnected');
        this.setState({ connected: false });
        this.handleReconnect();
      };
    } catch (error) {
      console.error('Failed to create WebSocket:', error);
      this.handleReconnect();
    }
  }

  disconnectWebSocket() {
    if (this.ws) {
      this.ws.close();
      this.ws = null;
    }
  }

  handleReconnect() {
    if (this.reconnectAttempts < this.maxReconnectAttempts) {
      this.reconnectAttempts++;
      const delay = Math.min(1000 * Math.pow(2, this.reconnectAttempts), 30000);

      this.reconnectTimeout = setTimeout(() => {
        console.log(`Reconnecting... Attempt ${this.reconnectAttempts}`);
        this.connectWebSocket();
      }, delay);
    } else {
      this.setState({
        error: 'Failed to reconnect after multiple attempts'
      });
    }
  }

  clearReconnectTimeout() {
    if (this.reconnectTimeout) {
      clearTimeout(this.reconnectTimeout);
      this.reconnectTimeout = null;
    }
  }

  handleMessage(message) {
    switch (message.type) {
      case 'update':
        this.setState(prevState => ({
          data: this.updateData(prevState.data, message.payload),
          lastUpdate: new Date()
        }));
        break;

      case 'delete':
        this.setState(prevState => ({
          data: prevState.data.filter(item => item.id !== message.payload.id),
          lastUpdate: new Date()
        }));
        break;

      case 'error':
        this.setState({ error: message.payload.message });
        break;

      default:
        console.log('Unknown message type:', message.type);
    }
  }

  updateData(data, payload) {
    // 更新或添加数据
    const index = data.findIndex(item => item.id === payload.id);

    if (index !== -1) {
      // 更新现有数据
      const newData = [...data];
      newData[index] = { ...newData[index], ...payload };
      return newData;
    } else {
      // 添加新数据
      return [...data, payload];
    }
  }

  startPolling() {
    this.pollingInterval = setInterval(() => {
      if (!this.state.connected) {
        this.pollData();
      }
    }, this.props.pollInterval || 5000);
  }

  stopPolling() {
    if (this.pollingInterval) {
      clearInterval(this.pollingInterval);
      this.pollingInterval = null;
    }
  }

  async pollData() {
    try {
      const response = await fetch(this.props.apiUrl);
      const data = await response.json();

      this.setState({
        data,
        lastUpdate: new Date()
      });
    } catch (error) {
      console.error('Polling error:', error);
      this.setState({ error: 'Failed to fetch data' });
    }
  }

  render() {
    const { data, connected, error, lastUpdate } = this.state;

    return (
      <div className="real-time-sync">
        <div className="status-bar">
          <span className={`status ${connected ? 'connected' : 'disconnected'}`}>
            {connected ? '● Connected' : '○ Disconnected'}
          </span>
          {lastUpdate && (
            <span className="last-update">
              Last update: {lastUpdate.toLocaleTimeString()}
            </span>
          )}
        </div>

        {error && (
          <div className="error-message">
            Error: {error}
            <button onClick={() => this.connectWebSocket()}>
              Retry
            </button>
          </div>
        )}

        <div className="data-list">
          {data.map(item => (
            <div key={item.id} className="data-item">
              <h3>{item.title}</h3>
              <p>{item.content}</p>
              <small>Updated: {new Date(item.updatedAt).toLocaleString()}</small>
            </div>
          ))}
        </div>
      </div>
    );
  }
}

// Hooks 版本
function RealTimeSyncComponent({ wsUrl, apiUrl, pollInterval = 5000 }) {
  const [data, setData] = useState([]);
  const [connected, setConnected] = useState(false);
  const [error, setError] = useState(null);
  const [lastUpdate, setLastUpdate] = useState(null);

  const wsRef = useRef(null);
  const pollingIntervalRef = useRef(null);
  const reconnectAttemptsRef = useRef(0);
  const reconnectTimeoutRef = useRef(null);

  // 连接 WebSocket
  useEffect(() => {
    const connectWebSocket = () => {
      try {
        const ws = new WebSocket(wsUrl);
        wsRef.current = ws;

        ws.onopen = () => {
          console.log('WebSocket connected');
          setConnected(true);
          setError(null);
          reconnectAttemptsRef.current = 0;
        };

        ws.onmessage = (event) => {
          const message = JSON.parse(event.data);
          handleMessage(message);
        };

        ws.onerror = (error) => {
          console.error('WebSocket error:', error);
          setError('Connection error');
        };

        ws.onclose = () => {
          console.log('WebSocket disconnected');
          setConnected(false);
          handleReconnect();
        };
      } catch (error) {
        console.error('Failed to create WebSocket:', error);
        handleReconnect();
      }
    };

    connectWebSocket();

    return () => {
      if (wsRef.current) {
        wsRef.current.close();
        wsRef.current = null;
      }
    };
  }, [wsUrl]);

  // 消息处理
  const handleMessage = useCallback((message) => {
    switch (message.type) {
      case 'update':
        setData(prevData => {
          const index = prevData.findIndex(item => item.id === message.payload.id);
          if (index !== -1) {
            const newData = [...prevData];
            newData[index] = { ...newData[index], ...message.payload };
            return newData;
          } else {
            return [...prevData, message.payload];
          }
        });
        setLastUpdate(new Date());
        break;

      case 'delete':
        setData(prevData => prevData.filter(item => item.id !== message.payload.id));
        setLastUpdate(new Date());
        break;

      case 'error':
        setError(message.payload.message);
        break;

      default:
        console.log('Unknown message type:', message.type);
    }
  }, []);

  // 重连逻辑
  const handleReconnect = useCallback(() => {
    const maxReconnectAttempts = 5;

    if (reconnectAttemptsRef.current < maxReconnectAttempts) {
      reconnectAttemptsRef.current++;
      const delay = Math.min(1000 * Math.pow(2, reconnectAttemptsRef.current), 30000);

      reconnectTimeoutRef.current = setTimeout(() => {
        console.log(`Reconnecting... Attempt ${reconnectAttemptsRef.current}`);
        if (wsRef.current) {
          wsRef.current = new WebSocket(wsUrl);
        }
      }, delay);
    } else {
      setError('Failed to reconnect after multiple attempts');
    }
  }, [wsUrl]);

  // 轮询备用方案
  useEffect(() => {
    if (connected) return;

    const pollData = async () => {
      try {
        const response = await fetch(apiUrl);
        const data = await response.json();
        setData(data);
        setLastUpdate(new Date());
      } catch (error) {
        console.error('Polling error:', error);
        setError('Failed to fetch data');
      }
    };

    pollingIntervalRef.current = setInterval(pollData, pollInterval);

    return () => {
      if (pollingIntervalRef.current) {
        clearInterval(pollingIntervalRef.current);
      }
    };
  }, [connected, apiUrl, pollInterval]);

  return (
    <div className="real-time-sync">
      <div className="status-bar">
        <span className={`status ${connected ? 'connected' : 'disconnected'}`}>
          {connected ? '● Connected' : '○ Disconnected'}
        </span>
        {lastUpdate && (
          <span className="last-update">
            Last update: {lastUpdate.toLocaleTimeString()}
          </span>
        )}
      </div>

      {error && (
        <div className="error-message">
          Error: {error}
        </div>
      )}

      <div className="data-list">
        {data.map(item => (
          <div key={item.id} className="data-item">
            <h3>{item.title}</h3>
            <p>{item.content}</p>
            <small>Updated: {new Date(item.updatedAt).toLocaleString()}</small>
          </div>
        ))}
      </div>
    </div>
  );
}
```

## 注意事项

### 1. 避免在生命周期中直接修改状态

```jsx
// ❌ 错误：在 render 中调用 setState
class BadComponent extends React.Component {
  state = { count: 0 };

  render() {
    if (this.props.shouldIncrement) {
      this.setState({ count: this.state.count + 1 }); // 会导致无限循环
    }
    return <div>{this.state.count}</div>;
  }

  // ✅ 正确：在 componentDidUpdate 中处理
  componentDidUpdate(prevProps) {
    if (this.props.shouldIncrement && !prevProps.shouldIncrement) {
      this.setState(prevState => ({ count: prevState.count + 1 }));
    }
  }
}

// ❌ 错误：在生命周期中进行副作用操作
class BadSideEffects extends React.Component {
  shouldComponentUpdate(nextProps) {
    if (nextProps.id !== this.props.id) {
      this.fetchData(); // 不应该在 shouldComponentUpdate 中进行副作用
    }
    return true;
  }

  // ✅ 正确：在 componentDidUpdate 中处理副作用
  componentDidUpdate(prevProps) {
    if (this.props.id !== prevProps.id) {
      this.fetchData();
    }
  }
}
```

### 2. 清理副作用

```jsx
// ❌ 错误：没有清理副作用
class BadCleanup extends React.Component {
  componentDidMount() {
    this.timer = setInterval(() => {
      console.log('Tick');
    }, 1000);

    window.addEventListener('resize', this.handleResize);
  }

  // 没有 componentWillUnmount 来清理

  // ✅ 正确：清理所有副作用
class GoodCleanup extends React.Component {
  timer = null;

  componentDidMount() {
    this.timer = setInterval(() => {
      console.log('Tick');
    }, 1000);

    window.addEventListener('resize', this.handleResize);
  }

  componentWillUnmount() {
    // 清理定时器
    if (this.timer) {
      clearInterval(this.timer);
      this.timer = null;
    }

    // 移除事件监听
    window.removeEventListener('resize', this.handleResize);

    // 取消网络请求
    if (this.abortController) {
      this.abortController.abort();
    }
  }
}

// Hooks 版本的清理
function GoodCleanupHooks() {
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('Tick');
    }, 1000);

    const handleResize = () => {
      console.log('Window resized');
    };

    window.addEventListener('resize', handleResize);

    // 清理函数
    return () => {
      clearInterval(timer);
      window.removeEventListener('resize', handleResize);
    };
  }, []);
}
```

### 3. 避免重复的副作用调用

```jsx
// ❌ 错误：可能重复调用副作用
class BadEffect extends React.Component {
  componentDidMount() {
    this.fetchData();
  }

  componentDidUpdate(prevProps) {
    if (this.props.id !== prevProps.id) {
      this.fetchData(); // 可能重复调用
    }
  }
}

// ✅ 正确：避免重复调用
class GoodEffect extends React.Component {
  componentDidMount() {
    this.fetchData();
  }

  componentDidUpdate(prevProps) {
    // 确保只在 id 变化时调用
    if (this.props.id !== prevProps.id) {
      this.fetchData();
    }
  }

  // 使用自定义方法避免重复代码
  fetchData() {
    // 数据获取逻辑
  }
}

// Hooks 版本：使用依赖数组避免重复调用
function GoodEffectHooks({ id }) {
  useEffect(() => {
    fetchData();
  }, [id]); // 只在 id 变化时调用
}
```

## 最佳实践

### 1. 使用 Hooks 替代类组件

```jsx
// ✅ 推荐：使用 Hooks
function ModernComponent({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let isMounted = true;

    const fetchUser = async () => {
      try {
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();

        if (isMounted) {
          setUser(data);
          setLoading(false);
        }
      } catch (error) {
        if (isMounted) {
          console.error('Error:', error);
          setLoading(false);
        }
      }
    };

    fetchUser();

    return () => {
      isMounted = false;
    };
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  return <div>{user?.name}</div>;
}

// 传统类组件
class LegacyComponent extends React.Component {
  state = {
    user: null,
    loading: true
  };

  componentDidMount() {
    this.fetchUser();
  }

  componentDidUpdate(prevProps) {
    if (this.props.userId !== prevProps.userId) {
      this.fetchUser();
    }
  }

  async fetchUser() {
    try {
      const response = await fetch(`/api/users/${this.props.userId}`);
      const data = await response.json();
      this.setState({ user: data, loading: false });
    } catch (error) {
      console.error('Error:', error);
      this.setState({ loading: false });
    }
  }

  render() {
    if (this.state.loading) return <div>Loading...</div>;
    return <div>{this.state.user?.name}</div>;
  }
}
```

### 2. 自定义 Hooks 复用生命周期逻辑

```jsx
// 自定义 Hook：数据获取
function useDataFetching(url, dependencies = []) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let isMounted = true;

    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        const data = await response.json();

        if (isMounted) {
          setData(data);
          setError(null);
        }
      } catch (error) {
        if (isMounted) {
          setError(error.message);
          setData(null);
        }
      } finally {
        if (isMounted) {
          setLoading(false);
        }
      }
    };

    fetchData();

    return () => {
      isMounted = false;
    };
  }, dependencies);

  return { data, loading, error };
}

// 自定义 Hook：窗口大小
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

// 使用自定义 Hooks
function UserProfile({ userId }) {
  const { data: user, loading, error } = useDataFetching(
    `/api/users/${userId}`,
    [userId]
  );

  const { width } = useWindowSize();

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>Window width: {width}px</p>
    </div>
  );
}
```

### 3. 错误边界处理

```jsx
class ErrorBoundary extends React.Component {
  state = {
    hasError: false,
    error: null,
    errorInfo: null
  };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    this.setState({
      error,
      errorInfo
    });

    // 记录错误到错误报告服务
    logErrorToService(error, errorInfo);
  }

  handleReset = () => {
    this.setState({
      hasError: false,
      error: null,
      errorInfo: null
    });
  };

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-fallback">
          <h2>Something went wrong</h2>
          <details>
            <summary>Error details</summary>
            <pre>{this.state.error?.toString()}</pre>
            <pre>{this.state.errorInfo?.componentStack}</pre>
          </details>
          <button onClick={this.handleReset}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// 使用错误边界
function App() {
  return (
    <ErrorBoundary>
      <ComponentThatMayError />
    </ErrorBoundary>
  );
}
```

## 总结

React 生命周期是理解组件行为的关键，掌握生命周期方法的使用对于构建高质量的 React 应用至关重要。

### 关键要点

1. **生命周期阶段**：理解挂载、更新、卸载三个阶段的特点和用途
2. **方法选择**：根据需求选择合适的生命周期方法
3. **副作用处理**：正确处理副作用并确保清理
4. **性能优化**：使用 shouldComponentUpdate 和 getDerivedStateFromProps 优化性能
5. **现代实践**：优先使用 Hooks 替代传统的类组件生命周期

### 学习建议

1. **理解机制**：深入理解 React 的渲染和更新机制
2. **实践为主**：通过实际项目练习各种生命周期场景
3. **性能意识**：学习识别和解决性能问题
4. **现代开发**：优先学习和使用 Hooks 相关的生命周期管理
5. **错误处理**：掌握错误边界和错误处理最佳实践

随着 React 的发展，生命周期管理方式也在不断演进。掌握传统的生命周期方法有助于理解 React 的工作原理，但现代 React 开发中应该优先使用 Hooks 来管理组件的生命周期和副作用。