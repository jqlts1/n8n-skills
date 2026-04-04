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
```json
{
  "id": "note_1",
  "name": "Sticky Note",
  "type": "n8n-nodes-base.stickyNote",
  "typeVersion": 1,
  "position": [250, 100],
  "parameters": {
    "content": "## 流程说明\n这是一个示例流程\n\n**主要步骤：**\n- 步骤1\n- 步骤2\n- 步骤3",
    "height": 200,
    "width": 300,
    "color": 1
  }
}
```

**Sticky Note 颜色值:**
| 值 | 颜色 | 用途 |
|----|------|------|
| 1 | 黄色 | 默认/一般说明 |
| 2 | 蓝色 | 信息/提示 |
| 3 | 粉色 | 重要/警告 |
| 4 | 绿色 | 完成/确认 |
| 5 | 白色 | 中性说明 |
| 6 | 灰色 | 备注/次要 |
| 7 | 黑色 | 标题/强调 |

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
