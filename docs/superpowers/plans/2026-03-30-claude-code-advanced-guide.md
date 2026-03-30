# Claude Code 高级个人使用指南 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 创建一份问题驱动的Claude Code高级个人使用指南，覆盖日常编码、学习新技术、团队协作三个场景。

**Architecture:** 单一Markdown文件，按「我想做X」的问题形式组织，每个问题包含基础解法、高级技巧、最佳实践三层。使用Obsidian wiki-link和锚点实现快速导航。

**Tech Stack:** Markdown, Obsidian wiki-link语法, YAML frontmatter

---

### Task 1: 创建主文档框架（目录 + 结构骨架）

**Files:**
- Create: `工具指南/Claude Code 高级使用指南.md`

- [ ] **Step 1: 创建文档骨架**

```markdown
---
title: Claude Code 高级使用指南
tags: [claude-code, 工具指南, AI]
created: 2026-03-30
updated: 2026-03-30
---

# Claude Code 高级使用指南

> 这是一份**问题驱动**的个人实战技巧手册。遇到具体问题时，直接跳到对应章节。

## 目录

### 🔧 日常编码场景
- [[#理解陌生代码|我想快速理解一段陌生代码]]
- [[#安全重构|我想重构一个复杂函数，但怕改坏]]
- [[#代码审查|我想做代码审查，找出潜在问题]]
- [[#调试Bug|我想调试一个诡异的Bug]]

### 📚 学习新技术场景
- [[#上手新框架|我想快速上手一个新框架/语言]]
- [[#理解设计模式|我想理解某个设计模式在实际代码中的应用]]
- [[#消化技术文章|我想消化一篇技术文章/论文]]

### 👥 团队协作场景
- [[#生成文档|我想生成规范的代码注释/文档]]
- [[#理解他人代码库|我想快速了解别人写的代码库]]
- [[#生成PR描述|我想生成PR描述/Commit message]]

### ⚡ 通用高级技巧
- [[#Context管理|Context管理]]
- [[#CLAUDE.md定制|CLAUDE.md：为项目定制AI行为]]
- [[#Prompt技巧|指令技巧：如何写出更精准的Prompt]]
- [[#多文件操作|多文件操作：如何安全地大范围修改]]
- [[#子Agent|子Agent和并行处理]]

### 📎 附录
- [[#常用命令速查|常用命令速查表]]

---

## 🔧 日常编码场景

## 理解陌生代码
<!-- TODO: 填充 -->

## 安全重构
<!-- TODO: 填充 -->

## 代码审查
<!-- TODO: 填充 -->

## 调试Bug
<!-- TODO: 填充 -->

---

## 📚 学习新技术场景

## 上手新框架
<!-- TODO: 填充 -->

## 理解设计模式
<!-- TODO: 填充 -->

## 消化技术文章
<!-- TODO: 填充 -->

---

## 👥 团队协作场景

## 生成文档
<!-- TODO: 填充 -->

## 理解他人代码库
<!-- TODO: 填充 -->

## 生成PR描述
<!-- TODO: 填充 -->

---

## ⚡ 通用高级技巧

## Context管理
<!-- TODO: 填充 -->

## CLAUDE.md定制
<!-- TODO: 填充 -->

## Prompt技巧
<!-- TODO: 填充 -->

## 多文件操作
<!-- TODO: 填充 -->

## 子Agent
<!-- TODO: 填充 -->

---

## 📎 附录

## 常用命令速查
<!-- TODO: 填充 -->
```

---

## 实现任务分解

由于内容量大，将分解为以下任务：

### Task 1: 创建主文档框架
- [ ] 创建文档骨架和目录
- [ ] 提交框架

### Task 2-5: 编写日常编码场景（4个问题）
- [ ] 理解陌生代码
- [ ] 安全重构
- [ ] 代码审查
- [ ] 调试Bug

### Task 6-8: 编写学习新技术场景（3个问题）
- [ ] 上手新框架
- [ ] 理解设计模式
- [ ] 消化技术文章

### Task 9-11: 编写团队协作场景（3个问题）
- [ ] 生成文档
- [ ] 理解他人代码库
- [ ] 生成PR描述

### Task 12-16: 编写通用高级技巧（5个部分）
- [ ] Context管理
- [ ] CLAUDE.md定制
- [ ] Prompt技巧
- [ ] 多文件操作
- [ ] 子Agent

### Task 17: 添加附录和优化
- [ ] 常用命令速查表
- [ ] 内部链接优化
- [ ] 最终审查