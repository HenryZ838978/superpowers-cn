# 可视化搭档指南

基于浏览器的可视化头脑风暴搭档，用于展示 mockup、示意图与选项。

## 何时使用

按**每个问题**决定，而非按整场会话。判断标准：**用户是「看」比「读」更容易理解吗？**

**内容本身是视觉化时用浏览器：**

- **UI mockup** — 线框、版式、导航结构、组件设计
- **架构图** — 系统组件、数据流、关系图
- **并排视觉对比** — 两种版式、配色或设计方向
- **视觉打磨** — 讨论观感、间距、视觉层级
- **空间关系** — 状态机、流程图、实体关系等以图呈现

**内容是文字或表格时用终端：**

- **需求与范围** — 「X 指什么？」「哪些功能在范围内？」
- **概念性 A/B/C** — 用文字描述的方案取舍
- **权衡列表** — 利弊、对比表
- **技术决策** — API 设计、数据建模、架构选型
- **澄清问题** — 答案是话语而非视觉偏好

「关于 UI 的问题」不等于「视觉问题」。「你想要哪种向导？」偏概念——用终端。「这几种向导版式哪个更对味？」偏视觉——用浏览器。

## 工作原理

服务器监听目录中的 HTML 文件，把**最新**的一个提供给浏览器。你写 HTML，用户在浏览器里查看并点击选择。选择会写入 `.events` 文件，你在下一轮读取。

**内容片段 vs 完整文档：** 若 HTML 以 `<!DOCTYPE` 或 `<html` 开头，服务器原样提供（仅注入 helper script）。否则服务器会自动用 frame 模板包裹——加上 header、主题 CSS、选中态与全部交互基础设施。**默认写内容片段。** 仅当你需要完全控制页面时再写完整文档。

## 启动会话

```bash
# Start server with persistence (mockups saved to project)
scripts/start-server.sh --project-dir /path/to/project

# Returns: {"type":"server-started","port":52341,"url":"http://localhost:52341",
#           "screen_dir":"/path/to/project/.superpowers/brainstorm/12345-1706000000"}
```

从响应中保存 `screen_dir`。告知用户打开 URL。

**查找连接信息：** 服务器将启动 JSON 写入 `$SCREEN_DIR/.server-info`。若在后台启动且未捕获 stdout，读该文件获取 URL 与 port。使用 `--project-dir` 时，在 `<project>/.superpowers/brainstorm/` 下查找会话目录。

**注意：** 传入项目根作为 `--project-dir`，mockup 会持久化在 `.superpowers/brainstorm/`，重启服务器后仍在。不传则文件在 `/tmp` 会被清理。若尚未忽略，提醒用户将 `.superpowers/` 加入 `.gitignore`。

**按平台启动服务器：**

**Claude Code（macOS / Linux）：**
```bash
# Default mode works — the script backgrounds the server itself
scripts/start-server.sh --project-dir /path/to/project
```

**Claude Code（Windows）：**
```bash
# Windows auto-detects and uses foreground mode, which blocks the tool call.
# Use run_in_background: true on the Bash tool call so the server survives
# across conversation turns.
scripts/start-server.sh --project-dir /path/to/project
```
通过 Bash 工具调用时设置 `run_in_background: true`。下一轮读取 `$SCREEN_DIR/.server-info` 获取 URL 与 port。

**Codex：**
```bash
# Codex reaps background processes. The script auto-detects CODEX_CI and
# switches to foreground mode. Run it normally — no extra flags needed.
scripts/start-server.sh --project-dir /path/to/project
```

**Gemini CLI：**
```bash
# Use --foreground and set is_background: true on your shell tool call
# so the process survives across turns
scripts/start-server.sh --project-dir /path/to/project --foreground
```

**其他环境：** 服务器必须在多轮对话间保持后台运行。若环境会回收 detached 进程，使用 `--foreground` 并结合本平台的后台执行机制。

若 URL 在你侧浏览器不可达（远程/容器常见），绑定非 loopback 主机：

```bash
scripts/start-server.sh \
  --project-dir /path/to/project \
  --host 0.0.0.0 \
  --url-host localhost
```

用 `--url-host` 控制返回的 URL JSON 中打印的主机名。

## 循环

1. **确认服务器存活**，然后在 `screen_dir` 中**写入新 HTML 文件**：
   - 每次写入前确认 `$SCREEN_DIR/.server-info` 存在。若不存在（或存在 `.server-stopped`），说明服务器已退出——先用 `start-server.sh` 重启再继续。服务器在无活动约 30 分钟后会自动退出。
   - 语义化文件名：`platform.html`、`visual-style.html`、`layout.html`
   - **禁止复用文件名** — 每个屏幕用新文件
   - 使用 Write 工具 — **不要用 cat/heredoc**（终端噪音大）
   - 服务器按修改时间自动提供最新文件

2. **告知用户预期并结束本轮：**
   - 每步都提醒 URL（不只第一步）
   - 简短文字说明屏幕上是什么（例如：「首页三种版式选项」）
   - 请用户在终端回复：「请看一下并告诉我想法。需要的话可以点击选项。」

3. **下一轮** — 用户在终端回复后：
   - 若存在则读取 `$SCREEN_DIR/.events` — 内含浏览器交互（点击、选择）的 JSON 行
   - 与用户终端文字合并理解
   - 终端消息是主反馈；`.events` 提供结构化交互数据

4. **迭代或推进** — 若反馈会改变当前屏幕，写新文件（如 `layout-v2.html`）。仅当当前步骤得到确认后再进入下一问。

5. **回到终端时卸载视觉** — 若下一步不需要浏览器（如澄清问题、权衡讨论），推一个等待页以清掉过时内容：

   ```html
   <!-- filename: waiting.html (or waiting-2.html, etc.) -->
   <div style="display:flex;align-items:center;justify-content:center;min-height:60vh">
     <p class="subtitle">Continuing in terminal...</p>
   </div>
   ```

   避免用户盯着已结束的选项而对话已转向。下一个视觉问题出现时照常推新内容文件。

6. 重复直至结束。

## 编写内容片段

只写放进页面里的内容。服务器会自动用 frame 模板包裹（header、主题 CSS、选中态与全部交互基础设施）。

**最小示例：**

```html
<h2>Which layout works better?</h2>
<p class="subtitle">Consider readability and visual hierarchy</p>

<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Single Column</h3>
      <p>Clean, focused reading experience</p>
    </div>
  </div>
  <div class="option" data-choice="b" onclick="toggleSelect(this)">
    <div class="letter">B</div>
    <div class="content">
      <h3>Two Column</h3>
      <p>Sidebar navigation with main content</p>
    </div>
  </div>
</div>
```

到此即可。不需要 `<html>`、自写 CSS 或 `<script>`。服务器会提供这些。

## 可用 CSS 类

frame 模板为你的内容提供以下 CSS 类：

### Options（A/B/C 选择）

```html
<div class="options">
  <div class="option" data-choice="a" onclick="toggleSelect(this)">
    <div class="letter">A</div>
    <div class="content">
      <h3>Title</h3>
      <p>Description</p>
    </div>
  </div>
</div>
```

**多选：** 在容器上加 `data-multiselect`，用户可多选。每次点击切换。指示条显示数量。

```html
<div class="options" data-multiselect>
  <!-- same option markup — users can select/deselect multiple -->
</div>
```

### Cards（视觉设计）

```html
<div class="cards">
  <div class="card" data-choice="design1" onclick="toggleSelect(this)">
    <div class="card-image"><!-- mockup content --></div>
    <div class="card-body">
      <h3>Name</h3>
      <p>Description</p>
    </div>
  </div>
</div>
```

### Mockup 容器

```html
<div class="mockup">
  <div class="mockup-header">Preview: Dashboard Layout</div>
  <div class="mockup-body"><!-- your mockup HTML --></div>
</div>
```

### Split view（并排）

```html
<div class="split">
  <div class="mockup"><!-- left --></div>
  <div class="mockup"><!-- right --></div>
</div>
```

### Pros/Cons

```html
<div class="pros-cons">
  <div class="pros"><h4>Pros</h4><ul><li>Benefit</li></ul></div>
  <div class="cons"><h4>Cons</h4><ul><li>Drawback</li></ul></div>
</div>
```

### Mock 元素（线框积木）

```html
<div class="mock-nav">Logo | Home | About | Contact</div>
<div style="display: flex;">
  <div class="mock-sidebar">Navigation</div>
  <div class="mock-content">Main content area</div>
</div>
<button class="mock-button">Action Button</button>
<input class="mock-input" placeholder="Input field">
<div class="placeholder">Placeholder area</div>
```

### 排版与区块

- `h2` — 页面标题
- `h3` — 小节标题
- `.subtitle` — 标题下辅助文字
- `.section` — 带底部间距的内容块
- `.label` — 小号大写标签文字

## 浏览器事件格式

用户在浏览器点击选项时，交互写入 `$SCREEN_DIR/.events`（每行一个 JSON 对象）。推送新屏幕时文件会自动清空。

```jsonl
{"type":"click","choice":"a","text":"Option A - Simple Layout","timestamp":1706000101}
{"type":"click","choice":"c","text":"Option C - Complex Grid","timestamp":1706000108}
{"type":"click","choice":"b","text":"Option B - Hybrid","timestamp":1706000115}
```

完整事件流反映用户的探索路径——可能在定案前点击多个选项。通常最后一条 `choice` 事件是最终选择，但点击模式也能反映犹豫或偏好，值得追问。

若不存在 `.events`，说明用户未在浏览器交互——仅依据终端文字即可。

## 设计建议

- **保真度匹配问题** — 版式用线框，观感问题再 polish
- **每页说明在问什么** — 「哪种版式更专业？」而不只是「选一个」
- **推进前先迭代** — 若反馈改变当前屏，写新版本
- **每屏最多 2–4 个选项**
- **该真实就真实** — 摄影作品集可用真实图（如 Unsplash）。占位图会掩盖设计问题。
- **mockup 保持简单** — 聚焦版式与结构，勿追求像素级完美

## 文件命名

- 语义化：`platform.html`、`visual-style.html`、`layout.html`
- 禁止复用文件名 — 每屏必须新文件
- 迭代可加版本后缀：`layout-v2.html`、`layout-v3.html`
- 服务器按修改时间取最新文件

## 清理

```bash
scripts/stop-server.sh $SCREEN_DIR
```

若会话用了 `--project-dir`，mockup 会保留在 `.superpowers/brainstorm/` 供日后查阅。仅 `/tmp` 会话会在 stop 时删除。

## 参考

- Frame 模板（CSS 参考）：`scripts/frame-template.html`
- Helper script（客户端）：`scripts/helper.js`
