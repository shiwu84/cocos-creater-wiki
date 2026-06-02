---
title: Jujutsu VCS
date: 2026-06-02
tags:
  - layer/tool
  - type/concept
  - status/stable
aliases:
  - jj
  - Jujutsu
---

# Jujutsu VCS

> [!abstract] 摘要
> Jujutsu (jj) 是一个实验性的分布式版本控制系统，兼容 Git 仓库，提供更简洁的变基工作流、操作日志、工作区管理等现代特性。

## 核心概念

Jujutsu 是一种与 Git 兼容的 VCS，但采用了不同的工作流模型：

- **变更（Change）** 是核心抽象，替代 Git 的暂存区/提交两阶段模型
- **操作日志（Operation Log）** 记录所有操作，支持 `undo`/`redo`，比 Git 的 reflog 更强大
- **工作区（Working Copy）** 自动快照，每次命令执行前自动保存工作区状态
- **不可变提交** 默认保护配置的不可变提交集，防止意外改写

### 与 Git 的关系

| Jujutsu | Git |
|---------|-----|
| 变更 (Change) | 提交 (Commit) |
| 操作日志 | reflog (但更结构化) |
| 自动快照 | 手动 `git add` |
| undo | 困难 (需 reflog + reset) |
| 一流变基支持 | 视情况而定 |

Jujutsu 后端使用 Git 仓库存储，`jj git push` / `jj git fetch` 与 Git 远程交互。

## 关键命令

### 工作区管理
- `jj new` — 创建新空变更并切换工作区
- `jj edit <revision>` — 切换工作区到指定版本
- `jj abandon <revision>` — 放弃变更
- `jj status` / `jj st` — 查看仓库状态
- `jj workspace` — 多工作区管理

### 历史操作
- `jj log` — 查看版本历史
- `jj show <revision>` — 查看提交详情和差异
- `jj diff` — 比较两个版本的差异
- `jj evolog` — 查看变更的演进历史

### 变更操作
- `jj rebase` — 变基到不同父提交
- `jj squash` — 合并变更到另一个变更
- `jj split` — 将一个变更拆分为两个
- `jj absorb` — 将工作区变更移入修订栈
- `jj parallelize` — 并行化修订
- `jj arrange` — 交互式编排提交图

### 元数据
- `jj describe` / `jj desc` — 更新变更描述或元数据
- `jj commit` / `jj ci` — 更新描述并创建新变更
- `jj metaedit` — 修改修订元数据而不改变内容
- `jj sign` / `jj unsign` — 密码签名

### 撤销/重做
- `jj undo` — 撤销最近一次操作
- `jj redo` — 重做已撤销的操作
- `jj op log` — 查看操作历史
- `jj op restore` — 恢复到指定操作

### Git 互操作
- `jj git push` — 推送到 Git 远程
- `jj git fetch` — 从 Git 远程获取
- `jj bookmark` / `jj b` — 管理书签（等价于 Git 分支）
- `jj tag` — 管理标签

## 关键细节

### 自动快照
每次命令执行前，Jujutsu 自动将工作区变更快照到当前工作区提交（`@`）。可使用 `--ignore-working-copy` 跳过此行为。

### 操作模型
- 操作可以在不集成到操作日志的情况下创建 (`--no-integrate-operation`)
- 可在历史操作点查看仓库状态 (`--at-operation <id>`)
- 操作日志中的发散操作永远不会合入当前状态

### 配置
- 支持 TOML 格式配置，通过 `--config name=value` 内联设置
- 支持多个配置文件 (`--config-file <path>`)

## 注意事项

- Jujutsu 仍处于实验阶段，设计可能变化
- 使用 `.jj/` 目录存储元数据，与 `.git/` 共存
- 不可变提交保护通过配置的 `immutable_heads()` revset 控制
- `--ignore-immutable` 可绕过不可变保护（根提交除外）
- 工作区路径可通过 sparse 配置管理，控制哪些文件可见

## 相关页面

- [[Git 与版本控制]] — 传统 Git 工作流对比
- [[Linux 概述]] — Jujutsu 运行在 Linux 环境，底层依赖文件系统和进程模型
- [[软件工程概述]] — 版本控制在软件工程实践中的角色与模式
- [[编辑器界面]] — Jujutsu 与编辑器/IDE 集成的开发工作流

## 原始来源

- [jj help](raw/jj_help.md)
