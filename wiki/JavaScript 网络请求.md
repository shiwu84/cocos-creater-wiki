---
title: JavaScript 网络请求
date: 2026-06-02
tags:
  - layer/js
  - browser
  - network
  - type/concept
  - status/stable
aliases: []
---

# JavaScript 网络请求

> [!abstract] 摘要
> JavaScript 提供从现代 `fetch` API 到传统的 `XMLHttpRequest`，再到实时通信的 `WebSocket` 和 `Server-Sent Events` 等多种网络请求方式。fetch 作为现代通用方案，基于 Promise 实现了两阶段响应处理（响应头 + 响应体），支持多种数据格式和灵活的请求配置。

## 核心概念

### Fetch API —— 现代网络请求

fetch 是发送 HTTP 请求的现代方法，基于 Promise：

```js
let response = await fetch(url, options);
let data = await response.json();
```

两阶段响应处理：
1. **响应头阶段**：fetch 返回的 Promise 解析为 Response 对象，可通过 `response.status`、`response.ok`（200-299）、`response.headers` 检查响应头
2. **响应体阶段**：通过 `response.text()`、`response.json()`、`response.formData()`、`response.blob()`、`response.arrayBuffer()` 等方法读取 body

响应体只能读取一次，重复读取会失败。

请求配置选项：
- `method`：HTTP 方法（GET、POST 等）
- `headers`：请求头对象（部分安全 header 不可设置）
- `body`：请求体，支持 String、FormData、Blob、BufferSource、URLSearchParams

### FormData —— 表单数据

`FormData` 对象用于以 `multipart/form-data` 格式发送 HTML 表单数据，也支持发送二进制文件。

### 请求控制

- **中止请求**：使用 `AbortController` 的 `signal` 选项
- **跟踪进度**：使用 `response.body`（ReadableStream）逐块读取
- **跨域请求**：支持 CORS 配置

### WebSocket —— 实时双向通信

WebSocket 协议（RFC 6455）在浏览器和服务器之间建立持久连接，支持双向数据交换：

```js
let socket = new WebSocket("wss://example.com");
socket.onopen = () => socket.send("Hello");
socket.onmessage = (event) => console.log(event.data);
socket.onclose = (event) => console.log(event.code, event.reason);
```

- 使用 `ws://`（明文）和 `wss://`（加密，推荐）
- 握手通过 HTTP Upgrade 机制完成（101 状态码）
- 支持文本和二进制数据（通过 `socket.binaryType` 配置）
- `socket.bufferedAmount` 监测缓冲区待发送数据量
- 无跨域限制，但自身不包含重连、认证等机制

### 其他请求方式

- **XMLHttpRequest**：传统的 AJAX 方案，支持文件上传进度跟踪
- **Server-Sent Events**：服务器向浏览器的单向推送
- **长轮询（Long Polling）**：模拟实时通信的兼容方案
- **可恢复上传**：通过跟踪已上传字节实现断点续传

## 关键细节

- `fetch` 仅在网络错误时 reject；HTTP 404/500 等错误状态不会导致 reject，需通过 `response.ok` 判断
- 禁止设置的请求头包括 `Cookie`、`Referer`、`Origin`、`Host` 等，由浏览器控制
- WebSocket 连接状态通过 `socket.readyState` 获取（0-3：CONNECTING/OPEN/CLOSING/CLOSED）
- WebSocket 关闭码常见值：`1000`（正常关闭）、`1006`（连接丢失）、`1001`（离开页面）、`1009`（消息过大）

## 与其他系统的关系

- 与 **[[JavaScript 二进制数据]]** 相关：fetch 和 WebSocket 可发送/接收 Blob 和 ArrayBuffer
- 与 **[[JavaScript 浏览器存储]]** 相关：cookie 随请求自动发送
- 与 **[[JavaScript Frame 与 Window]]** 相关：iframe 中的请求受同源策略约束

## 注意事项

- `wss://` 优于 `ws://`，后者数据未加密
- 响应体只能读取一次，重复读取会抛出异常
- 旧代理服务器可能不支持 WebSocket 协议升级
- `fetch` 跨域请求默认不携带 cookie，需设置 `credentials` 选项

## 相关页面

- [[JavaScript 教程概述]]
- [[JavaScript 二进制数据]]
- [[JavaScript 浏览器存储]]
- [[JavaScript Frame 与 Window]]
- [[Linux 网络协议栈]]

## 原始来源

- [网络请求](raw/zh.javascript.info/5-network/index.md)
- [Fetch](raw/zh.javascript.info/5-network/01-fetch/article.md)
- [FormData](raw/zh.javascript.info/5-network/02-formdata/article.md)
- [Fetch：下载进度](raw/zh.javascript.info/5-network/03-fetch-progress/article.md)
- [Fetch：中止](raw/zh.javascript.info/5-network/04-fetch-abort/article.md)
- [Fetch：跨域请求](raw/zh.javascript.info/5-network/05-fetch-crossorigin/article.md)
- [Fetch API](raw/zh.javascript.info/5-network/06-fetch-api/article.md)
- [URL 对象](raw/zh.javascript.info/5-network/07-url/article.md)
- [XMLHttpRequest](raw/zh.javascript.info/5-network/08-xmlhttprequest/article.md)
- [可恢复的文件上传](raw/zh.javascript.info/5-network/09-resume-upload/article.md)
- [长轮询](raw/zh.javascript.info/5-network/10-long-polling/article.md)
- [WebSocket](raw/zh.javascript.info/5-network/11-websocket/article.md)
- [Server Sent Events](raw/zh.javascript.info/5-network/12-server-sent-events/article.md)
