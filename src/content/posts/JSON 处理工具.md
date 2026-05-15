---
title: JSON 处理工具
published: 2025-04-23
description: 'jq 命令行工具的详细介绍和学习笔记'
image: ''
tags: ["JSON","工具"]
category: '开发工具'
draft: false
lang: 'zh-CN'
---

## 概述

JSON（JavaScript Object Notation）是一种轻量级的数据交换格式，易于人阅读和编写，同时也易于机器解析和生成。在开发过程中，我们经常需要处理 JSON 数据，而 jq 是一个强大的命令行 JSON 处理工具，它能够让我们高效地查询、过滤和转换 JSON 数据。

## jq 简介

### 什么是 jq

jq 是一个用 C 编写的命令行 JSON 处理器，它类似于 JSON 版本的 sed。jq 支持 JSON 数据的各种操作，包括查询、过滤、映射、排序等，并且语法简洁强大。

### jq 的特点

- **跨平台**：支持 Linux、macOS、Windows
- **功能强大**：支持复杂的 JSON 操作
- **性能高效**：处理大文件速度快
- **语法简洁**：易于学习和使用
- **可扩展**：支持自定义函数和模块

## 安装 jq

### macOS

```bash
# 使用 Homebrew
brew install jq
```

### Linux (Ubuntu/Debian)

```bash
# 使用 apt
sudo apt-get install jq

# 或者使用 snap
sudo snap install jq
```

### Windows

```powershell
# 使用 Chocolatey
choco install jq

# 或者使用 Scoop
scoop install jq

# 或者下载二进制文件
# 访问 https://stedolan.github.io/jq/download/
```

### 验证安装

```bash
jq --version
# 输出：jq-1.6 或类似版本号
```

## jq 基础语法

### 基本查询

jq 使用 `.` 作为基础操作符来访问 JSON 数据：

```bash
# 示例 JSON
echo '{"name": "张三", "age": 25, "city": "北京"}' | jq '.'
```

**输出：**
```json
{
  "name": "张三",
  "age": 25,
  "city": "北京"
}
```

### 访问字段

```bash
# 访问单个字段
echo '{"name": "张三", "age": 25}' | jq '.name'
# 输出： "张三"

# 访问多个字段
echo '{"name": "张三", "age": 25, "city": "北京"}' | jq '.name, .age'
# 输出：
# "张三"
# 25
```

### 嵌套访问

```bash
# 嵌套对象
echo '{"user": {"name": "张三", "info": {"age": 25}}}' | jq '.user.info.age'
# 输出： 25

# 访问整个嵌套对象
echo '{"user": {"name": "张三", "age": 25}}' | jq '.user'
# 输出：
# {"name": "张三", "age": 25}
```

### 数组操作

```bash
# 示例数组 JSON
echo '[{"name": "张三", "age": 25}, {"name": "李四", "age": 30}]' | jq '.[]'

# 输出：
# {"name": "张三", "age": 25}
# {"name": "李四", "age": 30}

# 访问数组元素
echo '[1, 2, 3, 4, 5]' | jq '.[0]'
# 输出： 1

echo '[1, 2, 3, 4, 5]' | jq '.[-1]'
# 输出： 5

# 数组切片
echo '[1, 2, 3, 4, 5]' | jq '.[1:3]'
# 输出： [2, 3]

# 获取数组长度
echo '[1, 2, 3, 4, 5]' | jq '. | length'
# 输出： 5
```

## jq 过滤和选择

### 条件过滤

```bash
# 等于
echo '{"name": "张三", "age": 25}' | jq 'select(.age == 25)'
# 输出： {"name": "张三", "age": 25}

# 不等于
echo '{"name": "张三", "age": 25}' | jq 'select(.age != 30)'
# 输出： {"name": "张三", "age": 25}

# 大于小于
echo '{"name": "张三", "age": 25}' | jq 'select(.age > 20)'
# 输出： {"name": "张三", "age": 25}

# 多条件
echo '{"name": "张三", "age": 25, "city": "北京"}' | jq 'select(.age > 20 and .city == "北京")'
# 输出： {"name": "张三", "age": 25, "city": "北京"}
```

### 数组过滤

```bash
# 过滤数组元素
echo '[{"name": "张三", "age": 25}, {"name": "李四", "age": 30}]' | jq '.[] | select(.age > 25)'
# 输出： {"name": "李四", "age": 30}

# 提取特定字段
echo '[{"name": "张三", "age": 25}, {"name": "李四", "age": 30}]' | jq '.[] | select(.age > 25) | .name'
# 输出： "李四"

# 多条件过滤
echo '[{"name": "张三", "age": 25, "city": "北京"}, {"name": "李四", "age": 30, "city": "上海"}]' | jq '.[] | select(.age > 20 and .city == "北京")'
# 输出： {"name": "张三", "age": 25, "city": "北京"}
```

## jq 数据转换

### 提取字段

```bash
# 提取单个字段
echo '{"name": "张三", "age": 25, "city": "北京"}' | jq '{name: .name}'
# 输出： {"name": "张三"}

# 提取多个字段并重命名
echo '{"name": "张三", "age": 25, "city": "北京"}' | jq '{username: .name, user_age: .age}'
# 输出： {"username": "张三", "user_age": 25}

# 计算新字段
echo '{"price": 100, "tax_rate": 0.1}' | jq '{price: .price, tax: .price * .tax_rate, total: .price * (1 + .tax_rate)}'
# 输出： {"price": 100, "tax": 10, "total": 110}
```

### 数组映射

```bash
# 映射数组元素
echo '[1, 2, 3, 4, 5]' | jq 'map(. * 2)'
# 输出： [2, 4, 6, 8, 10]

# 对象数组映射
echo '[{"name": "张三", "age": 25}, {"name": "李四", "age": 30}]' | jq 'map({name: .name, is_adult: .age >= 18})'
# 输出：
# [{"name": "张三", "is_adult": true}, {"name": "李四", "is_adult": true}]

# 提取对象数组的特定字段
echo '[{"name": "张三", "age": 25}, {"name": "李四", "age": 30}]' | jq 'map(.name)'
# 输出： ["张三", "李四"]
```

### 数组排序

```bash
# 数字排序
echo '[3, 1, 4, 1, 5, 9, 2, 6]' | jq 'sort'
# 输出： [1, 1, 2, 3, 4, 5, 6, 9]

# 反向排序
echo '[3, 1, 4, 1, 5, 9, 2, 6]' | jq 'sort | reverse'
# 输出： [9, 6, 5, 4, 3, 1, 1]

# 对象数组排序
echo '[{"name": "张三", "age": 25}, {"name": "李四", "age": 30}, {"name": "王五", "age": 20}]' | jq 'sort_by(.age)'
# 输出：
# [{"name": "王五", "age": 20}, {"name": "张三", "age": 25}, {"name": "李四", "age": 30}]

# 按字段反向排序
echo '[{"name": "张三", "age": 25}, {"name": "李四", "age": 30}]' | jq 'sort_by(.age) | reverse'
# 输出：
# [{"name": "李四", "age": 30}, {"name": "张三", "age": 25}]
```

### 分组和聚合

```bash
# 数组分组
echo '[{"category": "A", "value": 10}, {"category": "B", "value": 20}, {"category": "A", "value": 30}]' | jq 'group_by(.category)'
# 输出：
# [
#   [{"category": "A", "value": 10}, {"category": "A", "value": 30}],
#   [{"category": "B", "value": 20}]
# ]

# 求和
echo '[1, 2, 3, 4, 5]' | jq 'add'
# 输出： 15

# 平均值
echo '[1, 2, 3, 4, 5]' | jq 'add / length'
# 输出： 3

# 最大值和最小值
echo '[1, 2, 3, 4, 5]' | jq 'max'
# 输出： 5

echo '[1, 2, 3, 4, 5]' | jq 'min'
# 输出： 1

# 唯一值
echo '[1, 2, 2, 3, 3, 3, 4]' | jq 'unique'
# 输出： [1, 2, 3, 4]
```

## jq 高级用法

### 函数定义

```bash
# 定义简单函数
echo '[1, 2, 3, 4, 5]' | jq 'def double: . * 2; map(double)'
# 输出： [2, 4, 6, 8, 10]

# 定义复杂函数
echo '{"price": 100, "tax_rate": 0.1}' | jq 'def calculate_total: .price * (1 + .tax_rate); {total: calculate_total}'
# 输出： {"total": 110}

# 多参数函数
echo '[1, 2, 3, 4, 5]' | jq 'def multiply(x): . * x; map(multiply(3))'
# 输出： [3, 6, 9, 12, 15]
```

### 条件表达式

```bash
# 三元运算符
echo '{"age": 25}' | jq 'if .age >= 18 then "成年" else "未成年" end'
# 输出： "成年"

# 嵌套条件
echo '{"age": 15}' | jq 'if .age >= 18 then "成年" elif .age >= 13 then "青少年" else "儿童" end'
# 输出： "青少年"

# 数组中的条件
echo '[{"name": "张三", "age": 25}, {"name": "李四", "age": 15}]' | jq 'map({name: .name, status: if .age >= 18 then "成年" else "未成年" end})'
# 输出：
# [{"name": "张三", "status": "成年"}, {"name": "李四", "status": "未成年"}]
```

### 错误处理

```bash
# 使用 // 提供默认值
echo '{"name": "张三"}' | jq '.age // 0'
# 输出： 0

echo '{"name": "张三", "age": 25}' | jq '.age // 0'
# 输出： 25

# 使用 ? 避免错误
echo '{"name": "张三"}' | jq '.age?'
# 输出： null

# 使用 try-catch
echo '{"invalid": "data"}' | jq 'try .missing_field catch "字段不存在"'
# 输出： "字段不存在"
```

### 构造新 JSON

```bash
# 从头构建对象
echo '{}' | jq '{name: "张三", age: 25, city: "北京"}'
# 输出： {"name": "张三", "age": 25, "city": "北京"}

# 合并多个对象
echo '{"name": "张三"} {"age": 25} {"city": "北京"}' | jq '{name: .[0].name, age: .[1].age, city: .[2].city}'
# 输出： {"name": "张三", "age": 25, "city": "北京"}

# 使用 + 运算符合并对象
echo '{"name": "张三"}' | jq '. + {"age": 25} + {"city": "北京"}'
# 输出： {"name": "张三", "age": 25, "city": "北京"}
```

## 实际应用场景

### API 响应处理

```bash
# 假设 API 返回的 JSON
curl -s https://api.github.com/users/github | jq '{login: .login, name: .name, public_repos: .public_repos, followers: .followers}'
# 输出：
# {
#   "login": "github",
#   "name": "GitHub",
#   "public_repos": 数值,
#   "followers": 数值
# }

# 提取特定信息
curl -s https://api.github.com/users/github | jq '.login, .name, .location'
# 输出：
# "github"
# "GitHub"
# "San Francisco"
```

### 日志文件分析

```bash
# 假设有一个 JSON 格式的日志文件
cat app.log | jq '.timestamp, .level, .message'
# 输出日志的时间戳、级别和消息

# 过滤错误日志
cat app.log | jq 'select(.level == "ERROR")'
# 只输出错误级别的日志

# 统计各级别日志数量
cat app.log | jq 'group_by(.level) | map({level: .[0].level, count: length})'
# 输出各级别日志的统计信息
```

### 配置文件处理

```bash
# 读取配置文件
cat config.json | jq '.database.host, .database.port'
# 输出数据库主机和端口

# 更新配置
cat config.json | jq '.timeout = 30' > new_config.json
# 更新超时设置为 30

# 合并配置
jq '.base + .override' <(cat base.json) <(cat override.json)
# 合并两个配置文件
```

### 数据清理和转换

```bash
# 标准化字段名
echo '{"UserName": "张三", "userAge": 25}' | jq '{name: .UserName, age: .userAge}'
# 输出： {"name": "张三", "age": 25}

# 数据类型转换
echo '{"value": "123"}' | jq '{value: (.value | tonumber)}'
# 将字符串转换为数字

# 去除空值
echo '{"name": "张三", "age": null, "city": ""}' | jq 'with_entries(select(.value != null and .value != ""))'
# 输出： {"name": "张三"}

# 数组去重
echo '[1, 2, 2, 3, 3, 3, 4]' | jq 'unique'
# 输出： [1, 2, 3, 4]
```

## 其他 JSON 处理工具

### Python (jq 替代品)

```bash
# 使用 Python 处理 JSON
echo '{"name": "张三", "age": 25}' | python -m json.tool
# 格式化 JSON 输出

# 使用 Python 进行复杂处理
echo '[{"name": "张三", "age": 25}, {"name": "李四", "age": 30}]' | python -c "import sys, json; data = json.load(sys.stdin); print([item['name'] for item in data])"
# 输出： ["张三", "李四"]
```

### JavaScript (Node.js)

```bash
# 使用 Node.js 处理 JSON
echo '{"name": "张三", "age": 25}' | node -e "console.log(JSON.stringify(JSON.parse(require('fs').readFileSync(0)), null, 2))"
# 格式化 JSON 输出
```

### Online Tools

- **jq play** (https://jqplay.org/) - 在线 jq 测试工具
- **JSON Editor Online** (https://jsoneditoronline.org/) - 在线 JSON 编辑器
- **JSON Formatter** (https://jsonformatter.org/) - JSON 格式化工具

## 注意事项

1. **数据类型**：注意 JSON 数据类型的转换，特别是字符串和数字
2. **大文件处理**：处理大文件时要注意内存使用，可以考虑使用流式处理
3. **错误处理**：使用 `?` 和 `//` 操作符来避免和处理错误
4. **性能考虑**：复杂查询可能会影响性能，特别是对大文件
5. **版本差异**：不同版本的 jq 可能有一些语法差异
6. **编码问题**：处理非 ASCII 字符时要注意编码问题
7. **引号使用**：在 shell 中使用 jq 时要注意引号的转义
8. **空值处理**：正确处理 null 值和缺失字段

## 最佳实践

1. **格式化输出**：使用 `.`, `indent(4)` 等美化输出
2. **管道链式**：合理使用管道进行多步骤处理
3. **函数封装**：将常用查询封装成函数
4. **注释和文档**：为复杂的 jq 脚本添加注释
5. **测试验证**：先在小数据集上测试 jq 命令
6. **性能监控**：监控大文件处理的时间和资源使用
7. **错误日志**：记录处理过程中的错误和异常
8. **版本兼容**：确保 jq 脚本在不同版本中都能正常工作

## 总结

jq 是一个强大而灵活的 JSON 处理工具，掌握了它的基础语法和高级用法，可以大大提高 JSON 数据处理的效率。通过实际项目的练习，你将能够熟练运用 jq 解决各种 JSON 处理问题。

记住，jq 的优势在于：
- 语法简洁，易于学习和使用
- 功能强大，支持复杂的 JSON 操作
- 性能高效，适合处理大文件
- 跨平台支持，可以在各种环境中使用

持续练习和实践，你将能够发挥 jq 的最大潜力，成为 JSON 处理的专家。