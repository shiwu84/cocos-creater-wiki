---
title: JavaScript 表单与控件
date: 2026-06-02
tags:
  - layer/js
  - type/concept
  - status/stable
aliases: []
---

# JavaScript 表单与控件

> [!abstract] 摘要
> 表单和控件是 Web 应用中数据收集和用户输入的核心机制。本章涵盖表单元素的导航和属性访问、焦点管理（focus/blur）、输入变更事件（change/input/cut/copy/paste）、以及表单提交流程（submit）。掌握这些 API 是构建用户友好表单的必备技能。

## 核心概念

### 表单导航

表单和控件提供双向引用关系：

- `document.forms` — 获取文档中所有表单（可通过名字或索引访问）
- `form.elements` — 获取表单内的所有控件元素（含 `<fieldset>` 内的）
- `element.form` — 控件反向引用所属表单

```js
document.forms.my;       // name="my" 的表单
document.forms[0];       // 文档中的第一个表单
form.elements.one;       // name="one" 的控件
form.login;              // 等价于 form.elements.login（简写）
login.form;              // 控件所属的表单
```

`<fieldset>` 也有 `elements` 属性，可视为子表单。表单中的控件无论嵌套多深都可通过 `form.elements` 获取。

### 表单控件属性

**`<input>` 和 `<textarea>`**：
- `input.value` — 当前值（字符串）
- `input.checked` — 复选框/单选框的选中状态（布尔值）
- `textarea.value` — 当前文本（**不要用** `textarea.innerHTML`，它只反映初始 HTML）

**`<select>` 和 `<option>`**：
- `select.options` — `<option>` 集合
- `select.value` — 当前选中选项的值
- `select.selectedIndex` — 当前选中选项的索引（-1 无选中）
- 三种设置选中值的方式：
  ```js
  select.options[2].selected = true;
  select.selectedIndex = 2;
  select.value = 'banana';
  ```
- 多选（`multiple` 属性）：使用 `option.selected` 遍历获取
- `new Option(text, value, defaultSelected, selected)` — 快速创建选项元素

### 焦点管理：focus/blur

`focus` 事件在元素获得焦点时触发，`blur` 在失去焦点时触发。这些事件**不会冒泡**（但 `focusin`/`focusout` 会）。

- `elem.focus()` — 编程式设置焦点
- `elem.blur()` — 编程式移除焦点

任何可获得焦点的元素都可以聚焦，对 `tabindex` 设置 `-1` 的元素只能用 `focus()` 聚焦而不能用 `Tab` 键跳转。

```js
// 典型的 focus/blur 校验示例
input.onblur = function() {
  if (!input.value.includes('@')) {
    input.classList.add('invalid');
    error.innerHTML = '请输入有效邮箱';
    input.focus(); // 强制焦点留在输入框
  }
};
input.onfocus = function() {
  this.classList.remove('invalid');
  error.innerHTML = '';
};
```

`document.activeElement` 返回当前聚焦元素。当焦点丢失时（如点击空白处），`activeElement` 可能为 `null` 或 `<body>`。

**`tabindex`** 属性控制元素的 Tab 键导航顺序。`autofocus` 特性使元素在页面加载后自动获得焦点。

### 输入变更事件

| 事件 | 触发时机 |
|------|---------|
| `change` | 元素修改完成并失去焦点时（对于 `<select>`/`<input type="checkbox">`/`<input type="radio">` 在选择后立即触发；对于 `<textarea>` 和其他 `<input>` 在失焦时触发） |
| `input` | 每次值修改时立即触发（每次按键、粘贴等），无法通过 `preventDefault` 阻止 |
| `cut` / `copy` / `paste` | 剪切/复制/粘贴操作时触发，可通过 `preventDefault` 阻止默认行为 |
| `select` | 选中文本时触发（通过 `event.target.selectionStart/End` 获取选区） |

`input` 事件是现代跟踪用户输入的最佳方式，因为它捕获所有形式的变化（键盘、粘贴、拖放、语音输入等）。

### 表单提交：submit

表单提交的两种触发方式：
1. 点击 `<input type="submit">` 或 `<input type="image">`
2. 在表单内的 `<input>` 中按下 `Enter`

`submit` 事件在表单发送前触发，可用于数据验证。调用 `event.preventDefault()` 可阻止提交。

```js
form.onsubmit = function(event) {
  // 验证表单数据
  if (!form.elements.email.value.includes('@')) {
    alert('请输入有效邮箱');
    event.preventDefault();
    return false; // 也有效
  }
};
```

`form.submit()` 方法手动提交，但**不会触发** `submit` 事件（假设脚本已自行处理所有验证）。

> [!tip] `submit` 与 `click` 的关系
> 在输入框中按 `Enter` 提交表单时，会先在 `<input type="submit">` 上触发 `click` 事件。

## 关键细节

1. **同名多个控件**：当多个控件（通常是单选按钮）共享同名 `name` 时，`form.elements[name]` 返回**集合**而非单个元素。
2. **`select` 的三合一设置**：`options[2].selected`、`selectedIndex`、`value` 三种方式设置选中是等效的，选择最方便的方式即可。
3. **`input.value` vs `getAttribute('value')`**：修改 `input.value` 不会同步回 HTML 的 `value` 特性，因此可以通过 `getAttribute('value')` 获取原始的初始值。
4. **剪贴板事件**：`copy`/`cut`/`paste` 事件提供 `event.clipboardData` 对象，可在 `paste` 事件中通过 `clipboardData.getData()` 获取剪贴板内容。

## 注意事项

- **不要用 `textarea.innerHTML`** 获取当前值，它只反映初始 HTML 内容。始终使用 `textarea.value`。
- `focus`/`blur` **不冒泡**，但 `focusin`/`focusout` 会冒泡（event delegation 时可用后者）
- 键盘过滤（如限制只能输入数字）不可靠，应结合 `input`/`change` 事件做二次验证
- `form.submit()` 编程式提交不触发 `submit` 事件
- 浏览器的自动填充（autofill）在 `DOMContentLoaded` 时触发，可能被长时间运行的脚本延迟

## 相关页面
- [[JavaScript Document]]
- [[JavaScript 事件]]
- [[JavaScript 事件细节]]
- [[JavaScript 数据加载]]
- [[JavaScript 教程概述]]

## 原始来源
- [表单，控件](raw/zh.javascript.info/2-ui/4-forms-controls/index.md)
- [表单属性和方法](raw/zh.javascript.info/2-ui/4-forms-controls/1-form-elements/article.md)
- [聚焦：focus/blur](raw/zh.javascript.info/2-ui/4-forms-controls/2-focus-blur/article.md)
- [输入事件](raw/zh.javascript.info/2-ui/4-forms-controls/3-events-change-input/article.md)
- [表单提交](raw/zh.javascript.info/2-ui/4-forms-controls/4-forms-submit/article.md)
