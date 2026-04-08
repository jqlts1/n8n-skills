# 操作模式示例

这份文件保留常见节点的“操作感知配置”知识，但示例全部改为 SDK `node(...)` 写法。

使用顺序仍然是：

```text
search_nodes -> get_node_types -> 参考这里的模式 -> 写 node(...) -> validate_workflow
```

---

## HTTP Request

### GET

```javascript
node('n8n-nodes-base.httpRequest', {
  method: 'GET',
  url: 'https://api.example.com/users',
  authentication: 'none',
});
```

### POST JSON

```javascript
node('n8n-nodes-base.httpRequest', {
  method: 'POST',
  url: 'https://api.example.com/users',
  authentication: 'none',
  sendBody: true,
  body: {
    contentType: 'json',
    content: {
      name: '={{$json.name}}',
      email: '={{$json.email}}',
    },
  },
});
```

重点：

- `POST` / `PUT` / `PATCH` 常常要配 `sendBody`
- body 结构要按当前类型定义来，不要自己发明字段

---

## Webhook

```javascript
trigger('n8n-nodes-base.webhook', {
  path: 'my-webhook',
  httpMethod: 'POST',
});
```

常见坑：

- 下游取值时忘记 webhook 数据常在 `$json.body`
- 原型里只放接收节点，没有响应或后续处理

---

## Slack

### 发消息

```javascript
node('n8n-nodes-base.slack', {
  resource: 'message',
  operation: 'post',
  channel: '#general',
  text: '={{$json.message}}',
});
```

### 更新消息

```javascript
node('n8n-nodes-base.slack', {
  resource: 'message',
  operation: 'update',
  messageId: '={{$json.messageId}}',
  text: '={{$json.updatedText}}',
});
```

重点：

- `post` 和 `update` 的必填项不同
- 不要把发消息的字段直接复制到更新消息

---

## Gmail

### 发邮件

```javascript
node('n8n-nodes-base.gmail', {
  resource: 'message',
  operation: 'send',
  to: 'user@example.com',
  subject: '通知',
  message: '处理完成',
});
```

重点：

- 收件人、主题、正文是最小骨架
- 具体附件、格式化字段仍应回 `get_node_types`

---

## Postgres

### 执行查询

```javascript
node('n8n-nodes-base.postgres', {
  operation: 'executeQuery',
  query: 'SELECT * FROM users WHERE status = $1',
  options: {},
});
```

### 插入数据

```javascript
node('n8n-nodes-base.postgres', {
  operation: 'insert',
  schema: 'public',
  table: 'users',
});
```

重点：

- 查询和插入不是同一套字段
- 数据库节点尤其要按操作读取类型

---

## Set

```javascript
node('n8n-nodes-base.set', {
  mode: 'manual',
  assignments: {
    assignments: [
      {
        name: 'status',
        value: 'processed',
        type: 'string',
      },
    ],
  },
  includeOtherFields: true,
});
```

重点：

- 它常用来整理字段，不是替代复杂逻辑
- 原型阶段也很适合用它模拟输出形状

---

## Code

### JavaScript

```javascript
node('n8n-nodes-base.code', {
  language: 'javaScript',
  mode: 'runOnceForAllItems',
  jsCode: `
const items = $input.all();
return items.map(item => ({
  json: {
    processed: true,
    name: item.json.name
  }
}));
  `,
});
```

### Python

```javascript
node('n8n-nodes-base.code', {
  language: 'python',
  mode: 'runOnceForAllItems',
  pythonCode: `
all_data = _items
return [{"processed": True, "count": len(all_data)}]
  `,
});
```

重点：

- Code 节点是最后手段，不要先上来就写代码
- 优先考虑 Set、IF、Filter、Merge 等内置节点

---

## IF

```javascript
node('n8n-nodes-base.if', {
  conditions: {
    options: { version: 2 },
    combinator: 'and',
    conditions: [],
  },
});
```

重点：

- IF 适合清晰的二分逻辑
- 条件结构要按当前版本定义写

---

## Switch

```javascript
node('n8n-nodes-base.switch', {
  mode: 'rules',
  rules: {
    values: [],
  },
});
```

重点：

- 分支超过两个时优先考虑 Switch
- 别把多个 IF 串成难维护的链

---

## OpenAI / Chat Model

```javascript
node('@n8n/n8n-nodes-langchain.lmChatOpenAi', {
  model: 'gpt-4o-mini',
});
```

重点：

- AI 相关节点不能孤立看
- 需要结合 Agent、Parser、Memory 一起设计

---

## Schedule Trigger

```javascript
trigger('n8n-nodes-base.scheduleTrigger', {
  rule: {
    interval: [
      {
        field: 'hours',
        hoursInterval: 1,
      },
    ],
  },
});
```

重点：

- 定时规则先写最小版本
- 上线前确认时区和执行频率

---

## Data Table

Data Table 是 n8n 内置的持久化存储，原型阶段推荐优先使用，无需外部数据库。

### 两个版本

| 版本 | 节点 ID | 用途 |
|------|---------|------|
| 普通节点 | `n8n-nodes-base.dataTable` | 工作流中直接操作 |
| AI Tool | `n8n-nodes-base.dataTableTool` | 给 AI Agent 作为工具调用 |

### Row 操作

#### Insert

```javascript
node('n8n-nodes-base.dataTable', {
  resource: 'row',
  operation: 'insert',
  dataTableId: { mode: 'id', value: 'your-table-id' },
});
```

#### Get

```javascript
node('n8n-nodes-base.dataTable', {
  resource: 'row',
  operation: 'get',
  dataTableId: { mode: 'id', value: 'your-table-id' },
});
```

#### Upsert

```javascript
node('n8n-nodes-base.dataTable', {
  resource: 'row',
  operation: 'upsert',
  dataTableId: { mode: 'id', value: 'your-table-id' },
});
```

#### rowExists / rowNotExists（双输出端口）

```javascript
node('n8n-nodes-base.dataTable', {
  resource: 'row',
  operation: 'rowExists',  // 或 'rowNotExists'
  dataTableId: { mode: 'id', value: 'your-table-id' },
});
```

> ⚠️ 双输出端口类似 IF 节点，只传递输入项，不返回表数据。需要表数据时，后接 `get` 操作。

### Table 管理操作

```javascript
// 创建表
node('n8n-nodes-base.dataTable', {
  resource: 'table',
  operation: 'create',
});

// 列出所有表
node('n8n-nodes-base.dataTable', {
  resource: 'table',
  operation: 'list',
});
```

重点：

- `dataTableId` 是必填，表需要先在 n8n 界面创建
- `rowExists` / `rowNotExists` 是条件分支，不是数据查询
- 给 AI Agent 用时换成 `n8n-nodes-base.dataTableTool`
- 具体筛选条件、列映射等参数仍需回 `get_node_types` 确认

详细说明见 [docs/配置类/datatable.md](/docs/配置类/datatable.md)

---

## 什么时候回到类型定义

下面这些情况都不要继续照着示例硬写：

- 换了操作
- 换了认证方式
- 某字段突然出现或消失
- 某节点有多个模式
- 你不确定字段层级

一旦出现这些情况，立刻回到 `get_node_types`。

