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

## Mock vs Real Execution

### Prefer mock testing when

- The workflow calls external APIs
- Credentialed nodes are involved and you don't want to send real messages, write to a database, or consume quota
- Your main goal right now is to confirm the main path, branches, expressions, and field mappings

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

- Pin data structure doesn't match what the node expects
- Expression references a wrong or missing field
- Branch condition doesn't match the test input
- A logic node doesn't handle null, arrays, or objects correctly

### 3. If `execute_workflow` fails

Most likely causes:

- Real input structure differs from what you assumed
- Credential, permission, network, or external service issue
- Upstream node output doesn't match your mock data

### 4. If you only know "execution failed"

That's not enough information. You must continue:

```text
execute_workflow -> get_execution -> find the failing node -> go to the relevant Skill
```

---

## Failure Type → Next Action

| What you see | Next step |
|-------------|-----------|
| Can't read a field / expression error | Go to `n8n-expression-syntax` |
| Wrong node parameters | Go to `n8n-node-configuration` |
| Branch, loop, or merge behaving unexpectedly | Go to `n8n-node-pitfalls` |
| Full workflow code fails validation | Go to `n8n-validation-expert` |

---

## Common Misconceptions

### ❌ Running real execution immediately

When multiple external services are involved, failure sources mix together and debugging becomes slow.

### ❌ Assuming mock success means production is fine

Mock tests confirm that the main path and field mappings are roughly correct — not that real credentials, permissions, and external responses will work.

### ❌ Only looking at the final error message

The real insight is not a single error line — it's **which node failed first, what input it received, and what it produced**.

---

## Related Skills

- `n8n-mcp-tools-expert` — overview of all MCP tools
- `n8n-validation-expert` — how to break down validation errors
- `n8n-node-configuration` — how to fix node parameters
- `n8n-expression-syntax` — how to fix expressions
