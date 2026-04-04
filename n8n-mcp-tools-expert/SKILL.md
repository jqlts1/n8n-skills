---
name: n8n-mcp-tools-expert
description: |
  Use when choosing between n8n MCP tools, searching nodes, reading node types, validating SDK workflow code, or creating, updating, executing, and publishing workflows.
---

# n8n MCP Tools Expert

Complete guide for building workflows with n8n MCP tools.

---

## Tool Overview (18 tools, 8 categories)

| Category | Tool | Purpose |
|----------|------|---------|
| **Node Discovery** | `search_nodes` | Search nodes by keyword |
| | `get_node_types` | Fetch TypeScript type definitions for nodes (**required before writing code**) |
| | `get_suggested_nodes` | Get recommended nodes by workflow category |
| **SDK Reference** | `get_sdk_reference` | Fetch workflow SDK docs (patterns, expressions, rules, etc.) |
| **Workflow Management** | `validate_workflow` | Validate SDK code |
| | `create_workflow_from_code` | Create a workflow from SDK code |
| | `update_workflow` | Update a workflow with SDK code (full replacement) |
| | `get_workflow_details` | Get workflow details |
| | `search_workflows` | Search existing workflows |
| **Execution** | `execute_workflow` | Execute a workflow (supports chat/form/webhook input) |
| | `get_execution` | Get execution result details |
| **Testing** | `prepare_test_pin_data` | Generate test pin data schema for a workflow |
| | `test_workflow` | Run a workflow with pin data, bypassing external services |
| **Lifecycle** | `publish_workflow` | Publish (activate) a workflow |
| | `unpublish_workflow` | Unpublish (deactivate) a workflow |
| | `archive_workflow` | Archive a workflow |
| **Organization** | `search_projects` | Search projects |
| | `search_folders` | Search folders within a project |

---

## Core Workflows

### Building a new workflow (most common)

```
1. search_nodes({queries: ["keyword"]})         → find node IDs
2. get_node_types({nodeIds: [...]})              → get parameter type definitions
3. get_sdk_reference({section: "patterns"})     → learn SDK syntax
4. Write SDK code                               → TypeScript/JavaScript
5. validate_workflow({code: "..."})             → validate code
6. create_workflow_from_code({code: "..."})     → create workflow
7. publish_workflow({workflowId: "..."})        → activate
```

### Modifying an existing workflow

```
1. get_workflow_details({workflowId: "..."})    → check current state
2. Update SDK code
3. validate_workflow({code: "..."})             → validate
4. update_workflow({workflowId, code})          → update (full replacement)
```

### Finding and discovering workflows

```
search_workflows({query: "keyword"})            → find existing workflows
search_nodes({queries: ["slack", "webhook"]})   → find nodes
get_suggested_nodes({categories: ["chatbot"]})  → get recommendations by category
```

### Testing without hitting real external services

```
1. prepare_test_pin_data({workflowId: "..."})   → get expected data schema per node
2. Generate mock data based on the schema
3. test_workflow({workflowId, pinData})         → run with pin data, skip external calls
```

> **What gets pinned (replaced with mock data)?**
> - Trigger nodes
> - Nodes with credentials (e.g. HTTP Request with auth, Slack, etc.)
>
> **What runs normally?**
> - Logic nodes: Set, If, Code, Switch, etc.
> - Credential-free I/O nodes: Execute Command, file read/write, etc.

### Which entry point to use

| Goal | Primary entry | Notes |
|------|--------------|-------|
| Modify your own n8n instance | `n8n-mcp` | Can create, update, execute, and debug |
| Look up official SDK / node docs | `n8n-docs` | Official docs take priority |
| Find reference workflow examples | See `docs/API接口/参照资源接口.md` | This is a reference resource, not an MCP tool |

---

## Tool Details

### Detailed guides

- **Node discovery tools** → [SEARCH_GUIDE.md](SEARCH_GUIDE.md)
- **Validation tools** → [VALIDATION_GUIDE.md](VALIDATION_GUIDE.md)
- **Workflow management tools** → [WORKFLOW_GUIDE.md](WORKFLOW_GUIDE.md)

---

## Common Mistakes

### ❌ Mistake 1: Writing code without calling `get_node_types` first

```javascript
// ❌ Guessing parameter names → produces invalid workflows
node('n8n-nodes-base.httpRequest', { url: '...', type: 'POST' })

// ✅ Call get_node_types first to get the correct parameter names
get_node_types({nodeIds: [{nodeId: 'n8n-nodes-base.httpRequest'}]})
// Discover that the parameter is `method`, not `type`
node('n8n-nodes-base.httpRequest', { url: '...', method: 'POST' })
```

### ❌ Mistake 2: Creating without validating

```javascript
// ❌ Skip validation and create directly
create_workflow_from_code({code: "..."})  // may fail

// ✅ Validate first, then create
validate_workflow({code: "..."})          // catch errors
create_workflow_from_code({code: "..."})  // create once confirmed valid
```

### ❌ Mistake 3: Wrong `search_nodes` parameter format

```javascript
// ❌ Passing a single string
search_nodes({query: "slack"})

// ✅ queries is an array
search_nodes({queries: ["slack"]})

// ✅ Multiple queries in one call
search_nodes({queries: ["slack", "webhook", "schedule"]})
```

### ❌ Mistake 4: Calling `get_node_types` without discriminators

```javascript
// ❌ Only passing nodeId — returned type definition may be too broad
get_node_types({nodeIds: ["n8n-nodes-base.httpRequest"]})

// ✅ Include the discriminators returned by search_nodes
get_node_types({nodeIds: [{
  nodeId: "n8n-nodes-base.httpRequest",
  // include discriminators from search_nodes results
}]})
```

### ❌ Mistake 5: Trying to do partial updates

```javascript
// ❌ Old approach: patch a small piece of config
// ✅ New approach: replace the full code
// 1. Fetch current workflow
get_workflow_details({workflowId: "..."})
// 2. Edit the complete SDK code
// 3. Replace entirely
update_workflow({workflowId: "...", code: "complete new code"})
```

---

## Best Practices

### ✅ Do

- **Always call `get_node_types` before writing code** — never guess parameter names
- **Use `get_sdk_reference` to learn SDK syntax** — especially the first time
- **Always `validate_workflow` before create/update** — avoid failures
- **Pass discriminators from `search_nodes` to `get_node_types`** — get more precise types
- **Use `get_suggested_nodes` for technology selection** — covers 11 common categories
- **Decide between mock and real execution upfront** — `test_workflow` for safe validation, `execute_workflow` for final integration
- **Always check `get_execution` after failure** — find the first broken node before fixing

### ❌ Don't

- **Don't guess parameter names** — always get them from `get_node_types`
- **Don't skip validation** — `validate_workflow` is mandatory
- **Don't expect partial updates** — `update_workflow` is a full replacement
- **Don't forget `publish_workflow`** — workflows must be published to run in production mode

---

## Related Skills

- **n8n-expression-syntax** — writing expressions in workflow fields
- **n8n-workflow-patterns** — workflow architecture patterns
- **n8n-validation-expert** — interpreting validation errors
- **n8n-execution-testing** — choosing between mock and real execution, debugging failures
- **n8n-node-configuration** — operation-level node configuration guide
- **n8n-node-pitfalls** — common node traps
