---
title: JavaScript 防抖与节流
published: 2022-08-06
description: '性能优化的常用技巧的详细介绍和学习笔记'
image: ''
tags: ["JS","性能"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

防抖(Debounce)和节流(Throttle)是前端性能优化的重要技术,用于控制函数的执行频率。它们在处理频繁触发的事件(如滚动、输入、调整窗口大小)时特别有用,可以显著提升性能和用户体验。

## 核心概念

### 防抖(Debounce)

防抖是指在事件被触发后,等待一段时间,如果在这段时间内没有再次触发事件,才执行函数。如果事件在等待期间再次触发,则重新计时。

```javascript
// 防抖示例
function debounce(fn, delay) {
  let timer = null;
  
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}

// 使用
const handleInput = debounce((value) => {
  console.log('搜索:', value);
}, 300);

// 快速输入,只在停止输入后300ms执行一次
handleInput('a');
handleInput('ab');
handleInput('abc'); // 只会执行一次,输出: 搜索: abc
```

### 节流(Throttle)

节流是指在指定时间间隔内,无论事件触发多少次,都只执行一次函数。

```javascript
// 节流示例
function throttle(fn, delay) {
  let lastTime = 0;
  
  return function(...args) {
    const now = Date.now();
    
    if (now - lastTime >= delay) {
      fn.apply(this, args);
      lastTime = now;
    }
  };
}

// 使用
const handleScroll = throttle(() => {
  console.log('滚动位置:', window.scrollY);
}, 100);

// 快速滚动,每100ms最多执行一次
window.addEventListener('scroll', handleScroll);
```

## 基本实现

### 防抖实现

#### 基础版本

```javascript
function debounce(fn, delay = 300) {
  let timer = null;
  
  return function(...args) {
    const context = this;
    
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(context, args);
    }, delay);
  };
}
```

#### 立即执行版本

```javascript
function debounceImmediate(fn, delay = 300) {
  let timer = null;
  
  return function(...args) {
    const context = this;
    
    if (!timer) {
      fn.apply(context, args);
    }
    
    clearTimeout(timer);
    timer = setTimeout(() => {
      timer = null;
    }, delay);
  };
}

// 使用
const handleResize = debounceImmediate(() => {
  console.log('窗口大小改变');
}, 300);

// 第一次立即执行,后续300ms内不再执行
handleResize();
handleResize(); // 不会执行
```

#### 带取消功能的版本

```javascript
function debounceCancellable(fn, delay = 300) {
  let timer = null;
  
  function debounced(...args) {
    const context = this;
    
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(context, args);
    }, delay);
  }
  
  debounced.cancel = function() {
    clearTimeout(timer);
  };
  
  return debounced;
}

// 使用
const handleSearch = debounceCancellable((keyword) => {
  console.log('搜索:', keyword);
}, 500);

handleSearch('apple');
handleSearch.cancel(); // 取消执行
```

### 节流实现

#### 时间戳版本

```javascript
function throttle(fn, delay = 300) {
  let lastTime = 0;
  
  return function(...args) {
    const now = Date.now();
    
    if (now - lastTime >= delay) {
      fn.apply(this, args);
      lastTime = now;
    }
  };
}
```

#### 定时器版本

```javascript
function throttleTimer(fn, delay = 300) {
  let timer = null;
  
  return function(...args) {
    const context = this;
    
    if (!timer) {
      timer = setTimeout(() => {
        fn.apply(context, args);
        timer = null;
      }, delay);
    }
  };
}
```

#### 混合版本

```javascript
function throttleCombined(fn, delay = 300) {
  let timer = null;
  let lastTime = 0;
  
  return function(...args) {
    const context = this;
    const now = Date.now();
    
    // 首次立即执行
    if (now - lastTime >= delay) {
      clearTimeout(timer);
      fn.apply(context, args);
      lastTime = now;
    } else if (!timer) {
      // 保证最后一次也会执行
      timer = setTimeout(() => {
        fn.apply(context, args);
        lastTime = Date.now();
        timer = null;
      }, delay - (now - lastTime));
    }
  };
}
```

#### 带取消功能

```javascript
function throttleCancellable(fn, delay = 300) {
  let timer = null;
  let lastTime = 0;
  
  function throttled(...args) {
    const context = this;
    const now = Date.now();
    
    if (now - lastTime >= delay) {
      clearTimeout(timer);
      fn.apply(context, args);
      lastTime = now;
    } else if (!timer) {
      timer = setTimeout(() => {
        fn.apply(context, args);
        lastTime = Date.now();
        timer = null;
      }, delay - (now - lastTime));
    }
  }
  
  throttled.cancel = function() {
    clearTimeout(timer);
  };
  
  return throttled;
}
```

## 实际应用

### 1. 搜索输入

```javascript
// 防抖处理搜索输入
const searchInput = document.getElementById('search');

const handleSearch = debounce((keyword) => {
  fetch(`/api/search?q=${keyword}`)
    .then(res => res.json())
    .then(data => {
      renderResults(data);
    });
}, 300);

searchInput.addEventListener('input', (e) => {
  handleSearch(e.target.value);
});
```

### 2. 滚动加载

```javascript
// 节流处理滚动事件
const handleScroll = throttle(() => {
  const { scrollTop, scrollHeight, clientHeight } = document.documentElement;
  
  if (scrollTop + clientHeight >= scrollHeight - 100) {
    loadMoreContent();
  }
}, 200);

window.addEventListener('scroll', handleScroll);

// 移除监听器时记得清理
window.removeEventListener('scroll', handleScroll);
```

### 3. 窗口调整

```javascript
// 防抖处理窗口大小调整
const handleResize = debounce(() => {
  const width = window.innerWidth;
  const height = window.innerHeight;
  
  // 根据窗口大小调整布局
  if (width < 768) {
    document.body.classList.add('mobile');
  } else {
    document.body.classList.remove('mobile');
  }
}, 200);

window.addEventListener('resize', handleResize);
```

### 4. 按钮点击

```javascript
// 节流处理按钮点击,防止重复提交
const handleSubmit = throttle((formData) => {
  submitForm(formData);
}, 1000);

document.getElementById('submit').addEventListener('click', (e) => {
  const formData = getFormData();
  handleSubmit(formData);
});
```

### 5. 鼠标移动

```javascript
// 节流处理鼠标移动事件
const handleMouseMove = throttle((e) => {
  updateCursor(e.clientX, e.clientY);
}, 16); // 约60fps

document.addEventListener('mousemove', handleMouseMove);
```

### 6. 自动保存

```javascript
// 防抖处理表单自动保存
const handleAutoSave = debounce((formData) => {
  saveDraft(formData).then(() => {
    showSaveSuccess();
  });
}, 1000);

const form = document.getElementById('editor-form');
form.addEventListener('input', (e) => {
  const formData = new FormData(form);
  handleAutoSave(formData);
});
```

## 高级应用

### 1. 防抖与节流组合

```javascript
// 先防抖再节流,用于极端情况
function debounceThenThrottle(fn, debounceDelay, throttleDelay) {
  const debouncedFn = debounce(fn, debounceDelay);
  return throttle(debouncedFn, throttleDelay);
}

// 使用
const handleComplexEvent = debounceThenThrottle(
  (data) => processData(data),
  500,  // 防抖500ms
  1000  // 节流1000ms
);
```

### 2. 自适应延迟

```javascript
// 根据事件触发频率自适应调整延迟
function adaptiveDebounce(fn, baseDelay = 300, maxDelay = 1000) {
  let timer = null;
  let currentDelay = baseDelay;
  let lastTriggerTime = 0;
  
  return function(...args) {
    const context = this;
    const now = Date.now();
    
    clearTimeout(timer);
    
    // 根据触发频率调整延迟
    if (now - lastTriggerTime < 100) {
      currentDelay = Math.min(currentDelay * 2, maxDelay);
    } else {
      currentDelay = baseDelay;
    }
    
    lastTriggerTime = now;
    
    timer = setTimeout(() => {
      fn.apply(context, args);
      currentDelay = baseDelay;
    }, currentDelay);
  };
}

// 使用
const handleInput = adaptiveDebounce((value) => {
  console.log('处理输入:', value);
}, 300, 1000);
```

### 3. 状态跟踪

```javascript
// 跟踪防抖/节流状态
function debounceWithStatus(fn, delay = 300) {
  let timer = null;
  let pending = false;
  
  function debounced(...args) {
    const context = this;
    
    clearTimeout(timer);
    pending = true;
    
    timer = setTimeout(() => {
      fn.apply(context, args);
      pending = false;
    }, delay);
  }
  
  debounced.isPending = () => pending;
  debounced.cancel = () => {
    clearTimeout(timer);
    pending = false;
  };
  
  return debounced;
}

// 使用
const handleSearch = debounceWithStatus((keyword) => {
  console.log('搜索:', keyword);
}, 500);

handleSearch('test');
console.log(handleSearch.isPending()); // true

setTimeout(() => {
  console.log(handleSearch.isPending()); // false
}, 600);
```

### 4. 带参数的记忆化

```javascript
// 记忆化最后一次的参数
function debounceWithLastArgs(fn, delay = 300) {
  let timer = null;
  let lastArgs = null;
  
  function debounced(...args) {
    const context = this;
    
    lastArgs = args;
    clearTimeout(timer);
    
    timer = setTimeout(() => {
      fn.apply(context, lastArgs);
    }, delay);
  }
  
  debounced.flush = function() {
    clearTimeout(timer);
    if (lastArgs) {
      fn.apply(this, lastArgs);
    }
  };
  
  return debounced;
}

// 使用
const handleUpdate = debounceWithLastArgs((data) => {
  console.log('更新数据:', data);
}, 1000);

handleUpdate({ id: 1 });
handleUpdate({ id: 2 });

// 页面卸载时立即执行
window.addEventListener('beforeunload', () => {
  handleUpdate.flush();
});
```

## 性能优化

### 1. requestAnimationFrame 优化

```javascript
// 使用 rAF 优化滚动节流
function throttleRAF(fn) {
  let ticking = false;
  
  return function(...args) {
    const context = this;
    
    if (!ticking) {
      requestAnimationFrame(() => {
        fn.apply(context, args);
        ticking = false;
      });
      
      ticking = true;
    }
  };
}

// 使用
const handleScroll = throttleRAF(() => {
  console.log('滚动位置:', window.scrollY);
});

window.addEventListener('scroll', handleScroll);
```

### 2. 使用 WeakMap 存储定时器

```javascript
// 使用 WeakMap 避免内存泄漏
const debounceCache = new WeakMap();

function debounceWeakMap(fn, delay = 300) {
  return function(...args) {
    const context = this;
    
    if (!debounceCache.has(context)) {
      debounceCache.set(context, null);
    }
    
    const timer = debounceCache.get(context);
    clearTimeout(timer);
    
    const newTimer = setTimeout(() => {
      fn.apply(context, args);
      debounceCache.set(context, null);
    }, delay);
    
    debounceCache.set(context, newTimer);
  };
}
```

### 3. 批量处理

```javascript
// 批量收集事件,一次性处理
function batchEvents(fn, delay = 100) {
  let timer = null;
  let events = [];
  
  return function(event) {
    events.push(event);
    
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn(events);
      events = [];
    }, delay);
  };
}

// 使用
const handleMultipleClicks = batchEvents((clicks) => {
  console.log('批量处理点击:', clicks.length);
}, 200);

document.addEventListener('click', handleMultipleClicks);
```

## 注意事项

### 1. this 指向问题

```javascript
// 确保正确绑定 this
const obj = {
  name: 'test',
  
  handle: debounce(function() {
    console.log(this.name); // 'test'
  }, 300)
};

// 箭头函数会捕获外部 this
const obj2 = {
  name: 'test',
  
  handle: debounce(() => {
    console.log(this.name); // undefined,因为箭头函数没有自己的 this
  }, 300)
};
```

### 2. 内存泄漏

```javascript
// 记得在组件卸载时清理
class MyComponent {
  constructor() {
    this.handleResize = debounce(this.onResize.bind(this), 300);
    window.addEventListener('resize', this.handleResize);
  }
  
  onResize() {
    console.log('resize');
  }
  
  destroy() {
    window.removeEventListener('resize', this.handleResize);
    this.handleResize.cancel();
  }
}
```

### 3. 延迟时间选择

```javascript
// 根据场景选择合适的延迟
const settings = {
  input: 300,        // 输入框: 300ms
  scroll: 100,       // 滚动: 100ms
  resize: 200,       // 调整大小: 200ms
  click: 1000,       // 点击: 1000ms
  autoSave: 1000     // 自动保存: 1000ms
};

const handleInput = debounce(inputHandler, settings.input);
const handleScroll = throttle(scrollHandler, settings.scroll);
```

### 4. 性能测试

```javascript
// 测试防抖/节流的性能
function testPerformance() {
  let count = 0;
  
  const debouncedFn = debounce(() => {
    count++;
  }, 100);
  
  const startTime = performance.now();
  
  for (let i = 0; i < 10000; i++) {
    debouncedFn();
  }
  
  const endTime = performance.now();
  console.log(`执行时间: ${endTime - startTime}ms`);
  console.log(`实际执行次数: ${count}`);
}
```

## 对比总结

### 防抖 vs 节流

```javascript
// 防抖: 等待停止触发后才执行
const debounced = debounce(() => {
  console.log('防抖执行');
}, 300);

// 连续触发,只会执行一次
debounced();
debounced();
debounced(); // 只会执行最后一次

// 节流: 固定间隔执行
const throttled = throttle(() => {
  console.log('节流执行');
}, 300);

// 连续触发,每300ms执行一次
throttled();
throttled();
throttled(); // 会执行多次
```

### 选择指南

```javascript
// 使用防抖的场景
const debounceUseCases = [
  '搜索输入 - 等待用户停止输入',
  '窗口调整 - 等待调整完成',
  '自动保存 - 等待用户停止编辑',
  '表单验证 - 等待用户完成输入'
];

// 使用节流的场景
const throttleUseCases = [
  '滚动事件 - 控制滚动处理频率',
  '鼠标移动 - 控制动画更新频率',
  '按钮点击 - 防止重复提交',
  '拖拽操作 - 控制拖拽处理频率'
];
```

## 总结

防抖与节流的核心要点:

1. **防抖**: 等待停止触发后执行,适合输入、调整等场景
2. **节流**: 固定间隔执行,适合滚动、动画等场景
3. **实现方式**: 基础版本、立即执行、带取消功能、混合版本
4. **实际应用**: 搜索、滚动、窗口调整、按钮点击等
5. **性能优化**: rAF 优化、WeakMap 存储、批量处理
6. **注意事项**: this 指向、内存泄漏、延迟选择、性能测试
7. **选择原则**: 根据场景需求选择合适的技术

合理使用防抖和节流可以显著提升应用性能和用户体验。