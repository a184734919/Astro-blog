---
title: Vue 3 新特性概览
published: 2024-04-06
description: 'Vue 3 主要变化总结的详细介绍和学习笔记'
image: ''
tags: ["Vue3","新特性"]
category: 'Vue3'
draft: false
lang: 'zh-CN'
---

## 概述

Vue 3 是 Vue.js 框架的重大版本升级，在保持框架核心理念的同时，带来了大量新特性和改进。相比 Vue 2，Vue 3 在性能、开发体验、代码组织等方面都有显著提升，为现代前端开发提供了更好的解决方案。

## 核心概念

### 主要改进领域

Vue 3 的改进主要集中在以下几个方面：

- **性能优化**: 更快的渲染速度和更小的打包体积
- **API 改进**: 更灵活的组合式 API
- **类型支持**: 完整的 TypeScript 支持
- **架构升级**: 更好的代码组织和复用性
- **开发体验**: 更好的调试工具和开发体验

## 基本用法

### 1. 组合式 API (Composition API)

组合式 API 是 Vue 3 最重大的变化，提供了更灵活的代码组织方式。

#### 基本用法

```vue
<script setup>
import { ref, reactive, computed, watch } from 'vue';

// 响应式状态
const count = ref(0);
const user = reactive({
  name: '张三',
  age: 25
});

// 计算属性
const doubleCount = computed(() => count.value * 2);

// 方法
function increment() {
  count.value++;
}

// 侦听器
watch(count, (newVal, oldVal) => {
  console.log(`计数变化: ${oldVal} -> ${newVal}`);
});
</script>

<template>
  <div>
    <p>计数: {{ count }}</p>
    <p>双倍计数: {{ doubleCount }}</p>
    <p>用户: {{ user.name }}</p>
    <button @click="increment">增加</button>
  </div>
</template>
```

#### 组合式函数

```javascript
// useCounter.js
import { ref, computed } from 'vue';

export function useCounter(initialValue = 0) {
  const count = ref(initialValue);
  
  const doubleCount = computed(() => count.value * 2);
  
  function increment() {
    count.value++;
  }
  
  function decrement() {
    count.value--;
  }
  
  function reset() {
    count.value = initialValue;
  }
  
  return {
    count,
    doubleCount,
    increment,
    decrement,
    reset
  };
}

// 在组件中使用
<script setup>
import { useCounter } from './useCounter';

const { count, increment } = useCounter(10);
</script>
```

### 2. Teleport 传送门

Teleport 允许将组件的一部分"传送"到 DOM 的任意位置。

#### 模态框示例

```vue
<script setup>
import { ref } from 'vue';

const showModal = ref(false);

function openModal() {
  showModal.value = true;
}

function closeModal() {
  showModal.value = false;
}
</script>

<template>
  <div>
    <button @click="openModal">打开模态框</button>
    
    <teleport to="body">
      <div v-if="showModal" class="modal">
        <div class="modal-content">
          <h2>模态框标题</h2>
          <p>这是模态框内容</p>
          <button @click="closeModal">关闭</button>
        </div>
      </div>
    </teleport>
  </div>
</template>

<style scoped>
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
  max-width: 500px;
  width: 90%;
}
</style>
```

### 3. Fragments 片段

Vue 3 支持多个根节点，不再需要单个根元素。

```vue
<template>
  <!-- Vue 3 支持多个根节点 -->
  <header>头部</header>
  <main>主要内容</main>
  <footer>页脚</footer>
</template>
```

### 4. Suspense 异步组件

Suspense 允许等待异步组件时显示后备内容。

```vue
<script setup>
import { defineAsyncComponent } from 'vue';

// 异步组件
const AsyncComponent = defineAsyncComponent(() => 
  import('./AsyncComponent.vue')
);
</script>

<template>
  <Suspense>
    <template #default>
      <AsyncComponent />
    </template>
    
    <template #fallback>
      <div>加载中...</div>
    </template>
  </Suspense>
</template>
```

### 5. 新的生命周期钩子

组合式 API 提供了新的生命周期钩子。

```vue
<script setup>
import { 
  onBeforeMount, 
  onMounted, 
  onBeforeUpdate, 
  onUpdated,
  onBeforeUnmount,
  onUnmounted
} from 'vue';

onBeforeMount(() => {
  console.log('组件挂载前');
});

onMounted(() => {
  console.log('组件已挂载');
});

onBeforeUpdate(() => {
  console.log('组件更新前');
});

onUpdated(() => {
  console.log('组件已更新');
});

onBeforeUnmount(() => {
  console.log('组件卸载前');
});

onUnmounted(() => {
  console.log('组件已卸载');
});
</script>
```

### 6. 自定义指令 API 变化

Vue 3 的自定义指令 API 有所变化。

```javascript
// Vue 3 自定义指令
const myDirective = {
  created(el, binding, vnode, prevVnode) {
    // 指令元素首次创建时调用
  },
  beforeMount(el, binding, vnode, prevVnode) {
    // 指令元素挂载前调用
  },
  mounted(el, binding, vnode, prevVnode) {
    // 指令元素挂载后调用
  },
  beforeUpdate(el, binding, vnode, prevVnode) {
    // 指令元素更新前调用
  },
  updated(el, binding, vnode, prevVnode) {
    // 指令元素更新后调用
  },
  beforeUnmount(el, binding, vnode, prevVnode) {
    // 指令元素卸载前调用
  },
  unmounted(el, binding, vnode, prevVnode) {
    // 指令元素卸载后调用
  }
};

// 使用
app.directive('focus', {
  mounted(el) {
    el.focus();
  }
});
```

### 7. 全局 API 变化

Vue 3 的全局 API 更加模块化。

```javascript
// Vue 2
import Vue from 'vue';
Vue.use(router);
Vue.component('my-component', MyComponent);
Vue.mixin(myMixin);
Vue.prototype.$http = http;

// Vue 3
import { createApp } from 'vue';
const app = createApp(App);
app.use(router);
app.component('my-component', MyComponent);
app.mixin(myMixin);
app.config.globalProperties.$http = http;
app.mount('#app');
```

### 8. 更好的 TypeScript 支持

Vue 3 提供了完整的 TypeScript 类型定义。

```typescript
<script setup lang="ts">
import { ref, reactive, computed } from 'vue';

// 带类型的 ref
const count = ref<number>(0);
const name = ref<string>('');

// 带类型的 reactive
interface User {
  id: number;
  name: string;
  email: string;
}

const user = reactive<User>({
  id: 1,
  name: '张三',
  email: 'zhangsan@example.com'
});

// 带类型的计算属性
const userInfo = computed<string>(() => {
  return `${user.name} (${user.email})`;
});

// 带类型的方法
function updateUser(data: Partial<User>) {
  Object.assign(user, data);
}
</script>
```

## 实际应用

### 项目迁移示例

从 Vue 2 迁移到 Vue 3 的关键变化：

```javascript
// Vue 2 选项式 API
export default {
  data() {
    return {
      count: 0,
      user: {
        name: '张三'
      }
    };
  },
  
  computed: {
    doubleCount() {
      return this.count * 2;
    }
  },
  
  methods: {
    increment() {
      this.count++;
    }
  },
  
  mounted() {
    console.log('组件已挂载');
  }
};

// Vue 3 组合式 API
<script setup>
import { ref, reactive, computed, onMounted } from 'vue';

const count = ref(0);
const user = reactive({
  name: '张三'
});

const doubleCount = computed(() => count.value * 2);

function increment() {
  count.value++;
}

onMounted(() => {
  console.log('组件已挂载');
});
</script>
```

### 实际项目应用

```vue
<!-- 用户列表组件 -->
<script setup>
import { ref, computed, onMounted } from 'vue';

// 响应式数据
const users = ref([]);
const loading = ref(false);
const error = ref(null);
const searchQuery = ref('');

// 计算属性
const filteredUsers = computed(() => {
  if (!searchQuery.value) return users.value;
  return users.value.filter(user => 
    user.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});

// 方法
async function fetchUsers() {
  loading.value = true;
  error.value = null;
  
  try {
    const response = await fetch('https://api.example.com/users');
    users.value = await response.json();
  } catch (err) {
    error.value = '获取用户列表失败';
    console.error('获取用户失败:', err);
  } finally {
    loading.value = false;
  }
}

function searchUsers(query) {
  searchQuery.value = query;
}

// 生命周期
onMounted(() => {
  fetchUsers();
});
</script>

<template>
  <div>
    <h2>用户列表</h2>
    
    <!-- 搜索框 -->
    <div class="search">
      <input 
        v-model="searchQuery" 
        placeholder="搜索用户..."
        @input="searchUsers($event.target.value)"
      >
    </div>
    
    <!-- 加载状态 -->
    <div v-if="loading" class="loading">
      加载中...
    </div>
    
    <!-- 错误信息 -->
    <div v-if="error" class="error">
      {{ error }}
    </div>
    
    <!-- 用户列表 -->
    <ul v-if="!loading && !error">
      <li v-for="user in filteredUsers" :key="user.id">
        {{ user.name }} - {{ user.email }}
      </li>
    </ul>
    
    <!-- 空状态 -->
    <div v-if="!loading && !error && filteredUsers.length === 0" class="empty">
      没有找到用户
    </div>
  </div>
</template>

<style scoped>
.search {
  margin-bottom: 20px;
}

input {
  padding: 8px;
  width: 100%;
  max-width: 300px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.loading, .error, .empty {
  padding: 20px;
  text-align: center;
  color: #666;
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
</style>
```

## 注意事项

### 迁移注意事项

1. **破坏性变化**: Vue 3 有一些破坏性变化，需要仔细处理
2. **插件兼容性**: 确保使用的插件支持 Vue 3
3. **构建工具**: 使用 Vite 或更新 webpack 配置
4. **浏览器支持**: Vue 3 不支持 IE 浏览器

### 性能考虑

1. **响应式开销**: 合理使用 ref 和 reactive
2. **计算属性缓存**: 充分利用计算属性的缓存特性
3. **组件懒加载**: 使用异步组件优化初始加载
4. **虚拟列表**: 对于长列表使用虚拟滚动

### 最佳实践

1. **组合式 API**: 优先使用组合式 API 和 `<script setup>`
2. **TypeScript**: 在 TypeScript 项目中充分利用类型系统
3. **组件设计**: 保持组件的小型和单一职责
4. **状态管理**: 简单状态使用响应式 API，复杂状态使用 Pinia

## 总结

Vue 3 的新特性极大地提升了开发体验和应用性能：

### 核心优势

1. **性能提升**: 更快的渲染速度和更小的打包体积
2. **开发体验**: 更好的 TypeScript 支持和调试工具
3. **代码组织**: 更灵活的组合式 API
4. **功能增强**: Teleport、Fragments、Suspense 等新特性

### 学习建议

1. **循序渐进**: 从基础概念开始，逐步学习高级特性
2. **实践为主**: 通过实际项目练习新特性
3. **官方文档**: 参考官方文档获取最新信息
4. **社区资源**: 利用社区教程和示例代码

Vue 3 是一个强大的前端框架，新特性让开发更加高效和愉快。掌握这些新特性将帮助你构建更好的应用。