# MCP Server Pattern

**Use Case**: 将 n8n 工作流暴露为 MCP (Model Context Protocol) 工具，供 AI 客户端（Claude、Cursor 等）直接调用。

---

## Pattern Structure

```
MCP Server Trigger → [Tool 1: HTTP Request Tool]
                   → [Tool 2: Code Tool]
                   → [Tool 3: ...]
```

**Key Characteristic**: MCP Trigger 不是线性流程，而是**工具注册器**。每个连接的 Tool 节点都会成为 MCP 客户端可调用的独立工具。

---

## Core Components

### 1. MCP Server Trigger (v2)

**Purpose**: 创建 MCP Server endpoint，暴露工具给 AI 客户端

**Configuration**:
```javascript
const mcpTrigger = trigger({
  type: '@n8n/n8n-nodes-langchain.mcpTrigger',
  version: 2,
  config: {
    name: 'MCP Server Trigger',
    parameters: {
      path: 'my-service'    // MCP endpoint 路径
      // authentication: 'bearerAuth'  // 可选：Bearer Token 认证
    },
    subnodes: {
      tools: [tool1, tool2]  // 注册的工具列表
    },
    position: [96, 272]
  },
  output: [{}]
});
```

**版本说明**:
- ~~v1.1~~ — 旧版本，不推荐
- **v2** — 当前版本，推荐使用

### 2. Tool 节点（连接到 MCP Trigger）

MCP Trigger 使用 **Tool 版本**的节点（不是普通版本）：

| 普通节点 | Tool 版本 | 用途 |
|----------|----------|------|
| `n8n-nodes-base.httpRequest` | `n8n-nodes-base.httpRequestTool` | HTTP 请求工具 |
| `n8n-nodes-base.code` | `n8n-nodes-base.codeTool` | 自定义代码工具 |
| — | `@n8n/n8n-nodes-langchain.toolCalculator` | 计算器工具 |
| — | `@n8n/n8n-nodes-langchain.toolWikipedia` | Wikipedia 查询 |

**Critical**: 必须使用 `tool()` 函数包装，不是 `node()`。

### 3. `$fromAi()` — AI 参数传递

让 MCP 客户端动态传入参数：

```javascript
// 在 Tool 节点的参数中使用
jsonBody: `={"url": "{{ $fromAi('url', 'The URL to scrape, returns Markdown content') }}", "formats": ["markdown"]}`
```

**`$fromAi()` 语法**:
```javascript
$fromAi('参数名', '参数描述')
// 参数描述非常重要！MCP 客户端靠它理解如何调用
```

**描述最佳实践**:
- 说明参数是什么：`'The URL to scrape'`
- 说明返回什么：`'returns page content in Markdown format'`
- 说明格式要求：`'ISO 8601 date string, e.g. 2024-01-01'`

---

## Complete Example: Firecrawl 网页抓取

```javascript
import { workflow, trigger, tool, fromAi } from '@n8n/workflow-sdk';

const scrapeTool = tool({
  type: 'n8n-nodes-base.httpRequestTool',
  version: 4.4,
  config: {
    name: 'scrape',
    parameters: {
      method: 'POST',
      url: 'http://10.10.10.4:3002/v2/scrape',
      authentication: 'genericCredentialType',
      genericAuthType: 'httpBearerAuth',
      sendBody: true,
      specifyBody: 'json',
      jsonBody: `={"url": "{{ $fromAi('url', 'The URL to scrape, returns page content in Markdown format') }}", "formats": ["markdown"]}`,
      options: { timeout: 60000 }
    },
    credentials: {
      httpBearerAuth: { id: 'xxx', name: 'my-auth' }
    },
    position: [320, 528]
  }
});

const mcpTrigger = trigger({
  type: '@n8n/n8n-nodes-langchain.mcpTrigger',
  version: 2,
  config: {
    name: 'MCP Server Trigger',
    parameters: { path: 'firecrawl' },
    subnodes: { tools: [scrapeTool] },
    position: [96, 272]
  },
  output: [{}]
});

export default workflow('id', 'Firecrawl MCP 中转')
  .add(mcpTrigger);
```

---

## 多工具注册

一个 MCP Trigger 可以注册多个工具：

```javascript
const scrapeTool = tool({ ... });
const searchTool = tool({ ... });
const crawlTool = tool({ ... });

const mcpTrigger = trigger({
  type: '@n8n/n8n-nodes-langchain.mcpTrigger',
  version: 2,
  config: {
    name: 'MCP Server Trigger',
    parameters: { path: 'my-tools' },
    subnodes: {
      tools: [scrapeTool, searchTool, crawlTool]
    },
    position: [96, 272]
  },
  output: [{}]
});
```

---

## Common Mistakes

### ❌ 使用普通节点而非 Tool 版本

```javascript
// 错误：普通 HTTP Request 不能作为 MCP 工具
const scrape = node({
  type: 'n8n-nodes-base.httpRequest',  // ❌
  ...
});

// 正确：使用 Tool 版本
const scrape = tool({
  type: 'n8n-nodes-base.httpRequestTool',  // ✅
  ...
});
```

### ❌ 用 `node()` 包装 Tool 节点

```javascript
// 错误
const scrape = node({ type: 'n8n-nodes-base.httpRequestTool', ... });  // ❌

// 正确
const scrape = tool({ type: 'n8n-nodes-base.httpRequestTool', ... });  // ✅
```

### ❌ 使用 MCP Trigger v1.1

```javascript
// 过时
version: 1.1,  // ❌

// 推荐
version: 2,    // ✅
```

### ❌ `$fromAi()` 描述不清

```javascript
// 差：调用方不知道该传什么
$fromAi('url', '')  // ❌

// 好：清晰描述参数用途和返回格式
$fromAi('url', 'The URL to scrape, returns page content in Markdown format')  // ✅
```

### ❌ 忘记激活工作流

MCP Trigger 必须**激活工作流**后才能被 MCP 客户端发现和调用。

### ❌ 忘记配置凭证

通过 SDK 创建的工作流，凭证可能需要在 n8n 界面手动确认绑定。

---

## MCP 客户端连接

工作流激活后，MCP 客户端通过 SSE 连接：

```
https://your-n8n-instance.com/mcp/<path>/sse
```

在 MCP 客户端配置中添加：
```json
{
  "mcpServers": {
    "firecrawl": {
      "url": "https://your-n8n-instance.com/mcp/firecrawl/sse"
    }
  }
}
```

---

## Checklist

- [ ] MCP Trigger 使用 v2
- [ ] Tool 节点使用 `tool()` 函数和 Tool 版本节点类型
- [ ] `$fromAi()` 参数描述清晰（说明传什么、返回什么）
- [ ] 凭证已在 n8n 界面确认绑定
- [ ] 工作流已激活
- [ ] MCP 客户端能正常连接 SSE endpoint
- [ ] 工作流描述写明了工具功能和返回格式

---

## 适用场景

| 场景 | 适合用 MCP Server |
|------|-------------------|
| AI 客户端需要调用内部 API | ✅ |
| 将现有服务暴露给 AI | ✅ |
| 内网服务中转（无公网 IP） | ✅ |
| 多工具聚合（一个 endpoint 多个能力） | ✅ |
| 高并发实时请求 | ❌ 用 Webhook + 专用服务更合适 |
| 简单的 HTTP 转发 | ❌ 用 Nginx 反向代理更轻量 |
