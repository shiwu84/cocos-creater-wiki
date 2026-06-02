---
title: JavaScript Frame 与 Window
date: 2026-06-02
tags:
  - layer/js
  - type/concept
  - status/stable
aliases: []
---

# JavaScript Frame 与 Window

> [!abstract] 摘要
> JavaScript 提供了操作浏览器窗口和 iframe 的完整能力，包括弹窗管理、跨窗口通信以及针对点击劫持等安全威胁的防御机制。"同源策略"是跨窗口交互的核心安全模型。

## 核心概念

### 弹窗（Popup）

弹窗通过 `window.open(url, name, params)` 打开，返回对新窗口的引用。弹窗是一个独立的窗口，具有独立的 JavaScript 环境，常用于 OAuth 授权等场景。

- `params` 配置字符串控制窗口特征：`width/height`、`left/top`、`menubar`、`toolbar`、`location`、`status`、`resizable`、`scrollbars`
- 浏览器会阻止用户行为之外的 `open` 调用（非 onclick 触发的弹窗）
- 默认在新标签页打开；若提供了窗口大小参数，则打开独立弹窗

```js
let newWin = window.open("about:blank", "hello", "width=200,height=200");
newWin.document.write("Hello, world!");
```

弹窗可通过 `window.opener` 访问 opener 窗口，主窗口通过 `open()` 返回值访问弹窗。关闭弹窗使用 `win.close()`，检查状态用 `win.closed`。

### 跨窗口通信与同源策略

同源策略要求两个 URL 具有相同的**协议、域和端口**才被视为同源。非同源窗口之间无法访问对方的内容（变量、文档等），唯一例外是可以修改（但不能读取）对方的 `location`。

对于 iframe，使用 `iframe.contentWindow` 和 `iframe.contentDocument` 访问其内容。

跨源通信通过 `postMessage` 接口实现，允许窗口间安全地交换消息。

### 点击劫持（Clickjacking）

点击劫持是指恶意页面在透明 iframe 中加载目标网站，诱骗用户点击实际在目标网站上的按钮。防御方法包括：

- `X-Frame-Options` HTTP header（`DENY` / `SAMEORIGIN` / `ALLOW-FROM domain`）
- 使用覆盖 `div` 防止在 iframe 中交互
- `samesite` cookie 特性

## 关键细节

- 窗口移动/调整大小：`win.moveBy(x,y)`、`win.moveTo(x,y)`、`win.resizeBy(w,h)`、`win.resizeTo(w,h)`（仅对弹窗有效）
- 窗口聚焦：`window.focus()` / `window.blur()`，受浏览器严格限制
- 滚动窗口：`win.scrollBy(x,y)`、`win.scrollTo(x,y)`、`elem.scrollIntoView(top)`
- `window.onfocus/onblur` 事件用于跟踪窗口激活状态

## 与其他系统的关系

- 与 **[[JavaScript 网络请求]]** 相关：iframe 加载资源受同源策略影响
- 与 **[[JavaScript 浏览器存储]]** 相关：localStorage 在同源窗口间共享
- 与 **[[JavaScript 事件]]** 相关：postMessage 基于事件模型

## 注意事项

- 弹窗在移动设备上支持不佳，应尽量避免使用
- `setTimeout` 触发的弹窗在不同浏览器中有不同的信任策略
- `X-Frame-Options` 只能通过 HTTP header 设置，`<meta>` 标签中写无效
- 同源策略对 `location` 的写入是例外——允许重定向但不允许读取

## 相关页面

- [[JavaScript 教程概述]]
- [[JavaScript 浏览器存储]]
- [[JavaScript 网络请求]]

## 原始来源

- [Frame 和 Window](raw/zh.javascript.info/3-frames-and-windows/index.md)
- [弹窗和 window 的方法](raw/zh.javascript.info/3-frames-and-windows/01-popup-windows/article.md)
- [跨窗口通信](raw/zh.javascript.info/3-frames-and-windows/03-cross-window-communication/article.md)
- [点击劫持攻击](raw/zh.javascript.info/3-frames-and-windows/06-clickjacking/article.md)
