---
name: n8n-validation-expert
description: |
  Use when validate_workflow returns errors or warnings, when SDK workflow code does not pass validation, or when someone needs help understanding what to fix first.
---

# n8n Validation Expert

In the current MCP version, validation targets the **entire workflow code**, not individual node configs.

The correct mental model is:

```text
write code -> validate_workflow -> read errors -> fix code -> validate again
```

Going back and forth 2–3 rounds is normal.

---

## What to Look at First

### 1. Triage by severity

- **Errors**: Must fix — workflow cannot be created or updated until resolved
- **Warnings**: Workflow can run, but there's likely a real risk
- **Suggestions**: Not blocking — can be addressed in a later round

### 2. Identify the source

Common sources:

- Syntax error in code
- Wrong node type
- Wrong parameter structure
- Wrong node connections
- Expression error

Don't rewrite the whole file at once. First decide which category the error belongs to.

---

## Recommended Validation Loop

### Standard process

```javascript
const result = validate_workflow({ code });
```

Then fix in this order:

1. Syntax and structural errors first
2. Then node type and parameter type errors
3. Then connection and reference errors
4. Finally warnings and suggestions

### Why this order works

Early errors often produce false downstream errors.

For example:

- An unclosed bracket in a code block
- A typo in a node ID
- A parameter object nested at the wrong level

If you leave these unfixed, the error output gets noisier and harder to read.

---

## High-Frequency Errors

### 1. Syntax errors

Signs:

- Code cannot be parsed at all
- A flood of seemingly unrelated errors appear at once

Check first:

- Brackets and braces
- Commas
- String quotes
- Object and array nesting

### 2. Wrong node type

Signs:

- Node type does not exist
- Node name looks guessed

Fix:

```text
search_nodes -> get_node_types -> fill in the correct node ID
```

### 3. Wrong parameter type or structure

Signs:

- Field name is incorrect
- A field expects an object but received a string
- A field only exists under certain operations

Fix:

```text
Re-fetch the type definition for the current operation -> update node(...) accordingly
```

### 4. Connection errors

Signs:

- Isolated node with no connections
- Connected to the wrong branch
- AI node uses wrong connection type

Fix:

- Verify the main workflow path is complete end-to-end
- Check special connection types (e.g. `ai_languageModel`, `ai_tool`)

### 5. Expression errors

Signs:

- Missing `{{ }}`
- Referencing a field path that doesn't exist
- Field names with special characters written incorrectly

Fix:

- Go to `n8n-expression-syntax`
- Confirm the actual data structure the current node receives

---

## Deprecated Approaches

- No longer using legacy node-level validation APIs
- No longer distinguishing legacy profiles
- No longer relying on legacy auto-fix APIs
- No longer patching workflow fragments in isolation

The current path is: **validate the full code, fix the full code**.

---

## Fix Priority

### Must fix immediately

- Syntax errors
- Node type does not exist
- Required field missing
- Broken connection

### Fix in second round

- Warnings
- Suggestions
- Best-practice reminders

---

## Related Skills

- **n8n-mcp-tools-expert** — confirm which tool to call
- **n8n-node-configuration** — revisit fields and operations
- **n8n-expression-syntax** — fix expressions
- **n8n-node-pitfalls** — debug Loop, Merge, AI Agent traps
