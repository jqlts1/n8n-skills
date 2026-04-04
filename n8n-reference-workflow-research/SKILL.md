---
name: n8n-reference-workflow-research
description: |
  Use when looking for existing n8n workflow examples, asking for similar workflow references before building, comparing reusable patterns, or deciding whether to adapt a reference workflow or design one from scratch.
---

# n8n 参考工作流研究

这份 Skill 用来先找参考，再决定怎么做，不是一上来就从零搭。

---

## 什么时候用

- 用户说“有没有类似工作流可以参考”
- 用户说“先帮我找现成方案”
- 用户说“先搜几个示例再决定怎么搭”
- 你不确定这个流程该自己设计，还是先借鉴已有案例

---

## 资料优先级

### 1. 先看你自己的实例

优先用：

- `search_workflows`
- `get_workflow_details`

因为你自己实例里的工作流最接近真实可复用资产。

### 2. 再看参考资源接口

看这份文档：

- [docs/API接口/参照资源接口.md](/docs/API接口/参照资源接口.md)

它适合：

- 搜公开参考流程
- 看详情
- 下载 JSON
- 看 Mermaid 图

### 3. 最后用官方文档校对

优先用：

- `n8n-docs`

它不是给你找案例，而是用来确认：

- 节点能力是不是真的存在
- 某种搭法是不是官方支持
- 某些功能有没有限制

---

## 推荐流程

```text
1. 先抽出目标：触发方式、核心动作、输出结果
2. search_workflows 看自己实例里有没有类似的
3. 不够再查参照资源接口
4. 从候选里提炼可复用骨架
5. 用 n8n-docs 校对关键节点和能力
6. 再决定进入 n8n-prototype 还是 n8n-workflow-patterns
```

---

## 查找时要抓的关键信息

- 触发方式：Webhook / 定时 / 表单 / AI / 手动
- 核心动作：拉数据、写库、发消息、分类、审批
- 外部集成：Slack、飞书、数据库、HTTP API、AI 模型
- 复杂度：简单主链路、带分支、带循环、带缓存

不要只搜一个很宽的关键词。要尽量把“触发 + 动作 + 输出”一起带上。

---

## 输出时要给什么

研究结果至少要整理成这 4 块：

1. 最像的参考候选
2. 可以直接借用的流程骨架
3. 只能借思路、不能直接照搬的部分
4. 下一步应该进入哪个 Skill

### 常见下一步

- 结构还没定：转 `n8n-prototype`
- 模式还没定：转 `n8n-workflow-patterns`
- 已经知道要用哪些节点：转 `n8n-node-configuration`

---

## 重点提醒

### 参考流程不是可直接上线的成品

重点看的是：

- 结构有没有借鉴价值
- 节点组合是否合理
- 分支、循环、缓存、错误处理怎么设计

不要把外部凭证、字段名、路径、业务规则直接照搬。

### 参考资源接口不是实例 MCP

它只能帮你找示例，不会直接修改你自己的工作流。

### 先找骨架，再配细节

研究阶段的目标不是“把每个字段都抠出来”，而是先判断：

- 值不值得参考
- 哪些部分可以复用
- 下一步怎么落地最快

---

## 常见误区

### ❌ 只看标题像不像

标题相似不代表流程真的能复用。必须看触发方式、核心节点和输出。

### ❌ 直接下载 JSON 就照搬

参考工作流常常只适合借结构，不适合直接跑在你的实例里。

### ❌ 找到参考后还继续乱搜

一旦已经找到 1 到 3 个足够接近的案例，就该开始提炼骨架，不要无限搜。

---

## 与其他 Skill 的关系

- `n8n-prototype`：把研究结果变成可讨论的流程骨架
- `n8n-workflow-patterns`：确定该用哪种工作流模式
- `n8n-mcp-tools-expert`：需要回到实例 MCP 或官方文档时使用
- `n8n-node-configuration`：准备进入真实节点配置时使用
