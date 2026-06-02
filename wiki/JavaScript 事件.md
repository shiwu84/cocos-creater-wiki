---
title: JavaScript 事件
date: 2026-06-02
tags:
  - layer/js
  - browser
  - event
  - type/concept
  - status/stable
aliases: []
---

# JavaScript 事件

> [!abstract] 摘要
> 浏览器事件是 JavaScript 交互的核心机制。本章涵盖事件处理程序的三种分配方式、事件冒泡与捕获的传播机制、事件委托模式、阻止浏览器默认行为、以及自定义事件的派发。掌握这些基础模式是构建响应式 Web 应用的关键。

## 核心概念

### 事件处理程序的三种分配方式

1. **HTML 特性** (`onclick="..."`)：直接在 HTML 标签中编写，浏览器读取后自动创建函数写入 DOM 属性。仅限简单场景，不推荐。
2. **DOM 属性** (`elem.onclick = function`)：一个事件只能有一个处理程序，新的会覆盖旧的。
3. **`addEventListener`**：最灵活的方式，支持多个处理程序，支持对象作为处理程序（通过 `handleEvent` 方法），并可控制事件阶段。

```js
// 语法
element.addEventListener(event, handler[, options]);
// options: { once, capture, passive }
element.removeEventListener(event, handler[, options]);
```

> [!warning] 移除处理程序需要同一函数
> `removeEventListener` 必须传入与 `addEventListener` 完全相同的函数引用。匿名函数无法被移除。

### 事件对象
事件发生时，浏览器创建 `event` 对象并作为参数传递给处理程序：

- `event.type` — 事件类型（如 `"click"`）
- `event.currentTarget` — 处理事件当前所在的元素（等于 `this`）
- `event.clientX` / `event.clientY` — 指针相对于窗口的坐标

### 对象处理程序：handleEvent
可以将对象作为处理程序，事件触发时调用对象的 `handleEvent(event)` 方法。使用类配合 `handleEvent` 可以优雅地管理多个事件：

```js
class Menu {
  handleEvent(event) {
    let method = 'on' + event.type[0].toUpperCase() + event.type.slice(1);
    this[method](event);
  }
  onMousedown() { /* ... */ }
  onMouseup() { /* ... */ }
}
let menu = new Menu();
elem.addEventListener('mousedown', menu);
elem.addEventListener('mouseup', menu);
```

## 关键细节

### 事件传播：冒泡与捕获

DOM 事件传播分为三个阶段：
1. **捕获阶段**（Capturing）— 事件从 `window` 向下传递到目标元素
2. **目标阶段**（Target）— 事件到达目标元素，捕获和冒泡处理程序都会被触发
3. **冒泡阶段**（Bubbling）— 事件从目标元素向上冒泡到 `window`

默认情况下，`addEventListener` 的处理程序在冒泡阶段触发。要启用捕获阶段，设置 `{capture: true}`。

> [!important] `event.target` vs `this`
> - `event.target` — **始终**指向引发事件的最深层元素，在冒泡过程中不变
> - `this` (=`event.currentTarget`) — **当前**正在运行处理程序的元素

### 停止传播
- `event.stopPropagation()` — 阻止事件向上冒泡，但当前元素上的其他处理程序仍会执行
- `event.stopImmediatePropagation()` — 阻止冒泡且阻止当前元素上的其他处理程序

> [!warning] 谨慎使用 stopPropagation
> 阻止冒泡可能导致意外的"死区"（如分析系统无法捕获被阻止的点击）。大多数场景不需要阻止冒泡，可用其他方式（如自定义事件、在 event 对象中添加标记数据）替代。

### 事件委托

**事件委托**是利用冒泡模式的强大事件处理模式：不在每个子元素上单独添加处理程序，而是在共同祖先上设置单个处理程序，通过 `event.target` 判断事件来源并处理。

```js
// 在容器上处理所有内部元素的点击
table.onclick = function(event) {
  let td = event.target.closest('td'); // 找到最近的 td 祖先
  if (!td) return;
  if (!table.contains(td)) return; // 排除嵌套表格
  highlight(td);
};
```

**优势**：
- 简化初始化，节省内存（无需为每个元素添加处理程序）
- 动态添加/移除元素无需管理处理程序
- 配合 `innerHTML` 等批量修改时仍能正常工作

**限制**：
- 只适用于冒泡的事件（`focus` 等不会冒泡）
- 容器级处理程序会对所有事件做出响应，但 CPU 负载通常可忽略

### "行为"模式
事件委托的声明式应用：通过自定义 `data-*` 特性为元素添加行为，由文档级处理程序统一处理。

```html
<button data-counter>点击计数</button>
<button data-toggle-id="form1">显示/隐藏</button>

<script>
document.addEventListener('click', function(event) {
  // 计数器行为
  if (event.target.dataset.counter != undefined)
    event.target.value++;
  
  // 切换器行为
  let id = event.target.dataset.toggleId;
  if (id) {
    let elem = document.getElementById(id);
    elem.hidden = !elem.hidden;
  }
});
</script>
```

### 阻止浏览器默认行为
- `event.preventDefault()` — 阻止浏览器默认行为（如链接跳转、表单提交）
- 从 `on<event>` 处理程序返回 `false` 等价于 `preventDefault()`
- `passive: true` 选项告诉浏览器处理程序不会调用 `preventDefault()`，可提升滚动性能

### 自定义事件派发
可以通过 `CustomEvent` 构造函数和 `dispatchEvent` 创建自定义事件：

```js
let event = new CustomEvent('hello', {
  bubbles: true,
  cancelable: true,
  detail: { name: 'John' }
});
elem.dispatchEvent(event);
```

自定义事件必须设置 `bubbles: true` 才能冒泡。

## 注意事项

- 某些事件（如 `DOMContentLoaded`、`transitionend`）**只能**通过 `addEventListener` 分配，DOM 属性方式无效
- 文档级处理程序始终使用 `addEventListener` 而非 `document.on<event>`，避免覆盖
- `beforeunload` 中 `event.preventDefault()` 无效，应使用 `event.returnValue` 设置字符串或直接 `return` 值
- `focus` 事件不会冒泡，但 `focusin` 会冒泡
- 不要通过 `setAttribute` 分配处理程序（特性值总是字符串）

## 相关页面
- [[JavaScript Document]]
- [[JavaScript 事件细节]]
- [[JavaScript 表单与控件]]
- [[JavaScript 数据加载]]
- [[JavaScript 教程概述]]
- [[Linux 进程模型]]
- [[软件工程概述]]

## 原始来源
- [事件简介](raw/zh.javascript.info/2-ui/2-events/index.md)
- [浏览器事件简介](raw/zh.javascript.info/2-ui/2-events/01-introduction-browser-events/article.md)
- [冒泡和捕获](raw/zh.javascript.info/2-ui/2-events/02-bubbling-and-capturing/article.md)
- [事件委托](raw/zh.javascript.info/2-ui/2-events/03-event-delegation/article.md)
- [浏览器默认行为](raw/zh.javascript.info/2-ui/2-events/04-default-browser-action/article.md)
- [自定义事件派发](raw/zh.javascript.info/2-ui/2-events/05-dispatch-events/article.md)
