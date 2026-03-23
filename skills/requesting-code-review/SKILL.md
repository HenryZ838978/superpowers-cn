---
name: requesting-code-review
description: Use when completing tasks, implementing major features, or before merging to verify work meets requirements
---

# 请求 Code Review

派发 superpowers:code-reviewer subagent，在问题蔓延前发现问题。评审者获得精心编写的评估上下文——绝不使用本会话历史。这使评审聚焦工作产物而非你的思路，也为你保留上下文以便继续工作。

**核心原则：** 早评审、常评审。

## 何时请求评审

**必须：**
- subagent-driven development 中每个任务之后  
- 完成主要功能之后  
- merge 到 main 之前  

**可选但很有价值：**
- 卡住时（新视角）  
- 重构前（基线检查）  
- 修复复杂 bug 之后  

## 如何请求

**1. 获取 git SHA：**
```bash
BASE_SHA=$(git rev-parse HEAD~1)  # or origin/main
HEAD_SHA=$(git rev-parse HEAD)
```

**2. 派发 code-reviewer subagent：**

使用 Task 工具，类型为 superpowers:code-reviewer，按 `code-reviewer.md` 模板填写

**占位符：**
- `{WHAT_WAS_IMPLEMENTED}` — 你刚构建的内容  
- `{PLAN_OR_REQUIREMENTS}` — 应该做什么  
- `{BASE_SHA}` — 起始 commit  
- `{HEAD_SHA}` — 结束 commit  
- `{DESCRIPTION}` — 简短摘要  

**3. 处理反馈：**
- Critical 问题立即修  
- Important 问题在继续前修  
- Minor 问题记下稍后处理  
- 若评审有误，反驳并附理由  

## 示例

```
[Just completed Task 2: Add verification function]

You: Let me request code review before proceeding.

BASE_SHA=$(git log --oneline | grep "Task 1" | head -1 | awk '{print $1}')
HEAD_SHA=$(git rev-parse HEAD)

[Dispatch superpowers:code-reviewer subagent]
  WHAT_WAS_IMPLEMENTED: Verification and repair functions for conversation index
  PLAN_OR_REQUIREMENTS: Task 2 from docs/superpowers/plans/deployment-plan.md
  BASE_SHA: a7981ec
  HEAD_SHA: 3df7661
  DESCRIPTION: Added verifyIndex() and repairIndex() with 4 issue types

[Subagent returns]:
  Strengths: Clean architecture, real tests
  Issues:
    Important: Missing progress indicators
    Minor: Magic number (100) for reporting interval
  Assessment: Ready to proceed

You: [Fix progress indicators]
[Continue to Task 3]
```

## 与工作流集成

**Subagent-Driven Development：**
- **每个**任务后评审  
- 在问题叠加前抓住它们  
- 修完再进入下一任务  

**Executing Plans：**
- 每批（3 个任务）后评审  
- 获取反馈、应用后继续  

**临时开发：**
- merge 前评审  
- 卡住时评审  

## 危险信号

**绝不：**
- 因「很简单」跳过评审  
- 忽略 Critical 问题  
- Important 未修就继续  
- 与有效的技术反馈争辩  

**若评审有误：**
- 用技术理由反驳  
- 用代码/测试证明可用  
- 请求澄清  

模板见：requesting-code-review/code-reviewer.md
