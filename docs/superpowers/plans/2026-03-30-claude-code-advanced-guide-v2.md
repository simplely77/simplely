# Claude Code 高级个人使用指南 - 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 创建一份问题驱动的Claude Code高级个人使用指南，覆盖日常编码、学习新技术、团队协作三个场景。

**Architecture:** 单一Markdown文件，按「我想做X」问题形式组织，每个问题包含基础解法→高级技巧→最佳实践三层递进。使用Obsidian wiki-link实现快速导航。

**Tech Stack:** Markdown + Obsidian wiki-link语法

---

### Task 1: 创建主文档框架

**Files:**
- Create: `工具指南/Claude Code 高级使用指南.md`

- [ ] **Step 1: 创建文档骨架**

在Obsidian中创建新文件 `工具指南/Claude Code 高级使用指南.md`，内容为：

```markdown
---
title: Claude Code 高级使用指南
tags: [claude-code, 高级技巧, 个人指南]
created: 2026-03-30
---

# Claude Code 高级使用指南

> 这是一份**问题驱动**的个人实战技巧手册，遇到具体问题时直接跳到对应章节。

## 快速导航

### 🔧 日常编码（4个问题）
- [[#理解陌生代码|我想快速理解一段陌生代码]]
- [[#安全重构|我想重构一个复杂函数，但怕改坏]]
- [[#代码审查|我想做代码审查，找出潜在问题]]
- [[#调试Bug|我想调试一个诡异的Bug]]

### 📚 学习新技术（3个问题）
- [[#上手新框架|我想快速上手一个新框架/语言]]
- [[#理解设计模式|我想理解某个设计模式在实际代码中的应用]]
- [[#消化技术文章|我想消化一篇技术文章/论文]]

### 👥 团队协作（3个问题）
- [[#生成文档|我想生成规范的代码注释/文档]]
- [[#理解他人代码库|我想快速了解别人写的代码库]]
- [[#生成PR描述|我想生成PR描述/Commit message]]

### ⚡ 通用高级技巧（5个）
- [[#Context管理|Context管理]]
- [[#CLAUDE.md定制|CLAUDE.md：为项目定制AI行为]]
- [[#Prompt技巧|指令技巧：如何写出更精准的Prompt]]
- [[#多文件操作|多文件操作：如何安全地大范围修改]]
- [[#子Agent|子Agent和并行处理]]

---

## 🔧 日常编码场景

### 理解陌生代码
<!-- 待填充 -->

### 安全重构
<!-- 待填充 -->

### 代码审查
<!-- 待填充 -->

### 调试Bug
<!-- 待填充 -->

---

## 📚 学习新技术场景

### 上手新框架
<!-- 待填充 -->

### 理解设计模式
<!-- 待填充 -->

### 消化技术文章
<!-- 待填充 -->

---

## 👥 团队协作场景

### 生成文档
<!-- 待填充 -->

### 理解他人代码库
<!-- 待填充 -->

### 生成PR描述
<!-- 待填充 -->

---

## ⚡ 通用高级技巧

### Context管理
<!-- 待填充 -->

### CLAUDE.md定制
<!-- 待填充 -->

### Prompt技巧
<!-- 待填充 -->

### 多文件操作
<!-- 待填充 -->

### 子Agent
<!-- 待填充 -->

---

## 📎 附录：常用命令速查
<!-- 待填充 -->
```

- [ ] **Step 2: 提交框架**

```bash
git add 工具指南/Claude\ Code\ 高级使用指南.md
git commit -m "docs: scaffold Claude Code advanced guide"
```

---

### Task 2-6: 编写日常编码场景（4个问题）

**说明**：每个问题按「基础解法 → 高级技巧 → 最佳实践」三层组织。参考[[docs/superpowers/specs/2026-03-30-claude-code-advanced-guide-design.md|设计文档]]第3.1节了解具体内容。

- [ ] **Step 1: 填充「理解陌生代码」**
- [ ] **Step 2: 填充「安全重构」**
- [ ] **Step 3: 填充「代码审查」**
- [ ] **Step 4: 填充「调试Bug」**
- [ ] **Step 5: 逐个提交各章节**

---

### Task 7-9: 编写学习新技术场景（3个问题）

**说明**：参考[[docs/superpowers/specs/2026-03-30-claude-code-advanced-guide-design.md|设计文档]]第3.2节。

- [ ] **Step 1: 填充「上手新框架」**
- [ ] **Step 2: 填充「理解设计模式」**
- [ ] **Step 3: 填充「消化技术文章」**
- [ ] **Step 4: 提交三个章节**

---

### Task 10-12: 编写团队协作场景（3个问题）

**说明**：参考[[docs/superpowers/specs/2026-03-30-claude-code-advanced-guide-design.md|设计文档]]第3.3节。

- [ ] **Step 1: 填充「生成文档」**
- [ ] **Step 2: 填充「理解他人代码库」**
- [ ] **Step 3: 填充「生成PR描述」**
- [ ] **Step 4: 提交三个章节**

---

### Task 13-17: 编写通用高级技巧（5个部分）

**说明**：参考[[docs/superpowers/specs/2026-03-30-claude-code-advanced-guide-design.md|设计文档]]第3.4节。每个部分都包含「概念 → 实战 → 常见陷阱」。

- [ ] **Step 1: 填充「Context管理」**
- [ ] **Step 2: 填充「CLAUDE.md定制」**
- [ ] **Step 3: 填充「Prompt技巧」**
- [ ] **Step 4: 填充「多文件操作」**
- [ ] **Step 5: 填充「子Agent」**
- [ ] **Step 6: 提交五个章节**

---

### Task 18: 添加附录和优化

**Files:**
- Modify: `工具指南/Claude Code 高级使用指南.md`

- [ ] **Step 1: 填充附录「常用命令速查」**

添加常见的Claude Code指令和快捷键：
- `@file` 快速引用文件
- `/clear` 清空context
- Shift+Enter 换行
- Ctrl/Cmd+K 新对话

- [ ] **Step 2: 优化导航和格式**

- 确保所有wiki-link都能正确链接（使用`[[#锚点|显示文本]]`格式）
- 每个问题章节确保有「基础解法 → 高级技巧 → 最佳实践」三部分
- 代码示例添加语言标记（markdown、bash等）

- [ ] **Step 3: 最终审查**

从头读一遍整个指南：
- 是否覆盖了设计文档的所有要点？
- 每个问题的三层递进是否清晰？
- 是否有重复或冗余的内容？

- [ ] **Step 4: 最终提交**

```bash
git add 工具指南/Claude\ Code\ 高级使用指南.md
git commit -m "docs: complete Claude Code advanced guide with all sections"
```

---

## 实现说明

**内容来源**：每个问题的具体内容参考 [[docs/superpowers/specs/2026-03-30-claude-code-advanced-guide-design.md|设计文档]] 第3节。

**格式要求**：
- 使用`###`作为问题标题（三级标题）
- 基础解法用```包裹代码示例
- 高级技巧用**加粗**标注技巧名
- 最佳实践用无序列表呈现

**质量标准**：
- ✅ 每个问题都可以独立阅读
- ✅ 基础解法对初学者友好
- ✅ 高级技巧涉及Context、CLAUDE.md、Hooks、SubAgent等高级功能
- ✅ 最佳实践基于实际使用经验，包含常见陷阱
