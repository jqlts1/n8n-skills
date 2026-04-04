---
name: n8n-node-configuration
description: |
  Use when configuring n8n nodes, determining required fields for a specific resource or operation, or translating node parameter knowledge into SDK code.
---

# n8n Node Configuration

Separating "what fields does this node have" from "how do I write it in workflow code" leads to more reliable results.

In the current MCP version, node configuration no longer revolves around legacy node-level validation tools. The main path is:

```text
search_nodes -> get_node_types -> read type definitions -> write SDK code -> validate_workflow
```

---

## Core Principles

### 1. Configuration is operation-aware

The same node has different required fields depending on `resource` and `operation`.

**Example: Slack**

```javascript
// Send a message
node('n8n-nodes-base.slack', {
  resource: 'message',
  operation: 'post',
  channel: '#general',
  text: 'Hello',
});

// Update a message
node('n8n-nodes-base.slack', {
  resource: 'message',
  operation: 'update',
  messageId: '123',
  text: 'Updated',
});
```

Don't copy fields from one operation and mechanically paste them into another.

### 2. Field dependencies come from type definitions, not from a separate dependency tool

Old approach: "essentials first, then dependencies, then info."

New approach:

1. `search_nodes` to find the correct node
2. `get_node_types` with the discriminators returned by search
3. Read the TypeScript type definition for the specific operation
4. Write `node(...)` parameters based on those types

If a field only appears under certain conditions, understand it from the type definition for that operation — don't look for a removed dependency tool.

### 3. Write the minimum viable configuration first

Start with what's needed to express the current operation, then add optional fields.

**HTTP Request minimum POST example:**

```javascript
node('n8n-nodes-base.httpRequest', {
  method: 'POST',
  url: 'https://api.example.com/users',
  authentication: 'none',
  sendBody: true,
  body: {
    contentType: 'json',
    content: {
      name: '={{$json.name}}',
      email: '={{$json.email}}',
    },
  },
});
```

### 4. Validation targets the entire workflow code

The current MCP version has no fast path for "validate a single node in isolation." Put the node into the full workflow code and run:

```javascript
validate_workflow({ code: workflowCode });
```

This way errors reflect real workflow context.

---

## Recommended Workflow

### Standard process

```text
1. Identify which node and which operation you need
2. search_nodes({queries: [...]})
3. get_node_types({nodeIds: [...]})
4. Write a minimal node(...)
5. Place it in the full workflow code
6. validate_workflow({code})
7. Based on errors: add fields, fix types, fix expressions
```

### When to re-fetch type definitions

- Switching to a different `resource`
- Switching to a different `operation`
- Switching to a different authentication method on the same node
- A field only appears under specific conditions
- Unsure whether the field is named `method`, `type`, `mode`, or something else

---

## Common Scenarios

### HTTP Request

Confirm first:

- Request method
- Whether `sendBody` is needed
- Body content structure
- Authentication method

Common traps:

- Forgetting `sendBody: true` for POST / PUT / PATCH
- Guessing body field structure directly
- Writing expressions as plain strings

### Webhook

Confirm first:

- How it's triggered
- Path
- Response mode

Common traps:

- Forgetting that webhook data is often under `$json.body` downstream
- Building only the receive side without adding a response node or downstream processing

### Slack / Gmail / Database nodes

Confirm first:

- Resource type
- Specific operation
- Credential method
- Required fields specific to that operation

Common traps:

- Required fields differ significantly across operations of the same service
- Copying fields from an old example that no longer apply to the current operation

---

## Anti-patterns

### ❌ Anti-pattern 1: Guessing parameters, then fixing

This leads to repeated dead ends. Fetch `get_node_types` first, then write.

### ❌ Anti-pattern 2: Looking at just the node name, not the operation

"How do I configure the Slack node?" is too broad. Narrow it down to "which resource and which operation on Slack."

### ❌ Anti-pattern 3: Treating field shape docs as final code

Documents like [OPERATION_PATTERNS.md](OPERATION_PATTERNS.md) help you understand parameter structure — they don't replace calling `get_node_types`.

### ❌ Anti-pattern 4: Validating a node outside of workflow context

A node that looks correct in isolation may still fail inside a workflow. Always run `validate_workflow` with the complete code.

---

## Quick Decision Guide

### Use `search_nodes` first when

- Unsure of the node name
- Unsure of the official node ID
- Unsure which service node to use

### Use `get_node_types` first when

- You know the node name but not the fields
- You know the operation but not the required fields
- A field's existence depends on a condition

### Use validation results first when

- Code is already written
- Unsure whether the issue is a missing field, wrong type, or expression error
- Unsure whether the problem is in the node itself or in its connections

---

## Related Skills

- **n8n-mcp-tools-expert** — explains when to use which MCP tool
- **n8n-validation-expert** — interprets `validate_workflow` results
- **n8n-expression-syntax** — fixes expressions
- **n8n-node-pitfalls** — covers node-level common traps
- **HTTP pagination docs** — when pagination is needed, check [docs/节点操作类/HTTP分页.md](/docs/节点操作类/HTTP分页.md) before reaching for Loop
