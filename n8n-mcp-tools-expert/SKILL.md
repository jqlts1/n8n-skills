---
name: n8n-mcp-tools-expert
description: |
  Use when choosing between n8n MCP tools, searching nodes, reading node types, validating SDK workflow code, or creating, updating, executing, and publishing workflows.
---

# n8n MCP 工具专家

使用 n8n MCP 工具构建工作流的完整指南。

---

## 工具总览（18个工具，8大类）

| 类别 | 工具 | 用途 |
|------|------|------|
| **节点发现** | `search_nodes` | 按关键词搜索节点 |
| | `get_node_types` | 获取节点 TypeScript 类型定义（**写代码前必须调用**） |
| | `get_suggested_nodes` | 按工作流类别获取推荐节点 |
| **SDK 参考** | `get_sdk_reference` | 获取工作流 SDK 文档（模式、表达式、规则等） |
| **工作流管理** | `validate_workflow` | 验证 SDK 代码 |
| | `create_workflow_from_code` | 从 SDK 代码创建工作流 |
| | `update_workflow` | 用 SDK 代码更新工作流（整体替换） |
| | `get_workflow_details` | 获取工作流详情 |
| | `search_workflows` | 搜索已有工作流 |
| **执行监控** | `execute_workflow` | 执行工作流（支持 chat/form/webhook 输入） |
| | `get_execution` | 获取执行结果详情 |
| **测试** | `prepare_test_pin_data` | 为工作流生成测试 Pin 数据的 Schema |
| | `test_workflow` | 用 Pin 数据测试工作流（绕过外部服务） |
| **生命周期** | `publish_workflow` | 发布（激活）工作流 |
| | `unpublish_workflow` | 取消发布（停用） |
| | `archive_workflow` | 归档工作流 |
| **组织管理** | `search_projects` | 搜索项目 |
| | `search_folders` | 搜索项目内的文件夹 |

---

## 核心工作流程

### 构建新工作流（最常用）

```
1. search_nodes({queries: ["关键词"]})        → 找到节点 ID
2. get_node_types({nodeIds: [...]})            → 获取参数类型定义
3. get_sdk_reference({section: "patterns"})    → 学习 SDK 语法
4. 编写 SDK 代码                               → TypeScript/JavaScript
5. validate_workflow({code: "..."})            → 验证代码
6. create_workflow_from_code({code: "..."})    → 创建工作流
7. publish_workflow({workflowId: "..."})       → 发布激活
```

### 修改已有工作流

```
1. get_workflow_details({workflowId: "..."})   → 查看当前状态
2. 修改 SDK 代码
3. validate_workflow({code: "..."})            → 验证
4. update_workflow({workflowId, code})         → 更新（整体替换）
```

### 查找和发现

```
search_workflows({query: "关键词"})            → 找已有工作流
search_nodes({queries: ["slack", "webhook"]})  → 找节点
get_suggested_nodes({categories: ["chatbot"]}) → 按类别推荐
```

### 测试工作流（不触发真实外部服务）

```
1. prepare_test_pin_data({workflowId: "..."})  → 获取各节点期望的数据 Schema
2. 根据 Schema 生成模拟数据
3. test_workflow({workflowId, pinData})        → 用 Pin 数据运行，跳过外部调用
```

> **什么节点会被 Pin（用模拟数据）？**
> - 触发器节点（Trigger）
> - 有 Credentials 的节点（如 HTTP Request with Auth、Slack 等）
>
> **什么节点正常执行？**
> - 逻辑节点：Set、If、Code、Switch 等
> - 无需凭证的 I/O 节点：Execute Command、文件读写等

### 什么时候用哪个入口

| 目标 | 优先入口 | 说明 |
|------|----------|------|
| 改你自己的实例 | `n8n-mcp` | 可创建、更新、执行、调试 |
| 查官方 SDK / 节点说明 | `n8n-docs` | 官方文档优先 |
| 找参考工作流示例 | `docs/API接口/参照资源接口.md` | 这是参考资源入口，不是 MCP 工具 |

---

## 各工具详解

### 详细指南

- **节点发现工具** → [SEARCH_GUIDE.md](SEARCH_GUIDE.md)
- **验证工具** → [VALIDATION_GUIDE.md](VALIDATION_GUIDE.md)
- **工作流管理工具** → [WORKFLOW_GUIDE.md](WORKFLOW_GUIDE.md)

---

## 常见错误

### ❌ 错误 1：不调用 get_node_types 就写代码

```javascript
// ❌ 猜测参数名 → 创建出无效工作流
node('n8n-nodes-base.httpRequest', { url: '...', type: 'POST' })

// ✅ 先调用 get_node_types 获取正确参数名
get_node_types({nodeIds: [{nodeId: 'n8n-nodes-base.httpRequest'}]})
// 发现参数是 method 不是 type
node('n8n-nodes-base.httpRequest', { url: '...', method: 'POST' })
```

### ❌ 错误 2：不验证就创建

```javascript
// ❌ 跳过验证直接创建
create_workflow_from_code({code: "..."})  // 可能失败

// ✅ 先验证再创建
validate_workflow({code: "..."})          // 检查错误
create_workflow_from_code({code: "..."})  // 确认无误后创建
```

### ❌ 错误 3：search_nodes 参数格式错误

```javascript
// ❌ 传单个字符串
search_nodes({query: "slack"})

// ✅ queries 是数组
search_nodes({queries: ["slack"]})

// ✅ 可以一次搜多个
search_nodes({queries: ["slack", "webhook", "schedule"]})
```

### ❌ 错误 4：get_node_types 不带 discriminators

```javascript
// ❌ 只传 nodeId，返回的类型定义可能太泛
get_node_types({nodeIds: ["n8n-nodes-base.httpRequest"]})

// ✅ 带上 search_nodes 返回的 discriminators
get_node_types({nodeIds: [{
  nodeId: "n8n-nodes-base.httpRequest",
  // search_nodes 返回了这些 discriminators
}]})
```

### ❌ 错误 5：还在按旧思路做“局部补丁”

```javascript
// ❌ 旧思路：只改一小段配置
// ✅ 新方式：整体替换代码
// 1. 先获取当前工作流
get_workflow_details({workflowId: "..."})
// 2. 修改完整 SDK 代码
// 3. 整体替换
update_workflow({workflowId: "...", code: "完整的新代码"})
```

---

## 最佳实践

### ✅ Do

- **写代码前必须调用 `get_node_types`** — 参数名不能猜
- **用 `get_sdk_reference` 学习 SDK 语法** — 特别是第一次使用时
- **每次都先 `validate_workflow` 再创建/更新** — 避免失败
- **用 `search_nodes` 的 discriminators 传给 `get_node_types`** — 获取更精确的类型
- **用 `get_suggested_nodes` 做技术选型** — 11 个类别覆盖常见场景
- **先区分 mock 测试和真实执行** — `test_workflow` 适合安全验证，`execute_workflow` 适合真实联调
- **执行失败要回看 `get_execution`** — 先找第一个坏掉的节点，再修

### ❌ Don't

- **不要猜参数名** — 必须从 `get_node_types` 获取
- **不要跳过验证** — `validate_workflow` 是必须步骤
- **不要期望部分更新** — `update_workflow` 是整体替换
- **不要忘记 `publish_workflow`** — 创建后需要发布才能在生产模式执行

---

## 相关 Skills

- **n8n Expression Syntax** — 在工作流字段中写表达式
- **n8n Workflow Patterns** — 工作流架构模式
- **n8n Validation Expert** — 解读验证错误
- **n8n Execution Testing** — 决定 mock 还是真跑，并定位执行失败
- **n8n Node Configuration** — 操作级节点配置指导
- **n8n Node Pitfalls** — 节点易错点速查
