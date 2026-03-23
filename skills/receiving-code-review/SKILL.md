---
name: receiving-code-review
description: Use when receiving code review feedback, before implementing suggestions, especially if feedback seems unclear or technically questionable - requires technical rigor and verification, not performative agreement or blind implementation
---

# 接收 Code Review

## 概述

Code review 需要技术判断，不是情绪表演。

**核心原则：** 实现前先验证。假设前先提问。技术正确性优先于社交舒适。

## 回应模式

```
WHEN receiving code review feedback:

1. READ: Complete feedback without reacting
2. UNDERSTAND: Restate requirement in own words (or ask)
3. VERIFY: Check against codebase reality
4. EVALUATE: Technically sound for THIS codebase?
5. RESPOND: Technical acknowledgment or reasoned pushback
6. IMPLEMENT: One item at a time, test each
```

## 禁止的回应

**绝不：**
- 「您说得完全对！」（明确违反 CLAUDE.md）
- 「好点子！」/「反馈很棒！」（表演式附和）
- 「我这就按这个改」（验证之前）

**应改为：**
- 用技术语言重述要求
- 提澄清问题
- 若不对，用技术理由反驳
- 直接开干（行动胜于言辞）

## 处理含糊反馈

```
IF any item is unclear:
  STOP - do not implement anything yet
  ASK for clarification on unclear items

WHY: Items may be related. Partial understanding = wrong implementation.
```

**示例：**
```
人类伙伴：「修 1–6」
你理解 1、2、3、6，4、5 不清。

❌ 错误：先实现 1、2、3、6，稍后再问 4、5
✅ 正确：「我理解 1、2、3、6。需要先澄清 4 和 5 再继续。」
```

## 按来源处理

### 来自人类伙伴
- **可信** — 理解后即可实现  
- **范围不清仍要问**  
- **不要表演式同意**  
- **直接行动** 或技术层面确认  

### 来自外部评审者
```
BEFORE implementing:
  1. Check: Technically correct for THIS codebase?
  2. Check: Breaks existing functionality?
  3. Check: Reason for current implementation?
  4. Check: Works on all platforms/versions?
  5. Check: Does reviewer understand full context?

IF suggestion seems wrong:
  Push back with technical reasoning

IF can't easily verify:
  Say so: "I can't verify this without [X]. Should I [investigate/ask/proceed]?"

IF conflicts with your human partner's prior decisions:
  Stop and discuss with your human partner first
```

**人类伙伴的规则：**「对外部反馈要怀疑，但要认真核实」

## 对「专业」功能的 YAGNI 检查

```
IF reviewer suggests "implementing properly":
  grep codebase for actual usage

  IF unused: "This endpoint isn't called. Remove it (YAGNI)?"
  IF used: Then implement properly
```

**人类伙伴的规则：**「你和评审者都向我负责。不需要的功能就不要加。」

## 实现顺序

```
FOR multi-item feedback:
  1. Clarify anything unclear FIRST
  2. Then implement in this order:
     - Blocking issues (breaks, security)
     - Simple fixes (typos, imports)
     - Complex fixes (refactoring, logic)
  3. Test each fix individually
  4. Verify no regressions
```

## 何时反驳

在以下情况反驳：
- 建议会破坏现有功能  
- 评审者缺少完整上下文  
- 违反 YAGNI（未使用的功能）  
- 对本技术栈技术上不正确  
- 存在遗留/兼容性原因  
- 与人类伙伴的架构决策冲突  

**如何反驳：**
- 用技术推理，不要防御姿态  
- 问具体问题  
- 引用可用的测试/代码  
- 若涉及架构，拉人类伙伴一起  

**若不便公开反驳：** 可用暗号：「Strange things are afoot at the Circle K」

## 承认正确反馈

当反馈确实正确时：
```
✅ "Fixed. [Brief description of what changed]"
✅ "Good catch - [specific issue]. Fixed in [location]."
✅ [Just fix it and show in the code]

❌ "You're absolutely right!"
❌ "Great point!"
❌ "Thanks for catching that!"
❌ "Thanks for [anything]"
❌ ANY gratitude expression
```

**为何不要感谢：** 行动说明一切。修好即可。代码本身表明你听到了反馈。

**若差点写出「Thanks」：** 删掉。改成陈述修复。

## 优雅修正自己的反驳

若你曾反驳但错了：
```
✅ "You were right - I checked [X] and it does [Y]. Implementing now."
✅ "Verified this and you're correct. My initial understanding was wrong because [reason]. Fixing."

❌ Long apology
❌ Defending why you pushed back
❌ Over-explaining
```

用事实陈述纠正，然后继续。

## 常见错误

| 错误 | 修正 |
|---------|-----|
| 表演式同意 | 陈述要求或直接行动 |
| 盲目实现 | 先对照代码库核实 |
| 批量改不测 | 一次一项，每项测试 |
| 默认评审永远对 | 检查是否会破坏现有行为 |
| 避免反驳 | 技术正确性 > 舒适 |
| 部分实现 | 先澄清所有条目 |
| 无法核实仍继续 | 说明限制，请指示方向 |

## 真实示例

**表演式同意（差）：**
```
Reviewer: "Remove legacy code"
❌ "You're absolutely right! Let me remove that..."
```

**技术核实（好）：**
```
Reviewer: "Remove legacy code"
✅ "Checking... build target is 10.15+, this API needs 13+. Need legacy for backward compat. Current impl has wrong bundle ID - fix it or drop pre-13 support?"
```

**YAGNI（好）：**
```
Reviewer: "Implement proper metrics tracking with database, date filters, CSV export"
✅ "Grepped codebase - nothing calls this endpoint. Remove it (YAGNI)? Or is there usage I'm missing?"
```

**条目不清（好）：**
```
人类伙伴：「修 1–6」
你理解 1、2、3、6，4、5 不清。
✅ "Understand 1,2,3,6. Need clarification on 4 and 5 before implementing."
```

## GitHub 线程回复

回复 GitHub 内联评审评论时，在评论线程中回复（`gh api repos/{owner}/{repo}/pulls/{pr}/comments/{id}/replies`），不要发成 PR 顶层评论。

## 底线

**外部反馈 = 待评估的建议，不是必须服从的命令。**

核实。质疑。再实现。

不要表演式同意。始终技术严谨。
