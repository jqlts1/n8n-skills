---
name: n8n-workflow-patterns
description: |
  Use when designing a new n8n workflow, choosing a workflow structure, or deciding between webhook, API integration, database, AI agent, and scheduled task patterns.
---

# n8n Workflow Patterns

This Skill is about **how to structure the workflow**, not how to configure individual fields.

If you know the business goal but not what the workflow should look like, start here to pick a pattern.

---

## Five Core Patterns

### 1. Webhook Processing

```text
Webhook -> Validate -> Transform -> Respond / Notify
```

Best for:

- Receiving pushes from external systems
- Form submissions
- Instant business callbacks

### 2. HTTP API Integration

```text
Trigger -> HTTP Request -> Transform -> Downstream action
```

Best for:

- Calling third-party APIs
- Pulling data
- Syncing with external services

### 3. Database Operations

```text
Trigger -> Query / Write -> Transform -> Validate / Notify
```

Best for:

- Data synchronization
- ETL pipelines
- Periodic cleanup or reconciliation

> **Tip**: 原型阶段或轻量需求优先用 **Data Table**（n8n 内置，无需外部数据库）。API 缓存、去重、配置表等场景特别适合。详见 [database_operations.md](database_operations.md)

### 4. AI Agent Workflow

```text
Trigger -> AI Agent -> Tools / Memory / Structured Output -> Result
```

Best for:

- Intelligent Q&A
- Classification, summarization, extraction
- AI flows that require tool calls

### 5. Scheduled Tasks

```text
Schedule -> Fetch -> Process -> Output -> Log
```

Best for:

- Daily / weekly reports
- Periodic health checks
- Timed sync jobs

---

## General Checklist

### Planning phase

- [ ] Identify which of the five patterns fits
- [ ] List the trigger, core processing, and output nodes
- [ ] Decide how failures should be handled

### Implementation phase

- [ ] Use `search_nodes` to find nodes
- [ ] Use `get_node_types` to confirm operations and fields for key nodes
- [ ] Organize nodes into a clear main path
- [ ] Only add branches, loops, and merges when the logic actually requires them

### Validation phase

- [ ] Run `validate_workflow` on the full code
- [ ] Test the main path with sample data
- [ ] Check empty data, edge cases, and failure branches

### Publishing phase

- [ ] Review workflow settings
- [ ] Use `publish_workflow` when ready to go live
- [ ] Monitor the first few executions

---

## How This Skill Works with Others

- **n8n-mcp-tools-expert** — find tools and nodes
- **n8n-node-configuration** — configure individual nodes
- **n8n-expression-syntax** — write expressions
- **n8n-validation-expert** — interpret validation results
- **n8n-node-pitfalls** — debug special node traps

---

## Common Mistakes

### ❌ Piling on nodes before picking a pattern

Define the structure first, then add nodes.

### ❌ Treating an architecture problem as a configuration problem

Some flows are broken not because a field is missing, but because the whole structure is wrong.

### ❌ Ignoring the trigger and output

No matter how well the middle processing is built, a workflow without a clear entry point and exit is incomplete.

---

## Deep Dives

- [webhook_processing.md](webhook_processing.md)
- [http_api_integration.md](http_api_integration.md)
- [database_operations.md](database_operations.md)
- [ai_agent_workflow.md](ai_agent_workflow.md)
- [scheduled_tasks.md](scheduled_tasks.md)
