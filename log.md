# 操作日志

按时间顺序记录 Wiki 的所有操作（Ingest、Query、Lint 等）。

---

## [2026-06-02] setup | LLM Wiki 初始化

**操作**：根据 `llm-wiki.md` 模式文档，使用 Obsidian-flavored Markdown 初始化 LLM Wiki 的基础架构。

**创建的文件**：
- `AGENTS.md` — Schema 文件，定义 Wiki 结构、页面约定、工作流
- `index.md` — Wiki 内容目录
- `log.md` — 本文件，按时间顺序的操作记录
- `Cocos Creator 概述.md` — Cocos Creator 3.8 引擎整体概述（#overview）
- `引擎架构.md` — ECS 架构、子系统关系、更新循环（#overview #concept）
- `场景与节点系统.md` — 场景管理、节点属性和组件生命周期（#entity）
- `资源系统.md` — 资源类型、Asset Manager、加载策略（#concept）
- `图形渲染.md` — 渲染管线、相机、光照阴影、材质系统（#concept）

**创建的其他文件**：
- `脚本系统.md` — TypeScript 脚本、装饰器、生命周期、事件系统（#concept）
- `物理系统.md` — 3D 物理（Bullet）、2D 物理（Box2D）、刚体碰撞关节（#concept）
- `动画系统.md` — Animation 组件、Marionette 动画图、骨骼动画、Spine（#concept）
- `.obsidian/app.json` — Obsidian 配置（附件路径、链接格式）
- `.obsidian/appearance.json` — 外观配置（折叠标题、源码模式）
- `.obsidian/graph.json` — 图谱视图配置（按标签着色）

**分析的源文档**：
- `raw/index.md` — 用户手册首页
- `raw/SUMMARY.md` — 完整目录结构
- `raw/CONTRIBUTING.md` — 文档编写规范
- `raw/concepts/scene/index.md` — 场景制作工作流
- `raw/concepts/scene/node-component.md` — 节点和组件详解
- `raw/module-map/graphics.md` — 图形渲染模块图
- `raw/asset/index.md` — 资源系统概述

**状态**：Wiki 基础架构已就绪，已创建 8 个 Wiki 页面 + 完整 Obsidian 配置。可开始逐主题 Ingest。

---

## 后续计划

1. 逐步 Ingest 各子系统源文档（脚本系统、物理系统、动画系统等）
2. 补充待撰写的 Wiki 页面
3. 创建对比页面（如 Cocos Creator vs Unity）
4. 执行首次 Lint 检查

---

## [2026-06-02] restructure | 目录结构调整

**操作**：将 Wiki 页面移入 `wiki/` 子目录，创建 `output/` 目录，对齐 llm-wiki.md 三层架构。

**变更**：
- 创建 `wiki/` 目录，移入 8 个 Wiki 页面
- 创建 `output/` 目录（暂空，供后续生成物使用）
- 更新 `AGENTS.md` 架构图、Lint 死链检查命令路径

---

## [2026-06-02] ingest | 新源添加：Jujutsu VCS + JavaScript 教程

**操作**：用户向 `raw/` 添加了两个新源，按要求进行 Ingest 处理。

**新增源文件**：
- `raw/jj_help.md` — Jujutsu 版本控制工具帮助文档（143 行）
- `raw/zh.javascript.info/` — 《现代 JavaScript 教程》中文版（174 篇文章 + 习题）

**创建的 Wiki 页面**：
- [[Jujutsu VCS]] — Jujutsu (jj) 分布式版本控制系统概述，含核心概念、命令参考、与 Git 对比
- [[JavaScript 教程概述]] — JS 教程 9 大章节总览，含语言核心和浏览器 API 体系

**更新的文件**：
- `index.md` — 新增「工具与参考」分类，添加两个新页面条目，页面数 8→10

**状态**：两个新源已建立顶层 Wiki 页面。JavaScript 教程 14 个子章节可按需深入 Ingest。

---

## [2026-06-02] batch-ingest | 全面迭代：Cocos 子系统 + JS 教程 + 交叉引用

**操作**：按照 llm-wiki.md 方法论，对全部 raw/ 源文件进行增量构建，使用 9 个并行 Agent 分三轮处理。

**新创建页面**（55 个）：

### Cocos Creator 子系统（10 个）
- [[UI 系统]] — Canvas、多分辨率适配、Widget/Layout、渲染合批、20+ UI 组件
- [[粒子系统]] — 模块化架构、CPU vs GPU 渲染器、渲染模式、粒子剔除、自定义材质
- [[音频系统]] — AudioSource、AudioClip、Web Audio/DOM Audio 双模式
- [[缓动系统]] — Tween 链式 API、缓动函数、v3.8.4 自定义动作与翻转
- [[发布系统]] — 双内核跨平台、构建面板、Web/小游戏/原生
- [[Shader 系统]] — CCEffect/CCProgram、Surface/Legacy Shader、宏定义、全局 Uniform
- [[材质系统]] — EffectAsset 数据集、共享/实例材质、程序化操作
- [[编辑器界面]] — 编辑器面板布局、场景编辑器、层级管理器、属性检查器
- [[进阶主题]] — 引擎定制、热更新、JSB 桥接、动态合图
- [[Cocos Creator vs Unity]] — 架构/脚本/渲染/物理/动画全方位对比

### Cocos Creator 资源类型（13 个）
- [[场景资源]]、[[预制资源]]、[[材质资源]]、[[纹理贴图]]、[[精灵帧]]
- [[模型资源]]、[[音频资源]]、[[字体资源]]、[[动画剪辑]]、[[JSON 文本资源]]
- [[Spine 动画]]、[[DragonBones 动画]]、[[TiledMap]]

### Cocos Creator 版本（3 个）
- [[Cocos Creator 2.x]]、[[Cocos Creator 3.x]]、[[Cocos Creator 3D]]

### 工具（1 个）
- [[Git 与版本控制]]

### JavaScript 教程（28 个）
- 概述：[[JavaScript 教程概述]]
- Part 1 语言（14 章）：入门、基础知识、代码质量、对象基础、数据类型、函数进阶、对象属性、原型与继承、类、错误处理、Promise 与异步、Generator 与迭代器、模块、杂项
- Part 2 浏览器（5 章）：Document、事件、事件细节、表单与控件、数据加载
- Part 3-9（7 章）：Frame 与 Window、二进制数据、网络请求、浏览器存储、动画、Web Components、正则表达式

**修复的交叉引用**：
- 修正 JS 页面的 10 个错误 wikilink（`[[Environment]]`/`[[Prototype]]`/`[[HomeObject]]` → 代码格式）
- 补充资源系统下 13 个资源子页面的 wikilink 目标

**Lint 结果**：
- 0 orphan wikilinks
- 0 dead raw links
- Wiki 页面总数：10 → 63
