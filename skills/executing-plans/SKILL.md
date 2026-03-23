---
name: executing-plans
description: Use when you have a written implementation plan to execute in a separate session with review checkpoints
---

# 执行计划

## 概述

加载计划、批判性审阅、执行全部任务、完成后汇报。

**开始时声明：**「我正在使用 executing-plans skill 来实现本计划。」

**说明：** 请告诉人类伙伴，Superpowers 在能使用 subagent 的平台上效果好得多。若在支持 subagent 的平台（如 Claude Code 或 Codex）上运行，工作质量会明显更高。若可用 subagent，请用 superpowers:subagent-driven-development，而不要用本 skill。

## 流程

### 第 1 步：加载并审阅计划
1. 阅读计划文件  
2. 批判性审阅 — 标出对计划的疑问或顾虑  
3. 若有顾虑：开始前先与人类伙伴沟通  
4. 若无顾虑：创建 TodoWrite 并继续  

### 第 2 步：执行任务

对每个任务：
1. 标为 in_progress  
2. 严格按计划中的每一步执行（计划应为小步）  
3. 按计划运行验证  
4. 标为 completed  

### 第 3 步：完成开发

所有任务完成并验证后：
- 声明：「我正在使用 finishing-a-development-branch skill 来完成本工作。」
- **必选子 skill：** 使用 superpowers:finishing-a-development-branch  
- 按该 skill 验证测试、呈现选项、执行所选路径  

## 何时停止并求助

**出现以下情况立即停止执行：**
- 遇到阻塞（缺依赖、测试失败、指令不清）
- 计划有关键缺口无法开工
- 不理解某条指令
- 验证反复失败

**宁可澄清也不要猜。**

## 何时回到更早步骤

**回到审阅（第 1 步）当：**
- 伙伴根据你的反馈更新了计划
- 根本做法需要重新考虑

**不要硬闯阻塞** — 停下并提问。

## 记住
- 先批判性审阅计划  
- 严格按计划步骤执行  
- 不要跳过验证  
- 计划要求时引用对应 skill  
- 受阻时停下，不要猜  
- 未经用户明确同意，不要在 main/master 分支上开始实现  

## 集成

**所需工作流 skill：**
- **superpowers:using-git-worktrees** — 必选：开始前建立隔离工作区  
- **superpowers:writing-plans** — 生成本 skill 所执行的计划  
- **superpowers:finishing-a-development-branch** — 全部任务完成后收尾开发  
