---
title: JavaScript 数据加载
date: 2026-06-02
tags:
  - javascript
  - browser
aliases: []
---

# JavaScript 数据加载

> [!abstract] 摘要
> 本章涵盖页面生命周期事件、脚本加载策略以及资源加载追踪。核心包括 DOM 就绪（DOMContentLoaded）与完全加载（load）的区别、离开页面时的事件（beforeunload/unload）、async/defer 属性对脚本加载的影响、以及动态加载外部资源时的 onload/onerror 事件。

## 核心概念

### 页面生命周期事件

HTML 页面生命周期有三个核心事件，按执行顺序：

1. **`DOMContentLoaded`** — DOM 完全构建就绪，但外部资源（图片、样式）可能尚未加载完成
2. **`load`** — 页面**所有资源**（图片、样式、iframe 等）加载完成
3. **`beforeunload` / `unload`** — 用户即将离开/正在离开页面

```js
// DOMContentLoaded 只能通过 addEventListener 监听
document.addEventListener("DOMContentLoaded", () => {
  // DOM 就绪，可查找节点和初始化界面
  // 但图片大小此时为 0x0（未加载完成）
});

// load 可通过 onload 属性或 addEventListener
window.onload = () => {
  // 所有资源已加载完成，可获取图片真实大小
};

// beforeunload — 用户即将离开
window.onbeforeunload = () => {
  return false; // 或者在 addEventListener 中设置 event.returnValue
};

// unload — 用户正在离开，只能做轻量操作
window.addEventListener("unload", () => {
  navigator.sendBeacon("/analytics", JSON.stringify(analyticsData));
});
```

### DOMContentLoaded 的阻塞规则

- **同步 `<script>` 标签**：会阻塞 `DOMContentLoaded`，浏览器必须等待脚本下载并执行完成
- **`async` 脚本**：不会阻塞 `DOMContentLoaded`
- **动态创建的脚本**（`document.createElement('script')`）：默认不会阻塞 `DOMContentLoaded`
- **样式表在前、脚本在后**：脚本必须等待样式表加载完成才能执行（因为脚本可能需要读取样式信息），这会间接延迟 `DOMContentLoaded`

### 离开页面时的数据发送

`navigator.sendBeacon(url, data)` 专为 `unload` 场景设计：
- 以 POST 请求在后台发送数据
- 不阻塞页面跳转
- 数据大小限制约 64KB
- 请求完成后浏览器已离开页面，无法获取响应

```js
window.addEventListener("unload", function() {
  let analyticsData = { /* 收集的数据 */ };
  navigator.sendBeacon("/analytics", JSON.stringify(analyticsData));
});
```

`beforeunload` 中 `event.preventDefault()` 不起作用，应使用 `event.returnValue` 或直接 `return` 字符串值。现代浏览器不再显示自定义消息，仅显示默认的确认对话框。

### document.readyState

跟踪文档加载状态（三个值）：
- `"loading"` — 文档正在加载
- `"interactive"` — 文档已解析完毕（在 `DOMContentLoaded` 之前触发）
- `"complete"` — 文档和所有资源加载完毕（在 `window.onload` 之前触发）

可在代码中检查当前状态并决定是绑定事件还是直接执行：

```js
function work() { /* ... */ }
if (document.readyState == 'loading') {
  document.addEventListener('DOMContentLoaded', work);
} else {
  work(); // DOM 已就绪，立即执行
}
```

还有 `readystatechange` 事件在状态变化时触发，但现代开发中较少使用。

### 脚本加载策略：async vs defer

默认的 `<script>` 标签会阻塞 HTML 解析。`async` 和 `defer` 都能消除这种阻塞：

| 特性 | 执行顺序 | `DOMContentLoaded` 关系 |
|------|---------|------------------------|
| **`defer`** | 按文档中的顺序执行 | 在 `DOMContentLoaded` **之前**执行 |
| **`async`** | 加载优先（先完成先执行，与文档顺序无关） | 不相关，可能在之前或之后 |

**`defer`** 的特点：
- 在后台下载，不阻塞 HTML 解析
- DOM 构建完成后、`DOMContentLoaded` 事件前执行
- 多个 `defer` 脚本按文档顺序执行
- 仅适用于有 `src` 属性的外部脚本

**`async`** 的特点：
- 完全独立的脚本
- 加载完成后立即执行，不等待其他脚本和 DOM
- 脚本之间互不等待
- 适用于独立的第三方脚本（如广告、分析统计）

**动态脚本**（`document.createElement('script')`）默认行为为 `async`。可设置 `script.async = false` 使其按文档顺序执行（类似 `defer`）。

### 资源加载追踪：onload / onerror

对于动态加载的外部资源（脚本、图片、iframe 等），可使用 `load` 和 `error` 事件追踪：

```js
let script = document.createElement('script');
script.src = "https://example.com/library.js";

script.onload = () => {
  // 脚本加载完成，可使用其中的函数/变量
};
script.onerror = () => {
  // 加载失败（无法获取具体 HTTP 状态码，仅知失败）
};

document.head.append(script);
```

> [!warning] `onload`/`onerror` 仅追踪加载本身
> 脚本加载成功即触发 `onload`，即使脚本内有运行时错误也照常触发。运行时错误需通过 `window.onerror` 全局处理。

其他资源类似：
- **图片**：`img.onload` / `img.onerror`（图片加载成功/失败）
- **iframe**：`iframe.onload`（iframe 内容加载完成，但可能因跨域限制无法访问内容）
- **样式表**：可通过动态创建的 `<link>` 元素监听加载

## 关键细节

1. **典型的事件流顺序**：`readyState:loading` → `readyState:interactive` → `DOMContentLoaded` → `iframe onload` → `img onload` → `readyState:complete` → `window onload`
2. **浏览器的自动填充时机**：在 `DOMContentLoaded` 期间触发。若被长时间运行的脚本延迟，登录名/密码字段会出现延迟填充。
3. **`defer` 阻塞 `DOMContentLoaded`**：带 `defer` 的脚本需全部执行完毕，`DOMContentLoaded` 才会触发。
4. **页面可用性提示**：使用 `async` 或 `defer` 时，用户可能在脚本执行前看到页面。应添加加载提示并禁用尚未初始化的按钮。

## 注意事项

- 脚本放在 `<body>` 底部可访问上方所有 DOM 元素，但需要等整个 HTML 下载后才能开始下载脚本
- `async` 脚本适用独立第三方脚本；`defer` 适用有依赖关系的脚本（需要完整 DOM、特定执行顺序）
- 无脚本的页面也应保持基本可用，确保渐进增强
- `beforeunload` 的确认对话框不能自定义消息（现代浏览器限制）
- 从不同域加载脚本可能受 CORS 策略影响（`onerror` 获取的信息有限）

## 相关页面
- [[JavaScript Document]]
- [[JavaScript 事件]]
- [[JavaScript 事件细节]]
- [[JavaScript 表单与控件]]
- [[JavaScript 教程概述]]

## 原始来源
- [加载文档和其他资源](raw/zh.javascript.info/2-ui/5-loading/index.md)
- [页面生命周期](raw/zh.javascript.info/2-ui/5-loading/01-onload-ondomcontentloaded/article.md)
- [脚本 async/defer](raw/zh.javascript.info/2-ui/5-loading/02-script-async-defer/article.md)
- [资源加载 onload/onerror](raw/zh.javascript.info/2-ui/5-loading/03-onload-onerror/article.md)
