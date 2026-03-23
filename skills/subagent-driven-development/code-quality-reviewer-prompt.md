# 代码质量审阅者 Prompt 模板

派发代码质量审阅 subagent 时使用本模板。

**目的：** 确认实现质量过硬（清晰、有测试、可维护）

**仅在规格符合性审阅通过后再派发。**

```
Task tool (superpowers:code-reviewer):
  Use template at requesting-code-review/code-reviewer.md

  WHAT_WAS_IMPLEMENTED: [from implementer's report]
  PLAN_OR_REQUIREMENTS: Task N from [plan-file]
  BASE_SHA: [commit before task]
  HEAD_SHA: [current commit]
  DESCRIPTION: [task summary]
```

**除常规代码质量外，审阅者还应检查：**
- 每个文件是否职责单一、接口是否清晰？
- 单元是否拆到可独立理解与测试？
- 实现是否遵循计划中的文件结构？
- 本次实现是否新增已很大的文件，或显著撑大既有文件？（勿因历史体积挑刺——聚焦本改动带来的部分。）

**代码审阅者返回：** Strengths、Issues（Critical/Important/Minor）、Assessment
