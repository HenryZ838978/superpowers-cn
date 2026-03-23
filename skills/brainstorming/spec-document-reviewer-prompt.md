# 规格文档审阅者 Prompt 模板

派发规格文档审阅 subagent 时使用本模板。

**目的：** 确认规格完整、一致，且已可进入实现规划。

**派发时机：** 规格文档已写入 `docs/superpowers/specs/`

```
Task tool (general-purpose):
  description: "审阅规格文档"
  prompt: |
    你是规格文档审阅者。确认本规格已完整，可进入规划阶段。

    **待审规格：** [SPEC_FILE_PATH]

    ## 检查项

    | Category | What to Look For |
    |----------|------------------|
    | Completeness | TODO、占位符、「待定」、未完成章节 |
    | Consistency | 内部矛盾、冲突需求 |
    | Clarity | 需求是否模糊到可能让人做错产品 |
    | Scope | 是否聚焦在单一计划内——而非横跨多个独立子系统 |
    | YAGNI | 未要求的功能、过度设计 |

    ## 校准

    **只标记会在实现规划阶段造成实质问题的情况。**
    缺失章节、矛盾、或可被两种不同方式理解的需求——才算问题。轻微措辞、风格偏好、
    「某节不如其他节细」不算。

    除非存在会导致计划严重偏差的缺口，否则应予通过（Approved）。

    ## 输出格式

    ## Spec Review

    **Status:** Approved | Issues Found

    **Issues (if any):**
    - [Section X]: [具体问题] - [对规划的影响]

    **Recommendations (advisory, do not block approval):**
    - [改进建议]
```

**审阅者返回：** Status、Issues（如有）、Recommendations
