---
title: Fiddler 使用教程
published: 2025-01-26
description: 'HTTP 调试工具的详细介绍和学习笔记'
image: ''
tags: ["Fiddler","工具"]
category: '开发工具'
draft: false
lang: 'zh-CN'
---

## 概述

Fiddler 是一款功能强大的 HTTP 调试代理工具，广泛用于 Web 开发、API 测试、性能分析和安全审计。它能够捕获、检查、修改和重放 HTTP/HTTPS 流量，支持 Windows 平台，是前端开发中不可或缺的调试工具。本文将详细介绍 Fiddler 的核心功能和使用方法。

## 核心概念

### Fiddler 工作原理

Fiddler 基于系统代理架构：

- **系统代理**: 自动配置系统代理设置
- **流量捕获**: 监控进出系统的所有 HTTP 流量
- **HTTPS 解密**: 支持加密流量的解密和分析
- **请求处理**: 提供强大的请求处理和修改能力

### 核心功能模块

Fiddler 主要包含以下功能模块：

1. **流量捕获**: 实时捕获 HTTP/HTTPS 流量
2. **请求检查**: 详细检查请求和响应内容
3. **请求修改**: 拦截和修改请求参数
4. **性能分析**: 分析请求性能和网络状况
5. **自动响应**: 设置规则自动响应请求
6. **脚本扩展**: 使用脚本扩展功能

## 基本用法

### 1. 安装和配置

#### 安装步骤

```javascript
// 1. 下载 Fiddler
// 访问 Fiddler 官网: https://www.telerik.com/fiddler
// 下载免费版本或试用版本

// 2. 安装程序
// 运行安装程序
// 选择安装路径
// 完成安装

// 3. 首次启动
// 启动 Fiddler
// 同意许可协议
// 配置基本设置

// 4. 检查版本
// Help → About Fiddler
// 查看版本信息
```

#### 基本配置

```javascript
// 1. 系统代理设置
// Tools → Options → Connections
// 勾选 "Act as system proxy on startup"
// 默认端口: 8888

// 2. 浏览器配置
// Fiddler 会自动配置系统代理
// 大多数浏览器会自动使用系统代理
// 无需手动配置

// 3. 验证代理
// 打开浏览器访问任意网站
// Fiddler 中应该能看到请求记录

// 4. 基本设置
// Tools → Options
// 配置各种选项
// 设置代理端口、日志等
```

### 2. HTTPS 配置

#### 启用 HTTPS 解密

```javascript
// 1. 开启 HTTPS 解密
// Tools → Options → HTTPS
// 勾选 "Capture HTTPS CONNECTs"
// 勾选 "Decrypt HTTPS traffic"

// 2. 信任 Fiddler 证书
// 点击 "Actions" → "Trust Root Certificate"
// 确认信任证书

// 3. 配置浏览器证书
// Fiddler 会自动安装证书
// 确保浏览器信任 Fiddler 证书

// 4. 验证 HTTPS 配置
// 访问 HTTPS 网站
// Fiddler 中应该能看到解密后的内容
// 浏览器不应该显示证书错误
```

#### HTTPS 高级配置

```javascript
// 1. 配置 HTTPS 解密范围
// Tools → Options → HTTPS
// 选择 "from all processes" 或指定进程

// 2. 服务器证书验证
// 选择忽略证书错误
// 或验证服务器证书

// 3. 客户端证书
// 配置客户端证书
// 用于双向认证场景

// 4. 移除证书
// Tools → Options → HTTPS → Actions
// 点击 "Remove All Certificates"
// 清除所有证书
```

### 3. 流量捕获和分析

#### 基础捕获功能

```javascript
// 1. 开始捕获
// 点击工具栏的 "Capturing" 按钮
// 或按 F12 键

// 2. 停止捕获
// 再次点击 "Capturing" 按钮
// 或按 F12 键

// 3. 查看请求
// 左侧面板显示请求列表
// 点击请求查看详细信息

// 4. 请求分类
// Fiddler 会自动分类请求
// 可按会话类型过滤
```

#### 请求信息面板

```javascript
// 1. Inspectors 面板
// Headers: 请求和响应头
// TextView: 文本格式显示
// ImageView: 图片预览
// JSONView: JSON 格式显示
// XMLView: XML 格式显示
// WebView: 网页预览

// 2. 请求详情
// 请求行 (Method, URL, Version)
// 请求头 (Headers)
// 请求体 (Body)
// 查询参数 (Query String)

// 3. 响应详情
// 状态行 (Status Code, Reason)
// 响应头 (Headers)
// 响应体 (Body)
// 响应时间 (Timings)

// 示例: 分析 API 请求
// 1. 找到 API 请求
// 2. 查看 Headers 面板
// 3. 检查请求参数
// 4. 查看响应数据
```

#### 请求过滤和搜索

```javascript
// 1. 过滤请求
// 使用左侧的过滤器
// 可按会话类型、域名、状态码等过滤

// 2. 搜索功能
// 使用搜索框搜索
// 可搜索 URL、内容、头信息等

// 3. 高级过滤
// 使用过滤表达式
// 支持复杂的过滤条件

// 示例过滤表达式:
// 过滤特定域名:
// *.example.com

// 过滤状态码:
// Status Code == 404

// 过滤请求方法:
// Method == POST

// 组合过滤:
// *.example.com && Method == GET
```

#### 时间线分析

```javascript
// 1. 查看时间线
// 点击 "Timeline" 标签
// 查看请求的时间瀑布图

// 2. 时间分析
// 可以看到每个请求的加载时间
// 识别阻塞请求
// 分析加载顺序

// 3. 性能指标
// 请求总时间
// DNS 查询时间
// 连接时间
// 发送时间
// 等待时间
// 接收时间

// 示例: 识别慢请求
// 1. 查看时间线
// 2. 找出耗时最长的请求
// 3. 分析时间分解
// 4. 针对性优化
```

### 4. 请求修改和重放

#### 请求重放

```javascript
// 1. 重放单个请求
// 选择请求
// 按下 R 键
// 或右键 → Replay → Reissue

// 2. 重放多个请求
// 选择多个请求
// 右键 → Replay → Reissue Selected

// 3. 修改后重放
// 右键 → Replay → Reissue and Edit
// 修改请求内容
// 点击 "Execute" 重放

// 示例: 测试 API 参数
// 1. 找到 API 请求
// 2. 右键 → Replay → Reissue and Edit
// 3. 修改请求参数
// 4. 点击 Execute
// 5. 查看新的响应
```

#### 请求拦截

```javascript
// 1. 设置断点
// 选择请求
// 按 B 键
// 或右键 → Breakpoints → Request Breakpoint

// 2. 配置断点类型
// Before Requests: 请求断点
// After Responses: 响应断点

// 3. 执行请求
// 重新发送请求
// Fiddler 会在断点处暂停

// 4. 修改内容
// 查看并修改请求内容
// 点击 "Run to Completion" 继续

// 示例: 修改登录请求
// 1. 找到登录 API
// 2. 设置请求断点
// 3. 重新执行登录
// 4. 修改用户名密码
// 5. 查看不同响应
```

#### 自动响应规则

```javascript
// 1. 创建自动响应规则
// Rules → Automatic Breakpoints
// 配置自动断点规则

// 2. 配置条件
// 可以按 URL、方法、头信息等设置条件

// 3. 设置响应
// 设置固定的响应内容
// 或使用本地文件

// 示例: 自动响应错误
// Rules → Automatic Breakpoints
// 设置条件: URL 包含 "/api/error"
// 设置响应: 500 Internal Server Error

// 4. 启用/禁用规则
// Rules → Automatic Breakpoints → Disable
// 可以临时禁用规则
```

### 5. 高级功能使用

#### 脚本扩展

```javascript
// 1. 打开脚本编辑器
// Rules → Customize Rules
// 或按 Ctrl+R

// 2. 脚本基础结构
// Fiddler 使用 JScript.NET
// 主要函数: OnBeforeRequest, OnBeforeResponse

// 示例: 修改请求头
static function OnBeforeRequest(oSession: Session) {
    // 添加自定义头
    oSession.oRequest.headers["X-Custom-Header"] = "CustomValue";

    // 修改 User-Agent
    oSession.oRequest.headers["User-Agent"] = "My Custom User Agent";
}

// 示例: 修改响应
static function OnBeforeResponse(oSession: Session) {
    if (oSession.oResponse.headers.Exists("Content-Type") &&
        oSession.oResponse.headers["Content-Type"].Contains("json")) {

        // 解析 JSON 响应
        var responseJSON = Fiddler.WebFormats.JSON.JsonDecode(oSession.GetResponseBodyAsString());

        // 修改响应数据
        responseJSON.JSONObject["addedField"] = "custom value";

        // 保存修改后的响应
        oSession.utilSetResponseBody(responseJSON.JSONObject);
    }
}

// 示例: URL 重写
static function OnBeforeRequest(oSession: Session) {
    if (oSession.url.StartsWith("https://api.example.com")) {
        oSession.url = oSession.url.Replace("https://api.example.com", "https://test.example.com");
    }
}
```

#### 性能测试

```javascript
// 1. 模拟网络条件
// Rules → Performance → Simulate Modem Speeds
// 启用网络速度模拟

// 2. 配置网络速度
// 可以设置上传/下载速度
// 模拟不同的网络环境

// 3. 测试页面加载
// 启用网络模拟后访问网站
// 观察加载情况
// 识别性能问题

// 示例: 测试慢速网络
// 1. 启用 Modem Speeds
// 2. 访问网站
// 3. 观察加载情况
// 4. 优化资源
```

#### 域名映射

```javascript
// 1. 配置域名映射
// Tools → Hosts
// 添加域名到 IP 的映射

// 示例配置:
// 127.0.0.1 www.example.com
// 192.168.1.100 api.example.com

// 2. 使用场景
// 本地开发环境测试
// 测试不同环境
// 绕过 DNS 问题

// 3. 优先级
// Fiddler 的 hosts 文件优先于系统 hosts
// 只在 Fiddler 运行时生效
```

## 实际应用

### 1. Web 应用调试

```javascript
// 场景：调试 Web 应用 API 调用

// 1. 捕获请求
// 启动 Fiddler 捕获
// 在应用中执行操作
// 查看捕获的请求

// 2. 分析请求
// 检查请求 URL 和方法
// 查看请求头和参数
// 验证数据格式

// 3. 修改请求
// 使用断点功能
// 修改请求参数
// 测试不同场景

// 4. 验证响应
// 检查响应状态码
// 分析响应数据
// 验证业务逻辑

// 实际案例:
// 调试用户注册 API
// 1. 捕获注册请求
// 2. 查看请求参数
// 3. 测试边界条件
// 4. 验证错误处理
```

### 2. API 测试

```javascript
// 场景：API 接口测试

// 1. 测试 API 端点
// 使用 Composer 功能
// 手动构建 API 请求
// 发送并查看响应

// 2. 测试不同方法
// GET: 获取数据
// POST: 创建数据
// PUT: 更新数据
// DELETE: 删除数据

// 3. 测试认证
// 添加认证头
// 测试不同认证方式
// 验证权限控制

// 4. 测试错误处理
// 发送错误参数
// 验证错误响应
// 检查错误处理逻辑

// 示例: 测试 REST API
// 1. 打开 Composer
// 2. 选择方法: POST
// 3. 输入 URL: https://api.example.com/users
// 4. 添加请求头: Content-Type: application/json
// 5. 添加请求体: {"name":"Alice","email":"alice@example.com"}
// 6. 点击 Execute
// 7. 查看响应结果
```

### 3. 性能问题排查

```javascript
// 场景：排查网站加载慢的问题

// 1. 录制加载过程
// 清空 Fiddler 会话
// 启动捕获
// 加载网站

// 2. 分析时间线
// 查看 Timeline 面板
// 识别慢速请求
// 分析加载顺序

// 3. 识别瓶颈
// 查找最慢的资源
// 分析请求大小
// 检查时间分解

// 4. 测试优化方案
// 使用网络模拟
// 测试不同优化策略
// 验证优化效果

// 实际案例:
// 优化首页加载
// 1. 录制首页加载
// 2. 识别慢速资源
// 3. 优化图片和 CSS
// 4. 启用缓存
// 5. 验证性能提升
```

### 4. 跨域问题解决

```javascript
// 场景：解决开发环境跨域问题

// 1. 启用 CORS
// 使用脚本添加 CORS 头

static function OnBeforeResponse(oSession: Session) {
    oSession.oResponse.headers["Access-Control-Allow-Origin"] = "*";
    oSession.oResponse.headers["Access-Control-Allow-Methods"] = "GET, POST, PUT, DELETE, OPTIONS";
    oSession.oResponse.headers["Access-Control-Allow-Headers"] = "Content-Type, Authorization";
}

// 2. 修改响应头
// 针对特定请求添加 CORS 头
// 允许跨域访问

// 3. 测试跨域
// 保存脚本
// 重新加载页面
// 验证跨域问题解决

// 注意事项:
// 仅在开发环境使用
// 生产环境应正确配置 CORS
// 注意安全性
```

### 5. 安全审计

```javascript
// 场景：进行安全审计

// 1. 检查敏感信息
// 查看请求中的敏感数据
// 验证数据是否加密
// 检查是否有明文传输

// 2. 测试认证机制
// 修改认证令牌
// 测试会话管理
// 验证权限控制

// 3. 测试输入验证
// 修改请求参数
// 尝试 SQL 注入
// 测试 XSS 攻击

// 4. 测试安全头
// 检查安全头设置
// 验证 CSP、HSTS 等
// 测试 CSRF 保护

// 安全检查清单:
// [ ] HTTPS 是否正确配置
// [ ] 敏感数据是否加密
// [ ] 认证机制是否有效
// [ ] 输入验证是否完善
// [ ] 安全头是否设置
// [ ] CSRF 保护是否启用
```

## 注意事项

### 1. 使用限制

```javascript
// 1. 平台限制
// Fiddler 主要用于 Windows
// Mac 用户可以使用 Fiddler Everywhere
// Linux 用户可以考虑其他工具

// 2. 系统要求
// 需要 .NET Framework
// 建议使用较新版本的 Windows
// 确保系统资源充足

// 3. 浏览器兼容性
// 现代浏览器都支持系统代理
// 某些特殊浏览器可能需要手动配置
```

### 2. 性能影响

```javascript
// 1. 系统性能
// Fiddler 会占用系统资源
// 长时间运行可能影响性能
// 及时清理会话数据

// 2. 网络性能
// 代理会影响网络速度
// 解密 HTTPS 会增加延迟
// 测试时考虑性能影响

// 3. 资源管理
// 定期清理会话
// 关闭不必要的功能
// 优化配置以提高性能
```

### 3. 隐私和安全

```javascript
// 1. 数据隐私
// Fiddler 会记录所有网络请求
// 注意保护敏感信息
// 定期清理会话数据

// 2. 证书管理
// 妥善管理 Fiddler 证书
// 测试完成后移除证书
// 避免证书泄露风险

// 3. 使用环境
// 只在开发和测试环境使用
// 生产环境不要使用抓包工具
// 确保不会泄露用户数据
```

### 4. 常见问题解决

```javascript
// 1. 无法捕获 HTTPS 请求
// 检查 HTTPS 解密是否启用
// 确认证书已正确安装
// 验证浏览器信任证书

// 2. 某些应用无法抓包
// 应用可能使用证书固定
// 需要破解应用的证书验证
// 考虑使用其他抓包方法

// 3. 性能问题
// 关闭不必要的功能
// 清理会话数据
// 优化 Fiddler 配置

// 4. 证书错误
// 重新安装证书
// 确保证书信任设置
// 检查系统时间
```

### 5. 与 Charles 对比

```javascript
// Fiddler vs Charles 对比

// Fiddler 优势:
// - Windows 平台集成更好
// - 免费版本功能充足
// - 脚本功能强大
// - 界面熟悉

// Charles 优势:
// - 跨平台支持
// - 界面更现代
// - 移动设备支持更好
// - 性能优化更好

// 选择建议:
// - Windows 用户优先选择 Fiddler
// - Mac/Linux 用户选择 Charles
// - 需要跨平台选择 Charles
// - 需要强大脚本功能选择 Fiddler
```

## 总结

Fiddler 是功能强大的 HTTP 调试工具，掌握其使用能够：

1. **高效调试**: 详细分析 HTTP 请求和响应
2. **API 测试**: 方便测试和调试 API 接口
3. **性能优化**: 识别和解决性能问题
4. **安全审计**: 检查应用的安全性问题

**核心技能**:
- 配置 Fiddler 代理和 HTTPS
- 捕获和分析网络请求
- 修改和重放请求
- 使用脚本扩展功能
- 进行性能和安全分析

**应用场景**:
- Web 应用调试
- API 开发和测试
- 性能问题排查
- 跨域问题解决
- 安全审计

**学习建议**:
- 从基础功能开始学习
- 结合实际项目练习
- 学习脚本扩展功能
- 建立标准化工作流程
- 与团队分享经验

**进阶方向**:
- 学习高级脚本编写
- 掌握性能分析方法
- 深入理解 HTTP 协议
- 探索自动化测试
- 研究安全测试技术

通过掌握 Fiddler，你将获得强大的网络调试能力，能够更高效地开发和调试 Web 应用，确保应用的网络性能和安全性。记住，工具只是手段，理解其背后的原理才能真正发挥其价值。