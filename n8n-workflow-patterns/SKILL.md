---
name: n8n-workflow-patterns
description: |
  Use when designing a new n8n workflow, choosing a workflow structure, or deciding between webhook, API integration, database, AI agent, and scheduled task patterns.
---

# n8n 工作流模式

这份 Skill 讲的是“结构怎么搭”，不是“字段怎么抠”。

如果已经知道业务目标，但还不知道工作流应该长什么样，先来这里选模式。

---

## 五种核心模式

### 1. Webhook 处理

```text
Webhook -> 校验 -> 转换 -> 响应 / 通知
```

适合：

- 接外部系统推送
- 表单提交
- 即时触发的业务回调

### 2. HTTP API 集成

```text
触发器 -> HTTP Request -> 转换 -> 后续动作
```

适合：

- 调第三方接口
- 拉取数据
- 同步外部服务

### 3. 数据库操作

```text
触发器 -> 查询 / 写入 -> 转换 -> 校验 / 通知
```

适合：

- 数据同步
- ETL
- 定时清洗或对账

### 4. AI Agent 工作流

```text
触发器 -> AI Agent -> 工具 / 记忆 / 结构化输出 -> 结果
```

适合：

- 智能问答
- 分类、总结、抽取
- 需要工具调用的 AI 流程

### 5. 定时任务

```text
Schedule -> 拉取 -> 处理 -> 输出 -> 记录
```

适合：

- 日报、周报
- 周期巡检
- 定时同步

---

## 通用检查清单

### 规划阶段

- [ ] 先确定属于哪一种模式
- [ ] 列出触发器、核心处理、输出节点
- [ ] 想清楚失败时怎么处理

### 实现阶段

- [ ] 用 `search_nodes` 找节点
- [ ] 用 `get_node_types` 确认关键节点的操作和字段
- [ ] 把节点组织成清晰主链路
- [ ] 只在需要时加分支、循环、合并

### 验证阶段

- [ ] 用 `validate_workflow` 验证完整代码
- [ ] 用样例数据测试主路径
- [ ] 检查空数据、异常数据、失败分支

### 发布阶段

- [ ] 明确工作流设置
- [ ] 需要上线时使用 `publish_workflow`
- [ ] 观察前几次执行结果

---

## 怎么和其他 Skill 配合

- **n8n-mcp-tools-expert**：找工具和节点
- **n8n-node-configuration**：把节点配对
- **n8n-expression-syntax**：写表达式
- **n8n-validation-expert**：看验证结果
- **n8n-node-pitfalls**：排特殊节点坑

---

## 常见误区

### ❌ 一上来就堆节点

先定模式，再画主链路。

### ❌ 把架构问题当配置问题

有些流程不是字段没填对，而是整个结构就不顺。

### ❌ 忽略触发器和输出

中间处理写得再漂亮，没有清晰入口和出口，工作流仍然是不完整的。

---

## 继续深入

- [webhook_processing.md](webhook_processing.md)
- [http_api_integration.md](http_api_integration.md)
- [database_operations.md](database_operations.md)
- [ai_agent_workflow.md](ai_agent_workflow.md)
- [scheduled_tasks.md](scheduled_tasks.md)

