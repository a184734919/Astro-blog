---
title: Vue 3 计算属性
published: 2024-01-08
description: 'computed 函数的使用的详细介绍和学习笔记'
image: ''
tags: ["Vue3","computed"]
category: 'Vue3'
draft: false
lang: 'zh-CN'
---

## 概述

计算属性（Computed Properties）是 Vue 中非常重要的特性，它允许我们基于响应式数据派生出新的值。计算属性会自动缓存结果，只有在依赖的响应式数据发生变化时才会重新计算，这使得计算属性比方法调用更高效。

## 核心概念

### 计算属性的特点

1. **缓存机制**: 计算属性会缓存结果，避免重复计算
2. **响应式依赖**: 自动追踪依赖的响应式数据
3. **惰性求值**: 只有在访问时才会计算
4. **只读/可写**: 支持只读和可写两种模式

### 计算属性 vs 方法

```javascript
// 方法 - 每次调用都会重新执行
methods: {
  doubleCount() {
    console.log('计算双倍计数');
    return this.count * 2;
  }
}

// 计算属性 - 只在依赖变化时重新计算
computed: {
  doubleCount() {
    console.log('计算双倍计数');
    return this.count * 2;
  }
}
```

### 计算属性 vs 侦听器

- **计算属性**: 适用于派生数据，自动更新
- **侦听器**: 适用于副作用，需要手动处理

## 基本用法

### 1. 基础计算属性

```vue
<script setup>
import { ref, computed } from 'vue';

const firstName = ref('张');
const lastName = ref('三');
const age = ref(25);

// 只读计算属性
const fullName = computed(() => {
  return firstName.value + lastName.value;
});

const isAdult = computed(() => {
  return age.value >= 18;
});

const userInfo = computed(() => {
  return `${fullName.value}，${age.value}岁，${isAdult.value ? '成年' : '未成年'}`;
});
</script>

<template>
  <div>
    <input v-model="firstName" placeholder="姓">
    <input v-model="lastName" placeholder="名">
    <input v-model.number="age" type="number" placeholder="年龄">
    
    <p>全名: {{ fullName }}</p>
    <p>成年: {{ isAdult ? '是' : '否' }}</p>
    <p>用户信息: {{ userInfo }}</p>
  </div>
</template>
```

### 2. 可写计算属性

```vue
<script setup>
import { ref, computed } from 'vue';

const firstName = ref('张');
const lastName = ref('三');

// 可写计算属性
const fullName = computed({
  // getter
  get() {
    return firstName.value + lastName.value;
  },
  // setter
  set(newValue) {
    // 将新值分割为姓和名
    const names = newValue.split('');
    if (names.length > 0) {
      firstName.value = names[0];
      lastName.value = names.slice(1).join('');
    }
  }
});

function setFullName(name) {
  fullName.value = name;
}
</script>

<template>
  <div>
    <p>姓: {{ firstName }}</p>
    <p>名: {{ lastName }}</p>
    <p>全名: {{ fullName }}</p>
    
    <input 
      v-model="fullName" 
      placeholder="输入全名"
    >
    
    <button @click="setFullName('李四')">设置为李四</button>
  </div>
</template>
```

### 3. 计算属性的缓存

```vue
<script setup>
import { ref, computed } from 'vue';

const count = ref(0);

const expensiveComputation = computed(() => {
  console.log('执行复杂计算...');
  let result = 0;
  for (let i = 0; i < 1000000; i++) {
    result += i;
  }
  return result + count.value;
});

function accessComputed() {
  console.log('访问计算属性:', expensiveComputation.value);
}
</script>

<template>
  <div>
    <p>计数: {{ count }}</p>
    <p>计算结果: {{ expensiveComputation }}</p>
    
    <button @click="count++">增加计数</button>
    <button @click="accessComputed">访问计算属性</button>
  </div>
</template>
```

### 4. 计算属性的类型

```vue
<script setup>
import { ref, computed } from 'vue';

const numbers = ref([1, 2, 3, 4, 5]);

// 数组类型的计算属性
const evenNumbers = computed(() => {
  return numbers.value.filter(num => num % 2 === 0);
});

const sum = computed(() => {
  return numbers.value.reduce((acc, num) => acc + num, 0);
});

const average = computed(() => {
  return sum.value / numbers.value.length;
});

// 对象类型的计算属性
const statistics = computed(() => {
  return {
    count: numbers.value.length,
    sum: sum.value,
    average: average.value,
    min: Math.min(...numbers.value),
    max: Math.max(...numbers.value)
  };
});

function addNumber() {
  numbers.value.push(numbers.value.length + 1);
}
</script>

<template>
  <div>
    <h2>数字统计</h2>
    <p>原始数组: {{ numbers }}</p>
    <p>偶数: {{ evenNumbers }}</p>
    <p>总和: {{ sum }}</p>
    <p>平均值: {{ average }}</p>
    
    <h3>统计信息</h3>
    <ul>
      <li>数量: {{ statistics.count }}</li>
      <li>总和: {{ statistics.sum }}</li>
      <li>平均值: {{ statistics.average }}</li>
      <li>最小值: {{ statistics.min }}</li>
      <li>最大值: {{ statistics.max }}</li>
    </ul>
    
    <button @click="addNumber">添加数字</button>
  </div>
</template>
```

## 实际应用

### 1. 表单验证

```vue
<script setup>
import { ref, computed } from 'vue';

const formData = reactive({
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
});

const errors = reactive({
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
});

// 用户名验证
const usernameError = computed(() => {
  if (!formData.username) {
    return '用户名不能为空';
  }
  if (formData.username.length < 3) {
    return '用户名至少3个字符';
  }
  if (formData.username.length > 20) {
    return '用户名最多20个字符';
  }
  return '';
});

// 邮箱验证
const emailError = computed(() => {
  if (!formData.email) {
    return '邮箱不能为空';
  }
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(formData.email)) {
    return '邮箱格式不正确';
  }
  return '';
});

// 密码验证
const passwordError = computed(() => {
  if (!formData.password) {
    return '密码不能为空';
  }
  if (formData.password.length < 6) {
    return '密码至少6个字符';
  }
  if (!/[A-Z]/.test(formData.password)) {
    return '密码必须包含大写字母';
  }
  if (!/[0-9]/.test(formData.password)) {
    return '密码必须包含数字';
  }
  return '';
});

// 确认密码验证
const confirmPasswordError = computed(() => {
  if (!formData.confirmPassword) {
    return '请确认密码';
  }
  if (formData.confirmPassword !== formData.password) {
    return '两次密码不一致';
  }
  return '';
});

// 表单是否有效
const isFormValid = computed(() => {
  return formData.username && 
         formData.email && 
         formData.password && 
         formData.confirmPassword &&
         !usernameError.value && 
         !emailError.value && 
         !passwordError.value && 
         !confirmPasswordError.value;
});

function handleSubmit() {
  if (isFormValid.value) {
    console.log('表单提交:', formData);
    // 提交逻辑
  }
}
</script>

<template>
  <form @submit.prevent="handleSubmit" class="registration-form">
    <h2>用户注册</h2>
    
    <div class="form-group">
      <label>用户名:</label>
      <input 
        v-model="formData.username" 
        type="text"
        :class="{ error: usernameError }"
        placeholder="输入用户名"
      >
      <span v-if="usernameError" class="error-message">{{ usernameError }}</span>
    </div>
    
    <div class="form-group">
      <label>邮箱:</label>
      <input 
        v-model="formData.email" 
        type="email"
        :class="{ error: emailError }"
        placeholder="输入邮箱"
      >
      <span v-if="emailError" class="error-message">{{ emailError }}</span>
    </div>
    
    <div class="form-group">
      <label>密码:</label>
      <input 
        v-model="formData.password" 
        type="password"
        :class="{ error: passwordError }"
        placeholder="输入密码"
      >
      <span v-if="passwordError" class="error-message">{{ passwordError }}</span>
    </div>
    
    <div class="form-group">
      <label>确认密码:</label>
      <input 
        v-model="formData.confirmPassword" 
        type="password"
        :class="{ error: confirmPasswordError }"
        placeholder="确认密码"
      >
      <span v-if="confirmPasswordError" class="error-message">{{ confirmPasswordError }}</span>
    </div>
    
    <button type="submit" :disabled="!isFormValid">
      注册
    </button>
  </form>
</template>

<style scoped>
.registration-form {
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.form-group {
  margin-bottom: 20px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
}

input.error {
  border-color: red;
}

.error-message {
  color: red;
  font-size: 12px;
  margin-top: 5px;
  display: block;
}

button {
  width: 100%;
  padding: 12px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
}

button:disabled {
  background: #cccccc;
  cursor: not-allowed;
}
</style>
```

### 2. 购物车计算

```vue
<script setup>
import { ref, computed } from 'vue';

const products = ref([
  {
    id: 1,
    name: '商品 A',
    price: 100,
    quantity: 2,
    image: 'https://via.placeholder.com/100'
  },
  {
    id: 2,
    name: '商品 B',
    price: 200,
    quantity: 1,
    image: 'https://via.placeholder.com/100'
  },
  {
    id: 3,
    name: '商品 C',
    price: 150,
    quantity: 3,
    image: 'https://via.placeholder.com/100'
  }
]);

const discount = ref(0);
const shipping = ref(10);

// 计算每个商品的小计
const itemSubtotals = computed(() => {
  return products.value.map(product => ({
    ...product,
    subtotal: product.price * product.quantity
  }));
});

// 计算商品总价
const subtotal = computed(() => {
  return itemSubtotals.value.reduce((sum, item) => sum + item.subtotal, 0);
});

// 计算折扣金额
const discountAmount = computed(() => {
  return subtotal.value * (discount.value / 100);
});

// 计算小计（减去折扣）
const discountedSubtotal = computed(() => {
  return subtotal.value - discountAmount.value;
});

// 计算运费（满额免运费）
const finalShipping = computed(() => {
  return discountedSubtotal.value >= 500 ? 0 : shipping.value;
});

// 计算总价
const total = computed(() => {
  return discountedSubtotal.value + finalShipping.value;
});

// 购物车是否为空
const isEmpty = computed(() => {
  return products.value.length === 0;
});

// 商品数量
const itemCount = computed(() => {
  return products.value.reduce((sum, product) => sum + product.quantity, 0);
});

// 更新商品数量
function updateQuantity(productId, quantity) {
  const product = products.value.find(p => p.id === productId);
  if (product) {
    product.quantity = Math.max(1, quantity);
  }
}

// 移除商品
function removeProduct(productId) {
  const index = products.value.findIndex(p => p.id === productId);
  if (index > -1) {
    products.value.splice(index, 1);
  }
}

// 清空购物车
function clearCart() {
  products.value = [];
}
</script>

<template>
  <div class="shopping-cart">
    <h2>购物车</h2>
    
    <div v-if="isEmpty" class="empty-cart">
      <p>购物车是空的</p>
    </div>
    
    <div v-else class="cart-content">
      <div class="cart-items">
        <div v-for="item in itemSubtotals" :key="item.id" class="cart-item">
          <img :src="item.image" :alt="item.name" class="item-image">
          <div class="item-details">
            <h3>{{ item.name }}</h3>
            <p class="price">单价: ¥{{ item.price }}</p>
          </div>
          <div class="item-quantity">
            <button @click="updateQuantity(item.id, item.quantity - 1)">-</button>
            <span>{{ item.quantity }}</span>
            <button @click="updateQuantity(item.id, item.quantity + 1)">+</button>
          </div>
          <div class="item-subtotal">
            <p>小计: ¥{{ item.subtotal }}</p>
          </div>
          <button @click="removeProduct(item.id)" class="remove-btn">删除</button>
        </div>
      </div>
      
      <div class="cart-summary">
        <h3>订单摘要</h3>
        
        <div class="summary-row">
          <span>商品数量:</span>
          <span>{{ itemCount }} 件</span>
        </div>
        
        <div class="summary-row">
          <span>商品小计:</span>
          <span>¥{{ subtotal }}</span>
        </div>
        
        <div class="summary-row">
          <span>折扣:</span>
          <div class="discount-control">
            <input 
              v-model.number="discount" 
              type="number" 
              min="0" 
              max="100"
              placeholder="折扣百分比"
            >
            <span>%</span>
          </div>
        </div>
        
        <div class="summary-row discount-amount">
          <span>折扣金额:</span>
          <span>-¥{{ discountAmount }}</span>
        </div>
        
        <div class="summary-row">
          <span>运费:</span>
          <span>{{ finalShipping === 0 ? '免运费' : '¥' + finalShipping }}</span>
        </div>
        
        <div class="summary-row total">
          <span>总价:</span>
          <span>¥{{ total }}</span>
        </div>
        
        <div class="cart-actions">
          <button @click="clearCart" class="clear-btn">清空购物车</button>
          <button class="checkout-btn">去结算</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.shopping-cart {
  max-width: 1000px;
  margin: 0 auto;
  padding: 20px;
}

.empty-cart {
  text-align: center;
  padding: 40px;
  color: #666;
}

.cart-content {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 20px;
}

.cart-items {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 15px;
}

.cart-item {
  display: flex;
  align-items: center;
  padding: 15px;
  border-bottom: 1px solid #eee;
}

.cart-item:last-child {
  border-bottom: none;
}

.item-image {
  width: 80px;
  height: 80px;
  object-fit: cover;
  border-radius: 4px;
  margin-right: 15px;
}

.item-details {
  flex: 1;
}

.item-details h3 {
  margin: 0 0 5px 0;
}

.item-details .price {
  margin: 0;
  color: #666;
}

.item-quantity {
  display: flex;
  align-items: center;
  gap: 10px;
  margin: 0 15px;
}

.item-quantity button {
  width: 30px;
  height: 30px;
  border: 1px solid #ddd;
  background: white;
  border-radius: 4px;
  cursor: pointer;
}

.item-subtotal {
  margin: 0 15px;
  min-width: 100px;
  text-align: right;
}

.remove-btn {
  padding: 5px 10px;
  background: #f44336;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.cart-summary {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 15px;
  height: fit-content;
}

.cart-summary h3 {
  margin-top: 0;
  border-bottom: 1px solid #eee;
  padding-bottom: 10px;
}

.summary-row {
  display: flex;
  justify-content: space-between;
  margin: 15px 0;
}

.discount-control {
  display: flex;
  align-items: center;
  gap: 5px;
}

.discount-control input {
  width: 60px;
  padding: 5px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.discount-amount {
  color: #4CAF50;
}

.total {
  font-size: 18px;
  font-weight: bold;
  border-top: 2px solid #eee;
  padding-top: 15px;
  margin-top: 20px;
}

.cart-actions {
  margin-top: 20px;
  display: flex;
  gap: 10px;
}

.clear-btn {
  flex: 1;
  padding: 10px;
  background: #f44336;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.checkout-btn {
  flex: 2;
  padding: 10px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

@media (max-width: 768px) {
  .cart-content {
    grid-template-columns: 1fr;
  }
}
</style>
```

### 3. 搜索和过滤

```vue
<script setup>
import { ref, computed } from 'vue';

const items = ref([
  { id: 1, name: 'Apple', category: 'Fruit', price: 5, stock: 100 },
  { id: 2, name: 'Banana', category: 'Fruit', price: 3, stock: 150 },
  { id: 3, name: 'Carrot', category: 'Vegetable', price: 2, stock: 200 },
  { id: 4, name: 'Tomato', category: 'Vegetable', price: 4, stock: 80 },
  { id: 5, name: 'Chicken', category: 'Meat', price: 20, stock: 50 },
  { id: 6, name: 'Beef', category: 'Meat', price: 35, stock: 30 },
  { id: 7, name: 'Fish', category: 'Seafood', price: 25, stock: 40 },
  { id: 8, name: 'Shrimp', category: 'Seafood', price: 30, stock: 60 }
]);

const searchQuery = ref('');
const selectedCategory = ref('All');
const sortBy = ref('name');
const sortOrder = ref('asc');
const minPrice = ref(0);
const maxPrice = ref(100);

// 过滤后的商品
const filteredItems = computed(() => {
  let result = items.value;
  
  // 搜索过滤
  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase();
    result = result.filter(item => 
      item.name.toLowerCase().includes(query)
    );
  }
  
  // 分类过滤
  if (selectedCategory.value !== 'All') {
    result = result.filter(item => 
      item.category === selectedCategory.value
    );
  }
  
  // 价格范围过滤
  result = result.filter(item => 
    item.price >= minPrice.value && 
    item.price <= maxPrice.value
  );
  
  return result;
});

// 排序后的商品
const sortedItems = computed(() => {
  const result = [...filteredItems.value];
  
  result.sort((a, b) => {
    let comparison = 0;
    
    if (sortBy.value === 'name') {
      comparison = a.name.localeCompare(b.name);
    } else if (sortBy.value === 'price') {
      comparison = a.price - b.price;
    } else if (sortBy.value === 'stock') {
      comparison = a.stock - b.stock;
    }
    
    return sortOrder.value === 'asc' ? comparison : -comparison;
  });
  
  return result;
});

// 可用分类
const categories = computed(() => {
  const cats = ['All', ...new Set(items.value.map(item => item.category))];
  return cats;
});

// 统计信息
const statistics = computed(() => {
  return {
    total: items.value.length,
    filtered: filteredItems.value.length,
    categories: categories.value.length - 1,
    avgPrice: filteredItems.value.length > 0 
      ? (filteredItems.value.reduce((sum, item) => sum + item.price, 0) / filteredItems.value.length).toFixed(2)
      : 0
  };
});

// 重置过滤
function resetFilters() {
  searchQuery.value = '';
  selectedCategory.value = 'All';
  sortBy.value = 'name';
  sortOrder.value = 'asc';
  minPrice.value = 0;
  maxPrice.value = 100;
}
</script>

<template>
  <div class="product-list">
    <h2>商品列表</h2>
    
    <!-- 搜索和过滤 -->
    <div class="filters">
      <div class="filter-group">
        <label>搜索:</label>
        <input 
          v-model="searchQuery" 
          type="text" 
          placeholder="搜索商品名称..."
        >
      </div>
      
      <div class="filter-group">
        <label>分类:</label>
        <select v-model="selectedCategory">
          <option v-for="category in categories" :key="category" :value="category">
            {{ category }}
          </option>
        </select>
      </div>
      
      <div class="filter-group">
        <label>排序:</label>
        <select v-model="sortBy">
          <option value="name">名称</option>
          <option value="price">价格</option>
          <option value="stock">库存</option>
        </select>
        
        <select v-model="sortOrder">
          <option value="asc">升序</option>
          <option value="desc">降序</option>
        </select>
      </div>
      
      <div class="filter-group">
        <label>价格范围:</label>
        <input 
          v-model.number="minPrice" 
          type="number" 
          min="0" 
          placeholder="最低价"
        >
        <span>-</span>
        <input 
          v-model.number="maxPrice" 
          type="number" 
          min="0" 
          placeholder="最高价"
        >
      </div>
      
      <button @click="resetFilters" class="reset-btn">重置</button>
    </div>
    
    <!-- 统计信息 -->
    <div class="statistics">
      <p>总商品: {{ statistics.total }}</p>
      <p>筛选结果: {{ statistics.filtered }}</p>
      <p>分类数量: {{ statistics.categories }}</p>
      <p>平均价格: ¥{{ statistics.avgPrice }}</p>
    </div>
    
    <!-- 商品列表 -->
    <div class="products">
      <div v-if="sortedItems.length === 0" class="no-results">
        没有找到匹配的商品
      </div>
      
      <div v-else class="product-grid">
        <div v-for="item in sortedItems" :key="item.id" class="product-card">
          <h3>{{ item.name }}</h3>
          <p>分类: {{ item.category }}</p>
          <p>价格: ¥{{ item.price }}</p>
          <p>库存: {{ item.stock }}</p>
          <button :disabled="item.stock === 0">
            {{ item.stock > 0 ? '加入购物车' : '缺货' }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.product-list {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.filters {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
  margin-bottom: 20px;
  padding: 20px;
  background: #f5f5f5;
  border-radius: 8px;
}

.filter-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.filter-group label {
  font-weight: bold;
  white-space: nowrap;
}

.filter-group input,
.filter-group select {
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.reset-btn {
  padding: 8px 16px;
  background: #2196F3;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.statistics {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
  padding: 15px;
  background: #e3f2fd;
  border-radius: 8px;
}

.statistics p {
  margin: 0;
}

.no-results {
  text-align: center;
  padding: 40px;
  color: #666;
}

.product-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 20px;
}

.product-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  padding: 15px;
  text-align: center;
  transition: transform 0.2s;
}

.product-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 8px rgba(0,0,0,0.1);
}

.product-card h3 {
  margin-top: 0;
  color: #333;
}

.product-card p {
  margin: 8px 0;
  color: #666;
}

.product-card button {
  margin-top: 10px;
  padding: 8px 16px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.product-card button:disabled {
  background: #cccccc;
  cursor: not-allowed;
}
</style>
```

## 注意事项

### 最佳实践

1. **优先使用计算属性**: 对于派生数据，优先使用计算属性而非方法
2. **避免副作用**: 计算属性应该是纯函数，不应有副作用
3. **合理使用可写计算属性**: 只在需要时使用 setter
4. **性能考虑**: 对于复杂计算，考虑使用缓存或优化算法
5. **类型安全**: 在 TypeScript 中正确声明计算属性的类型

### 常见错误

```javascript
// 错误 1: 在计算属性中修改响应式数据
const count = ref(0);

const doubled = computed(() => {
  count.value++; // 错误：不要在计算属性中修改数据
  return count.value * 2;
});

// 正确做法
const doubled = computed(() => count.value * 2);

// 错误 2: 计算属性中使用异步操作
const data = computed(async () => {
  const response = await fetch('https://api.example.com/data');
  return response.json();
});

// 正确做法：使用侦听器或组合式函数
const data = ref(null);
const loading = ref(false);

onMounted(async () => {
  loading.value = true;
  const response = await fetch('https://api.example.com/data');
  data.value = await response.json();
  loading.value = false;
});

// 错误 3: 过度依赖计算属性导致性能问题
const expensiveValue = computed(() => {
  // 非常复杂的计算
  let result = 0;
  for (let i = 0; i < 1000000000; i++) {
    result += Math.sqrt(i);
  }
  return result;
});

// 正确做法：考虑使用缓存或优化算法
// 或者使用侦听器手动控制更新时机
```

### 性能优化

1. **减少依赖**: 尽量减少计算属性的依赖项
2. **避免深层嵌套**: 简化计算属性的逻辑
3. **使用 memoization**: 对于重复计算使用缓存
4. **合理使用可写计算属性**: 避免不必要的 setter
5. **考虑使用侦听器**: 对于复杂场景使用侦听器

## 总结

计算属性是 Vue 中非常强大和高效的特性，掌握其使用方法对于构建高性能应用至关重要：

### 核心优势

1. **缓存机制**: 自动缓存结果，提高性能
2. **响应式依赖**: 自动追踪和更新
3. **代码简洁**: 比方法调用更简洁
4. **类型安全**: 完整的 TypeScript 支持

### 使用场景

1. **数据转换**: 格式化、过滤、排序等
2. **表单验证**: 实时验证表单字段
3. **统计计算**: 计算总和、平均值等
4. **条件渲染**: 基于多个条件的复杂逻辑

### 选择建议

- **计算属性**: 适用于派生数据和缓存
- **方法**: 适用于不依赖缓存的简单操作
- **侦听器**: 适用于需要执行副作用的场景

计算属性是 Vue 响应式系统的重要组成部分，合理使用将大大提升应用性能和开发体验。