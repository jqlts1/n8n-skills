# n8n Node Configuration

用于回答“这个节点到底该怎么配”。

---

## 现在的主路线

```text
search_nodes -> get_node_types -> 写 node(...) -> validate_workflow
```

这份 Skill 不再依赖旧版的节点级发现和依赖工具。

---

## 文件说明

- [SKILL.md](SKILL.md): 入口说明，解释新版配置思路
- [DEPENDENCIES.md](DEPENDENCIES.md): 字段依赖怎么判断
- [OPERATION_PATTERNS.md](OPERATION_PATTERNS.md): 常见节点字段形状和使用习惯

---

## 适用问题

- 某个节点某个操作需要哪些字段
- 为什么换个操作后必填项变了
- HTTP Request / Webhook / Slack / 数据库节点怎么写
- 应该先看哪个工具，还是先看验证结果

---

## 边界

这份 Skill 重点讲“怎么理解节点配置”。

下面这些问题应交给其他 Skill：

- 表达式写法：`n8n-expression-syntax`
- 验证报错解释：`n8n-validation-expert`
- 节点通用坑位：`n8n-node-pitfalls`
- 工具怎么选：`n8n-mcp-tools-expert`

