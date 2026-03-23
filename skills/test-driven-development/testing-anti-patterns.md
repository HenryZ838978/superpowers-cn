# 测试反模式

**在以下情况加载本参考：** 编写或修改测试、添加 mock，或 tempted 向生产代码加「仅测试用」方法时。

## 概述

测试必须验证真实行为，而非 mock 行为。mock 用于隔离，不是被测对象。

**核心原则：** 测代码**做什么**，不要测 mock **做什么**。

**严格遵循 TDD 可避免这些反模式。**

## 铁律

```
1. NEVER test mock behavior
2. NEVER add test-only methods to production classes
3. NEVER mock without understanding dependencies
```

## 反模式 1：测 mock 行为

**违规：**
```typescript
// ❌ BAD: Testing that the mock exists
test('renders sidebar', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});
```

**为何错误：**
- 你在验证 mock 存在，而非组件行为
- mock 在就过、不在就挂
- 对真实行为零信息

**人类搭档的纠偏：**「我们是在测 mock 的行为吗？」

**修正：**
```typescript
// ✅ GOOD: Test real component or don't mock it
test('renders sidebar', () => {
  render(<Page />);  // Don't mock sidebar
  expect(screen.getByRole('navigation')).toBeInTheDocument();
});

// OR if sidebar must be mocked for isolation:
// Don't assert on the mock - test Page's behavior with sidebar present
```

### 门禁

```
BEFORE asserting on any mock element:
  Ask: "Am I testing real component behavior or just mock existence?"

  IF testing mock existence:
    STOP - Delete the assertion or unmock the component

  Test real behavior instead
```

## 反模式 2：生产类中的仅测试方法

**违规：**
```typescript
// ❌ BAD: destroy() only used in tests
class Session {
  async destroy() {  // Looks like production API!
    await this._workspaceManager?.destroyWorkspace(this.id);
    // ... cleanup
  }
}

// In tests
afterEach(() => session.destroy());
```

**为何错误：**
- 生产类被测试专用代码污染
- 生产误调用有危险
- 违反 YAGNI 与职责分离
- 混淆对象生命周期与实体生命周期

**修正：**
```typescript
// ✅ GOOD: Test utilities handle test cleanup
// Session has no destroy() - it's stateless in production

// In test-utils/
export async function cleanupSession(session: Session) {
  const workspace = session.getWorkspaceInfo();
  if (workspace) {
    await workspaceManager.destroyWorkspace(workspace.id);
  }
}

// In tests
afterEach(() => cleanupSession(session));
```

### 门禁

```
BEFORE adding any method to production class:
  Ask: "Is this only used by tests?"

  IF yes:
    STOP - Don't add it
    Put it in test utilities instead

  Ask: "Does this class own this resource's lifecycle?"

  IF no:
    STOP - Wrong class for this method
```

## 反模式 3：未理解就 mock

**违规：**
```typescript
// ❌ BAD: Mock breaks test logic
test('detects duplicate server', () => {
  // Mock prevents config write that test depends on!
  vi.mock('ToolCatalog', () => ({
    discoverAndCacheTools: vi.fn().mockResolvedValue(undefined)
  }));

  await addServer(config);
  await addServer(config);  // Should throw - but won't!
});
```

**为何错误：**
- 被 mock 的方法有测试依赖的副作用（写 config）
- 过度 mock「求稳」反而破坏真实行为
- 测试因错误原因通过或神秘失败

**修正：**
```typescript
// ✅ GOOD: Mock at correct level
test('detects duplicate server', () => {
  // Mock the slow part, preserve behavior test needs
  vi.mock('MCPServerManager'); // Just mock slow server startup

  await addServer(config);  // Config written
  await addServer(config);  // Duplicate detected ✓
});
```

### 门禁

```
BEFORE mocking any method:
  STOP - Don't mock yet

  1. Ask: "What side effects does the real method have?"
  2. Ask: "Does this test depend on any of those side effects?"
  3. Ask: "Do I fully understand what this test needs?"

  IF depends on side effects:
    Mock at lower level (the actual slow/external operation)
    OR use test doubles that preserve necessary behavior
    NOT the high-level method the test depends on

  IF unsure what test depends on:
    Run test with real implementation FIRST
    Observe what actually needs to happen
    THEN add minimal mocking at the right level

  Red flags:
    - "I'll mock this to be safe"
    - "This might be slow, better mock it"
    - Mocking without understanding the dependency chain
```

## 反模式 4：不完整 mock

**违规：**
```typescript
// ❌ BAD: Partial mock - only fields you think you need
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' }
  // Missing: metadata that downstream code uses
};

// Later: breaks when code accesses response.metadata.requestId
```

**为何错误：**
- **部分 mock 掩盖结构假设** — 只 mock 你知道的字段
- **下游可能依赖你未包含的字段** — 静默失败
- **测试过、集成挂** — mock 不完整，真实 API 完整
- **虚假信心** — 测试无法证明真实行为

**铁律：** mock **完整**数据结构，如现实 API，而非仅当前测试用到的字段。

**修正：**
```typescript
// ✅ GOOD: Mirror real API completeness
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' },
  metadata: { requestId: 'req-789', timestamp: 1234567890 }
  // All fields real API returns
};
```

### 门禁

```
BEFORE creating mock responses:
  Check: "What fields does the real API response contain?"

  Actions:
    1. Examine actual API response from docs/examples
    2. Include ALL fields system might consume downstream
    3. Verify mock matches real response schema completely

  Critical:
    If you're creating a mock, you must understand the ENTIRE structure
    Partial mocks fail silently when code depends on omitted fields

  If uncertain: Include all documented fields
```

## 反模式 5：集成测试当「事后补充」

**违规：**
```
✅ Implementation complete
❌ No tests written
"Ready for testing"
```

**为何错误：**
- 测试是实现的一部分，不是可选项
- TDD 本会拦住
- 无测试不能叫完成

**修正：**
```
TDD cycle:
1. Write failing test
2. Implement to pass
3. Refactor
4. THEN claim complete
```

## mock 过于复杂时

**警示信号：**
- mock 搭建比测试逻辑还长
- 为通过测试而 mock 一切
- mock 缺真实组件上的方法
- mock 一变测试就挂

**人类搭档的问题：**「这里真的需要 mock 吗？」

**考虑：** 用真实组件做集成测试往往比复杂 mock 更简单

## TDD 如何预防

**TDD 的帮助：**
1. **先写测试** → 迫使你想清到底在测什么
2. **先看失败** → 确认测的是真实行为而非 mock
3. **最小实现** → 不易渗入仅测试方法
4. **真实依赖** → mock 前你已看到测试真正需要什么

**若在测 mock 行为，你已违反 TDD** — 未先对真实代码看失败就加了 mock。

## 速查

| Anti-Pattern | Fix |
|--------------|-----|
| Assert on mock elements | Test real component or unmock it |
| Test-only methods in production | Move to test utilities |
| Mock without understanding | Understand dependencies first, mock minimally |
| Incomplete mocks | Mirror real API completely |
| Tests as afterthought | TDD - tests first |
| Over-complex mocks | Consider integration tests |

## 红旗

- 断言查 `*-mock` 这类 test ID
- 方法仅被测试文件调用
- mock 搭建占测试 >50%
- 去掉 mock 测试就挂
- 说不清为何需要 mock
- 「先 mock 保平安」

## 结论

**mock 是隔离工具，不是被测对象。**

若 TDD 让你发现自己在测 mock 行为，路走偏了。

修正：测真实行为，或反思是否根本不该在这 mock。
