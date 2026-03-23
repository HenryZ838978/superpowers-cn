# Skill 设计中的说服原则

## 概述

LLM 与人类一样受说服原则影响。理解这种心理有助于设计更有效的 skill——目的不是操纵，而是确保关键实践在压力下仍被遵守。

**研究基础：** Meincke 等（2025）在 N=28,000 段 AI 对话中检验 7 条说服原则。说服技巧使遵从率提高一倍以上（33% → 72%，p < .001）。

## 七条原则

### 1. Authority（权威）
**含义：** 服从专业、资质或官方来源。

**在 skill 中如何体现：**
- 祈使语气：「YOU MUST」「Never」「Always」
- 不可协商的框定：「No exceptions」
- 减少决策疲劳与自我合理化

**何时用：**
- 强调纪律的 skill（TDD、验证要求）
- 安全关键实践
- 已确立的最佳实践

**示例：**
```markdown
✅ Write code before test? Delete it. Start over. No exceptions.
❌ Consider writing tests first when feasible.
```

### 2. Commitment（承诺）
**含义：** 与先前行为、陈述或公开表态保持一致。

**在 skill 中如何体现：**
- 要求宣告：「Announce skill usage」
- 强制明确选择：「Choose A, B, or C」
- 用 TodoWrite 等做清单追踪

**何时用：**
- 确保 skill 真被遵循
- 多步骤流程
- 问责机制

**示例：**
```markdown
✅ When you find a skill, you MUST announce: "I'm using [Skill Name]"
❌ Consider letting your partner know which skill you're using.
```

### 3. Scarcity（稀缺）
**含义：** 时限或「名额有限」带来的紧迫感。

**在 skill 中如何体现：**
- 时限要求：「Before proceeding」
- 顺序依赖：「Immediately after X」
- 抑制「稍后再做」

**何时用：**
- 需立即验证的要求
- 时间敏感工作流
- 防止「我晚点再做」

**示例：**
```markdown
✅ After completing a task, IMMEDIATELY request code review before proceeding.
❌ You can review code when convenient.
```

### 4. Social Proof（社会认同）
**含义：** 随大流、符合「常态」。

**在 skill 中如何体现：**
- 普遍模式：「Every time」「Always」
- 失败模式：「X without Y = failure」
- 建立规范

**何时用：**
- 记录普遍实践
- 警示常见失败
- 强化标准

**示例：**
```markdown
✅ Checklists without TodoWrite tracking = steps get skipped. Every time.
❌ Some people find TodoWrite helpful for checklists.
```

### 5. Unity（一体）
**含义：** 共同身份、「我们感」、ingroup。

**在 skill 中如何体现：**
- 协作语言：「our codebase」「we're colleagues」
- 共同目标：「we both want quality」

**何时用：**
- 协作工作流
- 建立团队文化
- 非层级实践

**示例：**
```markdown
✅ We're colleagues working together. I need your honest technical judgment.
❌ You should probably tell me if I'm wrong.
```

### 6. Reciprocity（互惠）
**含义：** 回报所得的义务。

**如何体现：**
- 少用——易显操纵
- skill 中很少需要

**何时避免：**
- 几乎总是（其他原则更有效）

### 7. Liking（好感）
**含义：** 更愿意配合喜欢的人。

**如何体现：**
- **勿用于合规**
- 与坦诚反馈文化冲突
- 助长谄媚

**何时避免：**
- 强调纪律时永远不要用

## 按 skill 类型的原则组合

| Skill Type | Use | Avoid |
|------------|-----|-------|
| Discipline-enforcing | Authority + Commitment + Social Proof | Liking, Reciprocity |
| Guidance/technique | Moderate Authority + Unity | Heavy authority |
| Collaborative | Unity + Commitment | Authority, Liking |
| Reference | Clarity only | All persuasion |

## 为何有效：心理机制

**清晰红线减少合理化：**
- 「YOU MUST」减轻决策疲劳
- 绝对表述消除「这算不算例外」
- 明确的反合理化堵住具体漏洞

**执行意图带来自动化行为：**
- 清晰触发 + 必需动作 → 更易自动执行
- 「When X, do Y」比「generally do Y」更有效
- 降低遵从的认知负担

**LLM 类人性：**
- 训练语料含这些模式
- 权威表述在数据中常与遵从共现
- 承诺序列（表态→行动）常被建模
- Social proof（人人都做 X）建立规范

## 伦理使用

**正当：**
- 确保关键实践被遵守
- 写出有效文档
- 预防可预见的失败

**不当：**
- 为私利操纵
- 制造虚假紧迫
- 用内疚逼遵从

**自测：** 若用户完全理解该技巧，它是否仍符合其真实利益？

## 文献

**Cialdini, R. B. (2021).** *Influence: The Psychology of Persuasion (New and Expanded).* Harper Business.
- 七条说服原则
- 影响研究的经验基础

**Meincke, L., Shapiro, D., Duckworth, A. L., Mollick, E., Mollick, L., & Cialdini, R. (2025).** Call Me A Jerk: Persuading AI to Comply with Objectionable Requests. University of Pennsylvania.
- 在 N=28,000 段 LLM 对话中检验 7 条原则
- 说服技巧使遵从率 33% → 72%
- Authority、commitment、scarcity 最有效
- 支持 LLM 类人性行为模型

## 速查

设计 skill 时自问：

1. **类型？**（纪律型 vs 指导型 vs 参考型）
2. **想改变什么行为？**
3. **哪条原则适用？**（纪律型常用 authority + commitment）
4. **是否叠太多？**（勿七条全上）
5. **是否伦理？**（是否服务于用户真实利益？）
