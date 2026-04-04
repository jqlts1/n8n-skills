# n8n 工作流开发 Skills 集合

这是一组为 n8n 工作流自动化开发定制的专业 Skills，覆盖从流程设计、节点配置、表达式编写，到验证、测试、执行监控的完整开发生命周期。每个 Skill 对应一个具体的开发阶段或职责，通过组合使用完成复杂任务。

---

## 调用语法

```
/skill-name [可选描述]
```

示例：
```
/n8n-prototype 画一个订单处理流程
/n8n-validation-expert 这个验证错误怎么修
/n8n-execution-testing 工作流跑失败了，不知道哪里出问题
```

Skill 触发后会加载对应的 `SKILL.md` 文件进入上下文，AI 按其中的指引行动。

---

## 触发条件速查表

| Skill | 用这个 Skill 的时机 | 不要用的情况 |
|-------|-------------------|------------|
| `n8n-prototype` | 用户说"画个流程"、"设计工作流"、"搭原型"；把业务描述转为 n8n 可视化结构 | 已经在写真实 SDK 代码，不需要先画原型 |
| `n8n-workflow-patterns` | 不确定工作流应该用什么架构（Webhook/API/DB/AI/定时）；需要选结构模式 | 架构已经定了，只差具体节点配置 |
| `n8n-reference-workflow-research` | 用户想先看现成案例再决定怎么搭；不确定这个流程是否已有参考 | 需求独特，没有复用价值，直接设计即可 |
| `n8n-node-configuration` | 需要知道某个节点的必填字段；不确定参数结构；把参数知识转换成 SDK 代码 | 节点已经配好，只是在验证或调试 |
| `n8n-expression-syntax` | 写 `{{ }}` 表达式；遇到表达式报错；引用其他节点数据 | 不涉及表达式，只做节点配置 |
| `n8n-mcp-tools-expert` | 需要搜索节点、读类型定义、创建/更新/查找工作流；不确定用哪个 MCP 工具 | 只需要了解表达式或验证逻辑，不涉及 MCP 工具调用 |
| `n8n-validation-expert` | `validate_workflow` 返回错误或警告；不知道先修哪个、怎么修 | 代码还没写完，不到验证阶段 |
| `n8n-node-pitfalls` | 用到 Merge、Loop Over Items、IF、Switch 等流程控制节点；并行分支有问题；节点输出不符合预期 | 使用普通数据处理节点，没有流程控制结构 |
| `n8n-execution-testing` | 工作流已有，需要决定怎么测试；执行失败不知道先看哪里；拿到 `get_execution` 结果不知道下一步 | 工作流还没建好，还在写代码阶段 |
| `skill-creator` | 用户要创建新 Skill 或更新现有 Skill；需要了解 SKILL.md 的格式和写法规范 | 已有 Skill 能满足需求，不需要新建 |

---

## Skill 协作流程

### 场景 1：全新工作流开发

```
1. n8n-reference-workflow-research   先找有没有类似案例可以借鉴
         ↓
2. n8n-prototype                     画骨架原型，确认流程结构
         ↓
3. n8n-workflow-patterns             选定架构模式（Webhook/API/AI等）
         ↓
4. n8n-node-configuration            确认节点参数，写 SDK 代码
         ↓
5. n8n-expression-syntax             （有表达式时）确认语法正确
         ↓
6. n8n-mcp-tools-expert              validate → create_workflow_from_code
         ↓
7. n8n-validation-expert             修复验证错误（通常需要 2-3 轮）
         ↓
8. n8n-execution-testing             mock 测试 → 真实执行 → 定位失败
```

### 场景 2：修改已有工作流

```
1. n8n-mcp-tools-expert              search_workflows → get_workflow_details
         ↓
2. n8n-node-configuration            修改节点参数，更新 SDK 代码
         ↓
3. n8n-validation-expert             validate → 修复错误
         ↓
4. n8n-mcp-tools-expert              update_workflow
         ↓
5. n8n-execution-testing             验证修改效果
```

### 场景 3：调试失败的工作流

```
1. n8n-execution-testing             分析失败类型（节点/表达式/连接）
         ↓
2. n8n-node-pitfalls                 （如有流程控制节点）排查常见陷阱
      或
   n8n-expression-syntax             （如是表达式问题）修复语法
      或
   n8n-node-configuration            （如是参数问题）核对字段结构
         ↓
3. n8n-validation-expert             修复后重新验证
         ↓
4. n8n-mcp-tools-expert              update_workflow → 重新执行
```

### 场景 4：快速原型讨论

```
1. n8n-prototype                     用骨架/混合模式快速画出流程
         ↓
   （用户确认流程结构）
         ↓
2. n8n-node-configuration            把 noOp 占位节点替换为真实节点
         ↓
3. n8n-mcp-tools-expert              validate → create
```

---

## 各 Skill 详细说明

---

### `n8n-prototype` — 工作流原型设计

**用途**：快速把业务描述可视化为 n8n 工作流结构图，用 noOp 节点占位，Sticky Note 标注说明。

**用户典型输入**：
- "帮我设计一个处理订单的流程"
- "画一个 Webhook 接收数据后发通知的流程"
- "这个业务流程怎么用 n8n 实现"

**AI 应该做什么**：
1. 先询问用户选择原型模式（骨架/混合/可执行）
2. 用 `manualTrigger` 或用户指定的触发器
3. 业务逻辑用 noOp 占位，命名格式：`[类型] 动词+对象`，如 `[API] 调用支付接口`
4. 添加 Sticky Note 写整体说明
5. 调用 `n8n-mcp-tools-expert` 创建原型工作流

**noOp 命名类型标签**：`[API]` `[DB]` `[通知]` `[文件]` `[AI]` `[人工]`

---

### `n8n-workflow-patterns` — 工作流模式选择

**用途**：帮 AI 和用户确定工作流的架构模式，解决"这个流程该怎么搭"的问题。

**用户典型输入**：
- "我要接收 Webhook 数据然后处理"
- "我想搭一个 AI Agent 工作流"
- "每天定时同步数据库"

**AI 应该做什么**：根据用户描述匹配以下 5 种模式之一，并给出骨架结构：

| 模式 | 适用场景 |
|------|----------|
| Webhook 处理 | 接收外部推送、表单提交 |
| HTTP API 集成 | 调第三方 API、拉取/同步数据 |
| 数据库操作 | ETL、数据同步、定时对账 |
| AI Agent | 智能对话、多工具调用、结构化输出 |
| 定时任务 | 定期执行、批量处理、报告生成 |

---

### `n8n-reference-workflow-research` — 参考工作流研究

**用途**：在开始设计前，先找现有案例，决定是复用还是从头设计。

**用户典型输入**：
- "有没有类似的工作流可以参考"
- "先帮我搜几个示例再决定怎么搭"
- "这种流程有没有现成方案"

**AI 应该做什么**：
1. 先用 `search_workflows` 找自己实例里的类似工作流
2. 再查参照资源接口（见 `docs/API接口/参照资源接口.md`）获取公开示例
3. 最后用 `n8n-docs` 校对节点能力限制
4. 总结：哪些部分可以直接借用，哪些只能借思路

---

### `n8n-node-configuration` — 节点配置

**用途**：确认节点的正确参数结构，把参数知识转换为可执行的 SDK 代码。

**用户典型输入**：
- "Slack 节点发送消息需要哪些字段"
- "这个节点的参数怎么写"
- "怎么配置 HTTP Request 节点"

**AI 应该做什么**：
```
search_nodes({queries: ["节点关键词"]})
  → 获取节点 ID 和 discriminators
  → get_node_types({nodeIds: [...]})
  → 读 TypeScript 类型定义
  → 按操作类型写 node(...) 参数
```

**核心原则**：配置是**操作感知的**，`resource` + `operation` 不同，必填字段就不同，不要跨操作复制字段。

---

### `n8n-expression-syntax` — 表达式语法

**用途**：确认 n8n 表达式语法正确，修复常见语法错误。

**用户典型输入**：
- "怎么引用上一个节点的数据"
- "表达式报错，不知道哪里写错了"
- "这个 `{{...}}` 怎么写"

**核心规则**：

```
✅ {{ $json.email }}
✅ {{ $json["字段名"] }}
✅ {{ $node["节点名"].json.字段 }}
❌ $json.email         （缺花括号）
❌ { $json.email }     （单花括号）
```

常用变量：`$json`（当前节点）、`$node["名称"]`（指定节点）、`$now`（当前时间）、`$today`（今天日期）

---

### `n8n-mcp-tools-expert` — MCP 工具专家

**用途**：所有 n8n MCP 工具的使用指南，包括搜索节点、读类型、创建/更新/管理工作流。

**用户典型输入**：
- "怎么搜索某个节点"
- "怎么创建工作流"
- "不知道用哪个 MCP 工具"

**18 个工具 8 大类速查**：

| 类别 | 核心工具 |
|------|---------|
| 节点发现 | `search_nodes` `get_node_types` `get_suggested_nodes` |
| SDK 参考 | `get_sdk_reference` |
| 工作流管理 | `validate_workflow` `create_workflow_from_code` `update_workflow` `get_workflow_details` `search_workflows` |
| 执行监控 | `execute_workflow` `get_execution` |
| 测试 | `prepare_test_pin_data` `test_workflow` |
| 生命周期 | `publish_workflow` `unpublish_workflow` `archive_workflow` |
| 组织管理 | `search_projects` `search_folders` |

**重要**：`update_workflow` 是**整体替换**，不是局部补丁。创建或更新前必须先 `validate_workflow`。

---

### `n8n-validation-expert` — 验证专家

**用途**：解读 `validate_workflow` 返回的错误和警告，给出修复顺序和方法。

**用户典型输入**：
- "验证报错了，不知道怎么修"
- "这个 warning 是什么意思"
- "修了一个错误，又出来新错误"

**AI 应该做什么**：
1. 先按级别分类：Errors（必须修）> Warnings（有风险）> Suggestions（可选）
2. 修复顺序：语法错误 → 节点类型错误 → 参数结构错误 → 连接错误 → 表达式错误
3. 修完再跑一轮 `validate_workflow`，通常需要 2-3 轮

**注意**：前面的错误会"带出"后面的假问题，不要一次性大改全部代码。

---

### `n8n-node-pitfalls` — 节点易错点速查

**用途**：流程控制节点的常见陷阱和正确做法，防止常见配置错误。

**用户典型输入**：
- "Merge 节点为什么只收到部分数据"
- "Loop 节点为什么只执行一次"
- "并行分支怎么合并"

**覆盖节点**：

| 节点 | 常见陷阱 |
|------|---------|
| **Merge** | Number of Inputs 默认 2，合并多于 2 个分支必须手动改 |
| **Loop Over Items** | `loop` 端口（索引 1）必须回连到自身，否则只执行一次；大多数情况不需要用这个节点 |
| **IF / Switch** | — |

---

### `n8n-execution-testing` — 执行与测试

**用途**：决定如何测试工作流，以及执行失败后如何定位问题。

**用户典型输入**：
- "这个工作流该先 mock 测还是直接执行"
- "执行失败了不知道从哪里看"
- "拿到 get_execution 结果，下一步怎么办"

**测试决策**：

| 场景 | 推荐方式 |
|------|---------|
| 有外部 API，不想真的调用 | `prepare_test_pin_data` + `test_workflow` |
| 验证真实 webhook/chat 入口 | `execute_workflow` |
| 已失败，定位哪一段坏了 | `get_execution` 看节点级结果 |

**失败后的定位顺序**：先看哪个节点失败 → 看该节点的输入/输出 → 根据错误类型跳转到 `n8n-node-pitfalls` / `n8n-expression-syntax` / `n8n-node-configuration`

---

### `skill-creator` — Skill 创建指南（元技能）

**用途**：指导如何创建新 Skill 或更新现有 Skill 的结构和写法。

**用户典型输入**：
- "我想创建一个新的 Skill"
- "这个 Skill 需要更新，怎么修"
- "SKILL.md 怎么写"

**SKILL.md 必需结构**：
```yaml
---
name: skill-name
description: |
  一句话说明用途和触发场景。
---
```
正文用 Markdown，优先用示例代替长篇解释，保持精简（context window 是公共资源）。

**原则**：Claude 本身很聪明，只写它不知道的内容，不要解释它已经掌握的通用知识。
