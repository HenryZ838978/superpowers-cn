# 基于条件的等待

## 概述

脆弱测试常用任意延时去「猜」时机，容易产生竞态：快机器过、负载或 CI 下挂。

**核心原则：** 等待你真正关心的**条件**成立，而不是猜需要多久。

## 何时使用

```dot
digraph when_to_use {
    "Test uses setTimeout/sleep?" [shape=diamond];
    "Testing timing behavior?" [shape=diamond];
    "Document WHY timeout needed" [shape=box];
    "Use condition-based waiting" [shape=box];

    "Test uses setTimeout/sleep?" -> "Testing timing behavior?" [label="yes"];
    "Testing timing behavior?" -> "Document WHY timeout needed" [label="yes"];
    "Testing timing behavior?" -> "Use condition-based waiting" [label="no"];
}
```

**适用：**
- 测试里有任意延时（`setTimeout`、`sleep`、`time.sleep()`）
- 测试不稳定（有时过、负载下挂）
- 并行跑时超时
- 等待异步操作结束

**不适用：**
- 测的是真实时间行为（debounce、throttle 间隔）
- 若必须用任意超时，务必**文档说明原因**

## 核心模式

```typescript
// ❌ BEFORE: Guessing at timing
await new Promise(r => setTimeout(r, 50));
const result = getResult();
expect(result).toBeDefined();

// ✅ AFTER: Waiting for condition
await waitFor(() => getResult() !== undefined);
const result = getResult();
expect(result).toBeDefined();
```

## 速查模式

| Scenario | Pattern |
|----------|---------|
| Wait for event | `waitFor(() => events.find(e => e.type === 'DONE'))` |
| Wait for state | `waitFor(() => machine.state === 'ready')` |
| Wait for count | `waitFor(() => items.length >= 5)` |
| Wait for file | `waitFor(() => fs.existsSync(path))` |
| Complex condition | `waitFor(() => obj.ready && obj.value > 10)` |

## 实现

通用轮询函数：
```typescript
async function waitFor<T>(
  condition: () => T | undefined | null | false,
  description: string,
  timeoutMs = 5000
): Promise<T> {
  const startTime = Date.now();

  while (true) {
    const result = condition();
    if (result) return result;

    if (Date.now() - startTime > timeoutMs) {
      throw new Error(`Timeout waiting for ${description} after ${timeoutMs}ms`);
    }

    await new Promise(r => setTimeout(r, 10)); // Poll every 10ms
  }
}
```

本目录下 `condition-based-waiting-example.ts` 含完整实现及领域辅助函数（`waitForEvent`、`waitForEventCount`、`waitForEventMatch`），来自真实排障会话。

## 常见错误

**❌ 轮询过快：** `setTimeout(check, 1)` — 浪费 CPU  
**✅ 修正：** 约每 10ms 轮询一次

**❌ 无超时：** 条件永不成立则死循环  
**✅ 修正：** 始终设超时并给出清晰错误

**❌ 陈旧数据：** 在循环外缓存状态  
**✅ 修正：** 在循环内调用 getter 取最新值

## 何时任意超时才是对的

```typescript
// Tool ticks every 100ms - need 2 ticks to verify partial output
await waitForEvent(manager, 'TOOL_STARTED'); // First: wait for condition
await new Promise(r => setTimeout(r, 200));   // Then: wait for timed behavior
// 200ms = 2 ticks at 100ms intervals - documented and justified
```

**要求：**
1. 先等到触发条件
2. 基于**已知**时序（非瞎猜）
3. 注释说明 **WHY**

## 实际效果

来自排障会话（2025-10-03）：
- 修复 3 个文件中 15 个脆弱测试
- 通过率：60% → 100%
- 执行时间：约快 40%
- 消除竞态
