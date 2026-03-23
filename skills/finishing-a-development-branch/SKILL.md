---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to decide how to integrate the work - guides completion of development work by presenting structured options for merge, PR, or cleanup
---

# 完成开发分支

## 概述

通过呈现清晰选项并执行所选工作流，引导开发工作收尾。

**核心原则：** 验证测试 → 呈现选项 → 执行选择 → 清理。

**开始时声明：**「我正在使用 finishing-a-development-branch skill 来完成本工作。」

## 流程

### 第 1 步：验证测试

**在呈现选项前，确认测试通过：**

```bash
# Run project's test suite
npm test / cargo test / pytest / go test ./...
```

**若测试失败：**
```
Tests failing (<N> failures). Must fix before completing:

[Show failures]

Cannot proceed with merge/PR until tests pass.
```

停止，不要进入第 2 步。

**若测试通过：** 继续第 2 步。

### 第 2 步：确定基分支

```bash
# Try common base branches
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

或询问：「本分支是从 main 分出来的 — 对吗？」

### 第 3 步：呈现选项

**恰好**呈现以下 4 个选项：

```
Implementation complete. What would you like to do?

1. Merge back to <base-branch> locally
2. Push and create a Pull Request
3. Keep the branch as-is (I'll handle it later)
4. Discard this work

Which option?
```

**不要附加解释** — 保持选项简洁。

### 第 4 步：执行选择

#### 选项 1：本地 merge

```bash
# Switch to base branch
git checkout <base-branch>

# Pull latest
git pull

# Merge feature branch
git merge <feature-branch>

# Verify tests on merged result
<test command>

# If tests pass
git branch -d <feature-branch>
```

然后：清理 worktree（第 5 步）

#### 选项 2：Push 并创建 PR

```bash
# Push branch
git push -u origin <feature-branch>

# Create PR
gh pr create --title "<title>" --body "$(cat <<'EOF'
## Summary
<2-3 bullets of what changed>

## Test Plan
- [ ] <verification steps>
EOF
)"
```

然后：清理 worktree（第 5 步）

#### 选项 3：保持原状

汇报：「保留分支 `<name>`。worktree 仍在 `<path>`。」

**不要清理 worktree。**

#### 选项 4：丢弃

**先确认：**
```
This will permanently delete:
- Branch <name>
- All commits: <commit-list>
- Worktree at <path>

Type 'discard' to confirm.
```

等待用户准确输入确认。

若已确认：
```bash
git checkout <base-branch>
git branch -D <feature-branch>
```

然后：清理 worktree（第 5 步）

### 第 5 步：清理 Worktree

**对选项 1、2、4：**

检查是否在 worktree 中：
```bash
git worktree list | grep $(git branch --show-current)
```

若是：
```bash
git worktree remove <worktree-path>
```

**对选项 3：** 保留 worktree。

## 速查

| 选项 | Merge | Push | 保留 Worktree | 清理分支 |
|--------|-------|------|---------------|----------------|
| 1. 本地 merge | ✓ | - | - | ✓ |
| 2. 创建 PR | - | ✓ | ✓ | - |
| 3. 保持原状 | - | - | ✓ | - |
| 4. 丢弃 | - | - | - | ✓（强制） |

## 常见错误

**跳过测试验证**
- **问题：** merge 进坏代码、创建失败的 PR  
- **修正：** 提供选项前始终验证测试  

**开放式提问**
- **问题：**「接下来做什么？」→ 含糊  
- **修正：** 恰好给出 4 个结构化选项  

**自动清理 worktree**
- **问题：** 在仍需要时删掉 worktree（选项 2、3）  
- **修正：** 仅对选项 1 和 4 清理  

**丢弃无确认**
- **问题：** 误删工作  
- **修正：** 选项 4 要求输入「discard」确认  

## 危险信号

**绝不：**
- 在测试失败时继续  
- 未在 merge 结果上验证测试就 merge  
- 无确认就删除工作  
- 未经明确要求就 force-push  

**务必：**
- 提供选项前验证测试  
- 恰好给出 4 个选项  
- 选项 4 需打字确认「discard」  
- 仅对选项 1 与 4 清理 worktree  

## 集成

**由以下调用：**
- **subagent-driven-development**（第 7 步）— 全部任务完成后  
- **executing-plans**（第 5 步）— 全部批次完成后  

**搭配：**
- **using-git-worktrees** — 清理该 skill 创建的 worktree  
