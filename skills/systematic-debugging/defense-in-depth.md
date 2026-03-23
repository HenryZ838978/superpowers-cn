# 纵深校验（Defense-in-Depth Validation）

## 概述

因脏数据导致的 bug，只在某一处加校验往往觉得够了。但那条路径可能被别的调用链、重构或 mock 绕过。

**核心原则：** 在数据流经的**每一层**都校验。让这类 bug **在结构上不可能**再发生。

## 为何要多层

单层校验：「我们修了 bug」  
多层校验：「我们让这类 bug 不可能再出现」

不同层抓住不同情况：
- 入口校验抓住大部分问题
- 业务逻辑抓住边界情况
- 环境护栏挡住特定上下文下的危险
- 调试日志在其他层失效时帮助取证

## 四层

### Layer 1：入口校验
**目的：** 在 API 边界拒绝明显非法输入

```typescript
function createProject(name: string, workingDirectory: string) {
  if (!workingDirectory || workingDirectory.trim() === '') {
    throw new Error('workingDirectory cannot be empty');
  }
  if (!existsSync(workingDirectory)) {
    throw new Error(`workingDirectory does not exist: ${workingDirectory}`);
  }
  if (!statSync(workingDirectory).isDirectory()) {
    throw new Error(`workingDirectory is not a directory: ${workingDirectory}`);
  }
  // ... proceed
}
```

### Layer 2：业务逻辑校验
**目的：** 确保数据对本操作有意义

```typescript
function initializeWorkspace(projectDir: string, sessionId: string) {
  if (!projectDir) {
    throw new Error('projectDir required for workspace initialization');
  }
  // ... proceed
}
```

### Layer 3：环境护栏
**目的：** 在特定上下文中阻止危险操作

```typescript
async function gitInit(directory: string) {
  // In tests, refuse git init outside temp directories
  if (process.env.NODE_ENV === 'test') {
    const normalized = normalize(resolve(directory));
    const tmpDir = normalize(resolve(tmpdir()));

    if (!normalized.startsWith(tmpDir)) {
      throw new Error(
        `Refusing git init outside temp dir during tests: ${directory}`
      );
    }
  }
  // ... proceed
}
```

### Layer 4：调试埋点
**目的：** 为取证保留上下文

```typescript
async function gitInit(directory: string) {
  const stack = new Error().stack;
  logger.debug('About to git init', {
    directory,
    cwd: process.cwd(),
    stack,
  });
  // ... proceed
}
```

## 如何套用

发现 bug 时：

1. **追踪数据流** — 坏值从哪来？在哪被用？
2. **列出所有检查点** — 数据经过的每一处
3. **每层加校验** — 入口、业务、环境、调试
4. **逐层测** — 试着绕过第一层，确认第二层能拦住

## 会话中的例子

Bug：空的 `projectDir` 导致在源码目录执行 `git init`

**数据流：**
1. Test setup → 空字符串
2. `Project.create(name, '')`
3. `WorkspaceManager.createWorkspace('')`
4. `git init` 在 `process.cwd()` 执行

**四层补齐：**
- Layer 1：`Project.create()` 校验非空/存在/可写
- Layer 2：`WorkspaceManager` 校验 projectDir 非空
- Layer 3：`WorktreeManager` 在测试中拒绝 tmpdir 外的 git init
- Layer 4：git init 前打 stack trace 日志

**结果：** 1847 个测试全过，bug 无法复现

## 要点

四层都有必要。测试过程中每层都拦住了其他层漏掉的情况：
- 不同路径会绕过入口校验
- Mock 会绕过业务校验
- 不同平台边界情况需要环境护栏
- 调试日志暴露结构性误用

**不要停在一处校验。** 每一层都加检查。
