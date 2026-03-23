---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# 撰写计划

## 概述

撰写完整的实现计划，假定工程师对代码库**零上下文**、品味**可疑**。写清他们需要的全部信息：每个任务要动哪些文件、代码、测试、可能要看的文档、如何验证。把整个计划拆成小任务。DRY。YAGNI。TDD。频繁 commit。

假定他们是熟练开发者，但几乎不了解我们的工具链或问题域，且测试设计能力一般。

**开始时声明：**「我正在使用 writing-plans skill 来创建实现计划。」

**上下文：** 应在专用 worktree 中运行（由 brainstorming skill 创建）。

**计划保存到：** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- （用户对计划路径的偏好优先于本默认）

## 范围检查

若 spec 覆盖多个彼此独立的子系统，应在 brainstorming 阶段已拆成子项目 spec。若未拆，建议拆成多份计划 — 每个子系统一份。每份计划应能单独产出可运行、可测试的软件。

## 文件结构

在定义任务前，先列出将创建或修改的文件及各自职责。分解决策在此定型。

- 设计边界清晰、接口明确的单元。每个文件职责单一明确。  
- 你能最好地推理「一次能装入上下文」的代码，文件聚焦时编辑更可靠。优先小而专的文件，而非大包大揽。  
- 经常一起变的文件应放在一起。按职责拆分，而非按技术层次硬拆。  
- 在现有代码库中遵循既有模式。若库中惯用大文件，不要单方面大重构 — 但若你要改的文件已难以维护，在计划中包含拆分是合理的。  

该结构指导任务分解。每个任务应产生相对独立、自洽的变更。

## 任务粒度

**每一步是一个动作（约 2–5 分钟）：**
- 「写失败测试」— 一步  
- 「运行确认失败」— 一步  
- 「写最少代码使测试通过」— 一步  
- 「运行测试确认通过」— 一步  
- 「Commit」— 一步  

## 计划文档头部

**每份计划必须以该头部开头：**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## 任务结构

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## 记住
- 始终写确切文件路径  
- 计划中写完整代码（不要只写「加校验」）  
- 写确切命令与期望输出  
- 用 @ 语法引用相关 skill  
- DRY、YAGNI、TDD、频繁 commit  

## 计划评审循环

写完完整计划后：

1. 派发**单个** plan-document-reviewer subagent（见 plan-document-reviewer-prompt.md），附上精心编写的评审上下文 — 不要用本会话历史。使评审聚焦计划而非你的思路。  
   - 提供：计划文档路径、spec 文档路径  
2. 若 ❌ 发现问题：修复后**整份计划**再派发评审  
3. 若 ✅ 通过：进入执行交接  

**评审循环指引：**
- 同一份写计划的 agent 负责修改（保留上下文）  
- 若循环超过 3 轮，交由人类决策  
- 评审是建议性的 — 若你认为反馈不对，说明分歧  

## 执行交接

保存计划后，提供执行方式选择：

**「计划已完成并保存到 `docs/superpowers/plans/<filename>.md`。两种执行方式：**

**1. Subagent-Driven（推荐）** — 每任务派发全新 subagent，任务间评审，迭代快  

**2. Inline Execution** — 本会话用 executing-plans 执行，分批带检查点  

**选哪种？」**

**若选 Subagent-Driven：**
- **必选子 skill：** superpowers:subagent-driven-development  
- 每任务全新 subagent + 两阶段评审  

**若选 Inline Execution：**
- **必选子 skill：** superpowers:executing-plans  
- 分批执行，带评审检查点  
