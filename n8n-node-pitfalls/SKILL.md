---
name: n8n-node-pitfalls
description: |
  Quick reference for common mistakes and pitfalls in n8n nodes. Use when:
  - Working with Merge, Loop Over Items, IF, Switch, or other flow control nodes
  - Parallel branches are not merging correctly
  - Node output is not what you expected
  - You need to know the common traps for a specific node
---

# n8n Node Pitfalls

Common traps and correct patterns for frequently misused nodes.

---

## Merge Node

### Core Issue
> ⚠️ **Number of Inputs defaults to 2!** Merging more than 2 branches requires manual adjustment.

### Parameters

| Parameter | Default | Notes |
|-----------|---------|-------|
| Mode | Append | Merge strategy; Append is most common |
| Number of Inputs | 2 | **Must match the actual number of incoming branches** |

### Common Mistakes

```
❌ 5 branches connected to Merge, but Number of Inputs = 2
   → Only 2 input ports exist; the other 3 cannot connect

❌ Skipping Merge entirely — connecting multiple nodes directly to the same target
   → Each source node triggers the target independently
   → Target executes multiple times instead of waiting for all inputs

✅ Correct approach:
   1. Add a Merge node
   2. Settings → Number of Inputs = actual branch count
   3. Connect each branch to a different input port on Merge
   4. Merge waits for all inputs before producing output
```

### Where to Configure
- n8n UI → Merge node → Settings → Number of Inputs

---

## Loop Over Items Node

### Core Issue
> ⚠️ **You probably don't need this node!** n8n automatically runs each node once per item.
> Only use it when you need to **control batch size** or **add delays between batches**.

### Two Output Ports

| Port | Index | Description |
|------|-------|-------------|
| done | 0 (first) | Emits after the loop **completes** |
| loop | 1 (second) | Current batch — must be **wired back** to form the loop |

### Common Mistakes

```
❌ Only connecting the done port, forgetting to wire loop back
   → Executes only once, never loops

❌ Swapping done and loop
   → Logic is completely broken

❌ Wiring the loop back to the wrong input port
   → Infinite loop or skipped processing

✅ Correct wiring:
   Loop Over Items
       ├── [done:0] → post-loop processing
       └── [loop:1] → process item → Wait (optional) → back to Loop input
```

---

## IF Node

### Core Issue
> ⚠️ **Output port order matters!** `true` is first, `false` is second.

### Two Output Ports

| Port | Index | Description |
|------|-------|-------------|
| true | 0 (first) | Emits when condition is true |
| false | 1 (second) | Emits when condition is false |

### Common Mistakes

```
❌ Mixing up true/false when using sourceIndex
   → sourceIndex=0 is true, sourceIndex=1 is false

❌ Only connecting one output port
   → Data on the other branch is silently dropped

✅ When using MCP, prefer semantic parameters:
   branch="true" or branch="false"
   instead of sourceIndex=0 or sourceIndex=1
```

---

## Switch Node

### Core Issue
> ⚠️ **Output port count equals rule count!** Each rule gets its own output port.

### Common Mistakes

```
❌ Defining 3 rules but only connecting 2 outputs
   → Data matching the 3rd rule has nowhere to go

❌ Miscounting sourceIndex when wiring
   → Data is routed to the wrong branch

✅ When using MCP, prefer semantic parameters:
   case=0, case=1, case=2...
   instead of manually calculating sourceIndex
```

---

## HTTP Request Node

### Core Issues
> ⚠️ **Watch the output property name when downloading files!** Default is `data`, but it's configurable.
> ⚠️ **Use HTTP Request's built-in pagination first!** Don't reach for Loop before trying it.

### File Download Configuration

| Parameter | Path | Description |
|-----------|------|-------------|
| Response Format | options.response.response.responseFormat | Set to `file` |
| Output Property Name | options.response.response.outputPropertyName | Binary property name |

### Common Mistakes

```
❌ Downloading multiple files with the same outputPropertyName
   → Later files overwrite earlier ones

❌ Forgetting to set responseFormat: file
   → Returns text instead of binary

✅ Correct approach:
   - Use a unique outputPropertyName per download node
   - e.g.: image1, image2, image3...
```

### Pagination Note

- For multi-page APIs, use HTTP Request's built-in pagination config first
- Only use Loop Over Items when you need manual batching, delays, or throttling
- See [docs/节点操作类/HTTP分页.md](/docs/节点操作类/HTTP分页.md) for details

---

## Code Node

### Core Issue
> ⚠️ **The `mode` parameter determines execution behavior!**

### Two Modes

| Mode | Description |
|------|-------------|
| runOnceForEachItem | Runs once per item (default) |
| runOnceForAllItems | Runs once for all items together |

### Common Mistakes

```
❌ Trying to merge multiple items but using runOnceForEachItem
   → Each item is processed in isolation; merging is not possible

❌ Trying to process items individually but using runOnceForAllItems
   → Must write your own loop logic

✅ To merge multiple items:
   mode: "runOnceForAllItems"
   use $input.all() to access all items
```

### Merging Binary Data Example

```javascript
// mode: runOnceForAllItems
const allItems = $input.all();
const mergedBinary = {};

for (const item of allItems) {
  if (item.binary) {
    Object.assign(mergedBinary, item.binary);
  }
}

return [{
  json: { message: `Merged ${Object.keys(mergedBinary).length} files` },
  binary: mergedBinary
}];
```

---

## AI Agent Node

### Core Issue
> ⚠️ **Must connect a Chat Model!** Without a language model, the Agent cannot function.

### Required Connections

| Connection Type | Description |
|----------------|-------------|
| ai_languageModel | **Required** — connect a Chat Model node |
| ai_tool | Optional — connect tool nodes |
| ai_memory | Optional — connect memory nodes |
| ai_outputParser | Optional — connect an output parser |

### Common Mistakes

```
❌ Creating an AI Agent without connecting a Chat Model
   → Validation fails; workflow cannot run

❌ Connecting Chat Model using the main connection type
   → Must use ai_languageModel type instead

✅ Correct connection:
   sourceOutput: "ai_languageModel"
   not: sourceOutput: "main"
```

---

## Parallel Branches → Merge Pattern

When running multiple operations in parallel and merging results:

```
Trigger
    ├── Operation 1 ──┐
    ├── Operation 2 ──┤
    ├── Operation 3 ──┼──→ Merge (Number of Inputs = N) ──→ downstream
    ├── ...           ──┤
    └── Operation N ──┘
```

### Key Configuration

1. **Merge node**: Number of Inputs = number of parallel branches
2. **Each branch**: Connect to a different Merge input port (targetIndex 0, 1, 2...)
3. **Downstream node**: Connect only from Merge's output

---

## Quick Checklist

When using flow control nodes, verify:

- [ ] Merge: Number of Inputs matches the actual branch count?
- [ ] Loop: loop port is wired back?
- [ ] IF: both true and false branches are connected?
- [ ] Switch: every case has an output connection?
- [ ] Code: mode is set correctly?
- [ ] AI Agent: Chat Model is connected?
- [ ] HTTP download: outputPropertyName is unique per node?
