---
title: JavaScript 正则表达式
date: 2026-06-02
tags:
  - javascript
  - concept
  - regex
aliases: []
---

# JavaScript 正则表达式

> [!abstract] 摘要
> 正则表达式是文本搜索和替换的强大工具，在 JavaScript 中通过 `RegExp` 对象和字符串方法配合使用。提供 6 种修饰符（flags）、丰富的模式语法（字符类、量词、分组等）以及完善的方法集。从基本匹配到高级特性（捕获组、环视、Unicode 支持），是字符串处理不可或缺的技能。

## 核心概念

### 创建正则表达式

```js
let regexp = /pattern/gmi;                // 字面量语法（静态）
let regexp = new RegExp("pattern", "gi");  // 构造函数语法（动态字符串）
```

字面量语法更常见，构造函数适合动态生成模式。

### 六种修饰符（Flags）

| 修饰符 | 说明 |
|--------|------|
| `i` | 不区分大小写 |
| `g` | 全局搜索（所有匹配项） |
| `m` | 多行模式 |
| `s` | dotall 模式（`.` 匹配 `\n`） |
| `u` | 完整 Unicode 支持（正确处理代理对） |
| `y` | 粘滞模式（在指定位置精确搜索） |

### 基本方法

**字符串方法：**

| 方法 | 说明 |
|------|------|
| `str.match(regexp)` | 返回匹配项。有 `g` 返回所有匹配数组；无 `g` 返回首个匹配项（含捕获组）；无匹配返回 `null` |
| `str.matchAll(regexp)` | 返回可迭代对象，每个匹配项都含捕获组（需 `g`） |
| `str.replace(regexp, replacement)` | 替换匹配项。替换字符串支持 `$&`、`$\``、`$'`、`$n`、`$<name>` 等特殊符号 |
| `str.replaceAll(regexp, replacement)` | 替换所有匹配项 |
| `str.search(regexp)` | 返回首个匹配项位置，无匹配返回 `-1` |
| `str.split(regexp)` | 按模式分割字符串 |

**正则表达式方法：**

| 方法 | 说明 |
|------|------|
| `regexp.test(str)` | 测试是否存在匹配，返回 `true/false` |
| `regexp.exec(str)` | 返回匹配项详情。有 `g` 时从 `lastIndex` 开始搜索，可用于迭代匹配 |

### 字符类（Character Classes）

| 模式 | 匹配 |
|------|------|
| `\d` | 数字（0-9） |
| `\D` | 非数字 |
| `\w` | 单词字符（字母、数字、下划线） |
| `\W` | 非单词字符 |
| `\s` | 空白字符 |
| `\S` | 非空白字符 |
| `.` | 任意字符（默认不含 `\n`） |

### 量词与模式

| 模式 | 说明 |
|------|------|
| `+` | 1 个或多个 |
| `*` | 0 个或多个 |
| `?` | 0 个或 1 个 |
| `{n}` | 恰好 n 个 |
| `{n,m}` | n 到 m 个 |
| `{n,}` | n 个及以上 |
| `^` / `$` | 文本开头 / 结尾（多行模式下为行首/行尾） |
| `\b` | 词边界 |

贪婪 vs 懒惰：默认贪婪（尽可能多匹配），添加 `?` 变懒惰（尽可能少匹配）。如 `+?`、`*?`、`??`。

### 分组与引用

```js
/(\w+) (\w+)/              // 捕获组，可通过 $1、$2 引用
/(?<name>\w+)/             // 命名捕获组
/(?:abc)/                  // 非捕获组
/\1/                       // 反向引用第一个捕获组
/\k<name>/                 // 反向引用命名捕获组
```

### 环视断言（Lookaround）

| 模式 | 说明 |
|------|------|
| `(?=...)` | 正向前瞻（后面跟着...） |
| `(?!...)` | 负向前瞻（后面不跟着...） |
| `(?<=...)` | 正向后瞻（前面是...） |
| `(?<!...)` | 负向后瞻（前面不是...） |

### 字符集与转义

- `[abc]`：字符集中的任意一个
- `[a-z]`：范围
- `[^abc]`：排除字符集
- 特殊字符需转义：`[ \ ^ $ . | ? * + ( )`

### 交替（Alternation）

`|` 表示"或"，如 `html|css` 匹配 "html" 或 "css"。

## 关键细节

- `str.match(regexp)` 无匹配返回 `null`（非空数组），常见坑点
- `str.matchAll` 是较新的方法，返回可迭代对象（非数组），无匹配返回空迭代器
- 带 `g` 修饰符的正则表达式在 `test` 和 `exec` 中会维护 `lastIndex` 状态，同一正则重复使用需注意
- `regexp.exec` 可通过 `lastIndex` 手动控制搜索起始位置
- 带有 `y` 修饰符时，搜索严格从 `lastIndex` 开始，不会向前搜索
- 灾难性回溯：不当的正则模式（如 `(\w+)+`）可能在某些输入上导致指数级回溯
- `u` 修饰符是处理 Unicode（含 emoji 等）的必要选项

## 与其他系统的关系

- 与 **JavaScript 字符串处理** 相关：正则表达式与 String 方法深度集成
- 与 **[[JavaScript 事件]]** 相关：表单验证常用正则表达式
- 与 **[[JavaScript 网络请求]]** 相关：URL 解析、数据提取

## 注意事项

- 避免灾难性回溯模式，尤其是在处理用户输入时
- 测试前检查 `lastIndex` 状态，必要时重置为 0
- 处理 Unicode 文本时始终使用 `u` 修饰符
- `replace` 的第一个参数为普通字符串时仅替换第一个匹配项
- 字符集中仅需转义 `^`、`-`、`]`、`\`（以及开头的 `[`）
- `str.match` 无 `g` 时返回的数组包含捕获组，索引 0 为完整匹配

## 相关页面

- [[JavaScript 教程概述]]
- JavaScript 字符串处理
- [[JavaScript 网络请求]]

## 原始来源

- [正则表达式](raw/zh.javascript.info/9-regular-expressions/index.md)
- [模式（Patterns）和修饰符（flags）](raw/zh.javascript.info/9-regular-expressions/01-regexp-introduction/article.md)
- [字符类](raw/zh.javascript.info/9-regular-expressions/02-regexp-character-classes/article.md)
- [Unicode：修饰符 "u" 和 class \p{...}](raw/zh.javascript.info/9-regular-expressions/03-regexp-unicode/article.md)
- [锚点：字符串开始 ^ 和末尾 $](raw/zh.javascript.info/9-regular-expressions/04-regexp-anchors/article.md)
- [量词](raw/zh.javascript.info/9-regular-expressions/09-regexp-quantifiers/article.md)
- [贪婪量词和惰性量词](raw/zh.javascript.info/9-regular-expressions/10-regexp-greedy-and-lazy/article.md)
- [捕获组](raw/zh.javascript.info/9-regular-expressions/11-regexp-groups/article.md)
- [前瞻断言与后瞻断言](raw/zh.javascript.info/9-regular-expressions/14-regexp-lookahead-lookbehind/article.md)
- [灾难性回溯](raw/zh.javascript.info/9-regular-expressions/15-regexp-catastrophic-backtracking/article.md)
- [正则表达式和字符串的方法](raw/zh.javascript.info/9-regular-expressions/17-regexp-methods/article.md)
