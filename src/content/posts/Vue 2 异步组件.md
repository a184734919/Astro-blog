---
title: Vue 2 异步组件
published: 2023-09-02
description: '异步加载和代码分割的详细介绍和学习笔记'
image: ''
tags: ["Vue2","组件"]
category: 'Vue2'
draft: false
lang: 'zh-CN'
---

## Vue 2 组件基础

组件是 Vue.js 最核心的功能，它允许我们将 UI 拆分为独立可复用的部分。

## 组件注册

```javascript
// 全局注册
Vue.component('my-component', {
  template: '<div>自定义组件</div>'
});

// 局部注册
export default {
  components: {
    'my-component': {
      template: '<div>局部组件</div>'
    }
  }
};
```

## Props 传递

```javascript
// 父组件
<template>
  <child-component :message="msg" :count="10"></child-component>
</template>

// 子组件
export default {
  props: {
    message: {
      type: String,
      required: true
    },
    count: {
      type: Number,
      default: 0
    }
  }
};
```

## 自定义事件

```javascript
// 子组件
this.$emit('update', newValue);

// 父组件
<child-component @update="handleUpdate"></child-component>
```

## 插槽

```vue
<!-- 默认插槽 -->
<slot></slot>

<!-- 具名插槽 -->
<slot name="header"></slot>

<!-- 作用域插槽 -->
<slot :user="user"></slot>
```

## 总结

组件化开发让代码更易维护和复用。