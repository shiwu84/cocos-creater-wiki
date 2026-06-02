---
title: JavaScript 杂项
date: 2026-06-02
tags:
  - layer/js
  - type/concept
  - status/stable
aliases: []
---

# JavaScript 杂项

> [!abstract] 摘要
> 本章涵盖 JavaScript 中一些高级或独立的主题：Proxy 和 Reflect 用于拦截对象操作、`eval` 执行代码字符串、柯里化（Currying）函数变换技术、Reference Type 的内部机制、BigInt 大整数类型以及 Unicode 字符串编码原理。

## 核心概念

### Proxy 和 Reflect
- `new Proxy(target, handler)` 包装对象并拦截操作
- `handler` 通过"捕捉器（traps）"（如 `get`、`set`、`has`、`deleteProperty`、`apply`、`construct` 等）拦截对应内部方法
- 无捕捉器的代理是 `target` 的透明包装
- 常见应用：默认值（`get` 捕捉器返回默认值）、验证（`set` 捕捉器）、私有属性保护、可撤销代理
- 捕捉器必须遵守"不变量"（invariants），确保语言行为的一致性
- `Reflect` 对象提供与 Proxy 捕捉器对应的静态方法，用于实现转发和默认行为

### eval
- `eval(code)` 执行字符串形式的 JavaScript 代码
- 在**当前词法环境**中执行，可访问和修改外部变量
- 严格模式下 `eval` 有独立的词法环境，不能修改外部变量
- **安全性风险**：执行用户提供的代码极度危险，应尽量避免
- 现代开发中极少使用；用 `new Function` 或其他替代方案更安全

### 柯里化 (Currying)
- 将 `f(a, b, c)` 转换为 `f(a)(b)(c)` 的函数变换
- 核心目的：**部分应用函数（partial application）**，固定部分参数生成新函数
- 高级实现（如 lodash `_.curry`）同时支持正常调用和部分应用
- 典型场景：日志函数固定前缀、发送请求固定方法等

### Reference Type
- 是语言内部规范类型，不可直接访问
- 解释 `obj.method()` 能正常工作，但 `(obj.method)()` 丢失 `this` 的原因
- 当 `.` 操作返回 Reference Type 时，它携带了 `(base, name, strict)` 三元组信息
- 其他操作（如 `=`、`||`、三元运算符）会将 Reference Type 转为普通值，丢失 `this` 绑定

### BigInt
- 使用 `n` 后缀或 `BigInt()` 构造器创建：`123n` / `BigInt("123")`
- 支持任意长度整数，不会溢出
- 支持 `+`、`-`、`*`、`/`、`%`、`**` 运算（除法向零舍入）
- **不能与 Number 混合使用**，需显式转换（`BigInt(number)` 或 `Number(bigint)`）
- 不支持一元 `+` 操作符（`+1n` 报错）
- 比较运算符可与 Number 混用（`1n < 2` 有效）
- 在 `if`、`||`、`&&`、`!` 中行为与 Number 一致

### Unicode 字符串编码
- JavaScript 使用 UTF-16 编码
- 字符表示：`\xXX`（1 字节）、`\uXXXX`（2 字节）、`\u{X...XXXXXX}`（最多 6 字节）
- **代理对（Surrogate Pairs）**：超过 U+FFFF 的字符（如 emoji）用两个 16 位编码单元表示
- `String.fromCodePoint` / `codePointAt` 正确处理完整 Unicode 字符
- `length` 属性返回 UTF-16 编码单元数，对代理对字符不准确
- 使用 `for..of` 或扩展运算符 `[...str]` 可正确按字符迭代

## 关键细节

### Proxy 常用捕捉器
| 捕捉器 | 触发操作 |
|--------|----------|
| `get(target, prop, receiver)` | 读取属性 |
| `set(target, prop, value, receiver)` | 写入属性 |
| `has(target, prop)` | `in` 操作符 |
| `deleteProperty(target, prop)` | `delete` 操作符 |
| `apply(target, thisArg, args)` | 函数调用 |
| `construct(target, args)` | `new` 操作符 |
| `ownKeys(target)` | `Object.keys()` 等 |

### Proxy 应用场景
- **验证**：`set` 捕捉器中校验属性值
- **默认值**：`get` 捕捉器对不存在属性返回默认值
- **私有属性保护**：`get`/`set` 捕捉器中禁止外部访问以 `_` 开头的属性
- **可撤销代理**：`Proxy.revocable()` 创建可随时断开的代理

## 与其他系统的关系
- [[JavaScript 原型与继承]] 中 `Object.defineProperty` 与 Proxy 的 `defineProperty` 捕捉器互补
- [[JavaScript 类]] 的私有字段 `#` 与 Proxy 保护 `_` 属性形成私有化的不同方案
- [[JavaScript 教程概述]] 涵盖其他核心语言特性

## 注意事项
- `Proxy` 性能开销较大，不要过度使用
- Proxy 不能完全替代 `Object.defineProperty`（Vue 3 的响应式系统同时使用两者）
- `eval` 是现代开发中的反模式，应使用更安全的替代方案
- `BigInt` 不支持 `Math` 对象的方法，不能隐式转换为 Number 用于 JSON.stringify
- Unicode 处理中 `str.length` 和下标访问 `str[i]` 对 emoji 等代理对字符不准确，应使用 `[...str]` 或 `Array.from(str)`

## 相关页面
- [[JavaScript 教程概述]]
- [[JavaScript 原型与继承]]
- [[JavaScript 类]]

## 原始来源
- [杂项](raw/zh.javascript.info/1-js/99-js-misc/index.md)
- [Proxy 和 Reflect](raw/zh.javascript.info/1-js/99-js-misc/01-proxy/article.md)
- [Eval：执行代码字符串](raw/zh.javascript.info/1-js/99-js-misc/02-eval/article.md)
- [柯里化（Currying）](raw/zh.javascript.info/1-js/99-js-misc/03-currying-partials/article.md)
- [Reference Type](raw/zh.javascript.info/1-js/99-js-misc/04-reference-type/article.md)
- [BigInt](raw/zh.javascript.info/1-js/99-js-misc/05-bigint/article.md)
- [Unicode —— 字符串内幕](raw/zh.javascript.info/1-js/99-js-misc/06-unicode/article.md)
