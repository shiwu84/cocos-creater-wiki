---
title: JavaScript 动画
date: 2026-06-02
tags:
  - layer/js
  - browser
  - animation
  - type/concept
  - status/stable
aliases: []
---

# JavaScript 动画

> [!abstract] 摘要
> 浏览器支持三种动画实现方式：CSS 过渡/动画（声明式，性能最优）、JavaScript 动画（命令式，最灵活，可处理复杂路径和逻辑），以及贝塞尔曲线作为两者的数学基础。JS 动画通过 `requestAnimationFrame` 实现帧同步，可自定义任意时序函数（不受贝塞尔曲线限制）。

## 核心概念

### 贝塞尔曲线

贝塞尔曲线由控制点定义，曲线的阶次 = 控制点数 - 1。CSS 动画的 `transition-timing-function` 和 `animation-timing-function` 使用三次贝塞尔曲线（4 个控制点）。关键特性：

- 控制点不总在曲线上
- 曲线总是在控制点的凸包内部
- 通过移动控制点可以直观地改变曲线形态

### CSS 动画

#### CSS 过渡（Transition）

当属性值变化时，浏览器自动生成流畅的过渡动画：

```css
.animated {
  transition-property: background-color;
  transition-duration: 3s;
  transition-timing-function: ease;
  transition-delay: 0s;
}
/* 简写: transition: property duration timing-function delay */
```

#### CSS 动画（Animation）

通过 `@keyframes` 定义关键帧序列，可控制更复杂的动画：

```css
@keyframes slidein {
  from { transform: translateX(0); }
  to   { transform: translateX(100px); }
}
.animated {
  animation: slidein 3s ease infinite;
}
```

CSS 动画可结合 JavaScript 事件（`transitionend`、`animationend`）实现交互。

### JavaScript 动画 —— requestAnimationFrame

`requestAnimationFrame(callback)` 在浏览器下次重绘前调用回调，相比 `setInterval` 有三大优势：

1. 多个动画的回调合并为一次重绘，减少布局抖动
2. 后台标签页自动暂停动画，节省资源
3. 回调接收高精度时间戳（`performance.now()`）

结构化动画函数模板：

```js
function animate({timing, draw, duration}) {
  let start = performance.now();
  requestAnimationFrame(function animate(time) {
    let timeFraction = (time - start) / duration;
    if (timeFraction > 1) timeFraction = 1;
    let progress = timing(timeFraction);
    draw(progress);
    if (timeFraction < 1) requestAnimationFrame(animate);
  });
}
```

- `duration`：动画总时长（ms）
- `timing(timeFraction)`：时序函数，输入 0~1 的归一化时间，返回动画进度
- `draw(progress)`：实际绘制函数，`progress=0` 为初始状态，`progress=1` 为结束状态

### 时序函数（Timing Functions）

JS 动画可自定义任意时序函数，不受贝塞尔曲线限制：

| 函数 | 效果 |
|------|------|
| 幂函数 `n` 次方 | 加速动画 |
| `circ`（圆弧）| 圆弧形加速 |
| `back`（弓弦）| 先回退再前进 |
| `bounce`（弹跳）| 终止时弹跳 |
| `elastic`（弹性）| 终止时弹性震荡 |

变换函数：
- `makeEaseOut(timing)`：`1 - timing(1 - t)`，反转动画方向
- `makeEaseInOut(timing)`：前半段 easeIn，后半段 easeOut

### CSS 动画与 JS 动画对比

| 特性 | CSS 动画 | JS 动画 |
|------|---------|---------|
| 简单动画 | 声明式，简单 | 需编写代码 |
| 性能 | 可由 GPU 加速 | 主线程执行 |
| 灵活性 | 仅限贝塞尔时序 | 任意时序函数 |
| 动画目标 | 仅 CSS 属性 | 任意（DOM、Canvas、WebGL） |
| 控制力 | 通过类名切换 | 完全程序化控制 |

## 关键细节

- CSS 过渡在属性变化时自动触发，JS 只需改属性值
- `requestAnimationFrame` 通常在 10-20ms 间隔触发（约 60fps）
- 可通过 `cancelAnimationFrame(requestId)` 取消回调
- CSS 动画事件：`transitionend`、`animationstart`、`animationiteration`、`animationend`

## 与其他系统的关系

- 与 **Canvas 图形编程** 相关：JS 动画常用于 Canvas 动画
- 与 **JavaScript 性能优化** 相关：`requestAnimationFrame` 是高性能动画的核心
- 与 **CSS 布局与样式** 相关：CSS 过渡和动画依赖于 CSS 属性系统

## 注意事项

- 尽量使用 CSS 动画处理简单动画，性能更优
- `setInterval` 不适合动画，其间隔不精确且不会在后台暂停
- 避免在动画中使用触发强制布局的属性（如读取 `offsetHeight`）
- JS 动画的 `draw` 函数可用于任何东西，不仅仅是 CSS 属性
- `performance.now()` 比 `Date.now()` 更适合动画计时（更高精度且不受系统时间调整影响）

## 相关页面

- [[JavaScript 教程概述]]
- CSS 布局与样式
- [[Linux 进程模型]]

## 原始来源

- [动画](raw/zh.javascript.info/7-animation/index.md)
- [贝塞尔曲线](raw/zh.javascript.info/7-animation/1-bezier-curve/article.md)
- [CSS 动画](raw/zh.javascript.info/7-animation/2-css-animations/article.md)
- [JavaScript 动画](raw/zh.javascript.info/7-animation/3-js-animation/article.md)
