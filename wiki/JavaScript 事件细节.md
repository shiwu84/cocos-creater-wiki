---
title: JavaScript 事件细节
date: 2026-06-02
tags:
  - layer/js
  - browser
  - event
  - type/concept
  - status/stable
aliases: []
---

# JavaScript 事件细节

> [!abstract] 摘要
> 本章深入分析各种 UI 事件的具体属性和用法，包括鼠标事件（类型、按键、坐标）、鼠标移动与拖拽、指针事件、键盘事件（keydown/keyup、code vs key 区别）以及滚动事件。理解这些事件的细节属性是正确处理用户交互的前提。

## 核心概念

### 鼠标事件类型

| 事件 | 说明 |
|------|------|
| `mousedown` / `mouseup` | 鼠标按下/释放（在任何按键上都会触发） |
| `mouseover` / `mouseout` | 鼠标移入/移出元素（会冒泡） |
| `mousemove` | 鼠标在元素上的每次移动 |
| `click` | 鼠标左键在元素上按下并释放后触发 |
| `dblclick` | 短时间内双击同一元素（现代开发中较少使用） |
| `contextmenu` | 鼠标右键或特殊键盘按键触发 |

事件触发顺序：`mousedown` → `mouseup` → `click`（移动鼠标触发 `mousemove`）。

### 鼠标按键

`event.button` 属性区分按下的按键：
- `0` — 左键（主要按键）
- `1` — 中键（辅助按键）
- `2` — 右键（次要按键）
- `3` — X1（后退）
- `4` — X2（前进）

> [!warning] 不再使用 `event.which`
> `event.which` 已被弃用（`1`=左键，`2`=中键，`3`=右键），**不要**在新代码中使用。

### 组合键

所有鼠标事件都包含组合键状态属性（布尔值）：
- `shiftKey` — `Shift` 按下
- `altKey` — `Alt`（Mac 上为 `Opt`）按下
- `ctrlKey` — `Ctrl` 按下
- `metaKey` — Mac 的 `Cmd` 键按下

> [!tip] 跨平台组合键
> Windows/Linux 使用 `Ctrl`，Mac 使用 `Cmd`（`metaKey`）。要跨平台支持，应检查 `if (e.ctrlKey || e.metaKey)`。注意 Mac 上 `Ctrl+左键` 会被解释为右键单击。

### 坐标系统

两种坐标类型（所有鼠标事件都提供）：
- **`clientX` / `clientY`** — 相对于**当前窗口**左上角，类似 `position: fixed`，随滚动变化
- **`pageX` / `pageY`** — 相对于**文档**左上角，类似 `position: absolute`，不随滚动变化

### 鼠标移动事件：over/out vs enter/leave

- **`mouseover` / `mouseout`** — 移入/移出元素时会冒泡到父元素，移入子元素也会触发父元素的 `mouseout`
- **`mouseenter` / `mouseleave`** — 不冒泡，移入子元素不会触发父元素的事件（更符合直觉）

`mouseover/out` 有 `relatedTarget` 属性（鼠标从哪来/到哪去），`mouseenter/leave` 中为 `null`。

### 鼠标拖拽

实现拖拽的步骤：
1. `mousedown` → 记录起始坐标 → 绑定 `mousemove` 和 `mouseup`
2. `mousemove` → 计算偏移量 → 移动元素
3. `mouseup` → 解绑 `mousemove` 和 `mouseup`

关键技巧：使用 `document.addEventListener` 绑定 `mousemove`/`mouseup`（而非元素上），以确保鼠标移出元素时拖拽仍能继续。设置 `document.ondragstart = function() { return false; }` 防止浏览器默认拖拽行为干扰。

### 指针事件（Pointer Events）

指针事件是现代统一规范，将鼠标、触控笔、触摸统一为一个事件模型。主要事件：
- `pointerdown` / `pointerup` / `pointermove`
- `pointerover` / `pointerout` / `pointerenter` / `pointerleave`
- `pointercancel`

`event.pointerType` 返回 `"mouse"`、`"pen"` 或 `"touch"`。`event.pointerId` 在多触控点场景（如多点触摸）中区分不同指针。

### 键盘事件

两个主要事件：`keydown`（按下，长按会自动重复）和 `keyup`（释放）。

#### event.key vs event.code

| 场景 | `event.key` | `event.code` |
|------|-------------|--------------|
| 按 `Z` | `"z"` | `"KeyZ"` |
| `Shift+Z` | `"Z"` | `"KeyZ"` |
| 按 `F1` | `"F1"` | `"F1"` |
| 按左 `Shift` | `"Shift"` | `"ShiftLeft"` |

- **`event.key`** — 获取字符含义，受当前输入语言影响（如切换语言后 "Z" 可能变成 "Я"）
- **`event.code`** — 物理按键位置代码，始终不变（如 `"KeyZ"` 始终对应美式键盘 Z 键位置）

**选择依据**：
- 处理热键/快捷键 → 用 `event.code`（不受语言切换影响）
- 处理文字输入字符 → 用 `event.key`

> [!warning] 键盘布局差异
> 不同键盘布局中，同一物理位置可能对应不同字符（如德式键盘按 Y 时 `event.code` 为 `KeyZ`）。

#### 自动重复与默认行为
- `event.repeat` — 长按触发自动重复时为 `true`
- 阻止 `keydown` 默认行为可阻止大多数按键效果（但不能阻止 OS 级快捷键如 `Alt+F4`）
- 键盘过滤不 100% 可靠：可通过右键粘贴、语音输入等绕过的，应结合 `input`/`change` 事件验证

#### 移动端键盘
虚拟键盘/IME 下，`e.keyCode` 应为 `229`，`e.key` 应为 `"Unidentified"`。不能保证所有移动键盘都提供正确的按键值。

### 滚动事件（scroll）

`scroll` 事件在页面或元素滚动时触发。关键策略：
- `scroll` 事件触发频率极高，处理程序应轻量
- 使用 `throttle`（节流）或 `debounce`（防抖）限制处理频率
- 获取滚动位置：`window.pageXOffset` / `window.pageYOffset` 或 `document.documentElement.scrollLeft/scrollTop`
- `elem.scrollBy(x, y)` 和 `elem.scrollTo(pageX, pageY)` 控制滚动

## 关键细节

1. **防止选择**：双击或拖拽鼠标可能意外选中文本。在 `mousedown` 处理程序中 `return false` 可阻止或使用 `preventDefault()`。
2. **防止复制**：`oncopy` 事件中 `return false` 或 `preventDefault()` 可阻止复制。
3. **拖拽时的 `setPointerCapture`**：可确保即使鼠标移到元素外，目标元素仍能接收所有后续指针事件。
4. **触摸事件与 click 的 300ms 延迟**：移动端浏览器为了区分双击缩放，click 有约 300ms 延迟。可使用 pointer events 或 `touch-action: manipulation` CSS 消除。

## 注意事项

- `mouseenter`/`mouseleave` **不冒泡**，子元素移入/出时不会触发父元素这些事件
- 鼠标事件不仅来自鼠标设备，也来自模拟鼠标的触摸设备
- 跟踪 `<input>` 输入内容时优先使用 `input` 事件，而非键盘事件（键盘事件无法覆盖粘贴、语音输入等）
- 过去存在 `keypress` 事件和 `keyCode`/`charCode` 属性，已被弃用，不要在新代码中使用

## 相关页面
- [[JavaScript Document]]
- [[JavaScript 事件]]
- [[JavaScript 表单与控件]]
- [[JavaScript 数据加载]]
- [[JavaScript 教程概述]]

## 原始来源
- [UI 事件](raw/zh.javascript.info/2-ui/3-event-details/index.md)
- [鼠标事件](raw/zh.javascript.info/2-ui/3-event-details/1-mouse-events-basics/article.md)
- [移动鼠标事件](raw/zh.javascript.info/2-ui/3-event-details/3-mousemove-mouseover-mouseout-mouseenter-mouseleave/article.md)
- [鼠标拖拽](raw/zh.javascript.info/2-ui/3-event-details/4-mouse-drag-and-drop/article.md)
- [指针事件](raw/zh.javascript.info/2-ui/3-event-details/6-pointer-events/article.md)
- [键盘事件](raw/zh.javascript.info/2-ui/3-event-details/7-keyboard-events/article.md)
- [滚动事件](raw/zh.javascript.info/2-ui/3-event-details/8-onscroll/article.md)
