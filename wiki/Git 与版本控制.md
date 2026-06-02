---
title: Git 与版本控制
date: 2026-06-02
tags:
  - layer/tool
  - type/concept
  - status/stable
aliases:
  - Git
  - 版本控制
---

# Git 与版本控制

> [!abstract] 摘要
> Git 是当前最流行的分布式版本控制系统，通过快照模型管理项目历史。核心概念包括提交（commit）、分支（branch）、合并（merge）等。[[Jujutsu VCS]] 是与 Git 兼容的现代替代方案，提供了更直观的工作流。

## 核心概念

- **提交（Commit）**：项目在某一时刻的快照，每个提交有唯一的 SHA-1 哈希标识
- **分支（Branch）**：指向某个提交的可移动指针，用于并行开发
- **工作区 / 暂存区 / 仓库**：Git 的三层模型，修改需先 `git add` 到暂存区再 `git commit`
- **合并（Merge）**：将两个分支的历史合并到一起，产生合并提交
- **变基（Rebase）**：将一系列提交移动到新的基点上，保持线性历史

## 基础工作流

```
git init / git clone     # 初始化或克隆仓库
git add <file>           # 将修改加入暂存区
git commit -m "..."      # 创建提交
git push / git pull      # 与远程同步
git branch / git checkout # 创建和切换分支
git merge / git rebase   # 合并或变基
```

## 与 Jujutsu VCS 的差异

| 特性 | Git | [[Jujutsu VCS]] |
|------|-----|-----------------|
| 工作区模型 | 需手动 `git add` 到暂存区 | 自动快照，无需暂存区 |
| 撤销操作 | 依赖 reflog，操作复杂 | 内置 `undo`/`redo` |
| 变基支持 | `git rebase`，冲突处理繁琐 | 一流变基支持，操作粒度更细 |
| 不可变提交 | 无内置保护 | 默认保护配置的不可变提交集 |
| 仓库兼容 | 原生 `.git` | 兼容 Git 仓库，通过 `jj git push/fetch` 交互 |

> [!tip] 想了解更多？查看 [[Jujutsu VCS]] 获取完整命令参考和概念说明。

## 注意事项

- Git 是分布式系统，本地提交不会自动同步到远程
- `rebase` 改写历史，不应在公共分支上执行
- 提交信息应简洁描述变更内容

## 相关页面

- [[Jujutsu VCS]]

## 原始来源

- [Git 官网](https://git-scm.com/)
