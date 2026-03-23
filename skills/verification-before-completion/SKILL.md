---
name: verification-before-completion
description: Use when about to claim work is complete, fixed, or passing, before committing or creating PRs - requires running verification commands and confirming output before making any success claims; evidence before assertions always
---

# 完成前验证

## 概述

未经验证就宣称完成，是失信而非高效。

**核心原则：** 先有证据，再下结论。始终如此。

**违反本规则的字面规定，就是违反本规则的精神。**

## 铁律

```
NO COMPLETION CLAIMS WITHOUT FRESH VERIFICATION EVIDENCE
```

若本回合消息里尚未运行验证命令，不得声称通过。

## 门禁函数

```
BEFORE claiming any status or expressing satisfaction:

1. IDENTIFY: What command proves this claim?
2. RUN: Execute the FULL command (fresh, complete)
3. READ: Full output, check exit code, count failures
4. VERIFY: Does output confirm the claim?
   - If NO: State actual status with evidence
   - If YES: State claim WITH evidence
5. ONLY THEN: Make the claim

Skip any step = lying, not verifying
```

## 常见失败

| 宣称 | 需要 | 不足够 |
|-------|----------|--------------|
| 测试通过 | 测试命令输出：0 失败 | 上次跑过、「应该过」 |
| Linter 干净 | Linter 输出：0 错误 | 部分检查、外推 |
| 构建成功 | 构建命令：exit 0 | Linter 通过、日志「看起来 OK」 |
| Bug 已修 | 原症状测试：通过 | 代码改了、假定已好 |
| 回归测试有效 | 已验证 RED-GREEN 循环 | 测试只通过一次 |
| Agent 已完成 | VCS diff 显示变更 | Agent 报告「成功」 |
| 需求已满足 | 逐条核对清单 | 仅测试通过 |

## 危险信号 — 停

- 使用「应该」「大概」「看起来」  
- 验证前就表达满意（「太好了！」「完美！」「搞定！」等）  
- 未验证就要 commit/push/PR  
- 轻信 agent 成功报告  
- 依赖部分验证  
- 心想「就这一次」  
- 累了想赶紧结束  
- **任何未运行验证却暗示成功的措辞**  

## 防止自我辩解

| 借口 | 现实 |
|--------|---------|
| 「现在应该好了」 | **运行**验证命令 |
| 「我有信心」 | 信心 ≠ 证据 |
| 「就这一次」 | 无例外 |
| 「Linter 过了」 | Linter ≠ 编译器 |
| 「Agent 说成功」 | 独立再验 |
| 「我累了」 | 累不是借口 |
| 「部分检查够了」 | 部分证明不了什么 |
| 「换种说法就不算违规」 | 精神重于字面 |

## 关键模式

**测试：**
```
✅ [Run test command] [See: 34/34 pass] "All tests pass"
❌ "Should pass now" / "Looks correct"
```

**回归测试（TDD RED-GREEN）：**
```
✅ Write → Run (pass) → Revert fix → Run (MUST FAIL) → Restore → Run (pass)
❌ "I've written a regression test" (without red-green verification)
```

**构建：**
```
✅ [Run build] [See: exit 0] "Build passes"
❌ "Linter passed" (linter doesn't check compilation)
```

**需求：**
```
✅ Re-read plan → Create checklist → Verify each → Report gaps or completion
❌ "Tests pass, phase complete"
```

**委派给 agent：**
```
✅ Agent reports success → Check VCS diff → Verify changes → Report actual state
❌ Trust agent report
```

## 为何重要

来自 24 条失败记忆：
- 人类伙伴说「我不信你」— 信任破裂  
- 未定义函数上线 — 会崩溃  
- 缺需求就上线 — 功能不完整  
- 假完成浪费时间 → 纠正 → 返工  
- 违背：「诚实是核心价值。撒谎就会被换掉。」  

## 何时适用

**在以下之前就应适用：**
- **任何**成功/完成类宣称的变体  
- **任何**满意表达  
- **任何**对工作状态的正面陈述  
- commit、创建 PR、任务完成  
- 进入下一任务  
- 再委派给 agent  

**规则适用于：**
- 原话  
- 转述与同义词  
- 暗示成功  
- **任何**暗示完成/正确的沟通  

## 底线

**验证没有捷径。**

运行命令。读输出。**再**下结论。

不容讨价还价。
