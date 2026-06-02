---
title: JavaScript Web Components
date: 2026-06-02
tags:
  - layer/js
  - type/concept
  - status/stable
aliases: []
---

# JavaScript Web Components

> [!abstract] 摘要
> Web Components 是一组浏览器原生标准，用于创建可复用的自定义 HTML 元素。核心组成包括：Custom Elements（自定义元素类）、Shadow DOM（封装内部 DOM 和样式）、以及 `<template>` 和 `<slot>`（声明式模板和内容组合）。Web Components 的目标是让组件化的开发模式成为浏览器内置能力，而非依赖第三方框架。

## 核心概念

### 组件化架构

好的架构让复杂的事情简单化。组件应具有：
- 自己的 JavaScript 类
- 封装的 DOM 结构（外部不可直接操作）
- 隔离的 CSS 样式
- 明确的 API（事件、方法、属性）

Web Components 让浏览器原生支持这些特性。

### Custom Elements —— 自定义元素

通过继承 `HTMLElement` 定义自定义元素类，须用 `customElements.define` 注册，元素名**必须包含短横线 `-`**：

```js
class MyElement extends HTMLElement {
  constructor() { super(); /* 初始化 */ }
  connectedCallback() { /* 添加到文档时调用 */ }
  disconnectedCallback() { /* 从文档移除时调用 */ }
  static get observedAttributes() { return ['attr1', 'attr2']; }
  attributeChangedCallback(name, oldVal, newVal) { /* 属性变化时调用 */ }
  adoptedCallback() { /* 移动到新文档时调用（罕见）*/ }
}
customElements.define("my-element", MyElement);
```

两种类型：
1. **自主自定义元素**：继承 `HTMLElement`，全新的标签
2. **自定义内建元素**：继承内建元素（如 `HTMLButtonElement`），扩展已有标签

### Shadow DOM —— 封装内部 DOM

Shadow DOM 为组件创建隔离的 DOM 子树，外部 CSS 和 JS 无法直接穿透：

```js
let shadow = element.attachShadow({mode: 'open'});
shadow.innerHTML = `<style>p { color: red; }</style><p>Hello</p>`;
```

- `mode: 'open'`：可通过 `element.shadowRoot` 访问
- `mode: 'closed'`：外部无法访问 `shadowRoot`
- Shadow DOM 内的样式只作用于内部，不受外部 CSS 影响
- 事件重定向（Event Retargeting）：Shadow DOM 内的事件冒泡到外部时，`event.target` 指向宿主元素

### Template 元素 —— 声明式模板

`<template>` 标签内容不会被渲染，可克隆后动态插入：

```html
<template id="tmpl">
  <style>span { color: green; }</style>
  <span><slot name="text">默认文本</slot></span>
</template>
```

```js
let template = document.getElementById('tmpl');
let clone = template.content.cloneNode(true);
shadowRoot.appendChild(clone);
```

### Slot 插槽 —— 内容组合

`<slot>` 元素定义组件内部的占位区，允许外部内容投射到 Shadow DOM 内部：

- 具名插槽：`<slot name="header">`，外部用 `slot="header"` 匹配
- 默认插槽：`<slot>` 接收未指定 slot 的外部内容
- 插槽内容属于外部 DOM（light DOM），样式由外部控制

```html
<my-card>
  <h2 slot="header">标题</h2>
  <p>默认插槽内容</p>
</my-card>
```

### Shadow DOM 样式

Shadow DOM 提供额外的样式选择器：
- `:host`：选择宿主元素
- `:host(selector)`：宿主元素匹配选择器时应用
- `:host-context(selector)`：宿主祖先匹配选择器时应用
- `::slotted(selector)`：选择投射到插槽中的元素
- CSS 自定义属性（`--*`）可穿透 Shadow DOM 边界

## 关键细节

- Custom Elements 名称必须含 `-`，确保与内建 HTML 元素无冲突
- `connectedCallback` / `disconnectedCallback` 在元素反复添加/移除时多次调用
- `attributeChangedCallback` 仅在 `observedAttributes` 中的属性变化时触发
- Shadow DOM 内的事件（如 click）冒泡到外部时，`event.target` 会被重定向为宿主元素
- `<template>` 内容不会渲染，但可以被 JS 访问和克隆
- 插槽中的内容属于 light DOM，CSS 作用域在外部而非 Shadow DOM 内

## 与其他系统的关系

- 与 **[[JavaScript 事件]]** 相关：Shadow DOM 的事件重定向机制
- 与 **CSS 布局与样式** 相关：`:host`、`::slotted` 等 Shadow DOM 选择器
- 与 **[[JavaScript 浏览器存储]]** 相关：组件状态可通过 storage 持久化
- 与 **[[JavaScript 动画]]** 相关：组件内部可实现独立动画

## 注意事项

- Shadow DOM 影响 `document.querySelector` 等全局查询（查不到 Shadow DOM 内部元素）
- 某些旧浏览器不完全支持 Web Components，需 polyfill
- CSS 自定义属性（`--*`）是跨 Shadow DOM 边界传递样式的推荐方式
- Web Components 不替代框架路由和状态管理，可与框架组合使用
- `mode: 'closed'` 的 Shadow DOM 也会增加调试难度

## 相关页面

- [[JavaScript 教程概述]]
- [[JavaScript 事件]]
- CSS 布局与样式
- [[JavaScript 动画]]
- [[软件工程概述]]

## 原始来源

- [Web Components](raw/zh.javascript.info/8-web-components/index.md)
- [从星球轨道的高度讲起](raw/zh.javascript.info/8-web-components/1-webcomponents-intro/article.md)
- [Custom Elements](raw/zh.javascript.info/8-web-components/2-custom-elements/article.md)
- [Shadow DOM](raw/zh.javascript.info/8-web-components/3-shadow-dom/article.md)
- [Template 元素](raw/zh.javascript.info/8-web-components/4-template-element/article.md)
- [Shadow DOM 插槽](raw/zh.javascript.info/8-web-components/5-slots-composition/article.md)
- [Shadow DOM 样式](raw/zh.javascript.info/8-web-components/6-shadow-dom-style/article.md)
- [Shadow DOM 与事件](raw/zh.javascript.info/8-web-components/7-shadow-dom-events/article.md)
