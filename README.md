# n8n Skills Collection

**[中文文档](README.zh.md)**

A collection of specialized Skills for building n8n workflows with Claude Code. These Skills guide AI through the entire workflow development lifecycle — from prototyping to node configuration, expression writing, validation, testing, and execution debugging.

---

## What's Included

| Skill | Purpose |
|-------|---------|
| `n8n-prototype` | Rapidly sketch workflow structures using noOp placeholders before building the real thing |
| `n8n-workflow-patterns` | Choose the right architecture pattern (Webhook, API, Database, AI Agent, Scheduled) |
| `n8n-reference-workflow-research` | Find existing workflow examples before designing from scratch |
| `n8n-node-configuration` | Determine correct node parameters and translate them into SDK code |
| `n8n-expression-syntax` | Write and debug n8n `{{ }}` expressions |
| `n8n-mcp-tools-expert` | Complete guide to all 18 n8n MCP tools — search, create, update, test, publish |
| `n8n-validation-expert` | Interpret and fix `validate_workflow` errors and warnings |
| `n8n-node-pitfalls` | Quick reference for common mistakes in Merge, Loop, IF, and Switch nodes |
| `n8n-execution-testing` | Decide between mock testing and real execution; debug failed runs |

---

## Installation

### Option 1: Clone directly

```bash
git clone https://github.com/jqlts1/n8n-skills.git .codex/skills
```

Claude Code automatically picks up all Skills in `.codex/skills/`. No extra config needed.

### Option 2: Symlink (recommended for multiple projects)

Clone once to a central location, then symlink into each project. One `git pull` updates everything.

```bash
# Clone once
git clone https://github.com/jqlts1/n8n-skills.git ~/.local/share/n8n-skills

# Link into your project
SKILLS_REPO=~/.local/share/n8n-skills
SKILLS_DIR=your-project/.codex/skills

mkdir -p $SKILLS_DIR
for skill in n8n-prototype n8n-node-configuration n8n-expression-syntax \
             n8n-mcp-tools-expert n8n-workflow-patterns \
             n8n-reference-workflow-research n8n-validation-expert \
             n8n-node-pitfalls n8n-execution-testing; do
  ln -s $SKILLS_REPO/$skill $SKILLS_DIR/$skill
done

# Update all projects at once
cd ~/.local/share/n8n-skills && git pull
```

### Option 3: Git Submodule (team projects)

```bash
git submodule add https://github.com/jqlts1/n8n-skills.git .codex/skills
git submodule update --init
```

---

## How to Use

Invoke any Skill with a slash command:

```
/skill-name [optional description]
```

Examples:
```
/n8n-prototype design an order processing workflow
/n8n-validation-expert fix this validation error
/n8n-execution-testing workflow failed, where do I look first
```

When triggered, the Skill loads its `SKILL.md` into context and Claude follows the embedded instructions.

---

## When to Use Which Skill

| Skill | Use when... | Don't use when... |
|-------|-------------|-------------------|
| `n8n-prototype` | User says "draw a workflow", "design a flow", or describes a business process to automate | Already writing real SDK code |
| `n8n-workflow-patterns` | Unsure which architecture to use (Webhook / API / DB / AI / Scheduled) | Architecture is already decided |
| `n8n-reference-workflow-research` | Want to find similar examples before designing | Requirements are unique, no reference needed |
| `n8n-node-configuration` | Need to know required fields for a node; translating params into SDK code | Node is already configured, just debugging |
| `n8n-expression-syntax` | Writing `{{ }}` expressions; getting expression errors | No expressions involved |
| `n8n-mcp-tools-expert` | Searching nodes, reading type definitions, creating/updating/finding workflows | Only working with expressions or validation logic |
| `n8n-validation-expert` | `validate_workflow` returns errors or warnings; unsure what to fix first | Code isn't written yet |
| `n8n-node-pitfalls` | Using Merge, Loop Over Items, IF, or Switch; parallel branch issues | Using simple data processing nodes |
| `n8n-execution-testing` | Workflow is built; deciding how to test; execution failed | Still writing the workflow code |

---

## Workflow: Common Multi-Skill Sequences

### Building a new workflow from scratch

```
n8n-reference-workflow-research   → find similar examples
        ↓
n8n-prototype                     → sketch the structure, confirm flow
        ↓
n8n-workflow-patterns             → lock in the architecture pattern
        ↓
n8n-node-configuration            → configure nodes, write SDK code
        ↓
n8n-expression-syntax             → (if expressions needed) validate syntax
        ↓
n8n-mcp-tools-expert              → validate → create_workflow_from_code
        ↓
n8n-validation-expert             → fix errors (usually 2–3 rounds)
        ↓
n8n-execution-testing             → mock test → real execution → debug
```

### Modifying an existing workflow

```
n8n-mcp-tools-expert              → search_workflows → get_workflow_details
        ↓
n8n-node-configuration            → update node params
        ↓
n8n-validation-expert             → validate → fix
        ↓
n8n-mcp-tools-expert              → update_workflow
        ↓
n8n-execution-testing             → verify the change
```

### Debugging a failed workflow

```
n8n-execution-testing             → identify which node failed
        ↓
n8n-node-pitfalls                 → (flow control nodes) check common traps
   or
n8n-expression-syntax             → (expression errors) fix syntax
   or
n8n-node-configuration            → (wrong params) check field structure
        ↓
n8n-validation-expert             → re-validate after fixes
```

---

## Skill Reference

### `n8n-prototype`

Quickly visualize a business process as an n8n workflow using noOp placeholder nodes and Sticky Notes.

**Three prototype modes:**

| Mode | When to use |
|------|------------|
| Skeleton | Discuss flow logic, no tech decisions yet |
| Mixed ⭐ | Core nodes are real, business logic uses noOp |
| Executable | Flow is confirmed, build a runnable version directly |

**noOp naming convention:** `[Tag] Verb + Object`
- `[API] Call payment endpoint`
- `[DB] Query order records`
- `[Notify] Send Slack message`
- `[AI] Classify content`

---

### `n8n-workflow-patterns`

Five core patterns — use this Skill to decide the overall structure before writing any code.

| Pattern | Best for |
|---------|---------|
| Webhook processing | External push events, form submissions |
| HTTP API integration | Third-party API calls, data sync |
| Database operations | ETL, periodic cleanup, reconciliation |
| AI Agent workflow | Intelligent chat, multi-tool agents, structured output |
| Scheduled tasks | Periodic jobs, batch processing, reports |

---

### `n8n-node-configuration`

Configuration is **operation-aware**: the same node has different required fields depending on `resource` + `operation`. Always call `get_node_types` with discriminators before writing SDK code.

```
search_nodes → get_node_types → read TypeScript types → write node() params
```

---

### `n8n-expression-syntax`

All dynamic values use double curly braces:

```javascript
✅ {{ $json.email }}
✅ {{ $json["field name"] }}
✅ {{ $node["Node Name"].json.field }}
❌ $json.email          // no braces
❌ { $json.email }      // single braces
```

Key variables: `$json` (current node), `$node["name"]` (other nodes), `$now`, `$today`, `$input.all()`

---

### `n8n-mcp-tools-expert`

Full reference for all 18 n8n MCP tools across 8 categories:

| Category | Tools |
|----------|-------|
| Node discovery | `search_nodes` `get_node_types` `get_suggested_nodes` |
| SDK reference | `get_sdk_reference` |
| Workflow management | `validate_workflow` `create_workflow_from_code` `update_workflow` `get_workflow_details` `search_workflows` |
| Execution | `execute_workflow` `get_execution` |
| Testing | `prepare_test_pin_data` `test_workflow` |
| Lifecycle | `publish_workflow` `unpublish_workflow` `archive_workflow` |
| Organization | `search_projects` `search_folders` |

> `update_workflow` is a **full replacement**, not a patch. Always `validate_workflow` before creating or updating.

---

### `n8n-validation-expert`

Fix `validate_workflow` errors in the right order:

1. Syntax and structural errors first
2. Then node type and parameter type errors
3. Then connection and reference errors
4. Finally warnings and suggestions

Early errors often produce false downstream errors — fix top-down, not all at once.

---

### `n8n-node-pitfalls`

**Merge node:** `Number of Inputs` defaults to 2. If you're merging more than 2 branches, set it manually in Settings.

**Loop Over Items:** Most of the time you don't need this node — n8n automatically runs each node once per item. Only use it when you need to control batch size or add delays between batches. The `loop` output port (index 1) **must be wired back** to the Loop node itself, or it only runs once.

---

### `n8n-execution-testing`

| Situation | Recommended tool |
|-----------|-----------------|
| Don't want to hit real external services | `prepare_test_pin_data` + `test_workflow` |
| Need to verify real webhook / chat entry | `execute_workflow` |
| Already failed, need to find which node | `get_execution` → inspect node-level output |

Recommended order: `validate_workflow` → mock test → real execution → `get_execution` on failure

---

## Requirements

- [Claude Code](https://claude.ai/code) CLI or IDE extension
- An n8n instance with the [n8n MCP server](https://github.com/n8n-io/n8n-mcp) configured

---

## License

MIT
