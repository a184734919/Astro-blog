---
title: Vue 3 watch 函数
published: 2024-01-13
description: 'watch 监听响应式数据的详细介绍和学习笔记'
image: ''
tags: ["Vue3","watch"]
category: 'Vue3'
draft: false
lang: 'zh-CN'
---

## 概述

Vue 3 的 `watch` 函数是一个强大的响应式 API，用于监听响应式数据的变化并在变化时执行副作用。它是 Vue 2 中 `watch` 选项的 Composition API 版本，提供了更灵活、更强大的功能。

`watch` 的主要用途包括：
- 监听数据变化并执行相应操作
- 执行异步操作
- 数据验证
- 性能优化（防抖、节流）
- 复杂的状态管理逻辑

## 核心概念

### 基本语法

```javascript
import { watch, ref } from 'vue'

const count = ref(0)

// 基本用法
watch(count, (newValue, oldValue) => {
  console.log(`值从 ${oldValue} 变为 ${newValue}`)
})
```

### 监听类型

1. **监听单个 ref**：监听一个 ref 的变化
2. **监听多个 ref**：同时监听多个 ref 的变化
3. **监听 reactive 对象**：监听 reactive 对象的变化
4. **监听 getter 函数**：监听计算属性或复杂表达式的变化
5. **深度监听**：监听对象内部的嵌套变化

### 回调函数参数

- `newValue`：新值
- `oldValue`：旧值（仅对 ref 和深度监听有效）
- `onCleanup`：清理函数，用于清除副作用

## 基本用法

### 监听单个 ref

```vue
<template>
  <div>
    <h2>计数器示例</h2>
    <p>当前计数: {{ count }}</p>
    <button @click="increment">增加</button>
    <button @click="decrement">减少</button>
    <button @click="reset">重置</button>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const count = ref(0)

// 监听单个 ref
watch(count, (newValue, oldValue) => {
  console.log(`计数从 ${oldValue} 变为 ${newValue}`)
  
  // 根据不同的值执行不同的操作
  if (newValue === 10) {
    console.log('达到目标值 10')
  }
})

const increment = () => count.value++
const decrement = () => count.value--
const reset = () => count.value = 0
</script>
```

### 监听多个 ref

```vue
<template>
  <div>
    <h2>表单验证</h2>
    
    <div class="form-group">
      <label>用户名:</label>
      <input v-model="username" @input="validateUsername" />
      <span class="error" v-if="usernameError">{{ usernameError }}</span>
    </div>
    
    <div class="form-group">
      <label>邮箱:</label>
      <input v-model="email" @input="validateEmail" />
      <span class="error" v-if="emailError">{{ emailError }}</span>
    </div>
    
    <div class="form-group">
      <label>密码:</label>
      <input type="password" v-model="password" @input="validatePassword" />
      <span class="error" v-if="passwordError">{{ passwordError }}</span>
    </div>
    
    <button :disabled="!isFormValid" @click="handleSubmit">
      提交表单
    </button>
  </div>
</template>

<script setup>
import { ref, watch, computed } from 'vue'

const username = ref('')
const email = ref('')
const password = ref('')

const usernameError = ref('')
const emailError = ref('')
const passwordError = ref('')

// 监听多个字段
watch([username, email, password], ([newUsername, newEmail, newPassword]) => {
  console.log('表单字段变化:', { newUsername, newEmail, newPassword })
})

// 单独验证每个字段
watch(username, (newValue) => {
  if (newValue.length < 3) {
    usernameError.value = '用户名至少需要 3 个字符'
  } else {
    usernameError.value = ''
  }
})

watch(email, (newValue) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!emailRegex.test(newValue)) {
    emailError.value = '请输入有效的邮箱地址'
  } else {
    emailError.value = ''
  }
})

watch(password, (newValue) => {
  if (newValue.length < 6) {
    passwordError.value = '密码至少需要 6 个字符'
  } else {
    passwordError.value = ''
  }
})

// 计算表单是否有效
const isFormValid = computed(() => {
  return username.value.length >= 3 && 
         email.value.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/) &&
         password.value.length >= 6
})

const validateUsername = () => { /* 触发验证 */ }
const validateEmail = () => { /* 触发验证 */ }
const validatePassword = () => { /* 触发验证 */ }

const handleSubmit = () => {
  console.log('提交表单:', {
    username: username.value,
    email: email.value,
    password: password.value
  })
}
</script>

<style scoped>
.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
}

.form-group input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
}

.error {
  color: #f44336;
  font-size: 14px;
  margin-top: 5px;
  display: block;
}

button {
  padding: 10px 20px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:disabled {
  background: #ccc;
  cursor: not-allowed;
}
</style>
```

### 监听 reactive 对象

```vue
<template>
  <div>
    <h2>用户信息管理</h2>
    
    <div class="user-form">
      <div class="form-group">
        <label>姓名:</label>
        <input v-model="user.name" />
      </div>
      
      <div class="form-group">
        <label>年龄:</label>
        <input type="number" v-model="user.age" />
      </div>
      
      <div class="form-group">
        <label>邮箱:</label>
        <input v-model="user.email" />
      </div>
      
      <div class="form-group">
        <label>城市:</label>
        <input v-model="user.address.city" />
      </div>
    </div>
    
    <div class="user-info">
      <h3>当前用户信息:</h3>
      <pre>{{ JSON.stringify(user, null, 2) }}</pre>
    </div>
  </div>
</template>

<script setup>
import { reactive, watch } from 'vue'

const user = reactive({
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com',
  address: {
    city: '北京',
    street: '朝阳路'
  }
})

// 监听整个 reactive 对象
watch(() => ({ ...user }), (newValue, oldValue) => {
  console.log('用户信息变化:', { newValue, oldValue })
}, { deep: true })

// 监听对象的特定属性
watch(() => user.name, (newValue, oldValue) => {
  console.log(`姓名从 ${oldValue} 变为 ${newValue}`)
})

watch(() => user.address.city, (newValue, oldValue) => {
  console.log(`城市从 ${oldValue} 变为 ${newValue}`)
})
</script>

<style scoped>
.user-form {
  background: #f9f9f9;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: 500;
}

.form-group input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
}

.user-info {
  background: white;
  padding: 20px;
  border-radius: 8px;
  border: 1px solid #e0e0e0;
}

.user-info h3 {
  margin-top: 0;
}

.user-info pre {
  background: #f5f5f5;
  padding: 15px;
  border-radius: 4px;
  overflow-x: auto;
}
</style>
```

### 监听 getter 函数

```vue
<template>
  <div>
    <h2>购物车计算</h2>
    
    <div class="cart">
      <div class="cart-items">
        <div v-for="item in cart" :key="item.id" class="cart-item">
          <span>{{ item.name }}</span>
          <span>¥{{ item.price }}</span>
          <input 
            type="number" 
            v-model.number="item.quantity" 
            min="1"
            class="quantity-input"
          />
          <span>小计: ¥{{ item.price * item.quantity }}</span>
        </div>
      </div>
      
      <div class="cart-summary">
        <div class="summary-item">
          <span>商品总数:</span>
          <span>{{ totalItems }}</span>
        </div>
        <div class="summary-item">
          <span>商品总价:</span>
          <span>¥{{ totalPrice }}</span>
        </div>
        <div class="summary-item discount">
          <span>折扣优惠:</span>
          <span>-¥{{ discount }}</span>
        </div>
        <div class="summary-item total">
          <span>最终价格:</span>
          <span>¥{{ finalPrice }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

const cart = ref([
  { id: 1, name: '商品 A', price: 100, quantity: 2 },
  { id: 2, name: '商品 B', price: 200, quantity: 1 },
  { id: 3, name: '商品 C', price: 150, quantity: 3 }
])

// 计算属性
const totalItems = computed(() => {
  return cart.value.reduce((sum, item) => sum + item.quantity, 0)
})

const totalPrice = computed(() => {
  return cart.value.reduce((sum, item) => sum + item.price * item.quantity, 0)
})

const discount = computed(() => {
  if (totalPrice.value >= 1000) {
    return totalPrice.value * 0.1 // 10% 折扣
  } else if (totalPrice.value >= 500) {
    return totalPrice.value * 0.05 // 5% 折扣
  }
  return 0
})

const finalPrice = computed(() => {
  return totalPrice.value - discount.value
})

// 监听计算属性
watch(finalPrice, (newValue, oldValue) => {
  console.log(`最终价格从 ¥${oldValue} 变为 ¥${newValue}`)
  
  // 根据价格变化执行不同操作
  if (newValue > 1000) {
    console.log('提醒用户：订单金额较大')
  }
})

// 监听复杂的表达式
watch(
  () => cart.value.map(item => item.quantity).reduce((a, b) => a + b, 0),
  (newValue) => {
    console.log(`购物车商品总数变化: ${newValue}`)
  }
)
</script>

<style scoped>
.cart {
  max-width: 600px;
  margin: 0 auto;
}

.cart-items {
  background: #f9f9f9;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.cart-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid #e0e0e0;
}

.cart-item:last-child {
  border-bottom: none;
}

.quantity-input {
  width: 60px;
  padding: 5px;
  border: 1px solid #ddd;
  border-radius: 4px;
  text-align: center;
}

.cart-summary {
  background: white;
  padding: 20px;
  border-radius: 8px;
  border: 1px solid #e0e0e0;
}

.summary-item {
  display: flex;
  justify-content: space-between;
  padding: 10px 0;
  font-size: 16px;
}

.summary-item.discount {
  color: #4CAF50;
}

.summary-item.total {
  font-weight: bold;
  font-size: 18px;
  border-top: 2px solid #e0e0e0;
  padding-top: 15px;
  margin-top: 10px;
}
</style>
```

## 实际应用

### 搜索防抖

```vue
<template>
  <div>
    <h2>搜索示例</h2>
    
    <div class="search-container">
      <input
        v-model="searchQuery"
        placeholder="搜索用户..."
        class="search-input"
      />
      <div v-if="isLoading" class="loading">搜索中...</div>
    </div>
    
    <div class="results">
      <div v-if="results.length > 0" class="result-list">
        <div v-for="user in results" :key="user.id" class="result-item">
          <span class="user-name">{{ user.name }}</span>
          <span class="user-email">{{ user.email }}</span>
        </div>
      </div>
      
      <div v-else-if="searchQuery && !isLoading" class="no-results">
        未找到相关结果
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const searchQuery = ref('')
const results = ref([])
const isLoading = ref(false)

// 模拟 API 调用
const searchUsers = async (query) => {
  isLoading.value = true
  
  // 模拟网络延迟
  await new Promise(resolve => setTimeout(resolve, 500))
  
  // 模拟搜索结果
  const allUsers = [
    { id: 1, name: '张三', email: 'zhangsan@example.com' },
    { id: 2, name: '李四', email: 'lisi@example.com' },
    { id: 3, name: '王五', email: 'wangwu@example.com' },
    { id: 4, name: '赵六', email: 'zhaoliu@example.com' },
    { id: 5, name: '孙七', email: 'sunqi@example.com' }
  ]
  
  results.value = allUsers.filter(user => 
    user.name.includes(query) || user.email.includes(query)
  )
  
  isLoading.value = false
}

// 使用 watch 实现防抖搜索
let timeoutId = null

watch(searchQuery, (newValue) => {
  // 清除之前的定时器
  if (timeoutId) {
    clearTimeout(timeoutId)
  }
  
  // 如果搜索为空，清空结果
  if (!newValue.trim()) {
    results.value = []
    return
  }
  
  // 设置新的定时器
  timeoutId = setTimeout(() => {
    searchUsers(newValue)
  }, 300) // 300ms 防抖延迟
})
</script>

<style scoped>
.search-container {
  margin-bottom: 20px;
  position: relative;
}

.search-input {
  width: 100%;
  padding: 12px;
  font-size: 16px;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  box-sizing: border-box;
  transition: border-color 0.3s;
}

.search-input:focus {
  outline: none;
  border-color: #4CAF50;
}

.loading {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
  color: #999;
  font-size: 14px;
}

.results {
  min-height: 200px;
}

.result-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.result-item {
  display: flex;
  justify-content: space-between;
  padding: 15px;
  background: #f9f9f9;
  border-radius: 4px;
  transition: background 0.2s;
}

.result-item:hover {
  background: #f0f0f0;
}

.user-name {
  font-weight: 500;
  color: #333;
}

.user-email {
  color: #666;
  font-size: 14px;
}

.no-results {
  text-align: center;
  padding: 40px;
  color: #999;
}
</style>
```

### 数据持久化

```vue
<template>
  <div>
    <h2>自动保存的表单</h2>
    
    <form class="auto-save-form">
      <div class="form-group">
        <label>标题:</label>
        <input v-model="formData.title" placeholder="请输入标题" />
      </div>
      
      <div class="form-group">
        <label>内容:</label>
        <textarea 
          v-model="formData.content" 
          placeholder="请输入内容"
          rows="6"
        ></textarea>
      </div>
      
      <div class="form-group">
        <label>标签:</label>
        <input v-model="formData.tags" placeholder="用逗号分隔标签" />
      </div>
      
      <div class="save-status">
        <span v-if="saveStatus === 'saving'" class="saving">
          保存中...
        </span>
        <span v-else-if="saveStatus === 'saved'" class="saved">
          已保存
        </span>
        <span v-else-if="saveStatus === 'error'" class="error">
          保存失败
        </span>
      </div>
      
      <div class="form-actions">
        <button type="button" @click="handleManualSave" class="btn-save">
          手动保存
        </button>
        <button type="button" @click="handleReset" class="btn-reset">
          重置表单
        </button>
      </div>
    </form>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue'

const STORAGE_KEY = 'auto-save-form'

const formData = ref({
  title: '',
  content: '',
  tags: ''
})

const saveStatus = ref('idle') // idle, saving, saved, error

// 从本地存储加载数据
onMounted(() => {
  const savedData = localStorage.getItem(STORAGE_KEY)
  if (savedData) {
    try {
      formData.value = JSON.parse(savedData)
    } catch (error) {
      console.error('加载数据失败:', error)
    }
  }
})

// 保存数据到本地存储
const saveData = () => {
  saveStatus.value = 'saving'
  
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(formData.value))
    saveStatus.value = 'saved'
    
    // 2秒后重置状态
    setTimeout(() => {
      saveStatus.value = 'idle'
    }, 2000)
  } catch (error) {
    console.error('保存数据失败:', error)
    saveStatus.value = 'error'
  }
}

// 监听表单变化并自动保存
let saveTimeout = null

watch(formData, () => {
  // 清除之前的定时器
  if (saveTimeout) {
    clearTimeout(saveTimeout)
  }
  
  // 设置新的定时器，1秒后自动保存
  saveTimeout = setTimeout(() => {
    saveData()
  }, 1000)
}, { deep: true })

const handleManualSave = () => {
  saveData()
}

const handleReset = () => {
  if (confirm('确定要重置表单吗？')) {
    formData.value = {
      title: '',
      content: '',
      tags: ''
    }
    localStorage.removeItem(STORAGE_KEY)
    saveStatus.value = 'idle'
  }
}
</script>

<style scoped>
.auto-save-form {
  max-width: 600px;
  margin: 0 auto;
  padding: 30px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
  color: #333;
}

.form-group input,
.form-group textarea {
  width: 100%;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
  box-sizing: border-box;
  font-family: inherit;
}

.form-group input:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #4CAF50;
  box-shadow: 0 0 0 2px rgba(76, 175, 80, 0.2);
}

.save-status {
  text-align: center;
  padding: 10px;
  margin: 20px 0;
  border-radius: 4px;
}

.saving {
  color: #1976d2;
  background: #e3f2fd;
}

.saved {
  color: #4CAF50;
  background: #e8f5e9;
}

.error {
  color: #f44336;
  background: #ffebee;
}

.form-actions {
  display: flex;
  gap: 10px;
  justify-content: center;
}

.form-actions button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 500;
}

.btn-save {
  background: #4CAF50;
  color: white;
}

.btn-reset {
  background: #f44336;
  color: white;
}

.form-actions button:hover {
  opacity: 0.9;
}
</style>
```

### API 请求管理

```vue
<template>
  <div>
    <h2>用户列表</h2>
    
    <div v-if="loading" class="loading">
      <div class="spinner"></div>
      <p>加载中...</p>
    </div>
    
    <div v-else-if="error" class="error">
      <p>{{ error }}</p>
      <button @click="fetchUsers" class="retry-btn">重试</button>
    </div>
    
    <div v-else class="user-list">
      <div v-for="user in users" :key="user.id" class="user-card">
        <h3>{{ user.name }}</h3>
        <p>{{ user.email }}</p>
        <p>公司: {{ user.company.name }}</p>
      </div>
    </div>
    
    <div class="controls">
      <button @click="refreshUsers" class="refresh-btn">
        刷新数据
      </button>
      <select v-model="userFilter" @change="handleFilterChange">
        <option value="">全部用户</option>
        <option value="active">活跃用户</option>
        <option value="inactive">非活跃用户</option>
      </select>
    </div>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue'

const users = ref([])
const loading = ref(false)
const error = ref(null)
const userFilter = ref('')

// 模拟 API 请求
const fetchUsersFromAPI = async () => {
  loading.value = true
  error.value = null
  
  try {
    // 模拟网络请求
    await new Promise(resolve => setTimeout(resolve, 1000))
    
    // 模拟 API 响应
    const response = [
      {
        id: 1,
        name: '张三',
        email: 'zhangsan@example.com',
        company: { name: '科技公司' },
        status: 'active'
      },
      {
        id: 2,
        name: '李四',
        email: 'lisi@example.com',
        company: { name: '互联网公司' },
        status: 'active'
      },
      {
        id: 3,
        name: '王五',
        email: 'wangwu@example.com',
        company: { name: '软件公司' },
        status: 'inactive'
      }
    ]
    
    users.value = response
  } catch (err) {
    error.value = '加载数据失败: ' + err.message
  } finally {
    loading.value = false
  }
}

// 初始加载
onMounted(() => {
  fetchUsersFromAPI()
})

// 监听过滤器变化
watch(userFilter, (newValue) => {
  console.log('过滤器变化:', newValue)
  // 在实际应用中，这里可能会重新加载数据
})

const refreshUsers = () => {
  fetchUsersFromAPI()
}

const handleFilterChange = () => {
  // 处理过滤器变化
  console.log('过滤器改变为:', userFilter.value)
}
</script>

<style scoped>
.loading {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.error {
  text-align: center;
  padding: 40px;
  color: #f44336;
}

.retry-btn {
  margin-top: 10px;
  padding: 8px 16px;
  background: #f44336;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.user-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 20px;
  margin-bottom: 20px;
}

.user-card {
  padding: 20px;
  background: #f9f9f9;
  border-radius: 8px;
  border: 1px solid #e0e0e0;
}

.user-card h3 {
  margin-top: 0;
  color: #333;
}

.user-card p {
  margin: 8px 0;
  color: #666;
}

.controls {
  display: flex;
  justify-content: center;
  gap: 15px;
  align-items: center;
  padding: 20px 0;
}

.refresh-btn {
  padding: 10px 20px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.controls select {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}
</style>
```

## 高级特性

### watch 选项

```javascript
// 立即执行监听
watch(source, callback, {
  immediate: true, // 立即执行一次回调
  deep: true, // 深度监听
  flush: 'pre' // 'pre' | 'post' | 'sync'
})

// 深度监听 reactive 对象
const state = reactive({
  user: {
    profile: {
      name: 'John'
    }
  }
})

watch(
  () => state,
  (newValue) => {
    console.log('状态变化:', newValue)
  },
  { deep: true }
)

// 同步执行
watch(source, callback, {
  flush: 'sync' // 同步触发
})

// 前置执行（默认）
watch(source, callback, {
  flush: 'pre' // 在 DOM 更新前触发
})
```

### 清理副作用

```javascript
watch(searchQuery, (newValue, oldValue, onCleanup) => {
  // 创建一个定时器
  const timeoutId = setTimeout(() => {
    console.log('搜索:', newValue)
  }, 500)
  
  // 清理函数：在下次回调执行前或 watch 停止时调用
  onCleanup(() => {
    clearTimeout(timeoutId)
  })
})

// 取消监听
const stopWatch = watch(source, callback)

// 在需要时停止监听
stopWatch()
```

### 异步监听

```javascript
watch(userId, async (newUserId) => {
  loading.value = true
  
  try {
    const userData = await fetchUser(newUserId)
    user.value = userData
  } catch (error) {
    console.error('获取用户失败:', error)
  } finally {
    loading.value = false
  }
})
```

## 最佳实践

### 1. 合理使用 immediate 选项

```javascript
// 好的做法：需要在组件挂载时立即执行初始化操作
watch(data, async (newValue) => {
  await initializeData(newValue)
}, { immediate: true })

// 避免：如果不需要立即执行，不要使用 immediate
watch(data, (newValue) => {
  console.log('数据变化:', newValue)
})
```

### 2. 及时清理副作用

```javascript
watch(source, (newValue, oldValue, onCleanup) => {
  const controller = new AbortController()
  
  fetchData(newValue, { signal: controller.signal })
  
  onCleanup(() => {
    controller.abort() // 取消未完成的请求
  })
})
```

### 3. 避免无限循环

```javascript
// 好的做法：使用条件判断避免循环
watch(count, (newValue) => {
  if (newValue < 10) {
    count.value++ // 不会无限循环
  }
})

// 避免：无条件修改监听的数据
watch(count, (newValue) => {
  count.value++ // 可能导致无限循环
})
```

### 4. 合理使用 deep 选项

```javascript
// 好的做法：只监听需要的属性
watch(() => user.value.name, (newValue) => {
  console.log('用户名变化:', newValue)
})

// 避免：不必要的深度监听
watch(user, (newValue) => {
  console.log('用户变化:', newValue)
}, { deep: true }) // 性能开销较大
```

## 注意事项

1. **oldValue 的限制**：对于对象和数组，oldValue 可能与 newValue 引用相同。
2. **性能影响**：过度使用 watch 和 deep 监听可能影响性能。
3. **内存泄漏**：忘记清理副作用可能导致内存泄漏。
4. **异步操作**：处理异步操作时要注意竞态条件。
5. **响应式丢失**：确保监听的是响应式数据。
6. **清理监听器**：在组件卸载时手动清理监听器。

## 常见问题

### Q: watch 和 watchEffect 有什么区别？

A: watch 需要明确指定监听源，而 watchEffect 自动追踪依赖。watch 可以访问新旧值，watchEffect 不能。

### Q: 如何监听路由变化？

A: 使用 watch 监听路由对象的变化。

### Q: watch 会影响性能吗？

A: 合理使用不会明显影响性能，但过度使用或深度监听可能有影响。

### Q: 如何停止 watch？

A: watch 返回一个停止函数，调用它即可停止监听。

## 总结

Vue 3 的 watch 函数是一个功能强大的响应式 API，通过掌握其各种用法和最佳实践，我们可以：

1. **精确控制响应式行为**：选择合适的监听源和配置选项
2. **处理复杂的数据变化**：支持深度监听、多源监听等复杂场景
3. **优化性能**：合理使用防抖、节流等技术
4. **避免常见陷阱**：注意响应式丢失、内存泄漏等问题
5. **提升代码质量**：写出更清晰、更可维护的代码

在实际项目中，应该根据具体需求选择最合适的监听方式，既要保证功能的正确性，也要注意性能和代码的可维护性。