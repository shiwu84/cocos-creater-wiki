---
title: JavaScript 数据类型
date: 2026-06-02
tags:
  - layer/js
  - type/concept
  - status/stable
aliases:
  - JS 数据类型深入
---

# JavaScript 数据类型

> [!abstract] 摘要
> 本章深入探索 JavaScript 的核心数据结构：数字（含特殊值和 Math）、字符串（含模板字面量）、数组及其丰富的原型方法、可迭代对象与迭代器、Map/Set 集合、WeakMap/WeakSet、键值遍历、解构赋值、Date 时间处理和 JSON 序列化。是语言数据处理能力的完整图谱。

## 核心概念

### 数字类型
- 64 位 IEEE 754 双精度浮点数，整数安全范围 ±(2^53 - 1)
- 特殊值：`Infinity`、`NaN`、`-0`
- 不同进制：`0x`（十六进制）、`0o`（八进制）、`0b`（二进制）
- `num.toString(base)` 转换进制，`Math.floor/ceil/round/trunc` 取整
- 精度问题：`0.1 + 0.2 !== 0.3`，用 `toFixed(n)` 或 `Number.EPSILON` 比较
- `isNaN()` 和 `isFinite()` 做特殊值检查
- `parseInt(str, base)` / `parseFloat(str)` 从字符串提取数字
- `Math` 对象：`random()`、`max/min`、`pow`、`sqrt`、三角函数等

### 字符串
- 不可变：一旦创建无法原地修改，所有方法返回新字符串
- 三种引号：单引号、双引号、反引号（模板字面量）
- 模板字面量：支持 `${...}` 嵌入和跨行
- 索引访问 `str[0]`（只读），`for...of` 遍历字符（正确处理代理对）
- 常用方法：`indexOf/lastIndexOf`、`includes/startsWith/endsWith`、`slice/substring/substr`、`toLowerCase/toUpperCase`
- Unicode：`str.codePointAt(pos)` / `String.fromCodePoint(code)`

### 数组
- 有序集合，从 0 开始索引，支持混合类型元素
- `length` 属性可写（截断或扩展数组）
- `arr.at(-1)` 获取最后一个元素（现代浏览器）
- `push/pop`（末端增删，快），`shift/unshift`（首端增删，慢，需移动所有元素）
- 内部表示：引擎对连续存储的数组做优化，避免稀疏数组

### 数组方法分类
| 类别 | 方法 |
|------|------|
| 增删 | `splice(start, deleteCount, ...items)`、`slice(start, end)` |
| 合并 | `concat(...items)` |
| 查找 | `indexOf/lastIndexOf`、`includes`、`find/findIndex`（回调）、`findLast/findLastIndex` |
| 过滤 | `filter(fn)` |
| 转换 | `map(fn)`、`sort(fn)`、`reverse()`、`flat(depth)`、`flatMap(fn)` |
| 迭代 | `forEach(fn)` |
| 归约 | `reduce(fn, initial)`、`reduceRight(fn, initial)` |
| 检查 | `every(fn)`、`some(fn)` |
| 转换 | `join(sep)`、`split(sep)`（字符串方法，返回数组） |
| 其他 | `Array.from(iterable)`、`Array.isArray(value)`、`fill(value, start, end)`、`copyWithin()` |

### 可迭代对象
- `Symbol.iterator` 方法返回迭代器对象，该对象有 `next()` 方法返回 `{done, value}`
- 可迭代对象可在 `for...of` 循环中使用
- 内置可迭代对象：字符串、数组、Map、Set
- `Array.from(iterable)` 将可迭代对象/类数组转为数组
- 类数组：有索引和 `length` 属性的对象
- 迭代器本身也可迭代（返回自身）

### Map 和 Set
- **Map**：键值对集合，键可以是任意类型（包括对象）
  - 方法：`set(k,v)`、`get(k)`、`has(k)`、`delete(k)`、`clear()`、`size`
  - 遍历：`map.keys()`、`map.values()`、`map.entries()`，保有插入顺序
  - `Object.fromEntries(map)` 转回普通对象
- **Set**：唯一值集合（不允许重复）
  - 方法同 Map，但 `add(v)` 替代 `set`
- **WeakMap**：键必须是对象，键被垃圾回收后自动消失，不可迭代
- **WeakSet**：值必须是对象，不可迭代

### 解构赋值
- 数组解构：`let [a, b, ...rest] = arr`
- 对象解构：`let {prop1, prop2: alias, ...rest} = obj`
- 支持默认值、嵌套解构
- 常用于从函数参数中提取值：`function({name, age}) { ... }`

### Date 和 JSON
- **Date**：创建 `new Date()`、`new Date(milliseconds)`、`new Date("YYYY-MM-DD")`
  - 获取：`getFullYear()`、`getMonth()`（0-11）、`getDate()`、`getHours()` 等
  - 设置：`setFullYear()`、`setMonth()` 等（自动修正溢出）
  - 时间戳：`getTime()` 返回毫秒数；`Date.now()` 当前时间戳
  - 解析：`Date.parse(str)` 返回时间戳
- **JSON**：通用数据交换格式
  - `JSON.stringify(obj)`：对象→JSON 字符串（忽略函数、Symbol、`undefined` 属性）
  - `JSON.parse(str)`：JSON 字符串→对象
  - 排除/转换：自定义 `toJSON()` 方法或 `replacer`/`reviver` 函数

## 与其他系统的关系

- [[JavaScript 基础知识]] — 数据类型概述和基本类型
- [[JavaScript 对象基础]] — Object 是数组、Map/Set 等的基础原型
- [[JavaScript 对象属性]] — 属性描述符用于配置更精细的对象行为
- JSON 格式是前后端数据交换的基础，与 web API 紧密相关

## 注意事项

- 数组的 `typeof` 返回 `"object"`，用 `Array.isArray()` 判断
- `sort()` 默认按字符串排序，数字排序需提供比较函数 `(a,b) => a-b`
- 不要用 `==` 比较数组/对象，始终用引用比较
- `splice` 会修改原数组，`slice` 不会
- `forEach` 中不能用 `break`，要用 `for...of` 循环
- 解构时注意 `null` 和 `undefined` 会报错，可提供默认值 `= {}`

## 相关页面

- [[JavaScript 教程概述]]
- [[JavaScript 基础知识]]
- [[JavaScript 对象基础]]
- [[JavaScript 对象属性]]

## 原始来源

- [数据类型](raw/zh.javascript.info/1-js/05-data-types/index.md)
- [原始类型的方法](raw/zh.javascript.info/1-js/05-data-types/01-primitives-methods/article.md)
- [数字类型](raw/zh.javascript.info/1-js/05-data-types/02-number/article.md)
- [字符串](raw/zh.javascript.info/1-js/05-data-types/03-string/article.md)
- [数组](raw/zh.javascript.info/1-js/05-data-types/04-array/article.md)
- [数组方法](raw/zh.javascript.info/1-js/05-data-types/05-array-methods/article.md)
- [可迭代对象](raw/zh.javascript.info/1-js/05-data-types/06-iterable/article.md)
- [Map and Set](raw/zh.javascript.info/1-js/05-data-types/07-map-set/article.md)
- [WeakMap and WeakSet](raw/zh.javascript.info/1-js/05-data-types/08-weakmap-weakset/article.md)
- [Object.keys，values，entries](raw/zh.javascript.info/1-js/05-data-types/09-keys-values-entries/article.md)
- [解构赋值](raw/zh.javascript.info/1-js/05-data-types/10-destructuring-assignment/article.md)
- [日期和时间](raw/zh.javascript.info/1-js/05-data-types/11-date/article.md)
- [JSON 方法，toJSON](raw/zh.javascript.info/1-js/05-data-types/12-json/article.md)
