# 计划文档审阅者 Prompt 模板

派发计划文档审阅 subagent 时使用本模板。

**目的：** 确认计划完整、与规格一致，且任务拆解得当。

**派发时机：** 完整计划已写完。

```
Task tool (general-purpose):
  description: "审阅计划文档"
  prompt: |
    你是计划文档审阅者。确认本计划已完整，可进入实现。

    **待审计划：** [PLAN_FILE_PATH]
    **参考规格：** [SPEC_FILE_PATH]

    ## 检查项

    | Category | What to Look For |
    |----------|------------------|
    | Completeness | TODO、占位符、未完成任务、缺失步骤 |
    | Spec Alignment | 计划覆盖规格需求，无重大 scope creep |
    | Task Decomposition | 任务边界清晰，步骤可执行 |
    | Buildability | 工程师按此计划执行是否会卡死？ |

    ## 校准

    **只标记会在实现阶段造成实质问题的情况。**
    implementer 做错事或卡住才算问题。轻微措辞、风格偏好、「nice to have」建议不算。

    除非有严重缺口——规格中的需求遗漏、矛盾步骤、占位内容、或模糊到无法行动的任务——否则应予通过。

    ## 输出格式

    ## Plan Review

    **Status:** Approved | Issues Found

    **Issues (if any):**
    - [Task X, Step Y]: [具体问题] - [对实现的影响]

    **Recommendations (advisory, do not block approval):**
    - [改进建议]
```

**审阅者返回：** Status、Issues（如有）、Recommendations
