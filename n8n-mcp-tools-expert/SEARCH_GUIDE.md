# 节点发现指南

新版 MCP 下，节点发现不再是“查文档 + 猜字段”，而是标准三步：

```text
search_nodes -> get_node_types -> 写 SDK 代码
```

---

## 三个工具的职责

| 工具 | 作用 | 什么时候用 |
|------|------|------------|
| `search_nodes` | 搜节点、看节点能力范围 | 还没确定节点 ID |
| `get_node_types` | 取节点类型定义 | 已经准备写代码 |
| `get_suggested_nodes` | 按类别给推荐组合 | 还在做技术选型 |

---

## 1. search_nodes

### 参数

```javascript
search_nodes({
  queries: ["webhook", "slack"]
})
```

### 你要看什么

- `nodeId`
- 显示名称
- 描述
- 版本
- `discriminators`
- `@builderHint`
- `@relatedNodes`

### 最重要的是 discriminators

这决定了你后面取到的类型定义是否足够精确。

例如你搜索 Slack，结果可能不只是“这是 Slack 节点”，还会告诉你：

- `resource`
- `operation`
- `mode`

这些信息后面要直接传给 `get_node_types`。

### 常见用法

```javascript
search_nodes({queries: ["http request"]})
search_nodes({queries: ["postgres insert"]})
search_nodes({queries: ["chat trigger", "ai agent"]})
```

---

## 2. get_node_types

### 参数

```javascript
get_node_types({
  nodeIds: [
    {
      nodeId: "n8n-nodes-base.httpRequest",
      resource: "message",
      operation: "send",
      mode: "runOnceForAllItems",
      version: "2.1"
    }
  ]
})
```

### 为什么必须带 discriminators

只传 `nodeId`，拿到的类型往往太宽。

带上 `resource` / `operation` / `mode` 后，你拿到的才是接近当前场景的类型定义，这样才能知道：

- 当前操作到底需要哪些字段
- 哪些字段只是可选
- 哪些字段只在某种模式下出现

### 正确心智

不是“先把代码写出来，再拿类型核对”，而是：

```text
先拿类型定义 -> 再写 node(...)
```

### 批量取值

```javascript
get_node_types({
  nodeIds: [
    { nodeId: "n8n-nodes-base.webhook" },
    { nodeId: "n8n-nodes-base.set" },
    { nodeId: "n8n-nodes-base.httpRequest" }
  ]
})
```

如果你已经确定是一条完整链路，批量取常用节点的类型会更高效。

---

## 3. get_suggested_nodes

### 参数

```javascript
get_suggested_nodes({
  categories: ["chatbot", "notification"]
})
```

### 适合什么场景

- 用户只说“做个聊天机器人”
- 你知道要做“通知工作流”，但不确定具体节点
- 你需要先搭一个原型组合

### 常见类别

- `chatbot`
- `notification`
- `scheduling`
- `data_transformation`
- `data_persistence`
- `data_extraction`
- `document_processing`
- `form_input`
- `content_generation`
- `triage`
- `scraping_and_research`

### 正确用法

它适合做“起步选型”，不适合替代 `get_node_types`。

即使已经拿到推荐节点，正式写代码前仍然要回到：

```text
search_nodes / get_suggested_nodes -> get_node_types -> 写代码
```

---

## 推荐工作流

### 场景 1：已经知道要什么节点

```text
search_nodes -> get_node_types -> node(...)
```

### 场景 2：只知道业务，不知道节点

```text
get_suggested_nodes -> search_nodes -> get_node_types -> node(...)
```

### 场景 3：一个节点有很多操作

```text
search_nodes -> 拿具体 discriminators -> get_node_types -> 只写当前操作
```

---

## 完整示例

### 例子：Webhook 收消息后发 Slack

```javascript
// 1. 搜节点
search_nodes({queries: ["webhook", "slack"]})

// 2. 取类型
get_node_types({
  nodeIds: [
    { nodeId: "n8n-nodes-base.webhook" },
    {
      nodeId: "n8n-nodes-base.slack",
      resource: "message",
      operation: "post"
    }
  ]
})

// 3. 根据类型写 SDK 代码
```

---

## 常见错误

### 错误 1：把 `queries` 当成单字符串

```javascript
// ❌
search_nodes({query: "slack"})

// ✅
search_nodes({queries: ["slack"]})
```

### 错误 2：只搜节点，不取类型

搜索只能告诉你“可能用哪个节点”，不能替代参数定义。

### 错误 3：不带 discriminators

这样很容易误把别的操作字段当成当前操作字段。

### 错误 4：拿推荐结果直接写代码

推荐工具解决的是“用什么节点”，不是“字段怎么写”。

---

## 最佳实践

- 搜索时尽量带业务词和动作词
- 发现一个节点支持很多操作时，一定回传 discriminators
- 真正开始写代码前，必须跑 `get_node_types`
- 节点知识和参数知识分开看：先找节点，再看类型

