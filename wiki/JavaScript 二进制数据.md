---
title: JavaScript 二进制数据
date: 2026-06-02
tags:
  - javascript
  - concept
aliases: []
---

# JavaScript 二进制数据

> [!abstract] 摘要
> JavaScript 提供了多层级的二进制数据处理能力，从底层的 `ArrayBuffer` 到高级的 `Blob` 和 `File` 对象，通过类型化数组（TypedArray）和 `DataView` 实现灵活的数据视图，满足文件操作、图像处理、网络传输等场景的需求。

## 核心概念

### ArrayBuffer —— 二进制数据的基石

`ArrayBuffer` 是对固定长度连续内存空间的引用，创建时分配内存并用 0 填充：

```js
let buffer = new ArrayBuffer(16); // 创建 16 字节的 buffer
alert(buffer.byteLength); // 16
```

`ArrayBuffer` 不是数组，自身不可直接读写。必须通过**视图（View）**对象来操作。

### TypedArray —— 类型化数组视图

TypedArray 是一系列视图类型的总称（无 `TypedArray` 构造器），将 `ArrayBuffer` 中的字节解释为特定类型的数值：

| 类型 | 字节 | 范围 |
|------|------|------|
| `Uint8Array` | 1 | 0 ~ 255 |
| `Uint16Array` | 2 | 0 ~ 65535 |
| `Uint32Array` | 4 | 0 ~ 4294967295 |
| `Int8/16/32Array` | 1/2/4 | 有符号整数 |
| `Float32Array` | 4 | IEEE 754 单精度 |
| `Float64Array` | 8 | IEEE 754 双精度 |
| `Uint8ClampedArray` | 1 | 0 ~ 255（钳制） |

创建方式有 5 种变体：基于 ArrayBuffer、类数组对象、另一个 TypedArray、指定长度、或者无参创建空视图。

越界行为：写入超出范围的值时，多余的位被截断（对 2^8 取模），不报错。`Uint8ClampedArray` 例外，会钳制在 0~255。

### DataView —— 灵活视图

`DataView` 允许在同一 buffer 上以任意格式访问任意偏移量的数据，适合处理混合格式的二进制数据：

```js
let dataView = new DataView(buffer);
dataView.getUint8(0);    // 以 8 位读取
dataView.getUint16(0);   // 以 16 位读取
dataView.setUint32(0, 0); // 写入
```

### Blob —— 高级二进制对象

`Blob` 由可选的 MIME 类型和多个 `BlobPart`（Blob / String / BufferSource）组成，是不可变对象：

```js
let blob = new Blob(["<html>…</html>"], {type: 'text/html'});
```

- 支持 `slice()` 提取片段
- 可通过 `URL.createObjectURL(blob)` 生成临时 URL 用于 `<a>`、`<img>` 等标签
- 可作为 `fetch` 的 body 直接发送

### File —— 文件对象

`File` 继承自 `Blob`，增加了文件名和最后修改时间等属性。通常来自 `<input type="file">` 或拖放操作。

### TextDecoder / TextEncoder

用于二进制数据与字符串之间的编解码。

## 关键细节

- `buffer` 属性可访问 TypedArray 底层的 ArrayBuffer，实现视图切换
- TypedArray 有常规数组方法（`map`、`slice`、`find`、`reduce`），但没有 `splice` 和 `concat`
- `arr.set(fromArr, offset)` 批量复制；`arr.subarray(begin, end)` 创建新视图
- `BufferSource` 是 `ArrayBuffer` 或 `ArrayBufferView` 的总称，常用于 API 参数

## 与其他系统的关系

- 与 **[[JavaScript 网络请求]]** 相关：fetch 可发送/接收 Blob、ArrayBuffer
- 与 **JavaScript 文件操作** 相关：File 继承自 Blob，用于文件上传
- 与 **[[JavaScript 浏览器存储]]** 相关：IndexedDB 可存储二进制数据
- 与 **Canvas 图形编程** 相关：canvas 可通过 Blob 导出图像

## 注意事项

- `ArrayBuffer` 长度固定，不可动态增删
- Blob 是不可变的，修改需通过 `slice` 创建新 Blob
- TypedArray 越界写入不报错，调试时需注意
- `URL.createObjectURL` 创建的对象 URL 需手动 `revokeObjectURL` 释放

## 相关页面

- [[JavaScript 教程概述]]
- [[JavaScript 网络请求]]
- [[JavaScript 浏览器存储]]

## 原始来源

- [二进制数据，文件](raw/zh.javascript.info/4-binary/index.md)
- [ArrayBuffer，二进制数组](raw/zh.javascript.info/4-binary/01-arraybuffer-binary-arrays/article.md)
- [TextDecoder 和 TextEncoder](raw/zh.javascript.info/4-binary/02-text-decoder/article.md)
- [Blob](raw/zh.javascript.info/4-binary/03-blob/article.md)
- [File 和 FileReader](raw/zh.javascript.info/4-binary/04-file/article.md)
