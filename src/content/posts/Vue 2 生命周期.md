---
title: Vue 2 生命周期
published: 2023-09-05
description: '钩子函数详解和应用场景的详细介绍和学习笔记'
image: ''
tags: ["Vue2","生命周期"]
category: 'Vue2'
draft: false
lang: 'zh-CN'
---

## Vue 2 生命周期

每个 Vue 实例在被创建时都要经过一系列的初始化过程，这个过程就是生命周期。

## 生命周期钩子

```javascript
export default {
  beforeCreate() {
    // 实例初始化之后，数据观测和事件配置之前
  },
  created() {
    // 实例创建完成后调用，数据观测、属性和方法已配置
    // 但 DOM 还未生成
  },
  beforeMount() {
    // 挂载开始之前调用
  },
  mounted() {
    // DOM 挂载完成后调用
    // 可以访问 DOM 元素
  },
  beforeUpdate() {
    // 数据更新时调用，DOM 还未更新
  },
  updated() {
    // 数据更新后，DOM 已重新渲染
  },
  beforeDestroy() {
    // 实例销毁之前调用
  },
  destroyed() {
    // 实例销毁后调用
  }
};
```

## 使用场景

```javascript
export default {
  created() {
    // 发起 API 请求
    this.fetchData();
  },
  mounted() {
    // 操作 DOM
    this.initChart();
    // 添加事件监听
    window.addEventListener('resize', this.handleResize);
  },
  beforeDestroy() {
    // 清理工作
    window.removeEventListener('resize', this.handleResize);
  }
};
```

## 总结

理解生命周期对于正确使用 Vue 至关重要。