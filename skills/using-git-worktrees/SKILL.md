---
name: using-git-worktrees
description: Use when starting feature work that needs isolation from current workspace or before executing implementation plans - creates isolated git worktrees with smart directory selection and safety verification
---

# 使用 Git Worktrees

## 概述

Git worktree 创建共享同一仓库的隔离工作区，可同时工作在多个分支而无需频繁切换。

**核心原则：** 系统化选择目录 + 安全验证 = 可靠隔离。

**开始时声明：**「我正在使用 using-git-worktrees skill 来建立隔离工作区。」

## 目录选择流程

按以下优先级：

### 1. 检查已有目录

```bash
# Check in priority order
ls -d .worktrees 2>/dev/null     # Preferred (hidden)
ls -d worktrees 2>/dev/null      # Alternative
```

**若找到：** 使用该目录。若两者都存在，优先 `.worktrees`。

### 2. 检查 CLAUDE.md

```bash
grep -i "worktree.*director" CLAUDE.md 2>/dev/null
```

**若指定了偏好：** 直接使用，不必再问。

### 3. 询问用户

若既无目录也无 CLAUDE.md 偏好：

```
No worktree directory found. Where should I create worktrees?

1. .worktrees/ (project-local, hidden)
2. ~/.config/superpowers/worktrees/<project-name>/ (global location)

Which would you prefer?
```

## 安全验证

### 项目本地目录（.worktrees 或 worktrees）

**创建 worktree 前必须确认目录已被 ignore：**

```bash
# Check if directory is ignored (respects local, global, and system gitignore)
git check-ignore -q .worktrees 2>/dev/null || git check-ignore -q worktrees 2>/dev/null
```

**若未被 ignore：**

按 Jesse 的规则「坏了就立刻修」：
1. 在 .gitignore 中加入合适规则  
2. commit 该变更  
3. 再继续创建 worktree  

**为何关键：** 避免误将 worktree 内容 commit 进仓库。

### 全局目录（~/.config/superpowers/worktrees）

完全在项目外，无需 .gitignore 验证。

## 创建步骤

### 1. 检测项目名

```bash
project=$(basename "$(git rev-parse --show-toplevel)")
```

### 2. 创建 Worktree

```bash
# Determine full path
case $LOCATION in
  .worktrees|worktrees)
    path="$LOCATION/$BRANCH_NAME"
    ;;
  ~/.config/superpowers/worktrees/*)
    path="~/.config/superpowers/worktrees/$project/$BRANCH_NAME"
    ;;
esac

# Create worktree with new branch
git worktree add "$path" -b "$BRANCH_NAME"
cd "$path"
```

### 3. 运行项目初始化

自动检测并执行合适步骤：

```bash
# Node.js
if [ -f package.json ]; then npm install; fi

# Rust
if [ -f Cargo.toml ]; then cargo build; fi

# Python
if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
if [ -f pyproject.toml ]; then poetry install; fi

# Go
if [ -f go.mod ]; then go mod download; fi
```

### 4. 验证干净基线

运行测试，确保 worktree 从干净状态开始：

```bash
# Examples - use project-appropriate command
npm test
cargo test
pytest
go test ./...
```

**若测试失败：** 报告失败，询问是继续还是调查。

**若测试通过：** 报告就绪。

### 5. 报告位置

```
Worktree ready at <full-path>
Tests passing (<N> tests, 0 failures)
Ready to implement <feature-name>
```

## 速查

| 情况 | 动作 |
|-----------|--------|
| 存在 `.worktrees/` | 使用（验证已 ignore） |
| 存在 `worktrees/` | 使用（验证已 ignore） |
| 两者都存在 | 使用 `.worktrees/` |
| 都不存在 | 查 CLAUDE.md → 问用户 |
| 目录未 ignore | 加入 .gitignore 并 commit |
| 基线测试失败 | 报告失败并询问 |
| 无 package.json/Cargo.toml | 跳过依赖安装 |

## 常见错误

### 跳过 ignore 验证

- **问题：** worktree 内容被跟踪，污染 git status  
- **修正：** 创建项目本地 worktree 前始终 `git check-ignore`  

### 假定目录位置

- **问题：** 不一致，违反项目约定  
- **修正：** 按优先级：已有 > CLAUDE.md > 询问  

### 在测试失败时仍继续

- **问题：** 无法区分新 bug 与既有问题  
- **修正：** 报告失败，取得明确许可再继续  

### 硬编码安装命令

- **问题：** 在不同工具链的项目上失效  
- **修正：** 根据项目文件自动检测（package.json 等）  

## 示例工作流

```
You: I'm using the using-git-worktrees skill to set up an isolated workspace.

[Check .worktrees/ - exists]
[Verify ignored - git check-ignore confirms .worktrees/ is ignored]
[Create worktree: git worktree add .worktrees/auth -b feature/auth]
[Run npm install]
[Run npm test - 47 passing]

Worktree ready at /Users/jesse/myproject/.worktrees/auth
Tests passing (47 tests, 0 failures)
Ready to implement auth feature
```

## 危险信号

**绝不：**
- 项目本地 worktree 未验证 ignore 就创建  
- 跳过基线测试验证  
- 测试失败未询问就继续  
- 位置有歧义时自作主张  
- 跳过 CLAUDE.md 检查  

**务必：**
- 按优先级：已有 > CLAUDE.md > 询问  
- 项目本地须验证目录已 ignore  
- 自动检测并运行项目初始化  
- 验证干净测试基线  

## 集成

**由以下调用：**
- **brainstorming**（第 4 阶段）— 设计批准且随后实现时**必选**  
- **subagent-driven-development** — 执行任何任务前**必选**  
- **executing-plans** — 执行任何任务前**必选**  
- 任何需要隔离工作区的 skill  

**搭配：**
- **finishing-a-development-branch** — 工作完成后**必选**清理  
