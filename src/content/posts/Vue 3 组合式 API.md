---
title: Vue 3 组合式 API
published: 2024-04-09
description: 'setup 函数详解的详细介绍和学习笔记'
image: ''
tags: ["Vue3","Composition"]
category: 'Vue3'
draft: false
lang: 'zh-CN'
---

## 概述

组合式 API (Composition API) 是 Vue 3 的核心特性，它提供了一种更灵活、更可组合的方式来组织组件逻辑。相比选项式 API，组合式 API 允许我们将相关的功能代码组织在一起，而不是按照选项类型分离，从而提高了代码的可读性和可维护性。

## 核心概念

### 为什么要用组合式 API

组合式 API 解决了选项式 API 的几个痛点：

1. **逻辑复用困难**: 通过 Mixins 复用逻辑会导致命名冲突和来源不清晰
2. **大组件难以维护**: 相关逻辑分散在不同的选项中
3. **TypeScript 支持不佳**: 选项式 API 的类型推断不够完善
4. **代码组织不灵活**: 必须按照固定的选项结构组织代码

### 组合式 API 的优势

- **更好的逻辑复用**: 通过组合式函数实现逻辑复用
- **更灵活的代码组织**: 按功能而非选项类型组织代码
- **更好的 TypeScript 支持**: 完整的类型推断
- **更小的组件**: 通过拆分逻辑减少组件复杂度
- **更好的可测试性**: 逻辑函数更容易单元测试

## 基本用法

### 1. setup 函数基础

setup 函数是组合式 API 的入口点。

```vue
<script>
import { ref, reactive, computed, onMounted } from 'vue';

export default {
  setup() {
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

    function updateUserName(name) {
      user.name = name;
    }

    // 生命周期钩子
    onMounted(() => {
      console.log('组件已挂载');
    });

    // 返回给模板使用的数据和方法
    return {
      count,
      user,
      doubleCount,
      increment,
      updateUserName
    };
  }
};
</script>

<template>
  <div>
    <p>计数: {{ count }}</p>
    <p>双倍计数: {{ doubleCount }}</p>
    <p>用户: {{ user.name }}</p>
    <button @click="increment">增加计数</button>
    <button @click="updateUserName('李四')">修改用户名</button>
  </div>
</template>
```

### 2. `<script setup>` 语法糖

`<script setup>` 是 setup 函数的语法糖，提供了更简洁的写法。

```vue
<script setup>
import { ref, reactive, computed, onMounted } from 'vue';

// 直接定义响应式数据，无需 return
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

function updateUserName(name) {
  user.name = name;
}

// 生命周期钩子
onMounted(() => {
  console.log('组件已挂载');
});
</script>

<template>
  <div>
    <p>计数: {{ count }}</p>
    <p>双倍计数: {{ doubleCount }}</p>
    <p>用户: {{ user.name }}</p>
    <button @click="increment">增加计数</button>
    <button @click="updateUserName('李四')">修改用户名</button>
  </div>
</template>
```

### 3. 响应式 API

组合式 API 提供了多个响应式 API：

```vue
<script setup>
import { ref, reactive, toRef, toRefs, readonly, shallowRef, shallowReactive } from 'vue';

// ref - 包装基本类型为响应式对象
const count = ref(0);
const message = ref('Hello');

// reactive - 创建响应式对象
const user = reactive({
  name: '张三',
  age: 25,
  address: {
    city: '北京'
  }
});

// toRef - 为 reactive 对象的属性创建 ref
const userName = toRef(user, 'name');

// toRefs - 将 reactive 对象转换为普通对象，每个属性都是 ref
const { name, age } = toRefs(user);

// readonly - 创建只读的响应式对象
const original = reactive({ count: 0 });
const copy = readonly(original);

// shallowRef - 只有 .value 访问是响应式的
const shallowData = shallowRef({
  nested: { value: 1 }
});

// shallowReactive - 只有第一层属性是响应式的
const shallowState = shallowReactive({
  nested: { value: 1 }
});
</script>
```

### 4. 计算属性和侦听器

```vue
<script setup>
import { ref, computed, watch, watchEffect } from 'vue';

const count = ref(0);
const firstName = ref('张');
const lastName = ref('三');
const user = reactive({
  name: '张三',
  age: 25
});

// 计算属性 - 只读
const doubleCount = computed(() => count.value * 2);

// 计算属性 - 可写
const fullName = computed({
  get() {
    return firstName.value + lastName.value;
  },
  set(newValue) {
    [firstName.value, lastName.value] = newValue.split('');
  }
});

// watch - 侦听 ref
watch(count, (newVal, oldVal) => {
  console.log(`count 变化: ${oldVal} -> ${newVal}`);
});

// watch - 侦听 getter 函数
watch(() => user.name, (newVal, oldVal) => {
  console.log(`user.name 变化: ${oldVal} -> ${newVal}`);
});

// watch - 侦听多个源
watch([count, () => user.name], ([newCount, newName]) => {
  console.log(`多个值变化: count=${newCount}, name=${newName}`);
});

// watchEffect - 自动追踪依赖
watchEffect(() => {
  console.log(`count 的当前值是: ${count.value}`);
});
</script>
```

### 5. 生命周期钩子

组合式 API 中的生命周期钩子：

```vue
<script setup>
import {
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
  onErrorCaptured,
  onRenderTracked,
  onRenderTriggered
} from 'vue';

// 组件挂载前
onBeforeMount(() => {
  console.log('组件挂载前');
});

// 组件挂载后
onMounted(() => {
  console.log('组件挂载后');
  // 可以在这里进行 DOM 操作、API 请求等
});

// 组件更新前
onBeforeUpdate(() => {
  console.log('组件更新前');
});

// 组件更新后
onUpdated(() => {
  console.log('组件更新后');
});

// 组件卸载前
onBeforeUnmount(() => {
  console.log('组件卸载前');
  // 清理工作：取消订阅、清理定时器等
});

// 组件卸载后
onUnmounted(() => {
  console.log('组件卸载后');
});

// 捕获子组件错误
onErrorCaptured((err, instance, info) => {
  console.error('捕获到错误:', err);
  console.error('错误信息:', info);
  // 返回 false 阻止错误继续向上传播
  return false;
});

// 调试：跟踪虚拟 DOM 重新渲染
onRenderTracked((e) => {
  console.log('跟踪渲染:', e);
});

onRenderTriggered((e) => {
  console.log('触发渲染:', e);
});
</script>
```

## 实际应用

### 1. 组合式函数 (Composables)

组合式函数是逻辑复用的核心概念。

```javascript
// useCounter.js - 计数器逻辑
import { ref, computed } from 'vue';

export function useCounter(initialValue = 0) {
  const count = ref(initialValue);
  
  const doubleCount = computed(() => count.value * 2);
  const tripleCount = computed(() => count.value * 3);
  
  function increment() {
    count.value++;
  }
  
  function decrement() {
    count.value--;
  }
  
  function reset() {
    count.value = initialValue;
  }
  
  function setValue(value) {
    count.value = value;
  }
  
  return {
    count,
    doubleCount,
    tripleCount,
    increment,
    decrement,
    reset,
    setValue
  };
}

// useFetch.js - 数据获取逻辑
import { ref, onMounted, onUnmounted } from 'vue';

export function useFetch(url) {
  const data = ref(null);
  const loading = ref(false);
  const error = ref(null);
  
  let controller = null;
  
  async function fetchData() {
    if (controller) {
      controller.abort();
    }
    
    controller = new AbortController();
    loading.value = true;
    error.value = null;
    
    try {
      const response = await fetch(url, {
        signal: controller.signal
      });
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      data.value = await response.json();
    } catch (err) {
      if (err.name !== 'AbortError') {
        error.value = err.message;
      }
    } finally {
      loading.value = false;
    }
  }
  
  function abort() {
    if (controller) {
      controller.abort();
    }
  }
  
  onMounted(() => {
    fetchData();
  });
  
  onUnmounted(() => {
    abort();
  });
  
  return {
    data,
    loading,
    error,
    refetch: fetchData,
    abort
  };
}

// useLocalStorage.js - 本地存储逻辑
import { watch, ref } from 'vue';

export function useLocalStorage(key, defaultValue) {
  const storedValue = localStorage.getItem(key);
  const value = ref(storedValue ? JSON.parse(storedValue) : defaultValue);
  
  watch(value, (newValue) => {
    localStorage.setItem(key, JSON.stringify(newValue));
  }, { deep: true });
  
  return value;
}

// 在组件中使用
<script setup>
import { useCounter, useFetch, useLocalStorage } from './composables';

// 使用计数器逻辑
const { count, increment, decrement, reset } = useCounter(0);

// 使用数据获取逻辑
const { data: users, loading, error, refetch } = useFetch('https://api.example.com/users');

// 使用本地存储逻辑
const theme = useLocalStorage('theme', 'light');
</script>

<template>
  <div>
    <!-- 计数器 -->
    <div>
      <p>计数: {{ count }}</p>
      <button @click="increment">增加</button>
      <button @click="decrement">减少</button>
      <button @click="reset">重置</button>
    </div>
    
    <!-- 数据获取 -->
    <div>
      <div v-if="loading">加载中...</div>
      <div v-if="error">错误: {{ error }}</div>
      <ul v-if="data">
        <li v-for="user in data" :key="user.id">
          {{ user.name }}
        </li>
      </ul>
      <button @click="refetch">刷新</button>
    </div>
    
    <!-- 本地存储 -->
    <div>
      <p>当前主题: {{ theme }}</p>
      <button @click="theme = 'light'">浅色主题</button>
      <button @click="theme = 'dark'">深色主题</button>
    </div>
  </div>
</template>
```

### 2. 实际业务组件

```vue
<script setup>
import { ref, reactive, computed, watch, onMounted } from 'vue';

// Props 定义
const props = defineProps({
  userId: {
    type: Number,
    required: true
  }
});

// Emits 定义
const emit = defineEmits(['update', 'delete']);

// 响应式状态
const user = reactive({
  id: null,
  name: '',
  email: '',
  age: 0
});

const loading = ref(false);
const error = ref(null);
const isEditing = ref(false);

// 表单状态
const formData = reactive({
  name: '',
  email: '',
  age: 0
});

// 计算属性
const userInfo = computed(() => {
  return `${user.name} (${user.email})`;
});

const isAdult = computed(() => {
  return user.age >= 18;
});

const canEdit = computed(() => {
  return !loading.value && !isEditing.value;
});

// 方法
async function fetchUser() {
  loading.value = true;
  error.value = null;
  
  try {
    const response = await fetch(`https://api.example.com/users/${props.userId}`);
    const data = await response.json();
    
    Object.assign(user, data);
    Object.assign(formData, data);
  } catch (err) {
    error.value = '获取用户信息失败';
    console.error('获取用户失败:', err);
  } finally {
    loading.value = false;
  }
}

async function updateUser() {
  loading.value = true;
  error.value = null;
  
  try {
    const response = await fetch(`https://api.example.com/users/${user.id}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(formData)
    });
    
    const updatedUser = await response.json();
    Object.assign(user, updatedUser);
    
    emit('update', updatedUser);
    isEditing.value = false;
  } catch (err) {
    error.value = '更新用户信息失败';
    console.error('更新用户失败:', err);
  } finally {
    loading.value = false;
  }
}

async function deleteUser() {
  if (!confirm('确定要删除此用户吗？')) return;
  
  loading.value = true;
  error.value = null;
  
  try {
    await fetch(`https://api.example.com/users/${user.id}`, {
      method: 'DELETE'
    });
    
    emit('delete', user.id);
  } catch (err) {
    error.value = '删除用户失败';
    console.error('删除用户失败:', err);
  } finally {
    loading.value = false;
  }
}

function startEdit() {
  isEditing.value = true;
  Object.assign(formData, user);
}

function cancelEdit() {
  isEditing.value = false;
  Object.assign(formData, user);
}

// 侦听器
watch(() => props.userId, (newUserId) => {
  if (newUserId) {
    fetchUser();
  }
}, { immediate: true });

// 生命周期
onMounted(() => {
  console.log('用户组件已挂载');
});
</script>

<template>
  <div class="user-card">
    <!-- 加载状态 -->
    <div v-if="loading" class="loading">
      加载中...
    </div>
    
    <!-- 错误信息 -->
    <div v-if="error" class="error">
      {{ error }}
    </div>
    
    <!-- 用户信息显示 -->
    <div v-if="!loading && !error && !isEditing" class="user-info">
      <h2>用户信息</h2>
      <p><strong>姓名:</strong> {{ user.name }}</p>
      <p><strong>邮箱:</strong> {{ user.email }}</p>
      <p><strong>年龄:</strong> {{ user.age }} {{ isAdult ? '(成年)' : '(未成年)' }}</p>
      <p><strong>信息:</strong> {{ userInfo }}</p>
      
      <div class="actions">
        <button @click="startEdit" :disabled="!canEdit">编辑</button>
        <button @click="deleteUser" :disabled="loading">删除</button>
      </div>
    </div>
    
    <!-- 编辑表单 -->
    <div v-if="isEditing" class="edit-form">
      <h2>编辑用户</h2>
      <form @submit.prevent="updateUser">
        <div>
          <label for="name">姓名:</label>
          <input 
            id="name" 
            v-model="formData.name" 
            type="text" 
            required
          >
        </div>
        
        <div>
          <label for="email">邮箱:</label>
          <input 
            id="email" 
            v-model="formData.email" 
            type="email" 
            required
          >
        </div>
        
        <div>
          <label for="age">年龄:</label>
          <input 
            id="age" 
            v-model.number="formData.age" 
            type="number" 
            min="0" 
            required
          >
        </div>
        
        <div class="actions">
          <button type="submit" :disabled="loading">保存</button>
          <button type="button" @click="cancelEdit" :disabled="loading">取消</button>
        </div>
      </form>
    </div>
  </div>
</template>

<style scoped>
.user-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 20px;
  max-width: 400px;
  margin: 0 auto;
}

.loading, .error {
  text-align: center;
  padding: 20px;
}

.error {
  color: red;
}

.user-info, .edit-form {
  margin-top: 20px;
}

.user-info p {
  margin: 10px 0;
}

.actions {
  margin-top: 20px;
}

.actions button {
  margin-right: 10px;
  padding: 8px 16px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.actions button:disabled {
  background: #cccccc;
  cursor: not-allowed;
}

.actions button:hover:not(:disabled) {
  background: #45a049;
}

.edit-form form div {
  margin-bottom: 15px;
}

.edit-form label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.edit-form input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
}
</style>
```

### 3. 复杂的组合式函数

```javascript
// useForm.js - 表单管理逻辑
import { ref, reactive, computed, watch } from 'vue';

export function useForm(initialValues, validationRules = {}) {
  const form = reactive({ ...initialValues });
  const errors = reactive({});
  const touched = reactive({});
  const isSubmitting = ref(false);
  
  // 验证单个字段
  function validateField(field) {
    const rules = validationRules[field];
    if (!rules) return true;
    
    const value = form[field];
    
    for (const rule of rules) {
      if (rule.required && !value) {
        errors[field] = rule.message || '此字段为必填';
        return false;
      }
      
      if (rule.pattern && !rule.pattern.test(value)) {
        errors[field] = rule.message || '格式不正确';
        return false;
      }
      
      if (rule.min && value.length < rule.min) {
        errors[field] = rule.message || `最少 ${rule.min} 个字符`;
        return false;
      }
      
      if (rule.max && value.length > rule.max) {
        errors[field] = rule.message || `最多 ${rule.max} 个字符`;
        return false;
      }
      
      if (rule.validator) {
        const result = rule.validator(value);
        if (result !== true) {
          errors[field] = result || '验证失败';
          return false;
        }
      }
    }
    
    delete errors[field];
    return true;
  }
  
  // 验证整个表单
  function validateAll() {
    let isValid = true;
    
    for (const field in validationRules) {
      if (!validateField(field)) {
        isValid = false;
      }
    }
    
    return isValid;
  }
  
  // 重置表单
  function reset() {
    Object.assign(form, initialValues);
    Object.keys(errors).forEach(key => delete errors[key]);
    Object.keys(touched).forEach(key => delete touched[key]);
  }
  
  // 设置字段值
  function setField(field, value) {
    form[field] = value;
    touched[field] = true;
    validateField(field);
  }
  
  // 计算属性
  const isValid = computed(() => {
    return Object.keys(errors).length === 0;
  });
  
  const isDirty = computed(() => {
    return Object.keys(touched).length > 0;
  });
  
  // 监听表单变化
  watch(form, (newForm) => {
    // 自动验证已修改的字段
    for (const field in touched) {
      if (touched[field]) {
        validateField(field);
      }
    }
  }, { deep: true });
  
  return {
    form,
    errors,
    touched,
    isSubmitting,
    isValid,
    isDirty,
    validateField,
    validateAll,
    reset,
    setField
  };
}

// 使用示例
<script setup>
import { useForm } from './useForm';

const { form, errors, isValid, validateAll, reset } = useForm(
  {
    username: '',
    email: '',
    password: ''
  },
  {
    username: [
      { required: true, message: '用户名不能为空' },
      { min: 3, message: '用户名至少3个字符' }
    ],
    email: [
      { required: true, message: '邮箱不能为空' },
      { pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/, message: '邮箱格式不正确' }
    ],
    password: [
      { required: true, message: '密码不能为空' },
      { min: 6, message: '密码至少6个字符' }
    ]
  }
);

async function handleSubmit() {
  if (validateAll()) {
    console.log('表单提交:', form);
    // 提交逻辑
  }
}
</script>
```

## 注意事项

### 最佳实践

1. **使用 `<script setup>`**: 优先使用语法糖，代码更简洁
2. **逻辑复用**: 将通用逻辑抽取为组合式函数
3. **TypeScript**: 充分利用类型系统提高代码质量
4. **性能优化**: 合理使用 computed 和 watch
5. **命名规范**: 组合式函数使用 `use` 前缀

### 常见错误

```javascript
// 错误 1: 解构失去响应性
const user = reactive({ name: '张三' });
const { name } = user; // name 不是响应式的

// 正确做法
const { name } = toRefs(user); // name 是响应式的

// 错误 2: 在 setup 外部使用响应式 API
const count = ref(0); // 错误：不能在 setup 外部

// 正确做法：在 setup 函数或 <script setup> 中使用

// 错误 3: 过度使用 watch
const count = ref(0);
const doubled = ref(0);

watch(count, (newVal) => {
  doubled.value = newVal * 2;
});

// 正确做法：使用计算属性
const doubled = computed(() => count.value * 2);

// 错误 4: 忘记清理副作用
onMounted(() => {
  setInterval(() => {
    console.log('定时器');
  }, 1000);
});

// 正确做法
onMounted(() => {
  const timer = setInterval(() => {
    console.log('定时器');
  }, 1000);
  
  onUnmounted(() => {
    clearInterval(timer);
  });
});
```

### 性能优化

1. **合理使用 computed**: 利用计算属性的缓存特性
2. **避免过度响应式**: 不是所有数据都需要响应式
3. **使用 shallowRef/shallowReactive**: 对于大型数据结构
4. **优化 watch**: 设置合适的侦听选项
5. **按需引入**: 支持 Tree-shaking

## 总结

组合式 API 是 Vue 3 的革命性特性，它改变了我们组织组件代码的方式：

### 核心优势

1. **更好的逻辑复用**: 通过组合式函数实现代码复用
2. **更灵活的代码组织**: 按功能而非选项类型组织代码
3. **更好的 TypeScript 支持**: 完整的类型推断
4. **更好的可维护性**: 相关代码组织在一起
5. **更好的可测试性**: 纯函数更易测试

### 关键概念

1. **setup 函数**: 组合式 API 的入口点
2. **响应式 API**: ref、reactive、computed 等
3. **生命周期钩子**: 组合式 API 中的生命周期
4. **组合式函数**: 逻辑复用的核心

### 实践建议

1. **从简单开始**: 先掌握基本的响应式 API
2. **逐步深入**: 学习高级特性和最佳实践
3. **注重复用**: 将通用逻辑抽取为组合式函数
4. **保持简洁**: 避免过度设计和复杂化

组合式 API 让 Vue 开发更加灵活和强大，是现代 Vue 开发的推荐方式。掌握组合式 API 将帮助你构建更加可维护和可扩展的应用。