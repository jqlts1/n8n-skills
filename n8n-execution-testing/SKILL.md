---
name: n8n-execution-testing
description: |
  Use when testing n8n workflows, deciding between mock testing and real execution, debugging failed runs, or figuring out what to inspect after execute_workflow, test_workflow, or get_execution results.
---

# n8n Execution & Testing

This Skill answers one question: the workflow is built — how do I test it, and how do I find what broke?

---

## When to Use

- You have a workflow and want to validate it safely before going live
- You're unsure whether to use `test_workflow` or `execute_workflow`
- An execution failed and you don't know where to start looking
- You have `get_execution` results and aren't sure what to fix next

---

## Which Test to Run First

| Situation | Recommended tool | Reason |
|-----------|-----------------|--------|
| Don't want to hit real external services | `prepare_test_pin_data` + `test_workflow` | Run the main path with mock data first |
| Need to verify real triggers and real integrations | `execute_workflow` | See real inputs and real execution results |
| Already failed, need to find which part broke | `get_execution` | Inspect node-level results directly |

---

## Recommended Order

```text
1. validate_workflow
2. Mock first if possible: prepare_test_pin_data -> test_workflow
3. Use execute_workflow only when you need the real entry point
4. After any failure: get_execution immediately
5. Based on the failure location, return to node config, expressions, or connections
```

---

## Mock Testing: prepare_test_pin_data + test_workflow

### Pin Data 规则：哪些节点需要 pin，哪些不需要

| 节点类型 | 需要 pin？ | 说明 |
|---------|-----------|------|
| Trigger 节点 | ✅ 必须 | 模拟触发输入 |
| 有凭据 (credentials) 的节点 | ✅ 必须 | 跳过真实认证 |
| HTTP Request 节点 | ✅ 必须 | 跳过真实请求 |
| Set / If / Code / Filter 等逻辑节点 | ❌ 不需要 | 正常执行 |
| Execute Command / 文件读写（无凭据） | ❌ 不需要 | 正常执行 |

> 简单记：**需要外部连接的 pin，纯逻辑的不 pin**

### 操作流程

#### Step 1: 获取 pin data schema

```text
prepare_test_pin_data({ workflowId: "xxx" })
```

返回的是**每个需要 pin 的节点的 JSON Schema**（不是真实数据）。Schema 来源于：
- 历史执行的输出结构
- 或节点类型定义

#### Step 2: 根据 schema 生成模拟数据

> ⚠️ **关键格式要求：每个 item 必须包裹在 `{json: {...}}` 里！**

```javascript
// ✅ 正确
pinData = {
  "Webhook 触发": [{ "json": { "id": "123", "name": "测试用户", "email": "test@example.com" } }],
  "HTTP请求": [{ "json": { "status": "ok", "data": [] } }]
}

// ❌ 错误 — 缺少 json 包裹
pinData = {
  "Webhook 触发": [{ "id": "123", "name": "测试用户" }]
}
```

**没有 schema 的节点**用空默认值：

```javascript
"某个节点": [{ "json": {} }]
```

#### Step 3: 执行 mock 测试

```text
test_workflow({
  workflowId: "xxx",
  pinData: { ... }
})
```

**多触发器工作流**可以指定从哪个触发器开始：

```text
test_workflow({
  workflowId: "xxx",
  pinData: { ... },
  triggerNodeName: "Webhook 触发"
})
```

#### Step 4: 检查结果

```text
get_execution({
  workflowId: "xxx",
  executionId: "返回的执行ID",
  includeData: true
})
```

### Mock Testing 常见错误

| 错误 | 原因 | 修复 |
|------|------|------|
| pin data 格式报错 | item 没有 `{json: {...}}` 包裹 | 加上 json 包裹 |
| 节点收到空数据 | 该 pin 的节点漏了 | 检查 prepare_test_pin_data 返回的节点列表 |
| 逻辑节点行为不对 | pin data 的字段和表达式引用不匹配 | 检查表达式引用的字段名是否和 pin data 一致 |
| 多余的 pin 导致逻辑被跳过 | 不该 pin 的逻辑节点也 pin 了 | 只 pin Trigger/凭据节点/HTTP Request |

---

## Real Execution: execute_workflow

### 三种输入类型

根据工作流的触发器类型，传不同的 inputs 结构：

#### Chat 类型（Chat Trigger / Manual Chat Trigger）

```javascript
execute_workflow({
  workflowId: "xxx",
  inputs: {
    type: "chat",
    chatInput: "你好，帮我查询订单"
  }
})
```

#### Form 类型（Form Trigger）

```javascript
execute_workflow({
  workflowId: "xxx",
  inputs: {
    type: "form",
    formData: {
      name: "张三",
      email: "zhang@example.com"
    }
  }
})
```

#### Webhook 类型（Webhook Trigger）

```javascript
execute_workflow({
  workflowId: "xxx",
  inputs: {
    type: "webhook",
    webhookData: {
      method: "POST",
      body: { orderId: "12345", action: "refund" },
      headers: { "content-type": "application/json" },
      query: { source: "test" }
    }
  }
})
```

#### Manual Trigger / Schedule Trigger

不需要传 inputs，直接执行：

```javascript
execute_workflow({
  workflowId: "xxx"
})
```

### executionMode: manual vs production

| 模式 | 说明 | 用途 |
|------|------|------|
| `manual`（默认） | 执行当前版本（包括未发布的修改） | 开发调试 |
| `production` | 执行已发布 (active) 的版本 | 验证线上版本 |

```javascript
// 测试当前版本
execute_workflow({ workflowId: "xxx", executionMode: "manual" })

// 测试已发布版本
execute_workflow({ workflowId: "xxx", executionMode: "production" })
```

---

## Mock vs Real Execution

### Prefer mock testing when

- The workflow calls external APIs
- Credentialed nodes are involved and you don't want to send real messages, write to a database, or consume quota
- Your main goal right now is to confirm the main path, branches, expressions, and field mappings
- **原型阶段** — API 没配好、凭据没填，先验证流程逻辑

### Go straight to real execution when

- You need to verify a real webhook / chat / form entry point
- You need to confirm behavior before going live
- Mock tests have already passed and you're doing final end-to-end validation

---

## What to Look at After a Failure

### 1. If `validate_workflow` reports errors

- Fix code structure, node types, and parameter structure first
- Don't jump to execution until validation passes

### 2. If `test_workflow` fails

Most likely causes:

- Pin data 格式错误（缺少 `{json: {...}}` 包裹）
- 该 pin 的节点没 pin / 不该 pin 的 pin 了
- Expression references a wrong or missing field
- Branch condition doesn't match the test input
- A logic node doesn't handle null, arrays, or objects correctly

### 3. If `execute_workflow` fails

Most likely causes:

- 输入类型传错了（chat/form/webhook 不匹配）
- Real input structure differs from what you assumed
- Credential, permission, network, or external service issue
- Upstream node output doesn't match your mock data

### 4. If you only know "execution failed"

That's not enough information. You must continue:

```text
execute_workflow -> get_execution -> find the failing node -> go to the relevant Skill
```

---

## Failure Type -> Next Action

| What you see | Next step |
|-------------|-----------|
| Can't read a field / expression error | Go to `n8n-expression-syntax` |
| Wrong node parameters | Go to `n8n-node-configuration` |
| Branch, loop, or merge behaving unexpectedly | Go to `n8n-node-pitfalls` |
| Full workflow code fails validation | Go to `n8n-validation-expert` |
| Pin data format error | Check `{json: {...}}` wrapping |
| Input type mismatch on execute | Check trigger type → use matching inputs |

---

## Quick Reference: 原型测试循环

```text
搭建原型
  ↓
validate_workflow  ← 代码层面验证
  ↓
prepare_test_pin_data  ← 获取需要 mock 的节点 schema
  ↓
生成 pin data (每个 item 包裹 {json: {...}})
  ↓
test_workflow  ← mock 执行
  ↓
get_execution (includeData: true)  ← 看结果
  ↓
发现问题 → 改代码 → 回到 validate_workflow
  ↓
全部通过 → execute_workflow 做最终联调
```

---

## Common Misconceptions

### ❌ Running real execution immediately

When multiple external services are involved, failure sources mix together and debugging becomes slow.

### ❌ Assuming mock success means production is fine

Mock tests confirm that the main path and field mappings are roughly correct — not that real credentials, permissions, and external responses will work.

### ❌ Only looking at the final error message

The real insight is not a single error line — it's **which node failed first, what input it received, and what it produced**.

### ❌ Pin data 不包裹 json

每个 item 必须是 `{json: {...}}`，这是 n8n 数据格式的基本要求，不是可选的。

### ❌ 把逻辑节点也 pin 了

Set / If / Code 这些节点应该正常执行，pin 了反而跳过了你想测试的逻辑。

---

## Related Skills

- `n8n-mcp-tools-expert` — overview of all MCP tools
- `n8n-validation-expert` — how to break down validation errors
- `n8n-node-configuration` — how to fix node parameters
- `n8n-expression-syntax` — how to fix expressions
