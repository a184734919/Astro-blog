---
title: React useMemo
published: 2023-08-10
description: '深入理解 React useMemo Hook：计算结果缓存和性能优化'
image: ''
tags: ["React","Hooks"]
category: 'React'
draft: false
lang: 'zh-CN'
---

## 概述

useMemo 是 React 提供的性能优化 Hook，用于缓存计算结果。它返回一个 memoized（记忆化）的值，该值只在依赖项发生变化时才会重新计算。这对于避免昂贵的计算、优化渲染性能非常重要。

### 为什么需要 useMemo

在 React 组件中，每次渲染都会重新执行组件函数，包括其中的所有计算。这可能导致以下问题：

1. **昂贵的重复计算**：复杂的计算操作在每次渲染时都重复执行
2. **对象引用变化**：每次渲染创建新的对象/数组，导致子组件不必要的重渲染
3. **性能浪费**：在大型应用中，不必要的计算和渲染会影响性能

useMemo 通过缓存计算结果解决了这些问题。

## 核心概念

### 计算成本问题

```jsx
// ❌ 问题：每次渲染都进行昂贵的计算
function ExpensiveCalculation({ numbers }) {
  // 每次渲染都重新计算，即使 numbers 没有变化
  const result = numbers.reduce((sum, num) => {
    // 模拟昂贵操作
    return sum + Math.sqrt(num) * Math.sin(num);
  }, 0);
  
  return <div>计算结果: {result}</div>;
}
```

### useMemo 的工作原理

useMemo 缓存计算结果，只有在依赖项变化时才会重新计算：

```jsx
// ✅ 解决方案：使用 useMemo 缓存计算结果
function OptimizedCalculation({ numbers }) {
  // 只有 numbers 变化时才重新计算
  const result = useMemo(() => {
    console.log('执行昂贵计算');
    return numbers.reduce((sum, num) => {
      return sum + Math.sqrt(num) * Math.sin(num);
    }, 0);
  }, [numbers]);
  
  return <div>计算结果: {result}</div>;
}
```

### useMemo 签名

```jsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

- **第一个参数**：返回缓存值的函数（"创建函数"）
- **第二个参数**：依赖数组，值在这些值变化时重新计算

## 基本用法

### 基本语法

```jsx
import { useMemo, useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  const [multiplier, setMultiplier] = useState(2);
  
  // 使用 useMemo 缓存计算结果
  const doubled = useMemo(() => {
    console.log('计算双倍值');
    return count * 2;
  }, [count]);
  
  const multiplied = useMemo(() => {
    console.log('计算乘积');
    return count * multiplier;
  }, [count, multiplier]);
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Doubled: {doubled}</p>
      <p>Multiplied: {multiplied}</p>
      
      <button onClick={() => setCount(count + 1)}>增加 Count</button>
      <button onClick={() => setMultiplier(multiplier + 1)}>增加 Multiplier</button>
    </div>
  );
}
```

### 复杂计算缓存

```jsx
function ComplexCalculation({ data }) {
  const expensiveResult = useMemo(() => {
    console.log('执行复杂计算');
    
    // 模拟复杂计算
    return data.reduce((acc, item) => {
      const processed = {
        ...item,
        value: Math.sqrt(item.value) * Math.log(item.value + 1),
        normalized: (item.value - Math.min(...data.map(d => d.value))) / 
                    (Math.max(...data.map(d => d.value)) - Math.min(...data.map(d => d.value)))
      };
      return acc + processed.value;
    }, 0);
  }, [data]);
  
  return <div>计算结果: {expensiveResult.toFixed(2)}</div>;
}
```

### 数据转换缓存

```jsx
function DataTransformation({ users, searchQuery }) {
  // 缓存过滤和排序结果
  const filteredUsers = useMemo(() => {
    console.log('过滤用户');
    return users.filter(user =>
      user.name.toLowerCase().includes(searchQuery.toLowerCase())
    );
  }, [users, searchQuery]);
  
  // 缓存排序结果
  const sortedUsers = useMemo(() => {
    console.log('排序用户');
    return [...filteredUsers].sort((a, b) => 
      a.name.localeCompare(b.name)
    );
  }, [filteredUsers]);
  
  return (
    <ul>
      {sortedUsers.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### 对象和数组缓存

```jsx
function ObjectCaching({ theme, settings }) {
  // 缓存样式对象
  const buttonStyle = useMemo(() => ({
    backgroundColor: theme.primary,
    color: theme.text,
    padding: '10px 20px',
    borderRadius: settings.borderRadius,
    fontSize: settings.fontSize
  }), [theme, settings]);
  
  // 缓存配置数组
  const menuItems = useMemo(() => [
    { id: 1, label: '首页', icon: 'home' },
    { id: 2, label: '关于', icon: 'info' },
    { id: 3, label: '设置', icon: 'settings' }
  ], []);
  
  return (
    <div>
      <button style={buttonStyle}>点击按钮</button>
      <nav>
        {menuItems.map(item => (
          <a key={item.id} href="#">
            {item.icon} {item.label}
          </a>
        ))}
      </nav>
    </div>
  );
}
```

## 实际应用

### 列表渲染优化

```jsx
function OptimizedList({ items, filter, sortBy }) {
  // 缓存过滤结果
  const filteredItems = useMemo(() => {
    return items.filter(item => {
      if (filter.category && item.category !== filter.category) {
        return false;
      }
      if (filter.search && !item.name.includes(filter.search)) {
        return false;
      }
      return true;
    });
  }, [items, filter]);
  
  // 缓存排序结果
  const sortedItems = useMemo(() => {
    const sorted = [...filteredItems];
    sorted.sort((a, b) => {
      if (sortBy === 'name') {
        return a.name.localeCompare(b.name);
      } else if (sortBy === 'price') {
        return a.price - b.price;
      } else if (sortBy === 'date') {
        return new Date(b.date) - new Date(a.date);
      }
      return 0;
    });
    return sorted;
  }, [filteredItems, sortBy]);
  
  // 缓存分组结果
  const groupedItems = useMemo(() => {
    return sortedItems.reduce((groups, item) => {
      const category = item.category;
      if (!groups[category]) {
        groups[category] = [];
      }
      groups[category].push(item);
      return groups;
    }, {});
  }, [sortedItems]);
  
  return (
    <div>
      {Object.entries(groupedItems).map(([category, items]) => (
        <div key={category}>
          <h3>{category}</h3>
          <ul>
            {items.map(item => (
              <li key={item.id}>{item.name} - ${item.price}</li>
            ))}
          </ul>
        </div>
      ))}
    </div>
  );
}
```

### 表单数据处理

```jsx
function FormProcessor({ formData, validationRules }) {
  // 缓存验证结果
  const validationResult = useMemo(() => {
    const errors = {};
    
    Object.entries(formData).forEach(([field, value]) => {
      const rules = validationRules[field] || [];
      
      rules.forEach(rule => {
        if (rule.required && !value) {
          errors[field] = `${field} 是必填的`;
        }
        if (rule.minLength && value.length < rule.minLength) {
          errors[field] = `${field} 至少需要 ${rule.minLength} 个字符`;
        }
        if (rule.pattern && !rule.pattern.test(value)) {
          errors[field] = `${field} 格式不正确`;
        }
      });
    });
    
    return {
      isValid: Object.keys(errors).length === 0,
      errors
    };
  }, [formData, validationRules]);
  
  // 缓存计算属性
  const formProgress = useMemo(() => {
    const totalFields = Object.keys(formData).length;
    const filledFields = Object.values(formData).filter(value => value).length;
    return (filledFields / totalFields) * 100;
  }, [formData]);
  
  return (
    <div>
      <div className="progress-bar" style={{ width: `${formProgress}%` }} />
      {validationResult.isValid ? (
        <div className="success">表单有效</div>
      ) : (
        <div className="errors">
          {Object.entries(validationResult.errors).map(([field, error]) => (
            <div key={field} className="error">{error}</div>
          ))}
        </div>
      )}
    </div>
  );
}
```

### 图表数据处理

```jsx
function ChartDataProcessor({ data, timeRange, metrics }) {
  // 缓存数据聚合
  const aggregatedData = useMemo(() => {
    const grouped = {};
    
    data.forEach(item => {
      const date = new Date(item.timestamp);
      const key = getDateKey(date, timeRange); // 'day', 'week', 'month' etc.
      
      if (!grouped[key]) {
        grouped[key] = {
          date: key,
          values: [],
          count: 0
        };
      }
      
      grouped[key].values.push(item.value);
      grouped[key].count++;
    });
    
    // 计算统计信息
    return Object.values(grouped).map(group => ({
      date: group.date,
      avg: group.values.reduce((sum, v) => sum + v, 0) / group.values.length,
      max: Math.max(...group.values),
      min: Math.min(...group.values),
      count: group.count
    }));
  }, [data, timeRange]);
  
  // 缓存图表数据格式化
  const chartData = useMemo(() => {
    return {
      labels: aggregatedData.map(d => d.date),
      datasets: metrics.map(metric => ({
        label: metric.name,
        data: aggregatedData.map(d => d[metric.key]),
        borderColor: metric.color,
        backgroundColor: metric.color + '20'
      }))
    };
  }, [aggregatedData, metrics]);
  
  return <Chart data={chartData} />;
}

// 辅助函数
function getDateKey(date, range) {
  if (range === 'day') {
    return date.toISOString().split('T')[0];
  } else if (range === 'week') {
    const weekStart = new Date(date);
    weekStart.setDate(date.getDate() - date.getDay());
    return weekStart.toISOString().split('T')[0];
  } else if (range === 'month') {
    return `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, '0')}`;
  }
  return date.toISOString().split('T')[0];
}
```

### 虚拟列表优化

```jsx
function VirtualizedList({ items, itemHeight, containerHeight }) {
  const [scrollTop, setScrollTop] = useState(0);
  
  // 缓存可见项目计算
  const visibleItems = useMemo(() => {
    const startIndex = Math.floor(scrollTop / itemHeight);
    const endIndex = Math.min(
      startIndex + Math.ceil(containerHeight / itemHeight) + 1,
      items.length
    );
    
    const offset = startIndex * itemHeight;
    
    return {
      items: items.slice(startIndex, endIndex),
      startIndex,
      endIndex,
      offset,
      totalHeight: items.length * itemHeight
    };
  }, [items, scrollTop, itemHeight, containerHeight]);
  
  const handleScroll = (e) => {
    setScrollTop(e.target.scrollTop);
  };
  
  return (
    <div
      style={{
        height: containerHeight,
        overflow: 'auto',
        position: 'relative'
      }}
      onScroll={handleScroll}
    >
      <div style={{ height: visibleItems.totalHeight }}>
        <div
          style={{
            transform: `translateY(${visibleItems.offset}px)`,
            position: 'absolute',
            top: 0,
            left: 0,
            right: 0
          }}
        >
          {visibleItems.items.map((item, index) => (
            <div
              key={visibleItems.startIndex + index}
              style={{ height: itemHeight }}
            >
              {item.content}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}
```

## 注意事项

### 1. 不要过度使用 useMemo

```jsx
// ❌ 过度使用：简单的计算不需要缓存
function OverOptimized() {
  const [count, setCount] = useState(0);
  
  const doubled = useMemo(() => {
    return count * 2;
  }, [count]);
  
  return <div>{doubled}</div>;
}

// ✅ 合理使用：简单的直接计算
function SimpleCalculation() {
  const [count, setCount] = useState(0);
  
  const doubled = count * 2; // 直接计算
  
  return <div>{doubled}</div>;
}
```

### 2. 依赖数组的正确使用

```jsx
// ❌ 错误：依赖数组不完整
function IncompleteDeps({ data, multiplier }) {
  const result = useMemo(() => {
    return data.reduce((sum, item) => sum + item.value * multiplier, 0);
  }, [data]); // 缺少 multiplier 依赖
  
  return <div>{result}</div>;
}

// ✅ 正确：包含所有依赖
function CompleteDeps({ data, multiplier }) {
  const result = useMemo(() => {
    return data.reduce((sum, item) => sum + item.value * multiplier, 0);
  }, [data, multiplier]);
  
  return <div>{result}</div>;
}
```

### 3. 缓存引用 vs 缓存计算

```jsx
// useMemo 缓存计算结果
const calculatedValue = useMemo(() => {
  return expensiveCalculation(a, b);
}, [a, b]);

// useMemo 缓存引用（不推荐，应该使用其他方式）
const config = useMemo(() => ({
  apiUrl: '/api',
  timeout: 5000
}), []); // 这个对象永远不会变化，直接定义在外面更好

// 更好的方式
const CONFIG = {
  apiUrl: '/api',
  timeout: 5000
}; // 定义在组件外部
```

### 4. useMemo vs useCallback

```jsx
// useMemo 缓存计算结果
const expensiveValue = useMemo(() => {
  return complexCalculation(a, b);
}, [a, b]);

// useCallback 缓存函数
const memoizedCallback = useCallback(() => {
  return complexCalculation(a, b);
}, [a, b]);

// 实际上，useCallback(fn, deps) 相当于 useMemo(() => fn, deps)
```

### 5. 性能测量

```jsx
function PerformanceTest({ largeDataset }) {
  const [filter, setFilter] = useState('');
  
  // 不使用 useMemo
  const filteredData1 = largeDataset.filter(item =>
    item.name.includes(filter)
  );
  
  // 使用 useMemo
  const filteredData2 = useMemo(() => {
    return largeDataset.filter(item =>
      item.name.includes(filter)
    );
  }, [largeDataset, filter]);
  
  // 使用 React DevTools Profiler 测量性能差异
  console.time('filtering');
  const result = filteredData2;
  console.timeEnd('filtering');
  
  return (
    <div>
      <input value={filter} onChange={(e) => setFilter(e.target.value)} />
      {/* 使用 result */}
    </div>
  );
}
```

## 总结

useMemo 是 React 性能优化的重要工具，但需要正确使用：

### 核心要点

1. **结果缓存**：useMemo 缓存计算结果，只在依赖变化时重新计算
2. **性能优化**：主要针对昂贵的计算操作，避免重复计算
3. **引用稳定**：为对象、数组等提供稳定的引用，避免子组件不必要的重渲染
4. **理性使用**：不是所有计算都需要 useMemo，避免过度优化

### 使用场景

1. **昂贵的计算**：复杂数学运算、大数据处理等
2. **对象引用稳定**：传递给子组件的对象 prop
3. **数据转换**：过滤、排序、分组等数据操作
4. **计算属性**：派生数据的缓存

### 最佳实践

1. **按需使用**：只在计算成本高时使用 useMemo
2. **合理依赖**：确保依赖数组包含所有外部依赖
3. **性能测量**：使用 DevTools Profiler 测量实际性能提升
4. **避免过度优化**：简单的计算不需要缓存
5. **配合其他 Hooks**：与 useCallback、React.memo 等配合使用

### 性能优化策略

1. **识别瓶颈**：使用 Profiler 识别性能瓶颈
2. **针对性优化**：对真正的性能问题进行优化
3. **测量验证**：优化后测量实际性能提升
4. **代码可读性**：不要为了优化而牺牲代码可读性

useMemo 是一个强大的性能优化工具，但要记住"过早优化是万恶之源"。在确保代码正确性的基础上，针对性能瓶颈进行有针对性的优化。理解其工作原理和适用场景，才能发挥其真正的价值。性能优化应该基于实际测量，而不是猜测。