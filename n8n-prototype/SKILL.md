---
name: n8n-prototype
description: |
  Quickly sketch n8n workflow prototypes. Use before formal development to design flow structure using noOp placeholder nodes and Sticky Notes.

  Trigger when:
  - User asks to "draw a workflow", "design a flow", or "build a prototype"
  - User describes a business process that needs to be turned into an n8n workflow
  - User wants to quickly visualize the structure of an automation process
---

# n8n Workflow Prototyping

Quickly sketch an n8n workflow structure, confirm the flow first, then progressively replace placeholders with real nodes.

---

## Prototype Mode Selection

Before starting, confirm which prototype mode the user needs:

| Mode | Description | When to use |
|------|-------------|-------------|
| **Skeleton** | All noOp + Sticky Notes; structure only | Quickly discussing flow logic before any tech decisions |
| **Mixed** ⭐ recommended | Real nodes for core logic, noOp for business steps | Flow is roughly decided; need to validate feasibility |
| **Executable** | Real nodes wherever possible; can be tested immediately | Flow is confirmed; building a runnable version directly |

### Mixed Mode — Which Nodes Must Be Real

| Node type | Must use real node | Can use noOp |
|-----------|-------------------|--------------|
| Trigger | ✅ Always | — |
| Flow control (If / Switch / Merge) | ✅ Always | — |
| AI Agent | ✅ Recommended | Simple cases can use noOp |
| Data Table cache | ✅ Recommended | — |
| HTTP Request | Optional | ✅ Can use noOp |
| Third-party integrations (Slack, DingTalk, etc.) | — | ✅ Use noOp |
| Business logic processing | — | ✅ Use noOp |

---

## Core Rules

1. **noOp placeholders** — use `n8n-nodes-base.noOp` for unimplemented business logic; express purpose through naming
2. **Always include a trigger** — default to `n8n-nodes-base.manualTrigger`; use another if specified
3. **Clear naming** — format: `Verb + Object`, e.g. "Parse PDF", "Send Notification"
4. **Add Sticky Notes** — annotate the flow with sticky notes so the prototype is self-explanatory
5. **Layout convention** — start at [250, 300]; horizontal spacing 200px; vertical spacing 150px

---

## noOp Naming Convention

### Format
```
[Tag] Verb + Object
```

### Type Tags (optional)
| Tag | Meaning | Example |
|-----|---------|---------|
| `[API]` | Calls an external API | `[API] Fetch weather data` |
| `[DB]` | Database operation | `[DB] Query user record` |
| `[Notify]` | Sends a notification or message | `[Notify] Send Slack message` |
| `[File]` | File processing | `[File] Parse Excel` |
| `[AI]` | AI / LLM processing | `[AI] Classify content` |
| `[Manual]` | Requires human intervention | `[Manual] Review and approve` |

### Naming Examples
```
✅ Good names:
- [API] Call payment API
- [Notify] Send DingTalk message
- [DB] Update order status
- [File] Generate PDF report
- Calculate discount amount
- Format output data

❌ Avoid:
- Process (too vague)
- Step 1 (meaningless)
- TODO (unclear)
```

---

## From Prototype to Production: Replacement Guide

### Phase 1: Skeleton prototype
```
Manual Trigger → [API] Fetch data → [AI] Analyze → If condition → [Notify] Send result
                    (noOp)              (noOp)                          (noOp)
```

### Phase 2: Replace core nodes
```
Manual Trigger → HTTP Request → AI Agent → If condition → [Notify] Send result
                  (real node)   (real node)                    (noOp)
```

### Phase 3: Full implementation
```
Manual Trigger → HTTP Request → AI Agent → If condition → Slack / DingTalk
                  (configured)  (configured)               (configured)
```

### From prototype to workflow code

Once the prototype is confirmed, follow the new MCP implementation path:

```text
search_nodes -> get_node_types -> write SDK code -> validate_workflow -> create_workflow_from_code
```

The prototype confirms the structure; the actual creation is done by `create_workflow_from_code`.

### Replacement Reference

| noOp placeholder | Replace with | Key config |
|-----------------|-------------|------------|
| `[API] Call XXX` | HTTP Request | URL, Method, Headers, Body |
| `[DB] Query data` | Data Table / Postgres | Table name, query conditions |
| `[AI] Process` | AI Agent + Chat Model | Prompt, Output Parser |
| `[File] Parse PDF` | Extract from File | File path, parse format |
| `[File] Generate Excel` | Convert to File | Data mapping |
| `[Notify] Send DingTalk` | HTTP Request (DingTalk Webhook) | Webhook URL, message format |
| `[Notify] Send email` | Send Email | SMTP config, recipients |
| `[Manual] Review` | Wait + Form | Wait condition, form fields |

---

## Node Quick Reference

### Triggers

| Name | Type | Version | Notes |
|------|------|---------|-------|
| Manual Trigger | `n8n-nodes-base.manualTrigger` | 1 | Click to trigger; best for testing |
| Webhook | `n8n-nodes-base.webhook` | 2 | HTTP endpoint trigger |
| Schedule Trigger | `n8n-nodes-base.scheduleTrigger` | 1.2 | Cron-based scheduled tasks |
| Form Trigger | `n8n-nodes-base.formTrigger` | 2.2 | Web form submission |
| Chat Trigger | `@n8n/n8n-nodes-langchain.chatTrigger` | 1.4 | AI chat entry point |
| Manual Chat Trigger | `@n8n/n8n-nodes-langchain.manualChatTrigger` | 1 | Manual chat testing |
| Error Trigger | `n8n-nodes-base.errorTrigger` | 1 | Fires when another workflow errors |
| Execute Workflow Trigger | `n8n-nodes-base.executeWorkflowTrigger` | 1 | Fires when called by another workflow |

### Flow Control

| Name | Type | Version | Notes |
|------|------|---------|-------|
| If | `n8n-nodes-base.if` | 2.2 | Binary branch (true/false) |
| Switch | `n8n-nodes-base.switch` | 3.2 | Multi-condition branching |
| Filter | `n8n-nodes-base.filter` | 2.2 | Filter data items |
| Merge | `n8n-nodes-base.merge` | 3.1 | Merge multiple branches |
| Split In Batches | `n8n-nodes-base.splitInBatches` | 3.1 | Batch loop processing |
| Loop Over Items | `n8n-nodes-base.splitInBatches` | 3.1 | Per-item processing (same node) |
| Wait | `n8n-nodes-base.wait` | 1.1 | Pause / delay |
| Stop and Error | `n8n-nodes-base.stopAndError` | 1 | Throw error and stop |

### Data Processing

| Name | Type | Version | Notes |
|------|------|---------|-------|
| Set (Edit Fields) | `n8n-nodes-base.set` | 3.4 | Set / edit fields, mock data |
| Code | `n8n-nodes-base.code` | 2 | Run JS/Python code |
| Split Out | `n8n-nodes-base.splitOut` | 1 | Split array into multiple items |
| Aggregate | `n8n-nodes-base.aggregate` | 1 | Merge multiple items into one |
| Summarize | `n8n-nodes-base.summarize` | 1 | Sum / count / max etc. |
| Sort | `n8n-nodes-base.sort` | 1 | Sort items |
| Limit | `n8n-nodes-base.limit` | 1 | Limit item count |
| Remove Duplicates | `n8n-nodes-base.removeDuplicates` | 1 | Deduplicate |
| Compare Datasets | `n8n-nodes-base.compareDatasets` | 2.2 | Compare two datasets |
| Item Lists | `n8n-nodes-base.itemLists` | 3.1 | List manipulation utilities |

### Data Transformation

| Name | Type | Version | Notes |
|------|------|---------|-------|
| Date & Time | `n8n-nodes-base.dateTime` | 2 | Date and time processing |
| Crypto | `n8n-nodes-base.crypto` | 1 | Encryption / hashing / signing |
| HTML | `n8n-nodes-base.html` | 1.2 | HTML parsing and extraction |
| Markdown | `n8n-nodes-base.markdown` | 1 | Markdown ↔ HTML conversion |
| XML | `n8n-nodes-base.xml` | 1.1 | XML parsing and generation |
| JSON | n8n built-in expressions | — | Use `JSON.parse()` / `JSON.stringify()` |

### File Handling

| Name | Type | Version | Notes |
|------|------|---------|-------|
| Read Binary File | `n8n-nodes-base.readBinaryFile` | 1 | Read a local file |
| Write Binary File | `n8n-nodes-base.writeBinaryFile` | 1 | Write a local file |
| Extract from File | `n8n-nodes-base.extractFromFile` | 1 | Parse PDF / CSV / Excel etc. |
| Convert to File | `n8n-nodes-base.convertToFile` | 1.1 | Generate CSV / Excel / PDF etc. |
| Compression | `n8n-nodes-base.compression` | 1 | Compress / decompress ZIP |

### Network Requests

| Name | Type | Version | Notes |
|------|------|---------|-------|
| HTTP Request | `n8n-nodes-base.httpRequest` | 4.2 | General-purpose HTTP requests |
| Respond to Webhook | `n8n-nodes-base.respondToWebhook` | 1.1 | Send webhook response |
| GraphQL | `n8n-nodes-base.graphql` | 1 | GraphQL requests |
| FTP | `n8n-nodes-base.ftp` | 1 | FTP file transfer |
| SSH | `n8n-nodes-base.ssh` | 1 | SSH remote commands |

### Data Storage

| Name | Type | Version | Notes |
|------|------|---------|-------|
| Data Table | `n8n-nodes-base.dataTable` | 1.1 | n8n built-in data table (recommended for prototyping) |
| Postgres | `n8n-nodes-base.postgres` | 2.5 | PostgreSQL database |
| MySQL | `n8n-nodes-base.mySql` | 2.4 | MySQL database |
| MongoDB | `n8n-nodes-base.mongoDb` | 1.2 | MongoDB database |
| Redis | `n8n-nodes-base.redis` | 1 | Redis cache |

> ⚠️ **Use Data Table during prototyping** — no external database setup required

### AI Nodes

| Name | Type | Version | Notes |
|------|------|---------|-------|
| AI Agent | `@n8n/n8n-nodes-langchain.agent` | 3.1 | AI agent — the core node |
| Basic LLM Chain | `@n8n/n8n-nodes-langchain.chainLlm` | 1.5 | Simple LLM call |
| Structured Output Parser | `@n8n/n8n-nodes-langchain.outputParserStructured` | 1.3 | Structured output |
| OpenAI Chat Model | `@n8n/n8n-nodes-langchain.lmChatOpenAi` | 1.3 | OpenAI / compatible models |
| Anthropic Chat Model | `@n8n/n8n-nodes-langchain.lmChatAnthropic` | 1.3 | Claude models |
| Window Buffer Memory | `@n8n/n8n-nodes-langchain.memoryBufferWindow` | 1.3 | Windowed conversation memory |
| Chat Memory Manager | `@n8n/n8n-nodes-langchain.memoryManager` | 1.1 | Manage chat memory |
| Respond to Chat | `@n8n/n8n-nodes-langchain.chat` | 1 | Mid-conversation reply |

> ⚠️ **Important**: AI Agent must be connected to a Chat Model — never leave it unconnected!

### Prototype Utilities

| Name | Type | Version | Notes |
|------|------|---------|-------|
| No Operation | `n8n-nodes-base.noOp` | 1 | Placeholder node — does nothing |
| Sticky Note | `n8n-nodes-base.stickyNote` | 1 | Annotation note |
| Debug Helper | `n8n-nodes-base.debugHelper` | 1 | Generate test data / simulate errors |
| Execute Sub-workflow | `n8n-nodes-base.executeWorkflow` | 1.2 | Call a sub-workflow |

### Common Notification Channels (via HTTP Request)

| Platform | Implementation | Notes |
|----------|---------------|-------|
| DingTalk | HTTP Request → Webhook URL | Bot webhook |
| WeCom | HTTP Request → Webhook URL | Bot webhook |
| Feishu / Lark | HTTP Request → Webhook URL | Bot webhook |
| Slack | `n8n-nodes-base.slack` | Official node available |
| Telegram | `n8n-nodes-base.telegram` | Official node available |
| Email | `n8n-nodes-base.emailSend` | SMTP send |

---

## AI Node Configuration

### AI Agent required trio
```
AI Agent (hasOutputParser: true)
    │
    ├── [ai_languageModel] → Chat Model (required!)
    │
    └── [ai_outputParser] → Structured Output Parser (when JSON output is needed)
```

### Phrases that trigger structured output
- "Extract X fields"
- "Return JSON format"
- "Output structured data"
- "Return in a specific format"

### Common Mistakes
```
❌ Wrong: JSON.stringify({ field1: $json.field1, ... })
✅ Correct: JSON.stringify($json.output)

❌ Wrong: {{ $('NodeName').item.json.field }}  (referencing previous node)
✅ Correct: {{ $json.field }}
```

---

## Flow Patterns

### Linear
```
Trigger → A → B → C
```

### Conditional branch
```
Trigger → If
           ├── True  → A
           └── False → B
```

### Multi-branch
```
Trigger → Switch
           ├── Case 1 → A
           ├── Case 2 → B
           └── Default → C
```

### Branch and merge
```
Trigger → If
           ├── True  → A ─┐
           └── False → B ─┴→ Merge → C
```

### Cache pattern
```
Trigger → Data Table (rowNotExists) → Call API → Save cache
        → Data Table (rowExists) → Get cache
                                          ↓
                                  Merge → downstream
```

### Webhook async response
```
Webhook → Extract params → Respond to Webhook → Background processing (noOp)
```

### Loop processing (Loop Over Items)
```
Trigger → Loop Over Items ←────────────────────┐
              │                                │
              ├── [done] → post-loop handling  │
              │                                │
              └── [loop] → process item → Wait ─┘
```

---

## Loop Over Items — Detailed Guide

### Important
> ⚠️ **You probably don't need this node!** n8n automatically runs each node once per input item.
> Only use it when you need to **control batch size** or **add delays within the loop**.

### Node Info
| Attribute | Value |
|-----------|-------|
| Type | `n8n-nodes-base.splitInBatches` |
| Version | 3.1 |
| Alias | Split In Batches / Loop Over Items |

### Two Output Ports (Critical!)

| Port | Name | Index | Description |
|------|------|-------|-------------|
| First | `done` | 0 | Emits after **all batches complete** — connect to post-loop nodes |
| Second | `loop` | 1 | Current batch data — connect to nodes **inside the loop** |

### Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `batchSize` | number | 1 | Items per batch |
| `options.reset` | boolean | false | Restart from beginning (for new datasets) |

### Correct Wiring Diagram

```
                    ┌────────────────────────────┐
                    │                            │
                    ▼                            │
Input → Loop Over Items                          │
              │                                  │
              ├── [done:0] → loop complete → downstream
              │                                  │
              └── [loop:1] → HTTP Request → Wait(1s) ─┘
                              ▲
                              │
                         nodes inside the loop
```

### Common Mistakes

```
❌ Mistake 1: Only connecting one output
Loop Over Items → processing node → end
(missing the loop-back wire — runs only once)

✅ Correct: both outputs must be connected
Loop Over Items
    ├── [done] → end processing
    └── [loop] → processing node → back to Loop Over Items

❌ Mistake 2: Swapping done and loop
Loop Over Items
    ├── [done] → processing node → loop-back   (wrong! done is post-loop)
    └── [loop] → end processing                (wrong! loop is the loop body)

✅ Correct: done = after loop, loop = loop body
```

### When Do You Need Loop Over Items?

| Scenario | Needed? | Reason |
|----------|---------|--------|
| Batch API calls | ✅ Yes | Control concurrency, avoid rate limits |
| Per-item processing + delay | ✅ Yes | Add Wait between items |
| Standard data processing | ❌ No | n8n runs each node automatically per item |
| Simple transformation | ❌ No | Use Set / Code directly |

### Typical Use Case: Batch API Calls with Rate Limiting

```
Trigger → Loop Over Items (batchSize=1)
              │
              ├── [done] → Send completion notification
              │
              └── [loop] → HTTP Request → Wait(1s) → back to Loop
```

---

## Best Practices

### 1. Deduplication / Preventing Duplicate Processing
```
Webhook → Data Table (rowExists)
           ├── Exists → noOp (skip)
           └── Not exists → Process → Data Table (insert)
```

### 2. API Caching
```
Trigger → Data Table (get) → If (cache exists?)
           ├── True  → Return cached result
           └── False → HTTP Request → Data Table (upsert)
```

### 3. Batch Rate Limiting
```
Trigger → Split In Batches → HTTP Request → Wait(1s) → loop back
```

### 4. AI + Human Review
```
Trigger → AI Agent → If (confidence > 90%?)
           ├── True  → Auto-execute
           └── False → [Manual] Review
```

### 5. Error Handling
```
Trigger → Main processing
           ├── Success → Complete
           └── Failure → Wait → Retry → Notify if limit reached
```

---

## Expression Quick Reference

| Expression | Description |
|-----------|-------------|
| `{{ $json.field }}` | Field from the previous node |
| `{{ $json["field name"] }}` | Field with special characters in name |
| `{{ $('NodeName').item.json.field }}` | Field from a specific named node |
| `{{ $input.all() }}` | All data from the previous node |
| `{{ $now }}` | Current timestamp |
| `{{ $today }}` | Today's date |
| `{{ $runIndex }}` | Current execution index |
| `{{ $itemIndex }}` | Current item index |

---

## Example

**User**: "Build a resume screening flow: parse PDF, AI evaluation, notify HR or archive based on score"

**Skeleton mode**:
```
[Sticky Note: Resume Screening — score ≥ 80 notifies HR, otherwise archives]

Manual Trigger
    ↓
[File] Parse resume PDF (noOp)
    ↓
[AI] Evaluate resume (noOp)
    ↓
If: score ≥ 80?
    ├── True  → [Notify] Notify HR (noOp)
    └── False → [DB] Archive (noOp)
```

**Mixed mode**:
```
Manual Trigger
    ↓
[File] Parse resume PDF (noOp)
    ↓
AI Agent (evaluate resume)
    ↓
If: score ≥ 80?
    ├── True  → [Notify] Notify HR (noOp)
    └── False → Data Table (archive)
```
