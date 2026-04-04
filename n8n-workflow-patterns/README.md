# n8n Workflow Patterns

用于回答“这个业务流程该长成什么样”。

---

## 覆盖范围

- Webhook 处理
- HTTP API 集成
- 数据库操作
- AI Agent 工作流
- 定时任务

---

## 依赖关系

这份 Skill 主要提供结构，不负责工具细节。

实现时通常会联动：

- `n8n-mcp-tools-expert`
- `n8n-node-configuration`
- `n8n-expression-syntax`
- `n8n-validation-expert`

---

## 文件说明

- [SKILL.md](SKILL.md): 模式选择和通用检查清单
- [webhook_processing.md](webhook_processing.md): Webhook 模式
- [http_api_integration.md](http_api_integration.md): HTTP API 模式
- [database_operations.md](database_operations.md): 数据库模式
- [ai_agent_workflow.md](ai_agent_workflow.md): AI 模式
- [scheduled_tasks.md](scheduled_tasks.md): 定时任务模式

