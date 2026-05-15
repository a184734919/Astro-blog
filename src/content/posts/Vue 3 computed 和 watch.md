---
title: Vue 3 computed 和 watch
published: 2024-04-14
description: '计算属性和侦听器的详细介绍和学习笔记'
image: ''
tags: ["Vue3","响应式"]
category: 'Vue3'
draft: false
lang: 'zh-CN'
---

## 概述

在 Vue 3 的响应式系统中，`computed`（计算属性）和 `watch`（侦听器）是两个最重要的 API，它们都用于处理响应式数据，但在使用场景和设计理念上有显著区别。

理解两者的区别和使用场景对于编写高效、可维护的 Vue 3 应用至关重要。本文将深入分析这两个 API 的特性、用法和最佳实践。

## 核心概念

### Computed（计算属性）

计算属性是基于响应式数据计算得出的值，具有以下特点：

1. **缓存性**：计算结果会被缓存，只有依赖的数据变化时才会重新计算
2. **声明式**：声明性地描述如何从其他数据派生数据
3. **惰性求值**：只有在被访问时才会执行计算
4. **只读性**：默认是只读的，但也可以提供 setter

### Watch（侦听器）

侦听器用于在数据变化时执行副作用，具有以下特点：

1. **命令式**：命令式地描述在数据变化时要做什么
2. **无缓存**：每次数据变化都会执行回调
3. **副作用**：适合执行异步操作、DOM 操作等副作用
4. **灵活性**：支持深度监听、立即执行等多种配置

### 对比分析

| 特性 | Computed | Watch |
|------|----------|-------|
| 主要用途 | 派生数据 | 执行副作用 |
| 缓存 | 有 | 无 |
| 访问新旧值 | 不支持 | 支持 |
| 异步操作 | 不支持 | 支持 |
| 性能 | 更优（有缓存） | 较差（每次都执行） |
| 返回值 | 返回计算结果 | 返回停止函数 |
| 设置器 | 支持 | 不支持 |

## 基本用法

### Computed 基础用法

```vue
<template>
  <div>
    <h2>计算属性基础示例</h2>
    
    <div class="price-calculator">
      <div class="input-group">
        <label>商品价格:</label>
        <input type="number" v-model.number="price" />
      </div>
      
      <div class="input-group">
        <label>数量:</label>
        <input type="number" v-model.number="quantity" />
      </div>
      
      <div class="input-group">
        <label>折扣率 (%):</label>
        <input type="number" v-model.number="discount" />
      </div>
      
      <div class="results">
        <div class="result-item">
          <span>小计:</span>
          <strong>¥{{ subtotal.toFixed(2) }}</strong>
        </div>
        <div class="result-item">
          <span>折扣金额:</span>
          <strong>¥{{ discountAmount.toFixed(2) }}</strong>
        </div>
        <div class="result-item total">
          <span>总计:</span>
          <strong>¥{{ totalPrice.toFixed(2) }}</strong>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const price = ref(100)
const quantity = ref(2)
const discount = ref(10)

// 计算小计
const subtotal = computed(() => {
  console.log('计算小计') // 这个日志只在依赖变化时打印
  return price.value * quantity.value
})

// 计算折扣金额
const discountAmount = computed(() => {
  return subtotal.value * (discount.value / 100)
})

// 计算总价
const totalPrice = computed(() => {
  return subtotal.value - discountAmount.value
})
</script>

<style scoped>
.price-calculator {
  max-width: 500px;
  margin: 20px auto;
  padding: 30px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.input-group {
  margin-bottom: 20px;
}

.input-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
  color: #333;
}

.input-group input {
  width: 100%;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
  font-size: 16px;
}

.input-group input:focus {
  outline: none;
  border-color: #4CAF50;
  box-shadow: 0 0 0 2px rgba(76, 175, 80, 0.2);
}

.results {
  margin-top: 30px;
  padding: 20px;
  background: #f9f9f9;
  border-radius: 8px;
}

.result-item {
  display: flex;
  justify-content: space-between;
  padding: 10px 0;
  border-bottom: 1px solid #e0e0e0;
}

.result-item:last-child {
  border-bottom: none;
}

.result-item.total {
  margin-top: 15px;
  padding-top: 15px;
  border-top: 2px solid #4CAF50;
  border-bottom: none;
  font-size: 18px;
  font-weight: bold;
}

.result-item strong {
  color: #4CAF50;
}
</style>
```

### Watch 基础用法

```vue
<template>
  <div>
    <h2>侦听器基础示例</h2>
    
    <div class="search-demo">
      <div class="search-box">
        <input 
          v-model="searchQuery" 
          placeholder="搜索用户..."
          @keyup.enter="handleSearch"
        />
        <button @click="handleSearch">搜索</button>
      </div>
      
      <div v-if="isLoading" class="loading">
        <div class="spinner"></div>
        <p>搜索中...</p>
      </div>
      
      <div v-else-if="error" class="error">
        {{ error }}
      </div>
      
      <div v-else class="results">
        <div v-if="results.length > 0" class="result-list">
          <div v-for="user in results" :key="user.id" class="result-item">
            <div class="user-info">
              <strong>{{ user.name }}</strong>
              <span>{{ user.email }}</span>
            </div>
          </div>
        </div>
        
        <div v-else-if="searchQuery" class="no-results">
          未找到相关结果
        </div>
      </div>
      
      <div class="search-history">
        <h4>搜索历史:</h4>
        <div v-for="(query, index) in searchHistory" :key="index" class="history-item">
          <span @click="searchQuery = query">{{ query }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const searchQuery = ref('')
const results = ref([])
const isLoading = ref(false)
const error = ref(null)
const searchHistory = ref([])

// 模拟 API 调用
const searchUsers = async (query) => {
  isLoading.value = true
  error.value = null
  
  try {
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
    
    // 添加到搜索历史
    if (query && !searchHistory.value.includes(query)) {
      searchHistory.value.unshift(query)
      if (searchHistory.value.length > 5) {
        searchHistory.value.pop()
      }
    }
  } catch (err) {
    error.value = '搜索失败: ' + err.message
  } finally {
    isLoading.value = false
  }
}

// 防抖搜索
let searchTimeout = null

watch(searchQuery, (newValue) => {
  if (searchTimeout) {
    clearTimeout(searchTimeout)
  }
  
  if (!newValue.trim()) {
    results.value = []
    return
  }
  
  searchTimeout = setTimeout(() => {
    searchUsers(newValue)
  }, 300)
})

const handleSearch = () => {
  if (searchQuery.value.trim()) {
    searchUsers(searchQuery.value)
  }
}
</script>

<style scoped>
.search-demo {
  max-width: 600px;
  margin: 20px auto;
  padding: 30px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.search-box {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.search-box input {
  flex: 1;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
  box-sizing: border-box;
}

.search-box input:focus {
  outline: none;
  border-color: #4CAF50;
}

.search-box button {
  padding: 12px 24px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

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
  padding: 20px;
  color: #f44336;
  background: #ffebee;
  border-radius: 4px;
}

.result-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.result-item {
  padding: 15px;
  background: #f9f9f9;
  border-radius: 4px;
  border: 1px solid #e0e0e0;
}

.user-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.user-info strong {
  color: #333;
}

.user-info span {
  color: #666;
  font-size: 14px;
}

.no-results {
  text-align: center;
  padding: 40px;
  color: #999;
}

.search-history {
  margin-top: 30px;
  padding: 20px;
  background: #f9f9f9;
  border-radius: 8px;
}

.search-history h4 {
  margin-top: 0;
  margin-bottom: 15px;
  color: #333;
}

.history-item {
  display: inline-block;
  margin: 5px;
}

.history-item span {
  padding: 6px 12px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 20px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s;
}

.history-item span:hover {
  background: #4CAF50;
  color: white;
  border-color: #4CAF50;
}
</style>
```

## 实际应用

### 表单验证（Computed 优先）

```vue
<template>
  <div>
    <h2>表单验证示例</h2>
    
    <form class="validation-form" @submit.prevent="handleSubmit">
      <div class="form-group">
        <label>用户名:</label>
        <input 
          v-model="formData.username" 
          :class="{ 'error': !isUsernameValid }"
        />
        <span v-if="!isUsernameValid" class="error-message">{{ usernameError }}</span>
      </div>
      
      <div class="form-group">
        <label>邮箱:</label>
        <input 
          v-model="formData.email" 
          type="email"
          :class="{ 'error': !isEmailValid }"
        />
        <span v-if="!isEmailValid" class="error-message">{{ emailError }}</span>
      </div>
      
      <div class="form-group">
        <label>密码:</label>
        <input 
          v-model="formData.password" 
          type="password"
          :class="{ 'error': !isPasswordValid }"
        />
        <span v-if="!isPasswordValid" class="error-message">{{ passwordError }}</span>
        
        <div class="password-strength">
          <div 
            v-for="level in 4" 
            :key="level"
            class="strength-bar"
            :class="{ 'active': level <= passwordStrength }"
          ></div>
          <span class="strength-label">{{ passwordStrengthText }}</span>
        </div>
      </div>
      
      <div class="form-group">
        <label>确认密码:</label>
        <input 
          v-model="formData.confirmPassword" 
          type="password"
          :class="{ 'error': !isConfirmPasswordValid }"
        />
        <span v-if="!isConfirmPasswordValid" class="error-message">{{ confirmPasswordError }}</span>
      </div>
      
      <div class="form-actions">
        <button 
          type="submit" 
          :disabled="!isFormValid"
          class="submit-btn"
        >
          注册
        </button>
      </div>
      
      <div class="validation-summary">
        <h4>表单状态:</h4>
        <div class="status-indicator" :class="{ 'valid': isFormValid }">
          {{ isFormValid ? '✓ 表单有效，可以提交' : '✗ 表单无效，请检查输入' }}
        </div>
      </div>
    </form>
  </div>
</template>

<script setup>
import { reactive, computed } from 'vue'

const formData = reactive({
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
})

// 使用计算属性进行验证
const isUsernameValid = computed(() => {
  return formData.username.length >= 3 && formData.username.length <= 20
})

const usernameError = computed(() => {
  if (!formData.username) return '用户名不能为空'
  if (formData.username.length < 3) return '用户名至少需要3个字符'
  if (formData.username.length > 20) return '用户名不能超过20个字符'
  return ''
})

const isEmailValid = computed(() => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  return emailRegex.test(formData.email)
})

const emailError = computed(() => {
  if (!formData.email) return '邮箱不能为空'
  if (!isEmailValid.value) return '请输入有效的邮箱地址'
  return ''
})

const isPasswordValid = computed(() => {
  return formData.password.length >= 8 && 
         /[a-zA-Z]/.test(formData.password) && 
         /[0-9]/.test(formData.password)
})

const passwordError = computed(() => {
  if (!formData.password) return '密码不能为空'
  if (formData.password.length < 8) return '密码至少需要8个字符'
  if (!/[a-zA-Z]/.test(formData.password)) return '密码必须包含字母'
  if (!/[0-9]/.test(formData.password)) return '密码必须包含数字'
  return ''
})

const passwordStrength = computed(() => {
  const password = formData.password
  if (!password) return 0
  
  let strength = 0
  if (password.length >= 8) strength++
  if (/[a-z]/.test(password) && /[A-Z]/.test(password)) strength++
  if (/[0-9]/.test(password)) strength++
  if (/[^a-zA-Z0-9]/.test(password)) strength++
  
  return strength
})

const passwordStrengthText = computed(() => {
  const texts = ['', '很弱', '弱', '中等', '强', '很强']
  return texts[passwordStrength.value] || ''
})

const isConfirmPasswordValid = computed(() => {
  return formData.confirmPassword === formData.password && formData.confirmPassword !== ''
})

const confirmPasswordError = computed(() => {
  if (!formData.confirmPassword) return '请确认密码'
  if (!isConfirmPasswordValid.value) return '两次输入的密码不一致'
  return ''
})

const isFormValid = computed(() => {
  return isUsernameValid.value && 
         isEmailValid.value && 
         isPasswordValid.value && 
         isConfirmPasswordValid.value
})

const handleSubmit = () => {
  if (isFormValid.value) {
    console.log('表单提交:', formData)
    alert('注册成功！')
  }
}
</script>

<style scoped>
.validation-form {
  max-width: 500px;
  margin: 20px auto;
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

.form-group input {
  width: 100%;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
  font-size: 16px;
}

.form-group input.error {
  border-color: #f44336;
}

.error-message {
  display: block;
  color: #f44336;
  font-size: 14px;
  margin-top: 5px;
}

.password-strength {
  display: flex;
  align-items: center;
  gap: 5px;
  margin-top: 10px;
}

.strength-bar {
  flex: 1;
  height: 4px;
  background: #e0e0e0;
  border-radius: 2px;
}

.strength-bar.active:nth-child(1) { background: #f44336; }
.strength-bar.active:nth-child(2) { background: #ff9800; }
.strength-bar.active:nth-child(3) { background: #4CAF50; }
.strength-bar.active:nth-child(4) { background: #2196F3; }

.strength-label {
  font-size: 12px;
  color: #666;
  margin-left: 10px;
}

.form-actions {
  margin-top: 30px;
}

.submit-btn {
  width: 100%;
  padding: 12px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 500;
}

.submit-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.validation-summary {
  margin-top: 30px;
  padding: 20px;
  background: #f9f9f9;
  border-radius: 8px;
}

.validation-summary h4 {
  margin-top: 0;
  margin-bottom: 15px;
  color: #333;
}

.status-indicator {
  padding: 12px;
  border-radius: 4px;
  text-align: center;
  background: #ffebee;
  color: #f44336;
  font-weight: 500;
}

.status-indicator.valid {
  background: #e8f5e9;
  color: #4CAF50;
}
</style>
```

### 数据持久化（Watch 优先）

```vue
<template>
  <div>
    <h2>数据持久化示例</h2>
    
    <div class="persistence-demo">
      <div class="data-editor">
        <h3>用户信息编辑</h3>
        
        <div class="form-group">
          <label>姓名:</label>
          <input v-model="userData.name" />
        </div>
        
        <div class="form-group">
          <label>年龄:</label>
          <input type="number" v-model.number="userData.age" />
        </div>
        
        <div class="form-group">
          <label>邮箱:</label>
          <input v-model="userData.email" />
        </div>
        
        <div class="form-group">
          <label>个人简介:</label>
          <textarea v-model="userData.bio" rows="4"></textarea>
        </div>
      </div>
      
      <div class="persistence-status">
        <h4>保存状态:</h4>
        <div class="status-indicator" :class="saveStatus">
          <span v-if="saveStatus === 'saving'">💾 保存中...</span>
          <span v-else-if="saveStatus === 'saved'">✓ 已保存</span>
          <span v-else-if="saveStatus === 'error'">✗ 保存失败</span>
          <span v-else>⏳ 等待编辑</span>
        </div>
        
        <div class="save-info">
          <p>最后保存时间: {{ lastSaveTime || '未保存' }}</p>
          <p>自动保存: {{ autoSaveEnabled ? '开启' : '关闭' }}</p>
        </div>
        
        <div class="storage-actions">
          <button @click="toggleAutoSave" class="toggle-btn">
            {{ autoSaveEnabled ? '关闭自动保存' : '开启自动保存' }}
          </button>
          <button @click="manualSave" class="save-btn">立即保存</button>
          <button @click="clearStorage" class="clear-btn">清除存储</button>
        </div>
      </div>
      
      <div class="storage-preview">
        <h4>存储的数据:</h4>
        <pre>{{ JSON.stringify(storedData, null, 2) }}</pre>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, watch, onMounted } from 'vue'

const STORAGE_KEY = 'user-data-persistence'

const userData = reactive({
  name: '',
  age: 25,
  email: '',
  bio: ''
})

const saveStatus = ref('idle') // idle, saving, saved, error
const autoSaveEnabled = ref(true)
const lastSaveTime = ref('')
const storedData = ref(null)
let saveTimeout = null

// 从本地存储加载数据
onMounted(() => {
  loadData()
})

const loadData = () => {
  try {
    const savedData = localStorage.getItem(STORAGE_KEY)
    if (savedData) {
      const parsed = JSON.parse(savedData)
      Object.assign(userData, parsed.data)
      lastSaveTime.value = parsed.timestamp
      storedData.value = parsed
      console.log('数据已加载')
    }
  } catch (error) {
    console.error('加载数据失败:', error)
  }
}

// 保存数据到本地存储
const saveData = async () => {
  saveStatus.value = 'saving'
  
  try {
    const dataToSave = {
      data: { ...userData },
      timestamp: new Date().toLocaleString()
    }
    
    // 模拟异步保存
    await new Promise(resolve => setTimeout(resolve, 300))
    
    localStorage.setItem(STORAGE_KEY, JSON.stringify(dataToSave))
    storedData.value = dataToSave
    lastSaveTime.value = dataToSave.timestamp
    saveStatus.value = 'saved'
    
    // 2秒后重置状态
    setTimeout(() => {
      saveStatus.value = 'idle'
    }, 2000)
  } catch (error) {
    saveStatus.value = 'error'
    console.error('保存失败:', error)
  }
}

// 使用 watch 监听数据变化并自动保存
watch(
  () => ({ ...userData }), // 监听整个对象
  (newValue) => {
    if (!autoSaveEnabled.value) return
    
    // 清除之前的定时器
    if (saveTimeout) {
      clearTimeout(saveTimeout)
    }
    
    // 设置新的定时器，1秒后自动保存
    saveTimeout = setTimeout(() => {
      saveData()
    }, 1000)
  },
  { deep: true }
)

const toggleAutoSave = () => {
  autoSaveEnabled.value = !autoSaveEnabled.value
}

const manualSave = () => {
  saveData()
}

const clearStorage = () => {
  if (confirm('确定要清除存储的数据吗？')) {
    localStorage.removeItem(STORAGE_KEY)
    storedData.value = null
    lastSaveTime.value = ''
    saveStatus.value = 'idle'
    console.log('存储已清除')
  }
}
</script>

<style scoped>
.persistence-demo {
  max-width: 800px;
  margin: 20px auto;
  padding: 30px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.data-editor {
  margin-bottom: 30px;
}

.data-editor h3 {
  margin-top: 0;
  margin-bottom: 20px;
  color: #333;
}

.form-group {
  margin-bottom: 15px;
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
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
  font-family: inherit;
}

.form-group textarea {
  resize: vertical;
}

.persistence-status {
  background: #f9f9f9;
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.persistence-status h4 {
  margin-top: 0;
  margin-bottom: 15px;
  color: #333;
}

.status-indicator {
  padding: 12px;
  border-radius: 4px;
  text-align: center;
  margin-bottom: 15px;
  font-weight: 500;
}

.status-indicator.saving {
  background: #e3f2fd;
  color: #1976d2;
}

.status-indicator.saved {
  background: #e8f5e9;
  color: #4CAF50;
}

.status-indicator.error {
  background: #ffebee;
  color: #f44336;
}

.save-info {
  margin-bottom: 15px;
  color: #666;
  font-size: 14px;
}

.save-info p {
  margin: 5px 0;
}

.storage-actions {
  display: flex;
  gap: 10px;
}

.storage-actions button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
}

.toggle-btn {
  background: #6c757d;
  color: white;
}

.save-btn {
  background: #4CAF50;
  color: white;
}

.clear-btn {
  background: #f44336;
  color: white;
}

.storage-preview {
  background: #f5f5f5;
  padding: 20px;
  border-radius: 8px;
  overflow-x: auto;
}

.storage-preview h4 {
  margin-top: 0;
  margin-bottom: 15px;
  color: #333;
}

.storage-preview pre {
  background: white;
  padding: 15px;
  border-radius: 4px;
  overflow-x: auto;
  font-size: 14px;
  line-height: 1.5;
}
</style>
```

### 综合应用示例

```vue
<template>
  <div>
    <h2>综合应用：购物车</h2>
    
    <div class="shopping-cart">
      <!-- 商品列表 -->
      <div class="cart-items">
        <h3>购物车商品 ({{ cartItemCount }})</h3>
        
        <div v-if="cartItems.length === 0" class="empty-cart">
          <p>购物车是空的</p>
          <button @click="addSampleItems" class="add-sample-btn">添加示例商品</button>
        </div>
        
        <div v-else class="item-list">
          <div v-for="item in cartItems" :key="item.id" class="cart-item">
            <div class="item-info">
              <h4>{{ item.name }}</h4>
              <p class="item-price">¥{{ item.price.toFixed(2) }}</p>
            </div>
            
            <div class="item-quantity">
              <button @click="decrementQuantity(item)" :disabled="item.quantity <= 1">
                -
              </button>
              <span>{{ item.quantity }}</span>
              <button @click="incrementQuantity(item)">+</button>
            </div>
            
            <div class="item-subtotal">
              <strong>¥{{ (item.price * item.quantity).toFixed(2) }}</strong>
            </div>
            
            <button @click="removeItem(item)" class="remove-btn">
              删除
            </button>
          </div>
        </div>
      </div>
      
      <!-- 购物车摘要 -->
      <div class="cart-summary">
        <h3>订单摘要</h3>
        
        <div class="summary-row">
          <span>商品总数:</span>
          <strong>{{ cartItemCount }} 件</strong>
        </div>
        
        <div class="summary-row">
          <span>商品总价:</span>
          <strong>¥{{ subtotal.toFixed(2) }}</strong>
        </div>
        
        <div class="summary-row discount">
          <span>折扣优惠:</span>
          <strong>-¥{{ discount.toFixed(2) }}</strong>
        </div>
        
        <div class="summary-row shipping">
          <span>运费:</span>
          <strong>{{ shippingFee === 0 ? '免运费' : `¥${shippingFee.toFixed(2)}` }}</strong>
        </div>
        
        <div class="summary-row total">
          <span>总计:</span>
          <strong>¥{{ totalPrice.toFixed(2) }}</strong>
        </div>
        
        <!-- 优惠券输入 -->
        <div class="coupon-section">
          <input 
            v-model="couponCode" 
            placeholder="输入优惠券代码"
            @keyup.enter="applyCoupon"
          />
          <button @click="applyCoupon" :disabled="!couponCode">应用</button>
        </div>
        
        <div v-if="appliedCoupon" class="applied-coupon">
          <span>已应用优惠券: {{ appliedCoupon }}</span>
          <button @click="removeCoupon">移除</button>
        </div>
        
        <button 
          @click="checkout" 
          :disabled="cartItems.length === 0"
          class="checkout-btn"
        >
          结算 (¥{{ totalPrice.toFixed(2) }})
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

// 购物车数据
const cartItems = ref([])
const couponCode = ref('')
const appliedCoupon = ref(null)
const discountRate = ref(0)

// 示例商品
const sampleProducts = [
  { id: 1, name: '商品 A', price: 99.00 },
  { id: 2, name: '商品 B', price: 199.00 },
  { id: 3, name: '商品 C', price: 299.00 }
]

// 优惠券配置
const coupons = {
  'SAVE10': 0.1,  // 10% 折扣
  'SAVE20': 0.2,  // 20% 折扣
  'FREE50': 50    // 固定减免 50 元
}

// 使用 computed 计算派生数据
const cartItemCount = computed(() => {
  return cartItems.value.reduce((sum, item) => sum + item.quantity, 0)
})

const subtotal = computed(() => {
  return cartItems.value.reduce((sum, item) => sum + item.price * item.quantity, 0)
})

const discount = computed(() => {
  if (appliedCoupon.value && coupons[appliedCoupon.value]) {
    const couponValue = coupons[appliedCoupon.value]
    if (couponValue < 1) {
      // 百分比折扣
      return subtotal.value * couponValue
    } else {
      // 固定金额减免
      return Math.min(couponValue, subtotal.value)
    }
  }
  return 0
})

const shippingFee = computed(() => {
  // 满 299 免运费
  return subtotal.value >= 299 ? 0 : 10
})

const totalPrice = computed(() => {
  return Math.max(0, subtotal.value - discount.value + shippingFee.value)
})

// 使用 watch 监听购物车变化并保存
watch(
  cartItems,
  (newItems) => {
    console.log('购物车更新:', newItems)
    localStorage.setItem('shopping-cart', JSON.stringify(newItems))
  },
  { deep: true }
)

// 方法
const addSampleItems = () => {
  cartItems.value = sampleProducts.map(product => ({
    ...product,
    quantity: 1
  }))
}

const incrementQuantity = (item) => {
  item.quantity++
}

const decrementQuantity = (item) => {
  if (item.quantity > 1) {
    item.quantity--
  }
}

const removeItem = (item) => {
  const index = cartItems.value.indexOf(item)
  if (index > -1) {
    cartItems.value.splice(index, 1)
  }
}

const applyCoupon = () => {
  if (couponCode.value && coupons[couponCode.value]) {
    appliedCoupon.value = couponCode.value
    couponCode.value = ''
    console.log('优惠券已应用:', appliedCoupon.value)
  } else {
    alert('无效的优惠券代码')
  }
}

const removeCoupon = () => {
  appliedCoupon.value = null
}

const checkout = () => {
  if (cartItems.value.length === 0) return
  
  alert(`结算成功！总金额: ¥${totalPrice.value.toFixed(2)}`)
  cartItems.value = []
  appliedCoupon.value = null
}

// 初始化：从本地存储加载购物车
onMounted(() => {
  const savedCart = localStorage.getItem('shopping-cart')
  if (savedCart) {
    try {
      cartItems.value = JSON.parse(savedCart)
    } catch (error) {
      console.error('加载购物车失败:', error)
    }
  }
})
</script>

<style scoped>
.shopping-cart {
  max-width: 1000px;
  margin: 20px auto;
  padding: 30px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.cart-items {
  flex: 1;
  margin-bottom: 30px;
}

.cart-items h3 {
  margin-top: 0;
  margin-bottom: 20px;
  color: #333;
}

.empty-cart {
  text-align: center;
  padding: 40px;
  color: #999;
}

.add-sample-btn {
  margin-top: 15px;
  padding: 10px 20px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.item-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.cart-item {
  display: grid;
  grid-template-columns: 1fr auto auto auto;
  gap: 15px;
  padding: 20px;
  background: #f9f9f9;
  border-radius: 8px;
  align-items: center;
}

.item-info h4 {
  margin: 0 0 5px 0;
  color: #333;
}

.item-price {
  margin: 0;
  color: #666;
  font-size: 14px;
}

.item-quantity {
  display: flex;
  align-items: center;
  gap: 10px;
}

.item-quantity button {
  width: 30px;
  height: 30px;
  border: 1px solid #ddd;
  background: white;
  border-radius: 4px;
  cursor: pointer;
  font-size: 18px;
}

.item-quantity button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.item-quantity span {
  font-weight: 500;
  min-width: 30px;
  text-align: center;
}

.item-subtotal strong {
  color: #4CAF50;
  font-size: 18px;
}

.remove-btn {
  padding: 8px 16px;
  background: #f44336;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.cart-summary {
  background: #f9f9f9;
  padding: 25px;
  border-radius: 8px;
  min-width: 300px;
}

.cart-summary h3 {
  margin-top: 0;
  margin-bottom: 20px;
  color: #333;
}

.summary-row {
  display: flex;
  justify-content: space-between;
  padding: 10px 0;
  border-bottom: 1px solid #e0e0e0;
}

.summary-row.discount {
  color: #4CAF50;
}

.summary-row.shipping {
  color: #666;
}

.summary-row.total {
  font-size: 20px;
  font-weight: bold;
  border-top: 2px solid #333;
  border-bottom: none;
  padding-top: 15px;
  margin-top: 10px;
}

.coupon-section {
  display: flex;
  gap: 10px;
  margin: 20px 0;
}

.coupon-section input {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
}

.coupon-section button {
  padding: 10px 20px;
  background: #2196F3;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.coupon-section button:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.applied-coupon {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background: #e8f5e9;
  border-radius: 4px;
  margin-bottom: 15px;
}

.applied-coupon button {
  background: #f44336;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  padding: 5px 10px;
  font-size: 12px;
}

.checkout-btn {
  width: 100%;
  padding: 15px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 16px;
  font-weight: bold;
  margin-top: 10px;
}

.checkout-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.checkout-btn:hover:not(:disabled) {
  background: #45a049;
}
</style>
```

## 最佳实践

### 1. 选择合适的工具

```javascript
// 好的做法：使用 computed 处理派生数据
const fullName = computed(() => {
  return `${firstName.value} ${lastName.value}`
})

// 好的做法：使用 watch 处理副作用
watch(userId, async (newId) => {
  userData.value = await fetchUser(newId)
})

// 避免：在 computed 中执行副作用
const invalidComputed = computed(() => {
  console.log('副作用') // 不应该在 computed 中
  return someData.value * 2
})
```

### 2. 避免无限循环

```javascript
// 好的做法：使用条件判断
watch(data, (newValue) => {
  if (newValue !== someOtherValue.value) {
    someOtherValue.value = newValue
  }
})

// 避免：无条件的双向绑定
watch(data, (newValue) => {
  someOtherValue.value = newValue // 可能导致无限循环
})
```

### 3. 性能优化

```javascript
// 好的做法：利用 computed 的缓存
const expensiveComputed = computed(() => {
  return heavyCalculation(data1.value, data2.value)
})

// 避免：在 watch 中重复计算
watch([data1, data2], () => {
  const result = heavyCalculation(data1.value, data2.value) // 每次都计算
})
```

### 4. 类型安全（TypeScript）

```typescript
// Computed 类型定义
const count = ref(0)
const doubled = computed<number>(() => count.value * 2)

// Watch 类型定义
watch(
  count,
  (newValue: number, oldValue: number) => {
    console.log(`${oldValue} -> ${newValue}`)
  }
)
```

## 注意事项

1. **Computed 缓存**：computed 有缓存机制，只有依赖变化时才重新计算。
2. **Watch 执行时机**：watch 可以配置执行时机（pre、post、sync）。
3. **异步操作**：computed 不支持异步，watch 支持异步操作。
4. **性能考虑**：频繁变化的响应式数据使用 computed 更高效。
5. **内存泄漏**：watch 的副作用要及时清理，避免内存泄漏。
6. **响应式丢失**：确保在 computed 和 watch 中使用的是响应式数据。
7. **调试难度**：复杂的依赖关系可能增加调试难度。

## 常见问题

### Q: computed 和 watch 应该如何选择？

A: 需要派生数据时使用 computed，需要执行副作用时使用 watch。

### Q: computed 会影响性能吗？

A: 通常不会，因为 computed 有缓存机制，比 watch 更高效。

### Q: watch 可以监听多个数据源吗？

A: 可以，使用数组形式监听多个数据源。

### Q: 如何停止 watch？

A: watch 返回一个停止函数，调用它即可停止监听。

## 总结

Vue 3 的 computed 和 watch 各有优势，理解它们的区别和适用场景对于编写高效的 Vue 应用至关重要：

**Computed 适用于：**
- 派生数据的计算
- 需要缓存的计算结果
- 声明式的数据转换
- 性能敏感的场景

**Watch 适用于：**
- 执行异步操作
- 数据持久化
- 复杂的副作用逻辑
- 需要访问新旧值的场景

**最佳实践：**
1. 优先使用 computed 处理派生数据
2. 使用 watch 处理副作用和异步操作
3. 避免在 computed 中执行副作用
4. 注意性能优化和内存管理
5. 合理使用 watch 的配置选项

通过合理选择和使用这两个 API，我们可以构建出既高效又易于维护的 Vue 3 应用。