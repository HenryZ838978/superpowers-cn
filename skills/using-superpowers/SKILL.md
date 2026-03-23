---
name: using-superpowers
description: Use when starting any conversation - establishes how to find and use skills, requiring Skill tool invocation before ANY response including clarifying questions
---

<SUBAGENT-STOP>
若你是作为 subagent 被派发来执行特定任务，跳过本 skill。
</SUBAGENT-STOP>

<EXTREMELY-IMPORTANT>
若你认为某 skill 有哪怕 1% 可能适用于当前工作，**绝对必须**调用该 skill。

若 skill 适用于你的任务，**你没有选择**，**必须使用**。

不容讨价还价。不是可选项。不能自圆其说跳过。
</EXTREMELY-IMPORTANT>

## 指令优先级

Superpowers skill 覆盖默认系统提示行为，但**用户指令永远优先**：

1. **用户明确指令**（CLAUDE.md、GEMINI.md、AGENTS.md、直接要求）— 最高  
2. **Superpowers skill** — 与默认系统行为冲突时覆盖默认  
3. **默认系统提示** — 最低  

若 CLAUDE.md、GEMINI.md 或 AGENTS.md 写「不要用 TDD」，而 skill 写「始终用 TDD」，遵循用户指令。用户说了算。

## 如何访问 Skill

**在 Claude Code：** 使用 `Skill` 工具。调用 skill 后，其内容会加载给你——直接遵循。不要用 Read 工具读 skill 文件。

**在 Gemini CLI：** Skill 通过 `activate_skill` 工具激活。Gemini 在会话开始时加载 skill 元数据，按需加载完整内容。

**在其他环境：** 查阅平台文档了解 skill 如何加载。

## 平台适配

Skill 使用 Claude Code 工具名。非 CC 平台：见 `references/codex-tools.md`（Codex）了解工具对应关系。Gemini CLI 用户通过 GEMINI.md 自动获得工具映射。

# 使用 Skill

## 规则

**在回应或行动之前调用相关或被请求的 skill。** 哪怕只有 1% 可能适用，也应调用以确认。若调用后发现不适用，可不必按其执行。

```dot
digraph skill_flow {
    "User message received" [shape=doublecircle];
    "About to EnterPlanMode?" [shape=doublecircle];
    "Already brainstormed?" [shape=diamond];
    "Invoke brainstorming skill" [shape=box];
    "Might any skill apply?" [shape=diamond];
    "Invoke Skill tool" [shape=box];
    "Announce: 'Using [skill] to [purpose]'" [shape=box];
    "Has checklist?" [shape=diamond];
    "Create TodoWrite todo per item" [shape=box];
    "Follow skill exactly" [shape=box];
    "Respond (including clarifications)" [shape=doublecircle];

    "About to EnterPlanMode?" -> "Already brainstormed?";
    "Already brainstormed?" -> "Invoke brainstorming skill" [label="no"];
    "Already brainstormed?" -> "Might any skill apply?" [label="yes"];
    "Invoke brainstorming skill" -> "Might any skill apply?";

    "User message received" -> "Might any skill apply?";
    "Might any skill apply?" -> "Invoke Skill tool" [label="yes, even 1%"];
    "Might any skill apply?" -> "Respond (including clarifications)" [label="definitely not"];
    "Invoke Skill tool" -> "Announce: 'Using [skill] to [purpose]'";
    "Announce: 'Using [skill] to [purpose]'" -> "Has checklist?";
    "Has checklist?" -> "Create TodoWrite todo per item" [label="yes"];
    "Has checklist?" -> "Follow skill exactly" [label="no"];
    "Create TodoWrite todo per item" -> "Follow skill exactly";
}
```

## 危险信号

以下想法表示**停**——你在自我辩解：

| 想法 | 现实 |
|---------|---------|
| 「这只是个简单问题」 | 提问也是任务。查 skill。 |
| 「我需要先更多上下文」 | Skill 检查在澄清问题**之前**。 |
| 「我先快速看代码库」 | Skill 告诉你**如何**探索。先查。 |
| 「我可以快速看 git/文件」 | 文件缺少对话上下文。查 skill。 |
| 「我先收集信息」 | Skill 告诉你**如何**收集。 |
| 「这不需要正式 skill」 | 若有 skill，就要用。 |
| 「我记得这个 skill」 | Skill 会演进。读当前版本。 |
| 「这不算任务」 | 行动 = 任务。查 skill。 |
| 「Skill 小题大做」 | 简单事也会变复杂。要用。 |
| 「我先做这一件小事」 | **任何**事前先检查。 |
| 「这样感觉很高效」 | 无纪律的行动浪费时间。Skill 防止如此。 |
| 「我知道那是什么意思」 | 知道概念 ≠ 使用 skill。要调用。 |

## Skill 优先级

多个 skill 可能适用时，顺序如下：

1. **流程类 skill 优先**（brainstorming、debugging）— 决定**如何**接近任务  
2. **实现类 skill 其次**（frontend-design、mcp-builder）— 指导**如何**执行  

「我们来做 X」→ 先 brainstorming，再实现类 skill。  
「修这个 bug」→ 先 debugging，再领域相关 skill。

## Skill 类型

**刚性**（TDD、debugging）：严格遵循。不要削弱纪律。

**灵活**（模式）：按上下文调整原则。

Skill 正文会说明属于哪一类。

## 用户指令

指令说**做什么**，不说**如何**做。「加 X」或「修 Y」不表示可以跳过工作流。
