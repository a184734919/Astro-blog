---
title: JavaScript Service Worker
published: 2022-12-18
description: '离线缓存和 PWA的详细介绍和学习笔记'
image: ''
tags: ["PWA"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

Service Worker 是浏览器在后台运行的独立线程，充当 Web 应用程序与网络之间的代理服务器。它支持离线缓存、拦截网络请求、后台同步等功能，是构建 Progressive Web App（PWA）的核心技术。

## 核心概念

### Service Worker 特性

- **独立线程**：运行在与主线程分离的后台线程中
- **生命周期**：独立的安装、激活和更新流程
- **拦截请求**：可以拦截和处理网络请求
- **离线支持**：提供离线缓存能力
- **持久性**：即使用户关闭浏览器也能运行

### PWA 核心要素

- **可安装**：可以安装到设备主屏幕
- **离线可用**：在没有网络时仍可运行
- **响应式**：适配各种设备尺寸
- **安全**：必须通过 HTTPS 访问

## 基本用法

### 注册 Service Worker

```javascript
// main.js
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/service-worker.js')
      .then(registration => {
        console.log('Service Worker 注册成功:', registration.scope);

        // 检查是否有更新
        registration.addEventListener('updatefound', () => {
          const newWorker = registration.installing;
          newWorker.addEventListener('statechange', () => {
            if (newWorker.state === 'installed' && navigator.serviceWorker.controller) {
              console.log('发现新版本，提示用户刷新');
              // 可以在这里显示更新提示
            }
          });
        });
      })
      .catch(error => {
        console.error('Service Worker 注册失败:', error);
      });
  });
} else {
  console.warn('浏览器不支持 Service Worker');
}
```

### Service Worker 脚本

```javascript
// service-worker.js
const CACHE_NAME = 'my-site-cache-v1';
const urlsToCache = [
  '/',
  '/styles/main.css',
  '/scripts/main.js',
  '/images/logo.png'
];

// 安装事件：缓存静态资源
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => {
        console.log('缓存已打开');
        return cache.addAll(urlsToCache);
      })
  );
});

// 激活事件：清理旧缓存
self.addEventListener('activate', event => {
  const cacheWhitelist = [CACHE_NAME];

  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames.map(cacheName => {
          if (cacheWhitelist.indexOf(cacheName) === -1) {
            console.log('删除旧缓存:', cacheName);
            return caches.delete(cacheName);
          }
        })
      );
    })
  );
});

// 拦截请求事件
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(response => {
        // 缓存命中，返回缓存的资源
        if (response) {
          return response;
        }

        // 缓存未命中，发起网络请求
        return fetch(event.request).then(response => {
          // 检查是否为有效响应
          if (!response || response.status !== 200 || response.type !== 'basic') {
            return response;
          }

          // 克隆响应，因为响应流只能使用一次
          const responseToCache = response.clone();

          caches.open(CACHE_NAME)
            .then(cache => {
              cache.put(event.request, responseToCache);
            });

          return response;
        });
      })
  );
});
```

### 生命周期详解

Service Worker 有三个主要状态：

1. **Installing（安装中）**
   - 调用 `install` 事件
   - 适合预缓存静态资源

2. **Activated（已激活）**
   - 调用 `activate` 事件
   - 适合清理旧缓存

3. **Redundant（冗余）**
   - 被 Service Worker 替代时进入此状态

```javascript
self.addEventListener('install', event => {
  console.log('Service Worker 正在安装...');
  self.skipWaiting(); // 跳过等待，立即激活
});

self.addEventListener('activate', event => {
  console.log('Service Worker 已激活');
  event.waitUntil(self.clients.claim()); // 立即控制所有页面
});

self.addEventListener('fetch', event => {
  console.log('拦截请求:', event.request.url);
});
```

## 实际应用

### 离线优先策略

```javascript
// Cache First（缓存优先）
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request)
      .then(response => {
        return response || fetch(event.request);
      })
  );
});

// Network First（网络优先）
self.addEventListener('fetch', event => {
  event.respondWith(
    fetch(event.request)
      .then(response => {
        const responseToCache = response.clone();
        caches.open(CACHE_NAME)
          .then(cache => cache.put(event.request, responseToCache));
        return response;
      })
      .catch(() => caches.match(event.request))
  );
});

// Network Only（仅网络）
self.addEventListener('fetch', event => {
  event.respondWith(fetch(event.request));
});

// Cache Only（仅缓存）
self.addEventListener('fetch', event => {
  event.respondWith(caches.match(event.request));
});

// Stale While Revalidate（缓存优先，后台更新）
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request).then(cachedResponse => {
      const fetchPromise = fetch(event.request).then(networkResponse => {
        caches.open(CACHE_NAME).then(cache => {
          cache.put(event.request, networkResponse.clone());
        });
        return networkResponse;
      });
      return cachedResponse || fetchPromise;
    })
  );
});
```

### 动态缓存策略

```javascript
// 缓存 API 请求
self.addEventListener('fetch', event => {
  if (event.request.url.includes('/api/')) {
    event.respondWith(
      caches.open('api-cache').then(cache => {
        return cache.match(event.request).then(response => {
          return response || fetch(event.request).then(networkResponse => {
            cache.put(event.request, networkResponse.clone());
            return networkResponse;
          });
        });
      })
    );
  }
});

// 根据请求类型选择策略
function getCacheStrategy(request) {
  const url = new URL(request.url);

  // 静态资源使用缓存优先
  if (request.destination === 'style' ||
      request.destination === 'script' ||
      request.destination === 'image') {
    return 'cache-first';
  }

  // API 请求使用网络优先
  if (url.pathname.startsWith('/api/')) {
    return 'network-first';
  }

  // HTML 页面使用网络优先
  return 'network-first';
}

self.addEventListener('fetch', event => {
  const strategy = getCacheStrategy(event.request);

  if (strategy === 'cache-first') {
    event.respondWith(
      caches.match(event.request)
        .then(response => response || fetch(event.request))
    );
  } else {
    event.respondWith(
      fetch(event.request)
        .then(response => {
          const cacheResponse = response.clone();
          caches.open(CACHE_NAME)
            .then(cache => cache.put(event.request, cacheResponse));
          return response;
        })
        .catch(() => caches.match(event.request))
    );
  }
});
```

### 后台同步

```javascript
// 注册后台同步
navigator.serviceWorker.ready.then(registration => {
  return registration.sync.register('sync-data');
});

// Service Worker 中处理同步事件
self.addEventListener('sync', event => {
  if (event.tag === 'sync-data') {
    event.waitUntil(syncData());
  }
});

async function syncData() {
  try {
    // 获取 IndexedDB 中的数据
    const data = await getPendingData();

    // 发送到服务器
    await sendToServer(data);

    // 清除已发送的数据
    await clearPendingData(data.id);
  } catch (error) {
    console.error('同步失败:', error);
  }
}
```

### 推送通知

```javascript
// 订阅推送通知
navigator.serviceWorker.ready.then(registration => {
  return registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: 'YOUR_VAPID_PUBLIC_KEY'
  });
}).then(subscription => {
  console.log('推送订阅成功:', subscription);
  // 将订阅信息发送到服务器
  sendSubscriptionToServer(subscription);
});

// Service Worker 中处理推送事件
self.addEventListener('push', event => {
  const options = {
    body: event.data ? event.data.text() : '您有新消息',
    icon: '/images/icon-192x192.png',
    badge: '/images/badge-72x72.png',
    vibrate: [200, 100, 200],
    data: {
      dateOfArrival: Date.now(),
      primaryKey: 1
    }
  };

  event.waitUntil(
    self.registration.showNotification('新消息', options)
  );
});

// 处理通知点击
self.addEventListener('notificationclick', event => {
  event.notification.close();

  event.waitUntil(
    clients.openWindow('/notification-detail')
  );
});
```

### 处理离线页面

```javascript
// 创建离线页面
self.addEventListener('fetch', event => {
  event.respondWith(
    fetch(event.request)
      .then(response => {
        return response;
      })
      .catch(() => {
        // 返回离线页面
        return caches.match('/offline.html');
      })
  );
});

// 检测网络状态变化
window.addEventListener('online', () => {
  console.log('网络已恢复');
  // 可以在这里同步离线数据
});

window.addEventListener('offline', () => {
  console.log('网络已断开');
  // 显示离线提示
});
```

## PWA 配置

### Manifest 文件

```json
{
  "name": "我的 PWA 应用",
  "short_name": "MyPWA",
  "description": "一个 Progressive Web App 示例",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#4CAF50",
  "orientation": "portrait",
  "icons": [
    {
      "src": "/images/icon-72x72.png",
      "sizes": "72x72",
      "type": "image/png"
    },
    {
      "src": "/images/icon-96x96.png",
      "sizes": "96x96",
      "type": "image/png"
    },
    {
      "src": "/images/icon-128x128.png",
      "sizes": "128x128",
      "type": "image/png"
    },
    {
      "src": "/images/icon-144x144.png",
      "sizes": "144x144",
      "type": "image/png"
    },
    {
      "src": "/images/icon-152x152.png",
      "sizes": "152x152",
      "type": "image/png"
    },
    {
      "src": "/images/icon-192x192.png",
      "sizes": "192x192",
      "type": "image/png"
    },
    {
      "src": "/images/icon-384x384.png",
      "sizes": "384x384",
      "type": "image/png"
    },
    {
      "src": "/images/icon-512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    }
  ],
  "screenshots": [
    {
      "src": "/screenshots/home.png",
      "sizes": "1080x1920",
      "type": "image/png"
    }
  ],
  "categories": ["productivity", "education"],
  "related_applications": []
}
```

### 在 HTML 中引用

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>我的 PWA 应用</title>

  <!-- 引入 Manifest -->
  <link rel="manifest" href="/manifest.json">

  <!-- iOS 图标配置 -->
  <link rel="apple-touch-icon" href="/images/icon-152x152.png">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">

  <!-- 主题颜色 -->
  <meta name="theme-color" content="#4CAF50">
</head>
<body>
  <!-- 应用内容 -->
  <script src="/main.js"></script>
</body>
</html>
```

### Workbox 库使用

Workbox 是 Google 提供的 Service Worker 工具库，简化了 Service Worker 的开发。

```bash
npm install workbox-cli --save-dev
```

```javascript
// service-worker.js
importScripts('https://storage.googleapis.com/workbox-cdn/releases/6.5.4/workbox-sw.js');

// 预缓存静态资源
workbox.precaching.precacheAndRoute([
  { url: '/styles/main.css', revision: '123456' },
  { url: '/scripts/main.js', revision: '789012' }
]);

// 路由规则
workbox.routing.registerRoute(
  ({ request }) => request.destination === 'image',
  new workbox.strategies.CacheFirst({
    cacheName: 'images-cache',
    plugins: [
      new workbox.expiration.ExpirationPlugin({
        maxEntries: 60,
        maxAgeSeconds: 30 * 24 * 60 * 60, // 30 天
      }),
    ],
  })
);

workbox.routing.registerRoute(
  ({ url }) => url.pathname.startsWith('/api/'),
  new workbox.strategies.NetworkFirst({
    cacheName: 'api-cache',
    networkTimeoutSeconds: 10,
    plugins: [
      new workbox.cacheableResponse.CacheableResponsePlugin({
        statuses: [0, 200],
      }),
    ],
  })
);

workbox.routing.registerRoute(
  new workbox.strategies.StaleWhileRevalidate({
    cacheName: 'pages-cache',
  })
);

// 后台同步
workbox.routing.registerRoute(
  /\/api\/sync/,
  new workbox.strategies.NetworkOnly({
    plugins: [
      new workbox.backgroundSync.BackgroundSyncPlugin('sync-queue', {
        maxRetentionTime: 24 * 60, // 重试 24 小时
      }),
    ],
  })
);
```

## 调试技巧

### Chrome DevTools

```javascript
// 检查 Service Worker 状态
navigator.serviceWorker.getRegistration().then(registration => {
  if (registration) {
    console.log('Service Worker 状态:', registration.active.state);
    console.log('更新状态:', registration.updateViaCache);
  }
});

// 手动更新 Service Worker
navigator.serviceWorker.getRegistration().then(registration => {
  registration.update();
});

// 注销 Service Worker
navigator.serviceWorker.getRegistration().then(registration => {
  registration.unregister();
});

// 获取所有缓存
caches.keys().then(cacheNames => {
  console.log('所有缓存:', cacheNames);
});

// 清除所有缓存
caches.keys().then(cacheNames => {
  cacheNames.forEach(cacheName => {
    caches.delete(cacheName);
  });
});
```

### Service Worker 调试

1. **Application 面板**
   - 查看 Service Worker 状态
   - 检查缓存内容
   - 手动控制 Service Worker

2. **Console 面板**
   - 选择 Service Worker 上下文
   - 查看 Worker 日志输出

3. **Network 面板**
   - 查看请求是否被 Service Worker 拦截
   - 检查缓存策略是否生效

## 注意事项

### HTTPS 要求

Service Worker 必须在 HTTPS 环境下运行（localhost 除外）。

### 浏览器兼容性

```javascript
if ('serviceWorker' in navigator) {
  // 支持 Service Worker
} else {
  // 不支持，降级处理
}
```

### 缓存策略选择

- 静态资源：Cache First
- API 数据：Network First
- HTML 页面：Stale While Revalidate

### 性能优化

1. 合理设置缓存过期时间
2. 避免缓存过大文件
3. 及时清理无用缓存
4. 使用 CDN 加速资源加载

### 错误处理

```javascript
// Service Worker 错误处理
self.addEventListener('error', event => {
  console.error('Service Worker 错误:', event.error);
});

// 通信错误处理
navigator.serviceWorker.addEventListener('messageerror', event => {
  console.error('通信错误:', event.data);
});
```

## 总结

Service Worker 和 PWA 为 Web 应用提供了类似原生应用的体验。通过掌握以下内容：

- Service Worker 的生命周期和事件处理
- 多种缓存策略的选择和应用
- 离线支持和后台同步
- 推送通知功能
- PWA 的完整配置

可以构建出高性能、可靠、用户友好的 Web 应用。

关键要点：
- 理解 Service Worker 的工作原理
- 根据应用场景选择合适的缓存策略
- 做好版本管理和更新控制
- 优化性能和用户体验
- 注意安全性和兼容性

Service Worker 是现代 Web 开发的重要技术，值得深入学习和实践。