---
name: n8n-node-configuration
description: |
  Use when configuring n8n nodes, determining required fields for a specific resource or operation, or translating node parameter knowledge into SDK code.
---

# n8n 节点配置

把“节点知道什么字段”和“工作流代码怎么写出来”拆开看，会更稳定。

新版 MCP 下，节点配置不再围绕旧的节点级校验工具展开，而是围绕下面这条主线：

```text
search_nodes -> get_node_types -> 读类型定义 -> 写 SDK 代码 -> validate_workflow
```

---

## 核心原则

### 1. 配置是操作感知的

同一个节点，`resource` 和 `operation` 不同，必填项就不同。

**示例：Slack**

```javascript
// 发送消息
node('n8n-nodes-base.slack', {
  resource: 'message',
  operation: 'post',
  channel: '#general',
  text: 'Hello',
});

// 更新消息
node('n8n-nodes-base.slack', {
  resource: 'message',
  operation: 'update',
  messageId: '123',
  text: 'Updated',
});
```

不要把一个操作的字段，机械复制到另一个操作上。

### 2. 字段依赖来自类型定义，不再来自单独的依赖工具

旧思路是“先 essentials，再 dependencies，再 info”。

新思路是：

1. `search_nodes` 找到正确节点
2. `get_node_types` 带上搜索结果返回的 discriminators
3. 直接读该操作下的 TypeScript 类型定义
4. 按类型写 `node(...)` 参数

如果某个字段只有在特定条件下才出现，优先从该操作对应的类型定义里理解，不要再找已移除的依赖工具。

### 3. 先写最小可运行配置

先满足“这个节点能表达当前操作”，再加可选项。

**HTTP Request 最小 POST 思路：**

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

### 4. 验证对象是整段工作流代码

新版 MCP 没有“节点级验证一步到位”的主路径。应该把节点放进工作流代码后，再用：

```javascript
validate_workflow({ code: workflowCode });
```

这样看到的错误，才和真实工作流上下文一致。

---

## 推荐工作流

### 标准流程

```text
1. 明确要用哪个节点、哪个操作
2. search_nodes({queries: [...]})
3. get_node_types({nodeIds: [...]})
4. 先写一个最小 node(...)
5. 放入完整 workflow 代码
6. validate_workflow({code})
7. 根据报错补字段、改类型、修表达式
```

### 什么时候一定要重查类型

- 换了 `resource`
- 换了 `operation`
- 同一节点切到不同认证方式
- 某字段只在特定条件下出现
- 不确定字段名是 `method`、`type`、`mode` 还是别的

---

## 常见场景

### HTTP Request

重点先确认：

- 请求方法
- 是否需要 `sendBody`
- body 内容结构
- 认证方式

常见坑：

- `POST` / `PUT` / `PATCH` 忘记打开 `sendBody`
- 直接猜 body 字段结构
- 把表达式写成普通字符串

### Webhook

重点先确认：

- 触发方式
- 路径
- 响应模式

常见坑：

- 下游取值时忘了 webhook 数据常在 `$json.body`
- 只画了接收，没有补响应节点或后续处理

### Slack / Gmail / 数据库节点

重点先确认：

- 资源类型
- 具体操作
- 凭证方式
- 操作特有的必填项

常见坑：

- 同一服务不同操作的必填项差异很大
- 从旧示例复制字段，结果字段名已经不适配当前操作

---

## 反模式

### ❌ 反模式 1：先猜参数，再去修

会反复撞墙。先拿 `get_node_types` 再写。

### ❌ 反模式 2：只看节点名，不看操作

“Slack 节点怎么配”这个问题本身太大。要落到“Slack 的哪个资源、哪个操作”。

### ❌ 反模式 3：把字段形状当成最终代码

像 [OPERATION_PATTERNS.md](OPERATION_PATTERNS.md) 这类文档主要帮助理解参数结构，不代表你可以跳过 `get_node_types` 直接照抄。

### ❌ 反模式 4：脱离工作流上下文做验证

节点写得像对的，不代表放进工作流就对。始终用完整代码跑 `validate_workflow`。

---

## 快速判断

### 这种情况先看 `search_nodes`

- 不确定节点名
- 不确定官方节点 ID
- 不确定应该用哪个服务节点

### 这种情况先看 `get_node_types`

- 已经知道节点名，但不知道字段
- 已经知道操作，但不确定必填项
- 某个字段是否存在取决于条件

### 这种情况先看验证结果

- 代码已经写出来
- 不知道是字段缺失、类型错误还是表达式错误
- 不确定问题出在节点本身还是节点连接

---

## 与其他 Skill 的关系

- **n8n-mcp-tools-expert**：解释什么时候用哪个 MCP 工具
- **n8n-validation-expert**：解读 `validate_workflow` 返回的问题
- **n8n-expression-syntax**：修表达式
- **n8n-node-pitfalls**：补节点级常见坑
- **HTTP 分页文档**：需要分页时，优先看 [docs/节点操作类/HTTP分页.md](/docs/节点操作类/HTTP分页.md)，不要先上 Loop
