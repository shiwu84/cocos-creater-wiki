# Wiki 内容目录

Wiki 目录 — 所有 LLM 生成的 Wiki 页面索引。

> 本文件由 LLM 自动维护，每次 Ingest 操作后更新。

## 概述

| 页面 | 摘要 |
|------|------|
| [[Cocos Creator 概述]] | Cocos Creator 3.8 引擎的整体介绍、版本演进、核心特性、系统架构总览 |
| [[引擎架构]] | ECS 架构详解、子系统关系、更新循环、与 Unity 的对比 |
| [[Cocos Creator 2.x]] | Cocos Creator 2.x 与 3.x 的主要差异对比 |
| [[Cocos Creator 3.x]] | Cocos Creator 3.x 统一 2D/3D 开发的新架构 |
| [[Cocos Creator 3D]] | 已被 3.x 统一取代的独立版本说明 |

## 核心系统

| 页面 | 摘要 | 类别 |
|------|------|------|
| [[场景与节点系统]] | 场景管理、节点树、组件挂载、坐标系、渲染顺序 | 核心系统 |
| [[资源系统]] | 资源类型、导入工作流、Asset Manager、加载方式、分包策略 | 核心系统 |
| [[图形渲染]] | 渲染管线、相机、光照阴影、材质、辅助渲染特性 | 核心系统 |
| [[脚本系统]] | TypeScript 脚本、装饰器、生命周期、事件系统、资源加载 | 核心系统 |
| [[物理系统]] | 3D 物理（Bullet）、2D 物理（Box2D）、刚体、碰撞体、关节 | 核心系统 |
| [[动画系统]] | Animation 组件、动画剪辑、骨骼动画、Marionette 动画图、程序化动画 | 核心系统 |
| [[音频系统]] | AudioSource 组件、AudioClip 资源、Web Audio / DOM Audio 双模式、playOneShot 音效 | 核心系统 |
| [[缓动系统]] | Tween 链式 API、内置缓动函数、队列/并行/重复控制、v3.8.4 自定义动作与翻转 | 核心系统 |
| [[发布系统]] | 双内核跨平台发布、构建面板与选项、Web/小游戏/原生多平台、命令行自动化构建 | 核心系统 |
| [[Shader 系统]] | Cocos Shader（YAML+GLSL）、CCEffect/CCProgram 结构、Surface Shader、宏定义、内置 Uniform | 核心系统 |
| [[材质系统]] | 材质作为着色器数据集、EffectAsset 关联、共享材质与材质实例、程序化材质操作 | 核心系统 |
| [[UI 系统]] | Canvas 画布、多分辨率适配、Widget/Layout 布局、渲染合批、20+ UI 组件 | 核心系统 |
| [[粒子系统]] | 3D/2D 粒子、模块化架构（主模块、发射器、9 个动画子模块、渲染器）、CPU vs GPU 渲染 | 核心系统 |

## 资源类型

| 页面 | 摘要 |
|------|------|
| [[场景资源]] | 场景文件（.scene）的结构与属性 |
| [[预制资源]] | 预制体（Prefab）的创建、编辑与实例化 |
| [[材质资源]] | 材质的资源文件格式与属性面板 |
| [[纹理贴图]] | 图片资源与纹理的导入格式、过滤模式、压缩纹理 |
| [[精灵帧]] | 精灵帧资源与图集管理 |
| [[模型资源]] | 3D 模型（FBX/glTF）的导入、Mesh 与材质提取 |
| [[音频资源]] | 音频剪辑的格式兼容性与加载模式 |
| [[字体资源]] | 位图字体、TTF 字体与 LabelAtlas 资源 |
| [[动画剪辑]] | 动画剪辑资源的创建、编辑与曲线控制 |
| [[JSON 文本资源]] | JSON 和文本资源的加载与解析 |
| [[Spine 动画]] | Spine 骨骼动画资源的导入与使用 |
| [[DragonBones 动画]] | DragonBones 动画资源的导入与使用 |
| [[TiledMap]] | Tiled 地图编辑器导出的 TMX 地图资源 |

## 编辑器与工具

| 页面 | 摘要 | 类别 |
|------|------|------|
| [[编辑器界面]] | 编辑器面板布局、场景编辑器、层级管理器、资源管理器、属性检查器、偏好设置、项目设置 | 编辑器 |

## 进阶主题

| 页面 | 摘要 | 类别 |
|------|------|------|
| [[进阶主题]] | 引擎定制、热更新、数据存储、多语言国际化、网络通信、JSB 原生桥接、动态合图 | 进阶 |

## 对比分析

| 页面 | 摘要 | 类别 |
|------|------|------|
| [[Cocos Creator vs Unity]] | 架构设计、脚本语言、编辑器工作流、渲染管线、物理系统、动画系统的全方位对比 | 对比 |

## 工具与参考

| 页面 | 摘要 | 类别 |
|------|------|------|
| [[Jujutsu VCS]] | Jujutsu (jj) 实验性分布式版本控制系统，兼容 Git，操作日志、自动快照 | 工具 |
| [[Git 与版本控制]] | Git 基础工作流和与 Jujutsu 的对比 | 工具 |

## JavaScript 教程

| 页面 | 摘要 | 章节 |
|------|------|------|
| [[JavaScript 教程概述]] | 《现代 JavaScript 教程》中文版总览（174 篇文章），JS 语言 + 浏览器 API 完整体系 | 概述 |
| [[JavaScript 入门]] | 语言特性、浏览器能力与安全限制、JS 引擎（V8/SpiderMonkey）、开发环境 | Part 1 §1 |
| [[JavaScript 基础知识]] | 变量声明（let/const/var）、八种数据类型、运算符、条件与循环、函数基础、箭头函数 | Part 1 §2 |
| [[JavaScript 代码质量]] | 浏览器调试、编码风格、BDD 测试（Mocha/Chai）、Polyfill 与转译器 | Part 1 §3 |
| [[JavaScript 对象基础]] | 对象创建与属性、引用与拷贝、this 绑定、构造器、可选链、Symbol、类型转换 | Part 1 §4 |
| [[JavaScript 数据类型]] | 数字/Math、字符串、数组原型方法、Map/Set/WeakMap/WeakSet、解构、Date、JSON | Part 1 §5 |
| [[JavaScript 函数进阶]] | 递归、Rest/Spread、闭包与词法环境、var 提升、setTimeout/setInterval、call/apply/bind、装饰器 | Part 1 §6 |
| [[JavaScript 对象属性]] | 属性标志与描述符、Object 全局方法（freeze/seal）、访问器属性 getter/setter | Part 1 §7 |
| [[JavaScript 原型与继承]] | Prototype 链、F.prototype、原生原型、现代原型操作方法 | Part 1 §8 |
| [[JavaScript 类]] | class 语法糖、extends/super、静态成员、私有字段 #、instanceof、Mixin 模式 | Part 1 §9 |
| [[JavaScript 错误处理]] | try...catch...finally、throw、Error 层次体系、包装异常模式 | Part 1 §10 |
| [[JavaScript Promise 与异步]] | 回调→Promise 链→async/await、Promise API 六大静态方法、微任务队列 | Part 1 §11 |
| [[JavaScript Generator 与迭代器]] | function\* yield 双向通信、Symbol.asyncIterator、async generator 分页数据应用 | Part 1 §12 |
| [[JavaScript 模块]] | ES6 export/import 语法体系、命名/默认导出、模块作用域、动态 import() | Part 1 §13 |
| [[JavaScript 杂项]] | Proxy 拦截、eval、柯里化、Reference Type、BigInt、Unicode 编码 | Part 1 §14 |
| [[JavaScript Document]] | DOM 树结构、节点搜索与遍历、特性/属性、文档修改、样式操作、尺寸与坐标系统 | Part 2 §1 |
| [[JavaScript 事件]] | 事件处理程序分配、冒泡/捕获传播、事件委托、阻止默认行为、自定义事件 | Part 2 §2 |
| [[JavaScript 事件细节]] | 鼠标事件类型与坐标、键盘 event.key/code、移动事件与拖拽、滚动事件优化 | Part 2 §3 |
| [[JavaScript 表单与控件]] | 表单导航与属性、focus/blur 焦点管理、change/input 事件、提交与验证 | Part 2 §4 |
| [[JavaScript 数据加载]] | 页面生命周期事件、async vs defer 脚本加载、动态资源 onload/onerror | Part 2 §5 |
| [[JavaScript Frame 与 Window]] | 弹窗操作、同源策略、跨窗口通信（postMessage）、iframe、点击劫持防御 | Part 3 |
| [[JavaScript 二进制数据]] | ArrayBuffer/TypedArray/DataView、Blob/File、TextDecoder/Encoder 编解码 | Part 4 |
| [[JavaScript 网络请求]] | fetch API、WebSocket 实时通信、FormData、请求控制（中止/进度/跨域） | Part 5 |
| [[JavaScript 浏览器存储]] | Cookie、localStorage/sessionStorage、IndexedDB 事务性客户端数据库 | Part 6 |
| [[JavaScript 动画]] | 贝塞尔曲线、CSS 过渡/动画、requestAnimationFrame、自定义时序函数 | Part 7 |
| [[JavaScript Web Components]] | Custom Elements、Shadow DOM 封装、Template 元素、Slot 插槽 | Part 8 |
| [[JavaScript 正则表达式]] | 修饰符、字符类/量词/锚点/分组/环视断言、match/matchAll/replace 方法体系 | Part 9 |

---

*最后更新：2026-06-02 • Wiki 页面数：63*
