# Skill 撰写最佳实践

> 学习如何撰写能被 Claude 发现并成功使用的 Skill。

优秀的 Skill 简洁、结构清晰，并经真实使用验证。本指南给出可操作的撰写决策，帮助你写出 Claude 能有效发现与使用的 Skill。

关于 Skill 如何运作的概念背景，见 [Skills overview](/en/docs/agents-and-tools/agent-skills/overview)。

## 核心原则

### 简洁至上

[上下文窗口](https://platform.claude.com/docs/en/build-with-claude/context-windows)是公共资源。你的 Skill 与 Claude 还需掌握的一切共享该窗口，包括：

* 系统提示
* 对话历史
* 其他 Skill 的元数据
* 用户的实际请求

并非 Skill 里每个 token 都会立即计费。启动时只会预加载所有 Skill 的元数据（name 与 description）。Claude 仅在 Skill 相关时读取 SKILL.md，并按需读取其他文件。但 SKILL.md 仍宜简洁：一旦被加载，每个 token 都会与对话历史及其他上下文竞争。

**默认假设**：Claude 已经足够聪明

只补充 Claude 尚不具备的上下文。对每条信息自问：

* "Claude 真的需要这段解释吗？"
* "能否假设 Claude 已经知道？"
* "这段话是否对得起它的 token 成本？"

**好例子：简洁**（约 50 tokens）：

````markdown  theme={null}
## Extract PDF text

Use pdfplumber for text extraction:

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
````

**坏例子：冗长**（约 150 tokens）：

```markdown  theme={null}
## Extract PDF text

PDF (Portable Document Format) files are a common file format that contains
text, images, and other content. To extract text from a PDF, you'll need to
use a library. There are many libraries available for PDF processing, but we
recommend pdfplumber because it's easy to use and handles most cases well.
First, you'll need to install it using pip. Then you can use the code below...
```

简洁版默认 Claude 知道 PDF 是什么以及如何使用库。

### 设定合适的自由度

具体程度要与任务的脆弱性、可变性相匹配。

**高自由度**（纯文字说明）：

适用于：

* 多种做法都合理
* 决策依赖上下文
* 靠启发式指引路径

示例：

```markdown  theme={null}
## Code review process

1. Analyze the code structure and organization
2. Check for potential bugs or edge cases
3. Suggest improvements for readability and maintainability
4. Verify adherence to project conventions
```

**中自由度**（带参数的伪代码或脚本）：

适用于：

* 存在首选模式
* 允许一定变化
* 配置影响行为

示例：

````markdown  theme={null}
## Generate report

Use this template and customize as needed:

```python
def generate_report(data, format="markdown", include_charts=True):
    # Process data
    # Generate output in specified format
    # Optionally include visualizations
```
````

**低自由度**（固定脚本，参数很少或没有）：

适用于：

* 操作脆弱、易错
* 一致性至关重要
* 必须按固定顺序执行

示例：

````markdown  theme={null}
## Database migration

Run exactly this script:

```bash
python scripts/migrate.py --verify --backup
```

Do not modify the command or add additional flags.
````

**类比**：把 Claude 想成在探路的机器人：

* **两侧是悬崖的窄桥**：只有一条安全路径。给出明确护栏与精确指令（低自由度）。例：必须按固定顺序执行的 database migration。
* **无障碍的开阔地**：多条路都能成功。给大方向，相信 Claude 能选最佳路线（高自由度）。例：code review，最佳做法由上下文决定。

### 用你计划使用的所有模型来测

Skill 依附于模型之上，效果取决于底层模型。请用你计划搭配使用的每一个模型测试 Skill。

**按模型考虑的测试点**：

* **Claude Haiku**（快、省）：Skill 是否给出足够指引？
* **Claude Sonnet**（均衡）：Skill 是否清晰高效？
* **Claude Opus**（强推理）：Skill 是否避免过度解释？

对 Opus 完美的说明可能对 Haiku 需要更多细节。若 Skill 要跨多模型使用，应追求对各模型都友好的指令。

## Skill 结构

<Note>
  **YAML Frontmatter**：SKILL.md 的 frontmatter 仅支持两个字段：

  * `name` - Skill 的人类可读名称（最多 64 字符）
  * `description` - 一行说明 Skill 做什么、何时使用（最多 1024 字符）

  完整结构见 [Skills overview](/en/docs/agents-and-tools/agent-skills/overview#skill-structure)。
</Note>

### 命名惯例

用一致模式命名，便于引用与讨论。建议 Skill 名用**动名词形式**（动词 + -ing），直接描述活动或能力。

**动名词好例子**：

* "Processing PDFs"
* "Analyzing spreadsheets"
* "Managing databases"
* "Testing code"
* "Writing documentation"

**也可接受**：

* 名词短语："PDF Processing"、"Spreadsheet Analysis"
* 动作导向："Process PDFs"、"Analyze Spreadsheets"

**避免**：

* 模糊名："Helper"、"Utils"、"Tools"
* 过于泛化："Documents"、"Data"、"Files"
* 技能库内命名风格不统一

一致命名有助于：

* 在文档与对话中引用 Skill
* 一眼看懂 Skill 用途
* 整理与检索多个 Skill
* 保持专业、统一的 skill 库

### 撰写有效的 description

`description` 决定 Skill 能否被检索到，应同时说明**做什么**与**何时用**。

<Warning>
  **始终用第三人称**。description 会注入系统提示，人称不一致会导致检索问题。

  * **好：** "Processes Excel files and generates reports"
  * **避免：** "I can help you process Excel files"
  * **避免：** "You can use this to process Excel files"
</Warning>

**要具体，并包含关键词**。既写 Skill 的行为，也写触发使用的场景/上下文。

每个 Skill 只有一个 description。它对 skill 选择至关重要：Claude 可能从 100+ 个 Skill 里挑选，你的 description 必须足够具体，让 Claude 知道何时选它；SKILL.md 其余部分再承载实现细节。

有效示例：

**PDF 处理类 skill：**

```yaml  theme={null}
description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
```

**Excel 分析类 skill：**

```yaml  theme={null}
description: Analyze Excel spreadsheets, create pivot tables, generate charts. Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.
```

**Git commit 辅助类 skill：**

```yaml  theme={null}
description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
```

避免如下模糊 description：

```yaml  theme={null}
description: Helps with documents
```

```yaml  theme={null}
description: Processes data
```

```yaml  theme={null}
description: Does stuff with files
```

### 渐进式披露模式

SKILL.md 作为总览，按需指向详细材料，类似入职指南的目录。渐进式披露的原理见 overview 中 [How Skills work](/en/docs/agents-and-tools/agent-skills/overview#how-skills-work)。

**实践建议：**

* SKILL.md 正文宜控制在 500 行以内以利性能
* 接近上限时拆成多个文件
* 用下文模式组织说明、代码与资源

#### 视觉概览：从简到繁

最简 Skill 只有一个 SKILL.md，内含元数据与说明：

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=87782ff239b297d9a9e8e1b72ed72db9" alt="Simple SKILL.md file showing YAML frontmatter and markdown body" data-og-width="2048" width="2048" data-og-height="1153" height="1153" data-path="images/agent-skills-simple-file.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=c61cc33b6f5855809907f7fda94cd80e 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=90d2c0c1c76b36e8d485f49e0810dbfd 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=ad17d231ac7b0bea7e5b4d58fb4aeabb 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=f5d0a7a3c668435bb0aee9a3a8f8c329 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=0e927c1af9de5799cfe557d12249f6e6 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-simple-file.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=46bbb1a51dd4c8202a470ac8c80a893d 2500w" />

Skill 变大后，可打包额外内容，仅在需要时由 Claude 加载：

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=a5e0aa41e3d53985a7e3e43668a33ea3" alt="Bundling additional reference files like reference.md and forms.md." data-og-width="2048" width="2048" data-og-height="1327" height="1327" data-path="images/agent-skills-bundling-content.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=f8a0e73783e99b4a643d79eac86b70a2 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=dc510a2a9d3f14359416b706f067904a 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=82cd6286c966303f7dd914c28170e385 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=56f3be36c77e4fe4b523df209a6824c6 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=d22b5161b2075656417d56f41a74f3dd 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-bundling-content.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=3dd4bdd6850ffcc96c6c45fcb0acd6eb 2500w" />

完整目录结构可类似：

```
pdf/
├── SKILL.md              # Main instructions (loaded when triggered)
├── FORMS.md              # Form-filling guide (loaded as needed)
├── reference.md          # API reference (loaded as needed)
├── examples.md           # Usage examples (loaded as needed)
└── scripts/
    ├── analyze_form.py   # Utility script (executed, not loaded)
    ├── fill_form.py      # Form filling script
    └── validate.py       # Validation script
```

#### 模式 1：高层指南 + 引用文件

````markdown  theme={null}
---
name: PDF Processing
description: Extracts text and tables from PDF files, fills forms, and merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---

# PDF Processing

## Quick start

Extract text with pdfplumber:
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

## Advanced features

**Form filling**: See [FORMS.md](FORMS.md) for complete guide
**API reference**: See [REFERENCE.md](REFERENCE.md) for all methods
**Examples**: See [EXAMPLES.md](EXAMPLES.md) for common patterns
````

Claude 仅在需要时加载 FORMS.md、REFERENCE.md 或 EXAMPLES.md。

#### 模式 2：按领域组织

多领域 Skill 应按领域拆分，避免加载无关上下文。用户问销售指标时，Claude 只需读销售相关 schema，不必读财务或市场数据，从而降低 token、保持上下文聚焦。

```
bigquery-skill/
├── SKILL.md (overview and navigation)
└── reference/
    ├── finance.md (revenue, billing metrics)
    ├── sales.md (opportunities, pipeline)
    ├── product.md (API usage, features)
    └── marketing.md (campaigns, attribution)
```

````markdown SKILL.md theme={null}
# BigQuery Data Analysis

## Available datasets

**Finance**: Revenue, ARR, billing → See [reference/finance.md](reference/finance.md)
**Sales**: Opportunities, pipeline, accounts → See [reference/sales.md](reference/sales.md)
**Product**: API usage, features, adoption → See [reference/product.md](reference/product.md)
**Marketing**: Campaigns, attribution, email → See [reference/marketing.md](reference/marketing.md)

## Quick search

Find specific metrics using grep:

```bash
grep -i "revenue" reference/finance.md
grep -i "pipeline" reference/sales.md
grep -i "api usage" reference/product.md
```
````

#### 模式 3：按条件展开细节

先写基础内容，高级内容用链接：

```markdown  theme={null}
# DOCX Processing

## Creating documents

Use docx-js for new documents. See [DOCX-JS.md](DOCX-JS.md).

## Editing documents

For simple edits, modify the XML directly.

**For tracked changes**: See [REDLINING.md](REDLINING.md)
**For OOXML details**: See [OOXML.md](OOXML.md)
```

仅在用户需要对应功能时，Claude 才读 REDLINING.md 或 OOXML.md。

### 避免引用链过深

从「被引用的文件」再引用文件时，Claude 可能只部分读取；遇到嵌套引用时，可能用 `head -100` 等命令预览而非读全文件，导致信息不全。

**引用深度以 SKILL.md 为根保持一层**。所有参考文件应能从 SKILL.md 直接链到，以便需要时读完整文件。

**坏例子：过深**：

```markdown  theme={null}
# SKILL.md
See [advanced.md](advanced.md)...

# advanced.md
See [details.md](details.md)...

# details.md
Here's the actual information...
```

**好例子：仅一层**：

```markdown  theme={null}
# SKILL.md

**Basic usage**: [instructions in SKILL.md]
**Advanced features**: See [advanced.md](advanced.md)
**API reference**: See [reference.md](reference.md)
**Examples**: See [examples.md](examples.md)
```

### 长参考文件用目录组织

超过 100 行的参考文件，顶部加目录。即使只做部分预览，Claude 也能看到信息全貌。

**示例**：

```markdown  theme={null}
# API Reference

## Contents
- Authentication and setup
- Core methods (create, read, update, delete)
- Advanced features (batch operations, webhooks)
- Error handling patterns
- Code examples

## Authentication and setup
...

## Core methods
...
```

随后 Claude 可读完整文件或按需跳转章节。

基于文件系统的架构如何实现渐进式披露，见下文 Advanced 中的 [Runtime environment](#runtime-environment)。

## 工作流与反馈环

### 复杂任务使用工作流

把复杂操作拆成清晰、顺序的步骤。特别复杂时，提供清单，让 Claude 可复制到回复中逐项勾选。

**示例 1：研究综合工作流**（无代码类 Skill）：

````markdown  theme={null}
## Research synthesis workflow

Copy this checklist and track your progress:

```
Research Progress:
- [ ] Step 1: Read all source documents
- [ ] Step 2: Identify key themes
- [ ] Step 3: Cross-reference claims
- [ ] Step 4: Create structured summary
- [ ] Step 5: Verify citations
```

**Step 1: Read all source documents**

Review each document in the `sources/` directory. Note the main arguments and supporting evidence.

**Step 2: Identify key themes**

Look for patterns across sources. What themes appear repeatedly? Where do sources agree or disagree?

**Step 3: Cross-reference claims**

For each major claim, verify it appears in the source material. Note which source supports each point.

**Step 4: Create structured summary**

Organize findings by theme. Include:
- Main claim
- Supporting evidence from sources
- Conflicting viewpoints (if any)

**Step 5: Verify citations**

Check that every claim references the correct source document. If citations are incomplete, return to Step 3.
````

该例展示无代码的分析任务如何套用工作流。清单模式适用于任何复杂多步流程。

**示例 2：PDF 表单填写工作流**（含代码的 Skill）：

````markdown  theme={null}
## PDF form filling workflow

Copy this checklist and check off items as you complete them:

```
Task Progress:
- [ ] Step 1: Analyze the form (run analyze_form.py)
- [ ] Step 2: Create field mapping (edit fields.json)
- [ ] Step 3: Validate mapping (run validate_fields.py)
- [ ] Step 4: Fill the form (run fill_form.py)
- [ ] Step 5: Verify output (run verify_output.py)
```

**Step 1: Analyze the form**

Run: `python scripts/analyze_form.py input.pdf`

This extracts form fields and their locations, saving to `fields.json`.

**Step 2: Create field mapping**

Edit `fields.json` to add values for each field.

**Step 3: Validate mapping**

Run: `python scripts/validate_fields.py fields.json`

Fix any validation errors before continuing.

**Step 4: Fill the form**

Run: `python scripts/fill_form.py input.pdf fields.json output.pdf`

**Step 5: Verify output**

Run: `python scripts/verify_output.py output.pdf`

If verification fails, return to Step 2.
````

清晰步骤可避免 Claude 跳过关键校验。清单有助于你与 Claude 跟踪多步进度。

### 实现反馈环

**常见模式**：运行 validator → 修错 → 重复

该模式能明显提升输出质量。

**示例 1：风格指南合规**（无代码类 Skill）：

```markdown  theme={null}
## Content review process

1. Draft your content following the guidelines in STYLE_GUIDE.md
2. Review against the checklist:
   - Check terminology consistency
   - Verify examples follow the standard format
   - Confirm all required sections are present
3. If issues found:
   - Note each issue with specific section reference
   - Revise the content
   - Review the checklist again
4. Only proceed when all requirements are met
5. Finalize and save the document
```

这是用参考文档代替脚本的校验环。「validator」即 STYLE\_GUIDE.md，由 Claude 阅读并对照完成检查。

**示例 2：文档编辑流程**（含代码的 Skill）：

```markdown  theme={null}
## Document editing process

1. Make your edits to `word/document.xml`
2. **Validate immediately**: `python ooxml/scripts/validate.py unpacked_dir/`
3. If validation fails:
   - Review the error message carefully
   - Fix the issues in the XML
   - Run validation again
4. **Only proceed when validation passes**
5. Rebuild: `python ooxml/scripts/pack.py unpacked_dir/ output.docx`
6. Test the output document
```

校验环有助于尽早发现错误。

## 内容准则

### 避免时效性过强的信息

不要写会很快过时的内容：

**坏例子：强时效**（将失效）：

```markdown  theme={null}
If you're doing this before August 2025, use the old API.
After August 2025, use the new API.
```

**好例子**（用「旧模式」区块收纳）：

```markdown  theme={null}
## Current method

Use the v2 API endpoint: `api.example.com/v2/messages`

## Old patterns

<details>
<summary>Legacy v1 API (deprecated 2025-08)</summary>

The v1 API used: `api.example.com/v1/messages`

This endpoint is no longer supported.
</details>
```

「旧模式」区提供历史背景又不干扰正文。

### 术语一致

选定术语后在全 Skill 中统一使用：

**好——一致**：

* 始终用 "API endpoint"
* 始终用 "field"
* 始终用 "extract"

**坏——混用**：

* 混用 "API endpoint"、"URL"、"API route"、"path"
* 混用 "field"、"box"、"element"、"control"
* 混用 "extract"、"pull"、"get"、"retrieve"

一致性能帮助 Claude 理解与遵循说明。

## 常见模式

### 模板模式

提供输出格式模板。严格程度与需求匹配。

**要求严格时**（如 API 响应或数据格式）：

````markdown  theme={null}
## Report structure

ALWAYS use this exact template structure:

```markdown
# [Analysis Title]

## Executive summary
[One-paragraph overview of key findings]

## Key findings
- Finding 1 with supporting data
- Finding 2 with supporting data
- Finding 3 with supporting data

## Recommendations
1. Specific actionable recommendation
2. Specific actionable recommendation
```
````

**需要灵活时**（允许按情况调整）：

````markdown  theme={null}
## Report structure

Here is a sensible default format, but use your best judgment based on the analysis:

```markdown
# [Analysis Title]

## Executive summary
[Overview]

## Key findings
[Adapt sections based on what you discover]

## Recommendations
[Tailor to the specific context]
```

Adjust sections as needed for the specific analysis type.
````

### 示例模式

若输出质量依赖「看过例子」，像常规 prompting 一样提供输入/输出对：

````markdown  theme={null}
## Commit message format

Generate commit messages following these examples:

**Example 1:**
Input: Added user authentication with JWT tokens
Output:
```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware
```

**Example 2:**
Input: Fixed bug where dates displayed incorrectly in reports
Output:
```
fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation
```

**Example 3:**
Input: Updated dependencies and refactored error handling
Output:
```
chore: update dependencies and refactor error handling

- Upgrade lodash to 4.17.21
- Standardize error response format across endpoints
```

Follow this style: type(scope): brief description, then detailed explanation.
````

示例比单纯描述更能帮 Claude 把握期望风格与细节程度。

### 条件分支工作流

引导 Claude 经过决策点：

```markdown  theme={null}
## Document modification workflow

1. Determine the modification type:

   **Creating new content?** → Follow "Creation workflow" below
   **Editing existing content?** → Follow "Editing workflow" below

2. Creation workflow:
   - Use docx-js library
   - Build document from scratch
   - Export to .docx format

3. Editing workflow:
   - Unpack existing document
   - Modify XML directly
   - Validate after each change
   - Repack when complete
```

<Tip>
  若工作流步骤多、体积大，可拆到单独文件，并让 Claude 按当前任务读取对应文件。
</Tip>

## 评估与迭代

### 先做评估

**先建评估，再写大段文档。** 这样 Skill 解决的是真问题，而不是臆想需求。

**评估驱动开发：**

1. **找缺口**：不用 Skill，让 Claude 做代表性任务。记录具体失败或缺失上下文
2. **写评估**：构造三个场景覆盖这些缺口
3. **建基线**：测量无 Skill 时 Claude 的表现
4. **写最小说明**：只写够填坑、通过评估的内容
5. **迭代**：跑评估，对比基线，持续 refine

这样你解决的是实际问题，而非可能永远不会出现的需求。

**评估结构**：

```json  theme={null}
{
  "skills": ["pdf-processing"],
  "query": "Extract all text from this PDF file and save it to output.txt",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "Successfully reads the PDF file using an appropriate PDF processing library or command-line tool",
    "Extracts text content from all pages in the document without missing any pages",
    "Saves the extracted text to a file named output.txt in a clear, readable format"
  ]
}
```

<Note>
  该例展示数据驱动的评估与简单 testing rubric。我们目前不提供内置方式运行这些评估；用户可自建评估体系。评估是衡量 Skill 有效性的事实来源。
</Note>

### 与 Claude 迭代开发 Skill

最有效的 Skill 开发过程离不开 Claude 本身。用一个 Claude 实例（「Claude A」）编写将被其他实例（「Claude B」）使用的 Skill。Claude A 帮你设计与 refine 指令，Claude B 在真实任务中测试。可行是因为 Claude 既懂如何写有效的 agent 指令，也懂 agent 需要哪些信息。

**新建 Skill：**

1. **不用 Skill 完成一次任务**：与 Claude A 用普通 prompting 做完一件事。过程中你会自然提供上下文、偏好与流程知识。注意你**反复**提供了什么信息。

2. **提炼可复用模式**：任务结束后，找出哪些上下文对未来类似任务仍有价值。

   **示例**：若你刚做完 BigQuery 分析，可能提供了表名、字段定义、过滤规则（如 "always exclude test accounts"）和常用查询模式。

3. **请 Claude A 写 Skill**：例如：「把我们刚用的 BigQuery 分析模式做成 Skill，包含表 schema、命名约定和过滤测试账号的规则。」

   <Tip>
     Claude 模型原生理解 Skill 格式与结构。不需要特殊 system prompt 或额外的「writing skills」skill。直接请它创建 Skill，即可生成带合适 frontmatter 与正文的 SKILL.md。
   </Tip>

4. **检查简洁性**：确认 Claude A 没有堆砌多余解释。可要求：「删掉 win rate 含义的解释——Claude 已经知道。」

5. **改进信息架构**：请 Claude A 更有效组织内容。例如：「把表 schema 放到单独 reference 文件，以后可能加更多表。」

6. **在相似任务上测试**：用 Claude B（新会话、已加载 Skill）做相关用例。观察是否找对信息、规则是否用对、任务是否成功。

7. **据观察迭代**：若 Claude B 吃力或遗漏，带着具体现象回到 Claude A：「用这 Skill 时 Q4 忘了按日期过滤，要不要加一节讲日期过滤模式？」

**迭代已有 Skill：**

改进 Skill 时仍用同一层次节奏，在以下之间轮换：

* **与 Claude A 协作**（帮你 refine Skill 的专家）
* **与 Claude B 测试**（用 Skill 干实活的 agent）
* **观察 Claude B** 并把洞见带回 Claude A

1. **在真实工作流里用 Skill**：给 Claude B（已加载 Skill）真实任务，而非虚构测试场景

2. **观察 Claude B**：记下卡点、成功点、意外选择

   **观察示例**：「我要区域销售报表时，它写了查询但忘了过滤测试账号，尽管 Skill 里写了这条规则。」

3. **回到 Claude A 改进**：分享当前 SKILL.md 与观察。例如：「区域报表时 Claude B 忘了过滤测试账号。Skill 里有提过滤，是不是不够显眼？」

4. **评估 Claude A 的建议**：可能建议重组以突出规则、用更强措辞如 "MUST filter" 替代 "always filter"，或调整工作流章节结构。

5. **应用并再测**：按 Claude A 的 refine 更新 Skill，再用 Claude B 在相似请求上测

6. **随使用重复**：遇到新场景就继续观察—refine—测试。每次迭代基于真实 agent 行为，而非假设。

**收集团队反馈：**

1. 与队友共享 Skill 并观察使用
2. 询问：是否在预期时触发？说明是否清晰？还缺什么？
3. 纳入反馈，补上你自己使用中的盲区

**为何有效**：Claude A 懂 agent 需求，你提供领域知识，Claude B 在真实使用中暴露缺口，迭代 refine 基于观察而非臆测。

### 观察 Claude 如何浏览 Skill

迭代时留意 Claude 实际怎么用 Skill：

* **探索路径意外**：阅读顺序是否与你设想不同？可能说明结构不够直观
* **未跟链**：是否没跟到重要文件？链接需更明确或更醒目
* **过度依赖某文件**：若反复读同一文件，考虑是否应并入主 SKILL.md
* **从不打开某文件**：可能多余，或主说明未充分指向它

基于这些观察迭代，而非假设。元数据里的 `name` 与 `description` 尤其关键——Claude 靠它们判断是否应针对当前任务触发 Skill。务必清楚说明 Skill 做什么、何时该用。

## 应避开的反模式

### 避免 Windows 风格路径

路径始终用正斜杠，即使在 Windows 上：

* ✓ **好**：`scripts/helper.py`、`reference/guide.md`
* ✗ **避免**：`scripts\helper.py`、`reference\guide.md`

Unix 风格路径跨平台可用；Windows 风格在 Unix 上会出错。

### 避免提供过多选项

非必要不要并列多种做法：

````markdown  theme={null}
**Bad example: Too many choices** (confusing):
"You can use pypdf, or pdfplumber, or PyMuPDF, or pdf2image, or..."

**Good example: Provide a default** (with escape hatch):
"Use pdfplumber for text extraction:
```python
import pdfplumber
```

For scanned PDFs requiring OCR, use pdf2image with pytesseract instead."
````

## 进阶：含可执行代码的 Skill

以下章节针对包含可执行脚本的 Skill。若 Skill 仅有 markdown 说明，可跳到 [Checklist for effective Skills](#checklist-for-effective-skills)。

### 解决问题，勿甩给 Claude

为 Skill 写脚本时，应处理错误条件，而不是把烂摊子交给 Claude。

**好例子：明确处理错误**：

```python  theme={null}
def process_file(path):
    """Process a file, creating it if it doesn't exist."""
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        # Create file with default content instead of failing
        print(f"File {path} not found, creating default")
        with open(path, 'w') as f:
            f.write('')
        return ''
    except PermissionError:
        # Provide alternative instead of failing
        print(f"Cannot access {path}, using default")
        return ''
```

**坏例子：甩给 Claude**：

```python  theme={null}
def process_file(path):
    # Just fail and let Claude figure it out
    return open(path).read()
```

配置参数也应有理由与文档，避免「巫毒常数」（Ousterhout's law）。你若不知道合理取值，Claude 更难猜。

**好例子：自解释**：

```python  theme={null}
# HTTP requests typically complete within 30 seconds
# Longer timeout accounts for slow connections
REQUEST_TIMEOUT = 30

# Three retries balances reliability vs speed
# Most intermittent failures resolve by the second retry
MAX_RETRIES = 3
```

**坏例子：魔法数字**：

```python  theme={null}
TIMEOUT = 47  # Why 47?
RETRIES = 5   # Why 5?
```

### 提供工具脚本

即便 Claude 能写脚本，预制脚本仍有优势：

**工具脚本的好处**：

* 比现场生成的代码更可靠
* 省 token（不必把整段代码塞进上下文）
* 省时间（无需生成代码）
* 多次使用保持一致

<img src="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=4bbc45f2c2e0bee9f2f0d5da669bad00" alt="Bundling executable scripts alongside instruction files" data-og-width="2048" width="2048" data-og-height="1154" height="1154" data-path="images/agent-skills-executable-scripts.png" data-optimize="true" data-opv="3" srcset="https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=280&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=9a04e6535a8467bfeea492e517de389f 280w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=560&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=e49333ad90141af17c0d7651cca7216b 560w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=840&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=954265a5df52223d6572b6214168c428 840w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=1100&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=2ff7a2d8f2a83ee8af132b29f10150fd 1100w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=1650&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=48ab96245e04077f4d15e9170e081cfb 1650w, https://mintcdn.com/anthropic-claude-docs/4Bny2bjzuGBK7o00/images/agent-skills-executable-scripts.png?w=2500&fit=max&auto=format&n=4Bny2bjzuGBK7o00&q=85&s=0301a6c8b3ee879497cc5b5483177c90 2500w" />

上图展示可执行脚本如何与说明文件配合。说明文件（如 forms.md）引用脚本，Claude 可执行脚本而无需把其全文载入上下文。

**重要区分**：在说明中写清 Claude 应：

* **执行脚本**（最常见）："Run `analyze_form.py` to extract fields"
* **当作参考阅读**（复杂逻辑）："See `analyze_form.py` for the field extraction algorithm"

多数工具脚本优先执行，更可靠高效。脚本执行机制详见下文 [Runtime environment](#runtime-environment)。

**示例**：

````markdown  theme={null}
## Utility scripts

**analyze_form.py**: Extract all form fields from PDF

```bash
python scripts/analyze_form.py input.pdf > fields.json
```

Output format:
```json
{
  "field_name": {"type": "text", "x": 100, "y": 200},
  "signature": {"type": "sig", "x": 150, "y": 500}
}
```

**validate_boxes.py**: Check for overlapping bounding boxes

```bash
python scripts/validate_boxes.py fields.json
# Returns: "OK" or lists conflicts
```

**fill_form.py**: Apply field values to PDF

```bash
python scripts/fill_form.py input.pdf fields.json output.pdf
```
````

### 使用视觉分析

输入可渲染为图像时，可让 Claude 做视觉分析：

````markdown  theme={null}
## Form layout analysis

1. Convert PDF to images:
   ```bash
   python scripts/pdf_to_images.py form.pdf
   ```

2. Analyze each page image to identify form fields
3. Claude can see field locations and types visually
````

<Note>
  此例中你需要自行实现 `pdf_to_images.py` 脚本。
</Note>

Claude 的视觉能力有助于理解版式与结构。

### 创建可校验的中间产物

Claude 做复杂、开放任务时可能出错。「plan-validate-execute」模式通过：先让 Claude 用结构化格式产出计划，再用脚本校验计划，最后执行——从而尽早抓错。

**示例**：请 Claude 按表格更新 PDF 中 50 个表单字段。若无校验，可能引用不存在的字段、值冲突、漏必填字段或应用错误。

**做法**：沿用上文 PDF 表单工作流，但增加中间文件 `changes.json`，在应用变更前先校验。流程为：analyze → **create plan file** → **validate plan** → execute → verify。

**为何有效：**

* **尽早发现错误**：在写入前用 validation 发现问题
* **机器可验证**：脚本提供客观校验
* **计划可迭代**：Claude 可反复改计划而不动原件
* **调试清晰**：错误信息指向具体问题

**何时用**：批量操作、破坏性变更、复杂校验规则、高风险操作。

**实现提示**：校验脚本输出要具体，例如 "Field 'signature\_date' not found. Available fields: customer\_name, order\_total, signature\_date\_signed"，便于 Claude 自行修正。

### 包依赖

Skill 跑在代码执行环境中，各平台能力不同：

* **claude.ai**：可从 npm、PyPI 安装包，并从 GitHub 拉取
* **Anthropic API**：无网络、无运行时安装包

在 SKILL.md 列出所需包，并在 [code execution tool documentation](/en/docs/agents-and-tools/tool-use/code-execution-tool) 中确认可用。

### 运行时环境

Skill 运行在带文件系统、bash 与代码执行能力的环境中。架构概念见 overview 中 [The Skills architecture](/en/docs/agents-and-tools/agent-skills/overview#the-skills-architecture)。

**对撰写的影响：**

**Claude 如何访问 Skill：**

1. **元数据预加载**：启动时所有 Skill 的 YAML frontmatter 中 name、description 进入系统提示
2. **文件按需读取**：需要时 Claude 通过 bash Read 等工具从文件系统读 SKILL.md 及其他文件
3. **脚本高效执行**：工具脚本可通过 bash 执行，无需把全文载入上下文；仅输出消耗 token
4. **大文件无预载惩罚**：参考文件、数据或文档在实际读取前不占上下文 token

* **路径很重要**：Claude 像浏览文件系统一样浏览 skill 目录。用正斜杠（`reference/guide.md`），勿用反斜杠
* **文件名表意**：`form_validation_rules.md` 优于 `doc2.md`
* **为可发现性组织目录**：按领域或功能分目录
  * 好：`reference/finance.md`、`reference/sales.md`
  * 差：`docs/file1.md`、`docs/file2.md`
* **打包完整资源**：完整 API 文档、大量示例、大数据集均可；访问前不占上下文
* **确定性操作用脚本**：写 `validate_form.py`，勿让 Claude 现场生成校验代码
* **写清执行意图**：
  * "Run `analyze_form.py` to extract fields"（执行）
  * "See `analyze_form.py` for the extraction algorithm"（当参考读）
* **测访问路径**：用真实请求验证 Claude 能否导航你的目录结构

**示例：**

```
bigquery-skill/
├── SKILL.md (overview, points to reference files)
└── reference/
    ├── finance.md (revenue metrics)
    ├── sales.md (pipeline data)
    └── product.md (usage analytics)
```

用户问收入时，Claude 读 SKILL.md，看到 `reference/finance.md`，再调 bash 只读该文件。sales.md 与 product.md 仍在磁盘上，需要前不占上下文 token。这种基于文件系统的模型支撑 progressive disclosure，Claude 可按任务精确加载所需内容。

技术架构详见 Skills overview 中 [How Skills work](/en/docs/agents-and-tools/agent-skills/overview#how-skills-work)。

### MCP 工具引用

Skill 若使用 MCP（Model Context Protocol）工具，务必使用完全限定工具名，避免 "tool not found"。

**格式**：`ServerName:tool_name`

**示例**：

```markdown  theme={null}
Use the BigQuery:bigquery_schema tool to retrieve table schemas.
Use the GitHub:create_issue tool to create issues.
```

其中：

* `BigQuery`、`GitHub` 为 MCP server 名
* `bigquery_schema`、`create_issue` 为对应 server 内工具名

无 server 前缀时，Claude 可能找不到工具，尤其在多 MCP server 并存时。

### 勿默认工具已安装

勿假设包已可用：

````markdown  theme={null}
**Bad example: Assumes installation**:
"Use the pdf library to process the file."

**Good example: Explicit about dependencies**:
"Install required package: `pip install pypdf`

Then use it:
```python
from pypdf import PdfReader
reader = PdfReader("file.pdf")
```"
````

## 技术说明

### YAML frontmatter 要求

SKILL.md 的 frontmatter 仅含 `name`（最多 64 字符）与 `description`（最多 1024 字符）。完整结构见 [Skills overview](/en/docs/agents-and-tools/agent-skills/overview#skill-structure)。

### Token 预算

SKILL.md 正文宜在 500 行以内。超出则按前文渐进式披露拆文件。架构细节见 [Skills overview](/en/docs/agents-and-tools/agent-skills/overview#how-skills-work)。

## 有效 Skill 检查清单

分享 Skill 前请核对：

### 核心质量

* [ ] Description 具体且含关键词
* [ ] Description 同时说明做什么与何时用
* [ ] SKILL.md 正文少于 500 行
* [ ] 更多细节在独立文件中（如需要）
* [ ] 无强时效信息（或放在 "old patterns" 区）
* [ ] 全文术语一致
* [ ] 示例具体，非抽象
* [ ] 文件引用仅一层深
* [ ] 渐进式披露使用得当
* [ ] 工作流步骤清晰

### 代码与脚本

* [ ] 脚本解决问题，而非甩给 Claude
* [ ] 错误处理明确、有用
* [ ] 无「巫毒常数」（取值均有理由）
* [ ] 所需包已在说明中列出并确认可用
* [ ] 脚本有清晰文档
* [ ] 无 Windows 风格路径（均用正斜杠）
* [ ] 关键操作含 validation/verification 步骤
* [ ] 质量关键任务含反馈环

### 测试

* [ ] 至少三个 evaluation
* [ ] 用 Haiku、Sonnet、Opus 测过
* [ ] 用真实使用场景测过
* [ ] 已纳入团队反馈（如适用）

## 下一步

<CardGroup cols={2}>
  <Card title="Agent Skills 入门" icon="rocket" href="/en/docs/agents-and-tools/agent-skills/quickstart">
    创建你的第一个 Skill
  </Card>

  <Card title="在 Claude Code 中使用 Skills" icon="terminal" href="/en/docs/claude-code/skills">
    在 Claude Code 中创建与管理 Skills
  </Card>

  <Card title="通过 API 使用 Skills" icon="code" href="/en/api/skills-guide">
    以编程方式上传并使用 Skills
  </Card>
</CardGroup>
