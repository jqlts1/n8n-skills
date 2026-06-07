# 节点配置模板

所有节点的 JSON 配置模板，可直接用于 n8n MCP 工具创建节点。

## 触发器节点

### Manual Trigger
```json
{
  "id": "trigger_1",
  "name": "开始",
  "type": "n8n-nodes-base.manualTrigger",
  "typeVersion": 1,
  "position": [250, 300],
  "parameters": {}
}
```

### Webhook
```json
{
  "id": "webhook_1",
  "name": "Webhook",
  "type": "n8n-nodes-base.webhook",
  "typeVersion": 2,
  "position": [250, 300],
  "parameters": {
    "path": "my-webhook",
    "httpMethod": "POST"
  }
}
```

### Schedule Trigger
```json
{
  "id": "schedule_1",
  "name": "定时触发",
  "type": "n8n-nodes-base.scheduleTrigger",
  "typeVersion": 1.2,
  "position": [250, 300],
  "parameters": {
    "rule": {
      "interval": [{ "field": "hours", "hoursInterval": 1 }]
    }
  }
}
```

## 逻辑控制节点

### If 条件判断
```json
{
  "id": "if_1",
  "name": "判断条件",
  "type": "n8n-nodes-base.if",
  "typeVersion": 2,
  "position": [450, 300],
  "parameters": {
    "conditions": {
      "options": { "version": 2 },
      "combinator": "and",
      "conditions": []
    }
  }
}
```

### Switch 多分支
```json
{
  "id": "switch_1",
  "name": "多条件判断",
  "type": "n8n-nodes-base.switch",
  "typeVersion": 3,
  "position": [450, 300],
  "parameters": {
    "mode": "rules",
    "rules": { "values": [] }
  }
}
```

### Filter 过滤
```json
{
  "id": "filter_1",
  "name": "过滤数据",
  "type": "n8n-nodes-base.filter",
  "typeVersion": 2,
  "position": [450, 300],
  "parameters": {
    "conditions": {
      "options": { "version": 2 },
      "combinator": "and",
      "conditions": []
    }
  }
}
```

### Split In Batches 循环
```json
{
  "id": "batch_1",
  "name": "分批处理",
  "type": "n8n-nodes-base.splitInBatches",
  "typeVersion": 3,
  "position": [450, 300],
  "parameters": {
    "batchSize": 10
  }
}
```

### Merge 合并
```json
{
  "id": "merge_1",
  "name": "合并数据",
  "type": "n8n-nodes-base.merge",
  "typeVersion": 3,
  "position": [650, 375],
  "parameters": {
    "mode": "combine",
    "combinationMode": "mergeByPosition"
  }
}
```

### Wait 等待
```json
{
  "id": "wait_1",
  "name": "等待",
  "type": "n8n-nodes-base.wait",
  "typeVersion": 1.1,
  "position": [450, 300],
  "parameters": {
    "amount": 5,
    "unit": "seconds"
  }
}
```

## 数据处理节点

### Set 设置变量
```json
{
  "id": "set_1",
  "name": "设置变量",
  "type": "n8n-nodes-base.set",
  "typeVersion": 3.4,
  "position": [450, 300],
  "parameters": {
    "mode": "manual",
    "assignments": {
      "assignments": [
        {
          "id": "field_1",
          "name": "fieldName",
          "value": "fieldValue",
          "type": "string"
        }
      ]
    },
    "includeOtherFields": false
  }
}
```

### Code (Python)
```json
{
  "id": "code_1",
  "name": "Python处理",
  "type": "n8n-nodes-base.code",
  "typeVersion": 2,
  "position": [450, 300],
  "parameters": {
    "language": "python",
    "mode": "runOnceForAllItems",
    "pythonCode": "# 获取输入数据\ndata = _items[0][\"json\"]\n\n# 处理逻辑\nresult = {\"processed\": True}\n\nreturn [result]"
  }
}
```

### Code (JavaScript)
```json
{
  "id": "code_1",
  "name": "JS处理",
  "type": "n8n-nodes-base.code",
  "typeVersion": 2,
  "position": [450, 300],
  "parameters": {
    "language": "javaScript",
    "mode": "runOnceForAllItems",
    "jsCode": "// 获取输入数据\nconst data = $input.all();\n\n// 处理逻辑\nreturn [{json: {processed: true}}];"
  }
}
```

### Split Out 拆分数组
```json
{
  "id": "split_1",
  "name": "拆分数组",
  "type": "n8n-nodes-base.splitOut",
  "typeVersion": 1,
  "position": [450, 300],
  "parameters": {
    "fieldToSplitOut": "items"
  }
}
```

### Limit 限制数量
```json
{
  "id": "limit_1",
  "name": "限制数量",
  "type": "n8n-nodes-base.limit",
  "typeVersion": 1,
  "position": [450, 300],
  "parameters": {
    "maxItems": 10,
    "keep": "firstItems"
  }
}
```

**Limit 参数说明:**
- `maxItems`: 最大保留数量
- `keep`: `firstItems`(保留前N项) 或 `lastItems`(保留后N项)

### Crypto 加密
```json
{
  "id": "crypto_1",
  "name": "加密处理",
  "type": "n8n-nodes-base.crypto",
  "typeVersion": 1,
  "position": [450, 300],
  "parameters": {
    "action": "hash",
    "type": "SHA256",
    "value": "={{ $json.data }}",
    "dataPropertyName": "hash",
    "encoding": "hex"
  }
}
```

**Crypto 操作类型:**
| action | 说明 |
|--------|------|
| `hash` | 哈希 (MD5/SHA256/SHA512等) |
| `hmac` | HMAC 签名 |
| `sign` | 私钥签名 |
| `generate` | 生成随机字符串 |

## AI 节点

### HTTP Request
```json
{
  "id": "http_1",
  "name": "HTTP请求",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.3,
  "position": [450, 300],
  "parameters": {
    "method": "GET",
    "url": "https://api.example.com/data",
    "authentication": "none",
    "sendHeaders": false,
    "sendBody": false
  }
}
```

### HTTP Request (POST with JSON Body)
```json
{
  "id": "http_2",
  "name": "POST请求",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.3,
  "position": [450, 300],
  "parameters": {
    "method": "POST",
    "url": "https://api.example.com/submit",
    "authentication": "none",
    "sendBody": true,
    "contentType": "json",
    "bodyParameters": {
      "parameters": [
        { "name": "key", "value": "={{ $json.value }}" }
      ]
    }
  }
}
```

**HTTP Request 参数说明:**
| 参数 | 说明 |
|------|------|
| `method` | GET/POST/PUT/PATCH/DELETE |
| `url` | 请求地址 |
| `authentication` | none/predefinedCredentialType/genericCredentialType |
| `sendBody` | 是否发送请求体 |
| `contentType` | json/form-urlencoded/multipart-form-data/raw |
| `sendHeaders` | 是否发送自定义请求头 |

## 数据存储节点

### Data Table (插入数据)
```json
{
  "id": "datatable_1",
  "name": "保存数据",
  "type": "n8n-nodes-base.dataTable",
  "typeVersion": 1.1,
  "position": [450, 300],
  "parameters": {
    "resource": "row",
    "operation": "insert",
    "dataTableId": {
      "mode": "id",
      "value": "your-table-id"
    }
  }
}
```

### Data Table (查询数据)
```json
{
  "id": "datatable_2",
  "name": "查询数据",
  "type": "n8n-nodes-base.dataTable",
  "typeVersion": 1.1,
  "position": [450, 300],
  "parameters": {
    "resource": "row",
    "operation": "get",
    "dataTableId": {
      "mode": "id",
      "value": "your-table-id"
    }
  }
}
```

### Data Table (更新或插入)
```json
{
  "id": "datatable_3",
  "name": "更新或插入",
  "type": "n8n-nodes-base.dataTable",
  "typeVersion": 1.1,
  "position": [450, 300],
  "parameters": {
    "resource": "row",
    "operation": "upsert",
    "dataTableId": {
      "mode": "id",
      "value": "your-table-id"
    }
  }
}
```

**Data Table 操作类型:**
| resource | operation | 说明 |
|----------|-----------|------|
| row | insert | 插入新行 |
| row | get | 查询行 |
| row | update | 更新行 |
| row | upsert | 更新或插入 |
| row | deleteRows | 删除行 |
| row | rowExists | 判断行是否存在 |
| row | rowNotExists | 判断行是否不存在 |
| table | create | 创建表 |
| table | list | 列出所有表 |
| table | delete | 删除表 |
| table | update | 重命名表 |

> ⚠️ **重要**: 使用 Data Table 前，需要先在 n8n 界面中创建表格并定义列结构！

### AI Agent
```json
{
  "id": "agent_1",
  "name": "AI Agent",
  "type": "@n8n/n8n-nodes-langchain.agent",
  "typeVersion": 3.1,
  "position": [450, 300],
  "parameters": {
    "promptType": "define",
    "text": "请分析以下内容：{{ $json.content }}"
  }
}
```

**AI Agent 说明:**
- 需要连接 Language Model (如 OpenAI、Claude 等)
- 可选连接 Tools、Memory、Output Parser
- `promptType`: `auto`(从 Chat Trigger 获取) 或 `define`(手动定义)

### AI Agent (结构化输出)
当需要 AI 返回固定格式的 JSON 数据时，必须开启 `hasOutputParser` 并连接 Output Parser。
```json
{
  "id": "agent_2",
  "name": "AI Agent (结构化输出)",
  "type": "@n8n/n8n-nodes-langchain.agent",
  "typeVersion": 3.1,
  "position": [450, 300],
  "parameters": {
    "promptType": "define",
    "text": "请分析以下内容并提取关键信息：{{ $json.content }}",
    "hasOutputParser": true
  }
}
```

### Structured Output Parser
定义 AI 输出的 JSON 格式，需要连接到 AI Agent 的 Output Parser 端口。
```json
{
  "id": "parser_1",
  "name": "结构化输出解析",
  "type": "@n8n/n8n-nodes-langchain.outputParserStructured",
  "typeVersion": 1.3,
  "position": [450, 500],
  "parameters": {
    "schemaType": "fromJson",
    "jsonSchemaExample": "{\n\t\"name\": \"张三\",\n\t\"age\": 25,\n\t\"skills\": [\"Python\", \"JavaScript\"]\n}",
    "autoFix": true
  }
}
```

**Structured Output Parser 参数说明:**
| 参数 | 说明 |
|------|------|
| `schemaType` | `fromJson`(从示例生成) 或 `manual`(手写 JSON Schema) |
| `jsonSchemaExample` | JSON 示例，系统自动推断格式 |
| `inputSchema` | 手动定义的 JSON Schema（schemaType=manual 时使用）|
| `autoFix` | 输出格式错误时自动重试修复（会多一次 LLM 调用）|

**AI 结构化输出完整结构:**
```
AI Agent (hasOutputParser: true)
    │
    ├── [ai_languageModel] → OpenAI / Claude 等模型
    │
    └── [ai_outputParser] → Structured Output Parser
                              └── 定义输出 JSON 格式
```

### Chat Model 节点

AI Agent 必须连接 Chat Model 节点，以下是常用模型的配置模板：

#### OpenAI Chat Model
```json
{
  "id": "openai_1",
  "name": "OpenAI Chat Model",
  "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
  "typeVersion": 1.3,
  "position": [250, 500],
  "parameters": {
    "model": "gpt-4o",
    "options": {}
  }
}
```

#### DeepSeek Chat Model
```json
{
  "id": "deepseek_1",
  "name": "DeepSeek Chat Model",
  "type": "@n8n/n8n-nodes-langchain.lmChatDeepSeek",
  "typeVersion": 1,
  "position": [250, 500],
  "parameters": {
    "model": "deepseek-chat",
    "options": {}
  }
}
```

#### Anthropic Chat Model
```json
{
  "id": "anthropic_1",
  "name": "Anthropic Chat Model",
  "type": "@n8n/n8n-nodes-langchain.lmChatAnthropic",
  "typeVersion": 1,
  "position": [250, 500],
  "parameters": {
    "model": "claude-3-5-sonnet-20241022",
    "options": {}
  }
}
```

**Chat Model 连接方式:**
Chat Model 需要通过 `ai_languageModel` 端口连接到 AI Agent：
```json
{
  "AI Agent": {
    "ai_languageModel": [
      [{ "node": "OpenAI Chat Model", "type": "ai_languageModel", "index": 0 }]
    ]
  }
}
```

> ⚠️ **重要**: 原型阶段也必须创建 Chat Model 节点并连接，不能留空让用户自己加！

## 工具节点

### No Operation 占位
```json
{
  "id": "noop_1",
  "name": "【待实现】业务逻辑描述",
  "type": "n8n-nodes-base.noOp",
  "typeVersion": 1,
  "position": [450, 300],
  "parameters": {}
}
```

### Sticky Note 便签

Sticky Note 是工作流里的文档面板，用来写 API 说明、凭证提示、流程架构等。好的 Sticky Note 让别人（或未来的自己）一眼看懂工作流。

#### JSON 模板
```json
{
  "id": "note_1",
  "name": "Sticky Note",
  "type": "n8n-nodes-base.stickyNote",
  "typeVersion": 1,
  "position": [40, 100],
  "parameters": {
    "content": "## API 文档\n\n**Webhook:** `POST /webhook/xxx`\n\n**Body:**\n```json\n{ \"input\": \"内容\" }\n```",
    "height": 400,
    "width": 320,
    "color": 1
  }
}
```

#### SDK 写法
```javascript
import { sticky } from '@n8n/workflow-sdk';

const doc = sticky({
  config: {
    name: 'API 文档',
    parameters: {
      content: '## 标题\n\n**字段:** 说明\n\n- 列表项1\n- 列表项2',
      width: 320,
      height: 400
    },
    position: [40, 100]
  }
});

// 在 workflow 中添加（不参与连线，只是画布上的便签）
export default workflow('id', 'name')
  .add(doc)
  .add(trigger)
  .to(node1)
  ...
```

#### 参数说明

| 参数 | 类型 | 说明 |
|------|------|------|
| `content` | string | Markdown 内容，支持 `##` 标题、`**粗体**`、`` `代码` ``、代码块、列表等 |
| `width` | number | 宽度（像素），最小 60，建议 260–400 |
| `height` | number | 高度（像素），最小 60，根据内容调整 |
| `color` | number | 背景颜色，见下表 |

#### 颜色值

| 值 | 颜色 | 推荐用途 |
|----|------|----------|
| 1 | 黄色 | 默认/一般说明 |
| 2 | 蓝色 | API 文档/信息 |
| 3 | 粉色 | 重要警告/注意事项 |
| 4 | 绿色 | 完成确认/凭证配置 |
| 5 | 白色 | 中性说明 |
| 6 | 灰色 | 备注/次要信息 |
| 7 | 黑色 | 标题/强调 |

#### 位置策略

n8n 画布坐标系：X 向右增大，Y 向下增大。一个标准节点占约 200×100 像素。

**定位原则：**
1. **先确定工作流节点的范围** — 看所有节点的 position，找出最小 X/Y 和最大 X/Y
2. **Sticky Note 放在流程上方或左侧** — 不遮挡节点和连线
3. **多个 Sticky Note 按主题分区** — 文档类放左上，凭证提示放左下

**常用位置公式：**

```
工作流起点 trigger 位置通常在 [240, 300] ~ [240, 400]

┌─────────────────────┐
│ Sticky Note         │  position: [trigger.x - 200, trigger.y - 300]
│ [40, 100]           │  即 trigger 左上方
│ width: 320          │
│ height: 400         │
└─────────────────────┘
         ↓
    [trigger] → [node1] → [node2] → ...
    [240,400]   [480,400]  [720,400]
```

**按场景推荐：**

| 场景 | 位置 | 尺寸 | 颜色 |
|------|------|------|------|
| API 文档（主说明） | trigger 左上 `[40, 100]` | 320 × 400~500 | 1(黄) 或 2(蓝) |
| 凭证配置提示 | trigger 左下 `[40, trigger.y + 200]` | 300 × 200~280 | 4(绿) |
| 流程架构图 | 所有节点上方居中 | 400~600 × 200 | 5(白) |
| 警告/注意事项 | 相关节点附近 | 260 × 150 | 3(粉) |

#### 尺寸估算

内容行数 → 高度的经验公式：

| 内容量 | 推荐 height |
|--------|-------------|
| 标题 + 3~5 行 | 200 |
| 标题 + 8~12 行 | 350~400 |
| 标题 + 代码块 + 列表 | 400~500 |
| 完整 API 文档 | 500~600 |

宽度一般固定 **300~400**，太宽会挤占节点空间。

#### 内容模板示例

**API 文档型：**
```markdown
## Workflow Name API

**Webhook:** `POST /webhook/path`

**Body:**
\`\`\`json
{ "input": "内容", "type": "可选参数" }
\`\`\`

**逻辑:**
- 条件A → 分支1
- 条件B → 分支2

**集合:** DailyLog
**时区:** UTC+8
```

**凭证配置型：**
```markdown
## 需手动配置凭证

- HTTP节点: httpBearerAuth → Mem API
- X请求: httpHeaderAuth
- LLM: openAiApi (Newapi)

> 创建后需通过 REST API 绑定凭证
```

**流程说明型：**
```markdown
## 流程架构

Webhook → 搜索 → IF(存在?)
  ├─ Yes: 读取 → 拼接 → 更新
  └─ No:  准备 → 创建

LLM 调用: 0次（纯 API 操作）
```

### Execute Sub-workflow 调用子流程
```json
{
  "id": "subflow_1",
  "name": "调用子流程",
  "type": "n8n-nodes-base.executeWorkflow",
  "typeVersion": 1,
  "position": [450, 300],
  "parameters": {
    "workflowId": "workflow_id_here"
  }
}
```

### Respond to Webhook 响应
```json
{
  "id": "respond_1",
  "name": "返回响应",
  "type": "n8n-nodes-base.respondToWebhook",
  "typeVersion": 1.1,
  "position": [650, 300],
  "parameters": {
    "respondWith": "json"
  }
}
```
