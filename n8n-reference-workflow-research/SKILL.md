---
name: n8n-reference-workflow-research
description: |
  Use when looking for existing n8n workflow examples, asking for similar workflow references before building, comparing reusable patterns, or deciding whether to adapt a reference workflow or design one from scratch.
---

# n8n Reference Workflow Research

This Skill is about **finding references first, then deciding how to build** — not jumping straight into designing from scratch.

---

## When to Use

- User says "is there a similar workflow I can reference?"
- User says "find me an existing solution first"
- User says "search for a few examples before I decide how to build this"
- You're unsure whether to design the flow yourself or start from an existing example

---

## Source Priority

### 1. Your own instance first

Prefer:

- `search_workflows`
- `get_workflow_details`

Your own instance has the most relevant, reusable assets.

### 2. Reference resource API next

Check this document:

- [docs/API接口/参照资源接口.md](/docs/API接口/参照资源接口.md)

Useful for:

- Searching public reference workflows
- Viewing details
- Downloading JSON
- Viewing Mermaid diagrams

### 3. Official docs last — for validation, not discovery

Use:

- `n8n-docs`

Not for finding examples. Use it to confirm:

- Whether a node capability actually exists
- Whether a particular approach is officially supported
- Whether there are known limitations

---

## Recommended Process

```text
1. Extract the goal: trigger type, core action, output result
2. search_workflows — check if your own instance has something similar
3. If not enough, check the reference resource API
4. Extract a reusable skeleton from the candidates
5. Use n8n-docs to verify key nodes and capabilities
6. Decide whether to proceed to n8n-prototype or n8n-workflow-patterns
```

---

## Key Information to Look For

- **Trigger type**: Webhook / Schedule / Form / AI / Manual
- **Core actions**: Pull data, write to DB, send message, classify, approve
- **External integrations**: Slack, DingTalk, databases, HTTP APIs, AI models
- **Complexity**: Simple main path, branching, looping, caching

Don't search with a broad single keyword. Include trigger + action + output together.

---

## What to Deliver as Output

Research results should be organized into these 4 parts:

1. The closest matching reference candidates
2. Flow skeleton that can be directly reused
3. Parts that are only useful as inspiration — not for direct copying
4. Which Skill to go to next

### Common next steps

- Structure not decided yet → `n8n-prototype`
- Pattern not decided yet → `n8n-workflow-patterns`
- Already know which nodes to use → `n8n-node-configuration`

---

## Important Reminders

### Reference workflows are not production-ready

Focus on:

- Whether the structure has reuse value
- Whether the node combination is reasonable
- How branching, looping, caching, and error handling are designed

Don't copy external credentials, field names, paths, or business rules directly.

### The reference resource API is not your instance's MCP

It only helps you find examples — it does not modify your own workflows.

### Find the skeleton first, configure details later

The goal of the research phase is not to extract every field. The goal is to judge:

- Whether a reference is worth using
- Which parts can be reused
- What's the fastest path to implementation

---

## Common Mistakes

### ❌ Judging only by title similarity

Similar titles don't mean the flow is actually reusable. Always check trigger type, core nodes, and output.

### ❌ Downloading JSON and copying it directly

Reference workflows are usually suitable for borrowing structure — not for running directly in your instance.

### ❌ Continuing to search after finding good candidates

Once you have 1–3 sufficiently close examples, start extracting the skeleton. Don't keep searching indefinitely.

---

## Related Skills

- `n8n-prototype` — turn research results into a discussable flow skeleton
- `n8n-workflow-patterns` — decide which workflow pattern to use
- `n8n-mcp-tools-expert` — when you need to go back to your instance MCP or official docs
- `n8n-node-configuration` — when you're ready to configure real nodes
