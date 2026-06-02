---
title: JavaScript Document
date: 2026-06-02
tags:
  - javascript
  - browser
  - dom
aliases: []
---

# JavaScript Document

> [!abstract] 摘要
> Document 对象模型（DOM）是浏览器将 HTML/XML 文档表示为树形结构的编程接口。本章涵盖浏览器环境、DOM 树结构、节点搜索与遍历、特性与属性的区别、文档修改、样式与类的操作、元素尺寸与滚动、以及坐标系统，是 JavaScript 操作网页的基础。

## 核心概念

### 浏览器环境
JavaScript 在浏览器中运行时，宿主环境提供了一系列全局对象和 API。`window` 是全局根对象，扮演双重角色：既是 JavaScript 代码的全局对象，也代表浏览器窗口并提供控制浏览器窗口的方法。BOM（浏览器对象模型）是浏览器提供的用于处理文档之外所有内容的对象和方法，而 DOM 则专注于处理文档内容。

### DOM 树
HTML 文档被浏览器解析后形成 DOM 树，每个 HTML 标签都是树中的一个**元素节点**，标签内的文本形成**文本节点**（标记为 `#text`），注释形成**注释节点**（标记为 `#comment`）。DOM 中共有 12 种节点类型，实践中常用 4 种：`document`（入口点）、元素节点、文本节点和注释节点。

关键特性：
- 浏览器会自动纠正不规范的 HTML（如自动添加 `<html>`、`<body>`、`<tbody>`）
- `<head>` 之前的空格和换行符被忽略
- `</body>` 之后的内容会被自动移到 `<body>` 内部

### DOM 导航
节点间通过以下属性进行导航：
- **顶级节点**：`document.documentElement`（`<html>`）、`document.body`、`document.head`
- **子节点**：`childNodes`（含文本节点）、`children`（仅元素）、`firstChild`/`lastChild`、`firstElementChild`/`lastElementChild`
- **兄弟节点**：`nextSibling`/`previousSibling`、`nextElementSibling`/`previousElementSibling`
- **父节点**：`parentNode`、`parentElement`

### 搜索元素
六种主要搜索方法：

| 方法 | 搜索方式 | 可在元素上调用 | 实时集合 |
|------|---------|:---:|:---:|
| `querySelector` | CSS 选择器 | ✔ | - |
| `querySelectorAll` | CSS 选择器 | ✔ | - |
| `getElementById` | id | - | - |
| `getElementsByName` | name | - | ✔ |
| `getElementsByTagName` | 标签名 | ✔ | ✔ |
| `getElementsByClassName` | 类名 | ✔ | ✔ |

辅助方法：
- `elem.matches(css)` — 检查元素是否匹配 CSS 选择器
- `elem.closest(css)` — 查找最近的匹配祖先（含自身）
- `elemA.contains(elemB)` — 检查 elemB 是否是 elemA 的后代

> [!warning] 实时集合 vs 静态集合
> `getElementsBy*` 返回**实时集合**，会随 DOM 变化自动更新。`querySelectorAll` 返回**静态集合**，是查询时的快照。

### 特性（Attribute）与属性（Property）
- **特性（attribute）**：写在 HTML 标签中的内容，值始终是字符串，名称大小写不敏感
- **属性（property）**：DOM 对象中的内容，值可以是任意类型，名称大小写敏感

操作特性的方法：
- `elem.hasAttribute(name)` / `getAttribute(name)` / `setAttribute(name, value)` / `removeAttribute(name)`
- `elem.attributes` — 所有特性的可迭代集合

标准特性（如 `id`）会自动映射为 DOM 属性，但映射不总是双向的。典型例外：`input.value` 只能从特性同步到属性（反向不通）。

### dataset 和非标准特性
以 `data-` 开头的自定义特性可通过 `elem.dataset` 安全访问：
- `data-about` → `elem.dataset.about`
- `data-order-state` → `elem.dataset.orderState`（驼峰命名）

### 修改文档
创建元素：`document.createElement(tag)`、`document.createTextNode(text)`

插入方法：
- `node.append(...)` — 末尾插入
- `node.prepend(...)` — 开头插入
- `node.before(...)` — 前面插入
- `node.after(...)` — 后面插入
- `node.replaceWith(...)` — 替换节点

移除节点：`node.remove()`
克隆节点：`node.cloneNode(true/false)`（true 为深克隆）
插入 HTML：`elem.insertAdjacentHTML(position, html)`，position 可以为 `"beforebegin"`、`"afterbegin"`、`"beforeend"`、`"afterend"`

### 样式与类
- `elem.className` — 完整的类字符串
- `elem.classList` — 提供 `add/remove/toggle/contains` 方法操作单个类，且可迭代
- `elem.style` — 直接操作内联样式，使用驼峰命名（如 `backgroundColor`）
- `getComputedStyle(elem)` — 获取最终应用样式（含 CSS 级联），只读

原则：优先使用 CSS 类而非直接修改 `style` 属性。

### 元素尺寸与滚动
- **几何属性**：`offsetWidth/Height`（含边框）、`clientWidth/Height`（不含边框和滚动条）、`scrollWidth/Height`（含溢出内容）
- **位置属性**：`offsetLeft/Top`（相对最近定位祖先）、`clientLeft/Top`（左边框/上边框宽度）、`scrollLeft/Top`（滚动距离）
- 窗口信息：`window.innerWidth/innerHeight`（窗口内容区）、`document.documentElement.scrollLeft/scrollTop`（文档滚动距离）

### 坐标系统
- **相对于窗口**（`clientX/Y`）：以窗口可视区域左上角为原点
- **相对于文档**（`pageX/Y`）：以文档左上角为原点，不随滚动改变
- **`elem.getBoundingClientRect()`**：返回元素相对于窗口的坐标和尺寸
- **`document.elementFromPoint(x, y)`**：返回给定坐标处的最顶层元素

## 关键细节

1. **DOM 中的空白文本节点**：HTML 中的换行和缩进会产生 `#text` 节点，除 `<head>` 前和 `</body>` 后的空白外，所有空白都会成为 DOM 的一部分。
2. **`id` 全局变量的陷阱**：浏览器会用 `id` 创建同名全局变量，可能与 JS 变量冲突，因此始终推荐使用 `document.getElementById`。
3. **innerHTML vs createElement**：`innerHTML` 替换内容会重建整个内部 DOM（原元素的事件处理程序丢失），而 `insertAdjacentHTML` 不会移除已有内容。
4. **table 的自动 `<tbody>`**：浏览器会自动为 `<table>` 插入 `<tbody>`，DOM 导航时需要注意。

## 注意事项

- 不要通过 HTML 特性（如 `onclick="..."`）分配事件处理程序，它与 DOM 属性方式的混用容易导致覆盖
- `getElementsByTagName` 注意复数 `s`（`getElementById` 单数）
- `scrollTop/scrollLeft` 可以写，其他几何属性只读
- 修改 `style` 属性切勿忘记加单位（如 `px`）
- DOM 操作是昂贵的，尽量批量操作而非逐条修改

## 相关页面
- [[JavaScript 事件]]
- [[JavaScript 事件细节]]
- [[JavaScript 表单与控件]]
- [[JavaScript 数据加载]]
- [[JavaScript 教程概述]]

## 原始来源
- [Document 章节](raw/zh.javascript.info/2-ui/1-document/index.md)
- [DOM 树](raw/zh.javascript.info/2-ui/1-document/02-dom-nodes/article.md)
- [搜索元素](raw/zh.javascript.info/2-ui/1-document/04-searching-elements-dom/article.md)
- [特性和属性](raw/zh.javascript.info/2-ui/1-document/06-dom-attributes-and-properties/article.md)
- [修改文档](raw/zh.javascript.info/2-ui/1-document/07-modifying-document/article.md)
- [样式和类](raw/zh.javascript.info/2-ui/1-document/08-styles-and-classes/article.md)
