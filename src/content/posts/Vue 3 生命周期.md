---
title: Vue 3 生命周期
published: 2024-05-29
description: '生命周期钩子函数的详细介绍和学习笔记'
image: ''
tags: ["Vue3","基础"]
category: '前端框架'
draft: false
lang: 'zh-CN'
---

## 概述

Vue 3 组件生命周期是指组件从创建到销毁的整个过程，包括创建、挂载、更新、销毁等阶段。理解生命周期对于掌握 Vue 组件的工作原理、正确处理副作用、优化性能等方面都非常重要。

## 核心概念

### 生命周期阶段

Vue 3 组件的生命周期主要分为以下阶段：

1. **创建阶段**: 组件实例创建和初始化
2. **挂载阶段**: 将组件渲染到 DOM
3. **更新阶段**: 响应式数据变化导致重新渲染
4. **销毁阶段**: 组件从 DOM 中移除和清理

### 选项式 API vs 组合式 API

Vue 3 同时支持选项式 API 和组合式 API 的生命周期钩子：

#### 选项式 API 生命周期

```javascript
export default {
  beforeCreate() {
    console.log('beforeCreate: 实例初始化之后');
  },
  
  created() {
    console.log('created: 实例创建完成');
  },
  
  beforeMount() {
    console.log('beforeMount: 挂载开始之前');
  },
  
  mounted() {
    console.log('mounted: 挂载完成');
  },
  
  beforeUpdate() {
    console.log('beforeUpdate: 数据更新之前');
  },
  
  updated() {
    console.log('updated: 数据更新完成');
  },
  
  beforeUnmount() {
    console.log('beforeUnmount: 卸载之前');
  },
  
  unmounted() {
    console.log('unmounted: 卸载完成');
  }
};
```

#### 组合式 API 生命周期

```javascript
import {
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted
} from 'vue';

export default {
  setup() {
    onBeforeMount(() => {
      console.log('onBeforeMount: 挂载开始之前');
    });
    
    onMounted(() => {
      console.log('onMounted: 挂载完成');
    });
    
    onBeforeUpdate(() => {
      console.log('onBeforeUpdate: 数据更新之前');
    });
    
    onUpdated(() => {
      console.log('onUpdated: 数据更新完成');
    });
    
    onBeforeUnmount(() => {
      console.log('onBeforeUnmount: 卸载之前');
    });
    
    onUnmounted(() => {
      console.log('onUnmounted: 卸载完成');
    });
    
    return {};
  }
};
```

## 基本用法

### 1. 创建阶段

创建阶段包括 `beforeCreate` 和 `created` 钩子。

```vue
<script>
export default {
  data() {
    return {
      message: 'Hello Vue 3',
      count: 0
    };
  },
  
  beforeCreate() {
    console.log('beforeCreate');
    console.log('data:', this.$data); // undefined
    console.log('methods:', this.increment); // undefined
    // 此时组件实例刚刚初始化，data 和 methods 还不可用
  },
  
  created() {
    console.log('created');
    console.log('data:', this.$data); // 可以访问
    console.log('methods:', this.increment); // 可以访问
    // 此时组件实例创建完成，data 和 methods 可用，但 DOM 还未挂载
    // 适合进行数据初始化、API 请求等操作
    this.fetchData();
  },
  
  methods: {
    increment() {
      this.count++;
    },
    
    fetchData() {
      console.log('获取数据...');
    }
  }
};
</script>
```

### 2. 挂载阶段

挂载阶段包括 `beforeMount` 和 `mounted` 钩子。

```vue
<script>
export default {
  data() {
    return {
      message: 'Hello Vue 3'
    };
  },
  
  beforeMount() {
    console.log('beforeMount');
    console.log('DOM 元素:', this.$el); // undefined
    // 此时模板编译完成，但还未挂载到 DOM
    // 无法访问 DOM 元素
  },
  
  mounted() {
    console.log('mounted');
    console.log('DOM 元素:', this.$el); // 可以访问
    // 此时组件已挂载到 DOM，可以访问 DOM 元素
    // 适合进行 DOM 操作、第三方库初始化等
    this.initializeChart();
  },
  
  methods: {
    initializeChart() {
      console.log('初始化图表...');
      // 可以在这里初始化图表、地图等需要 DOM 的第三方库
    }
  }
};
</script>

<template>
  <div>
    <h1>{{ message }}</h1>
  </div>
</template>
```

### 3. 更新阶段

更新阶段包括 `beforeUpdate` 和 `updated` 钩子。

```vue
<script>
export default {
  data() {
    return {
      count: 0,
      previousCount: 0
    };
  },
  
  beforeUpdate() {
    console.log('beforeUpdate');
    console.log('当前 count:', this.count);
    console.log('DOM 中的 count:', this.$el.querySelector('.count').textContent);
    // 此时数据已更新，但 DOM 还未重新渲染
    // 可以在更新前访问旧 DOM
    this.previousCount = parseInt(this.$el.querySelector('.count').textContent);
  },
  
  updated() {
    console.log('updated');
    console.log('当前 count:', this.count);
    console.log('DOM 中的 count:', this.$el.querySelector('.count').textContent);
    // 此时 DOM 已重新渲染完成
    // 可以在更新后访问新 DOM
    console.log(`count 从 ${this.previousCount} 变为 ${this.count}`);
  },
  
  methods: {
    increment() {
      this.count++;
    }
  }
};
</script>

<template>
  <div>
    <p class="count">Count: {{ count }}</p>
    <button @click="increment">增加</button>
  </div>
</template>
```

### 4. 销毁阶段

销毁阶段包括 `beforeUnmount` 和 `unmounted` 钩子。

```vue
<script>
export default {
  data() {
    return {
      timer: null,
      intervalId: null
    };
  },
  
  mounted() {
    // 创建定时器
    this.timer = setTimeout(() => {
      console.log('定时器触发');
    }, 5000);
    
    this.intervalId = setInterval(() => {
      console.log('间隔定时器触发');
    }, 1000);
  },
  
  beforeUnmount() {
    console.log('beforeUnmount');
    // 此时组件即将卸载，但仍然可以访问组件实例
    // 适合进行清理工作：取消订阅、清理定时器等
    this.cleanup();
  },
  
  unmounted() {
    console.log('unmounted');
    // 此时组件已完全卸载，无法访问组件实例
    // 所有的事件监听器和子组件都已清理
    console.log('组件已完全卸载');
  },
  
  methods: {
    cleanup() {
      // 清理定时器
      if (this.timer) {
        clearTimeout(this.timer);
        this.timer = null;
      }
      
      if (this.intervalId) {
        clearInterval(this.intervalId);
        this.intervalId = null;
      }
      
      console.log('清理完成');
    }
  }
};
</script>
```

### 5. 错误捕获

Vue 3 提供了错误捕获钩子。

```vue
<script>
export default {
  // 选项式 API
  errorCaptured(err, instance, info) {
    console.error('捕获到错误:', err);
    console.error('组件实例:', instance);
    console.error('错误信息:', info);
    
    // 返回 false 阻止错误继续向上传播
    // 返回 true 或不返回则继续传播
    return false;
  }
};
</script>

<script setup>
import { onErrorCaptured } from 'vue';

// 组合式 API
onErrorCaptured((err, instance, info) => {
  console.error('捕获到错误:', err);
  console.error('组件实例:', instance);
  console.error('错误信息:', info);
  
  // 返回 false 阻止错误继续向上传播
  return false;
});

function triggerError() {
  throw new Error('这是一个测试错误');
}
</script>

<template>
  <div>
    <button @click="triggerError">触发错误</button>
  </div>
</template>
```

## 实际应用

### 1. 数据获取和初始化

```vue
<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';

const users = ref([]);
const loading = ref(false);
const error = ref(null);
let abortController = null;

async function fetchUsers() {
  loading.value = true;
  error.value = null;
  
  // 取消之前的请求
  if (abortController) {
    abortController.abort();
  }
  
  abortController = new AbortController();
  
  try {
    const response = await fetch('https://api.example.com/users', {
      signal: abortController.signal
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    users.value = await response.json();
  } catch (err) {
    if (err.name !== 'AbortError') {
      error.value = err.message;
      console.error('获取用户失败:', err);
    }
  } finally {
    loading.value = false;
  }
}

function refreshUsers() {
  fetchUsers();
}

// 组件挂载时获取数据
onMounted(() => {
  fetchUsers();
  console.log('组件已挂载，开始获取用户数据');
});

// 组件卸载前清理
onBeforeUnmount(() => {
  if (abortController) {
    abortController.abort();
  }
  console.log('组件即将卸载，清理资源');
});
</script>

<template>
  <div>
    <h2>用户列表</h2>
    
    <div v-if="loading" class="loading">
      加载中...
    </div>
    
    <div v-if="error" class="error">
      错误: {{ error }}
    </div>
    
    <ul v-if="!loading && !error">
      <li v-for="user in users" :key="user.id">
        {{ user.name }} - {{ user.email }}
      </li>
    </ul>
    
    <button @click="refreshUsers" :disabled="loading">
      刷新
    </button>
  </div>
</template>

<style scoped>
.loading, .error {
  padding: 20px;
  text-align: center;
}

.error {
  color: red;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  padding: 10px;
  border-bottom: 1px solid #eee;
}

button {
  margin-top: 20px;
  padding: 8px 16px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:disabled {
  background: #cccccc;
  cursor: not-allowed;
}
</style>
```

### 2. DOM 操作和第三方库集成

```vue
<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';
import * as echarts from 'echarts';

const chartRef = ref(null);
let chartInstance = null;

function initChart() {
  if (!chartRef.value) return;
  
  // 初始化图表
  chartInstance = echarts.init(chartRef.value);
  
  // 配置图表
  const option = {
    title: {
      text: '用户统计'
    },
    tooltip: {},
    xAxis: {
      data: ['一月', '二月', '三月', '四月', '五月', '六月']
    },
    yAxis: {},
    series: [{
      name: '用户数',
      type: 'bar',
      data: [5, 20, 36, 10, 10, 20]
    }]
  };
  
  chartInstance.setOption(option);
}

function updateChart() {
  if (!chartInstance) return;
  
  // 更新图表数据
  const newData = [Math.random() * 50, Math.random() * 50, Math.random() * 50, 
                    Math.random() * 50, Math.random() * 50, Math.random() * 50];
  
  chartInstance.setOption({
    series: [{
      data: newData
    }]
  });
}

// 组件挂载后初始化图表
onMounted(() => {
  initChart();
  console.log('图表已初始化');
  
  // 监听窗口大小变化，调整图表大小
  window.addEventListener('resize', handleResize);
});

function handleResize() {
  if (chartInstance) {
    chartInstance.resize();
  }
}

// 组件卸载前清理
onBeforeUnmount(() => {
  if (chartInstance) {
    chartInstance.dispose();
    chartInstance = null;
  }
  
  window.removeEventListener('resize', handleResize);
  console.log('图表资源已清理');
});
</script>

<template>
  <div>
    <h2>ECharts 图表示例</h2>
    <div ref="chartRef" style="width: 100%; height: 400px;"></div>
    <button @click="updateChart">更新数据</button>
  </div>
</template>
```

### 3. WebSocket 连接管理

```vue
<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';

const messages = ref([]);
const connectionStatus = ref('disconnected');
const messageInput = ref('');
let socket = null;
let reconnectTimer = null;

function connectWebSocket() {
  connectionStatus.value = 'connecting';
  
  try {
    socket = new WebSocket('wss://echo.websocket.org');
    
    socket.onopen = () => {
      connectionStatus.value = 'connected';
      console.log('WebSocket 连接已建立');
    };
    
    socket.onmessage = (event) => {
      const message = {
        id: Date.now(),
        text: event.data,
        time: new Date().toLocaleTimeString(),
        type: 'received'
      };
      messages.value.push(message);
      console.log('收到消息:', event.data);
    };
    
    socket.onerror = (error) => {
      console.error('WebSocket 错误:', error);
      connectionStatus.value = 'error';
    };
    
    socket.onclose = () => {
      connectionStatus.value = 'disconnected';
      console.log('WebSocket 连接已关闭');
      
      // 自动重连
      startReconnect();
    };
  } catch (error) {
    console.error('WebSocket 连接失败:', error);
    connectionStatus.value = 'error';
  }
}

function sendMessage() {
  if (!messageInput.value.trim()) return;
  
  if (socket && socket.readyState === WebSocket.OPEN) {
    const message = {
      id: Date.now(),
      text: messageInput.value,
      time: new Date().toLocaleTimeString(),
      type: 'sent'
    };
    
    messages.value.push(message);
    socket.send(messageInput.value);
    messageInput.value = '';
  } else {
    console.warn('WebSocket 未连接');
  }
}

function disconnectWebSocket() {
  if (socket) {
    socket.close();
    socket = null;
  }
  
  if (reconnectTimer) {
    clearTimeout(reconnectTimer);
    reconnectTimer = null;
  }
}

function startReconnect() {
  if (reconnectTimer) return;
  
  reconnectTimer = setTimeout(() => {
    console.log('尝试重新连接...');
    connectWebSocket();
    reconnectTimer = null;
  }, 3000);
}

// 组件挂载时建立连接
onMounted(() => {
  connectWebSocket();
});

// 组件卸载前断开连接
onBeforeUnmount(() => {
  disconnectWebSocket();
  console.log('WebSocket 资源已清理');
});
</script>

<template>
  <div class="websocket-container">
    <h2>WebSocket 聊天</h2>
    
    <div class="status">
      连接状态: 
      <span :class="connectionStatus">
        {{ connectionStatus === 'connected' ? '已连接' : 
           connectionStatus === 'connecting' ? '连接中...' :
           connectionStatus === 'error' ? '错误' : '未连接' }}
      </span>
    </div>
    
    <div class="messages">
      <div 
        v-for="message in messages" 
        :key="message.id"
        :class="['message', message.type]"
      >
        <span class="time">{{ message.time }}</span>
        <span class="text">{{ message.text }}</span>
      </div>
    </div>
    
    <div class="input-area">
      <input 
        v-model="messageInput" 
        @keyup.enter="sendMessage"
        placeholder="输入消息..."
        :disabled="connectionStatus !== 'connected'"
      >
      <button 
        @click="sendMessage"
        :disabled="connectionStatus !== 'connected' || !messageInput.trim()"
      >
        发送
      </button>
    </div>
  </div>
</template>

<style scoped>
.websocket-container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.status {
  margin-bottom: 20px;
  padding: 10px;
  background: #f5f5f5;
  border-radius: 4px;
}

.status .connected {
  color: green;
  font-weight: bold;
}

.status .connecting {
  color: orange;
}

.status .error {
  color: red;
}

.status .disconnected {
  color: gray;
}

.messages {
  height: 300px;
  overflow-y: auto;
  border: 1px solid #eee;
  border-radius: 4px;
  padding: 10px;
  margin-bottom: 20px;
}

.message {
  margin-bottom: 10px;
  padding: 8px;
  border-radius: 4px;
}

.message.sent {
  background: #e3f2fd;
  text-align: right;
}

.message.received {
  background: #f5f5f5;
}

.message .time {
  font-size: 12px;
  color: #666;
  margin-right: 10px;
}

.message .text {
  font-size: 14px;
}

.input-area {
  display: flex;
  gap: 10px;
}

.input-area input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.input-area button {
  padding: 8px 16px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.input-area button:disabled {
  background: #cccccc;
  cursor: not-allowed;
}
</style>
```

### 4. 完整的生命周期示例

```vue
<script setup>
import { ref, onBeforeMount, onMounted, onBeforeUpdate, onUpdated, onBeforeUnmount, onUnmounted } from 'vue';

const count = ref(0);
const message = ref('生命周期示例');
const lifecycleLog = ref([]);

function addLog(message) {
  lifecycleLog.value.push({
    time: new Date().toLocaleTimeString(),
    message: message
  });
}

// 创建阶段
onBeforeMount(() => {
  addLog('onBeforeMount: 组件即将挂载');
  console.log('onBeforeMount - DOM 元素还未创建');
});

onMounted(() => {
  addLog('onMounted: 组件已挂载');
  console.log('onMounted - DOM 元素已创建');
  // 可以在这里进行 DOM 操作、API 请求等
});

// 更新阶段
onBeforeUpdate(() => {
  addLog('onBeforeUpdate: 组件即将更新');
  console.log('onBeforeUpdate - 数据已变化，DOM 未更新');
});

onUpdated(() => {
  addLog('onUpdated: 组件已更新');
  console.log('onUpdated - DOM 已更新');
  // 注意：避免在这里修改状态，可能导致无限循环
});

// 销毁阶段
onBeforeUnmount(() => {
  addLog('onBeforeUnmount: 组件即将卸载');
  console.log('onBeforeUnmount - 清理资源');
  // 清理定时器、取消订阅等
});

onUnmounted(() => {
  addLog('onUnmounted: 组件已卸载');
  console.log('onUnmounted - 组件已完全销毁');
});

function increment() {
  count.value++;
}

function resetLog() {
  lifecycleLog.value = [];
}
</script>

<template>
  <div class="lifecycle-demo">
    <h2>Vue 3 生命周期演示</h2>
    
    <div class="demo-area">
      <p>计数: {{ count }}</p>
      <p>消息: {{ message }}</p>
      <button @click="increment">增加计数</button>
    </div>
    
    <div class="log-area">
      <h3>生命周期日志</h3>
      <div class="log-controls">
        <button @click="resetLog">清空日志</button>
      </div>
      <div class="log-content">
        <div v-for="(log, index) in lifecycleLog" :key="index" class="log-entry">
          <span class="time">[{{ log.time }}]</span>
          <span class="message">{{ log.message }}</span>
        </div>
        <div v-if="lifecycleLog.length === 0" class="empty-log">
          暂无日志
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.lifecycle-demo {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.demo-area {
  margin-bottom: 20px;
  padding: 20px;
  background: #f5f5f5;
  border-radius: 8px;
}

.demo-area p {
  margin: 10px 0;
  font-size: 18px;
}

.demo-area button {
  padding: 8px 16px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.log-area {
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
}

.log-area h3 {
  margin: 0;
  padding: 15px;
  background: #333;
  color: white;
}

.log-controls {
  padding: 10px 15px;
  background: #f5f5f5;
  border-bottom: 1px solid #ddd;
}

.log-controls button {
  padding: 6px 12px;
  background: #2196F3;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.log-content {
  max-height: 300px;
  overflow-y: auto;
  padding: 15px;
}

.log-entry {
  padding: 8px 0;
  border-bottom: 1px solid #eee;
}

.log-entry .time {
  color: #666;
  margin-right: 10px;
  font-size: 12px;
}

.log-entry .message {
  color: #333;
}

.empty-log {
  text-align: center;
  color: #999;
  padding: 20px;
}
</style>
```

## 注意事项

### 最佳实践

1. **合理选择生命周期钩子**: 根据需求选择合适的钩子
2. **及时清理资源**: 在 `onBeforeUnmount` 中清理定时器、事件监听器等
3. **避免副作用**: 避免在 `onUpdated` 中修改状态
4. **错误处理**: 使用 `errorCaptured` 捕获子组件错误
5. **异步操作**: 在 `onMounted` 中进行异步操作

### 常见错误

```javascript
// 错误 1: 在 created 钩子中访问 DOM
export default {
  created() {
    console.log(this.$el); // undefined
    // DOM 还未挂载，无法访问
  }
};

// 正确做法
export default {
  mounted() {
    console.log(this.$el); // 可以访问
  }
};

// 错误 2: 在 onUpdated 中修改状态导致无限循环
import { ref, onUpdated } from 'vue';

const count = ref(0);

onUpdated(() => {
  count.value++; // 错误：会导致无限循环
});

// 错误 3: 忘记清理定时器
export default {
  mounted() {
    this.timer = setInterval(() => {
      console.log('定时器');
    }, 1000);
  }
  // 错误：没有清理定时器
};

// 正确做法
export default {
  mounted() {
    this.timer = setInterval(() => {
      console.log('定时器');
    }, 1000);
  },
  
  beforeUnmount() {
    if (this.timer) {
      clearInterval(this.timer);
    }
  }
};

// 错误 4: 在 setup 外部使用生命周期钩子
import { onMounted } from 'vue';

onMounted(() => {
  console.log('错误：不能在 setup 外部使用');
});

// 正确做法
export default {
  setup() {
    onMounted(() => {
      console.log('正确：在 setup 中使用');
    });
  }
};
```

### 性能优化

1. **减少不必要的更新**: 使用计算属性和侦听器优化
2. **合理使用异步操作**: 避免阻塞主线程
3. **清理大对象**: 在销毁时清理大型数据结构
4. **避免重复初始化**: 检查是否已经初始化
5. **使用防抖节流**: 对于频繁触发的事件

## 总结

Vue 3 生命周期是组件开发的重要组成部分，掌握生命周期钩子的使用对于构建高质量的应用至关重要：

### 核心要点

1. **创建阶段**: `beforeCreate` → `created`
2. **挂载阶段**: `beforeMount` → `mounted`
3. **更新阶段**: `beforeUpdate` → `updated`
4. **销毁阶段**: `beforeUnmount` → `unmounted`
5. **错误捕获**: `errorCaptured`

### 实践建议

1. **数据初始化**: 在 `created` 或 `onMounted` 中进行
2. **DOM 操作**: 在 `mounted` 或 `onMounted` 中进行
3. **资源清理**: 在 `beforeUnmount` 或 `onBeforeUnmount` 中进行
4. **第三方库**: 在 `mounted` 中初始化，在 `beforeUnmount` 中清理
5. **API 请求**: 在 `onMounted` 中进行，记得取消请求

### 选择指南

- **数据初始化**: `created` / `onCreated` (组合式 API 中没有直接对应)
- **DOM 操作**: `mounted` / `onMounted`
- **数据更新监听**: `beforeUpdate` / `onBeforeUpdate`, `updated` / `onUpdated`
- **资源清理**: `beforeUnmount` / `onBeforeUnmount`
- **错误处理**: `errorCaptured` / `onErrorCaptured`

理解并正确使用生命周期钩子，将帮助你构建更加稳定、高效和可维护的 Vue 3 应用。