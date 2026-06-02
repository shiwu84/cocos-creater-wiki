---
title: JavaScript 浏览器存储
date: 2026-06-02
tags:
  - layer/js
  - browser
  - storage
  - type/concept
  - status/stable
aliases: []
---

# JavaScript 浏览器存储

> [!abstract] 摘要
> 浏览器提供了三种主要的客户端数据存储方案：Cookie（HTTP 协议原生支持，随请求自动发送）、Web Storage（localStorage/sessionStorage，JS 接口，不随请求发送）、以及 IndexedDB（功能强大的客户端数据库）。三者适用于不同的场景：身份认证、轻量键值存储、大规模结构化数据。

## 核心概念

### Cookie —— HTTP 原生存储

Cookie 是直接存储在浏览器中的一小串数据，由 HTTP 协议定义（RFC 6265）：

- 服务器通过 `Set-Cookie` HTTP response header 设置
- 浏览器通过 `Cookie` HTTP request header 自动附加到同域请求中
- 通过 `document.cookie` 访问和设置（访问器属性，非普通属性）

Cookie 选项（设置时以 `;` 分隔）：
- `path=/`：Cookie 可访问的路径
- `domain=site.com`：Cookie 可访问的域
- `expires/max-age`：过期时间
- `secure`：仅 HTTPS 传输
- `samesite`：防 CSRF（`strict` / `lax` / `none`）
- `httpOnly`：禁止 JS 访问（仅服务端设置）

单个 Cookie 的大小限制约为 4KB，每个域约 20 个。

### Web Storage —— localStorage 和 sessionStorage

两个对象提供相同的 API，绑定到源（协议/域/端口）：

```js
localStorage.setItem('key', 'value');
localStorage.getItem('key');
localStorage.removeItem('key');
localStorage.clear();
localStorage.key(index);
localStorage.length;
```

| 特性 | localStorage | sessionStorage |
|------|-------------|----------------|
| 生命周期 | 永久（浏览器重启后仍保留） | 当前标签页（关闭后清除） |
| 共享范围 | 同源所有标签页 | 仅当前标签页（含同源 iframe） |
| 大小限制 | 约 5MB+ | 约 5MB+ |

键和值都必须是字符串。存储对象可用类对象方式访问（`localStorage.key`），但不推荐（可能覆盖内建属性且不触发 storage 事件）。

**storage 事件**：当其他窗口修改 storage 时触发，携带 `key`、`oldValue`、`newValue`、`url`、`storageArea` 等属性。修改数据的窗口自身不会收到该事件。

### IndexedDB —— 客户端数据库

IndexedDB 是一个内建的事务性数据库，比 localStorage 功能强大得多：

- 支持多种键类型和几乎任意值类型
- 支持事务、键值范围查询和索引
- 可存储远超 5MB 的数据
- 适合离线应用，常与 ServiceWorker 结合使用

基于事件的 API，也可使用 `idb` 等 Promise 包装器：

```js
let openRequest = indexedDB.open("mydb", 1);
openRequest.onupgradeneeded = () => { /* 创建对象存储 */ };
openRequest.onsuccess = () => {
  let db = openRequest.result;
  let tx = db.transaction("store", "readwrite");
  let store = tx.objectStore("store");
  store.add({key: "value"});
};
```

IndexedDB 具有内建的模式版本控制（schema versioning）机制。

## 关键细节

- Cookie 随每次 HTTP 请求发送到服务器，Web Storage 不会
- `document.cookie` 的读取返回所有 cookie（`name=value; name2=value2;...`），写入只更新指定的 cookie
- localStorage 不可迭代，使用 `Object.keys(localStorage)` 或 `for` 循环 + `key(index)` 遍历
- IndexedDB 绑定到源，不同网站数据互相隔离
- IndexedDB 的操作大多是异步的

## 与其他系统的关系

- 与 **[[JavaScript 网络请求]]** 相关：Cookie 随请求自动发送，影响 fetch 的 `credentials` 选项
- 与 **[[JavaScript Frame 与 Window]]** 相关：localStorage 在同源窗口间共享，storage 事件可用于跨窗口通信
- 与 **[[JavaScript 二进制数据]]** 相关：IndexedDB 可存储 Blob 等二进制数据

## 注意事项

- Cookie 每次网络请求都发送，不宜存储大量数据
- Cookie 中敏感信息应设置 `secure` 和 `httpOnly`
- `sessionStorage` 在新标签页中不共享数据
- IndexedDB 原生接口基于事件，异步操作较复杂，推荐使用 Promise 包装器
- 隐私模式下部分存储 API 可能受限或行为不同

## 相关页面

- [[JavaScript 教程概述]]
- [[JavaScript 网络请求]]
- [[JavaScript Frame 与 Window]]
- [[JavaScript 二进制数据]]
- [[Linux 文件系统]]

## 原始来源

- [在浏览器中存储数据](raw/zh.javascript.info/6-data-storage/index.md)
- [Cookie，document.cookie](raw/zh.javascript.info/6-data-storage/01-cookie/article.md)
- [LocalStorage，sessionStorage](raw/zh.javascript.info/6-data-storage/02-localstorage/article.md)
- [IndexedDB](raw/zh.javascript.info/6-data-storage/03-indexeddb/article.md)
