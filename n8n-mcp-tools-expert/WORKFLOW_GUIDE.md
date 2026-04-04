# 工作流管理指南

新版 MCP 的工作流管理主线很清楚：

```text
写 SDK 代码 -> validate_workflow -> create / update -> publish -> execute -> get_execution
```

它不再围绕旧版的 JSON 局部补丁操作展开。

---

## 工作流工具总览

| 工具 | 用途 |
|------|------|
| `validate_workflow` | 验证完整代码 |
| `create_workflow_from_code` | 从代码创建工作流 |
| `update_workflow` | 用完整代码整体替换 |
| `get_workflow_details` | 查看工作流详情 |
| `search_workflows` | 搜索工作流 |
| `execute_workflow` | 执行工作流 |
| `get_execution` | 查看执行结果 |
| `publish_workflow` | 发布 |
| `unpublish_workflow` | 停用 |
| `archive_workflow` | 归档 |
| `search_projects` | 找项目 |
| `search_folders` | 找文件夹 |

---

## 创建工作流

### 标准流程

```text
search_nodes
-> get_node_types
-> get_sdk_reference
-> 写代码
-> validate_workflow
-> create_workflow_from_code
-> publish_workflow
```

### create_workflow_from_code

```javascript
create_workflow_from_code({
  code,
  name,
  description,
  projectId,
  folderId
})
```

### 什么时候要指定项目和文件夹

如果工作流需要落到特定位置，先查：

```javascript
search_projects({query: "自动化"})
search_folders({query: "n8n"})
```

拿到 ID 后再创建。

---

## 更新工作流

### 关键点

`update_workflow` 是**整体替换**，不是局部补丁。

### 正确流程

```text
get_workflow_details
-> 修改完整 SDK 代码
-> validate_workflow
-> update_workflow
```

### update_workflow

```javascript
update_workflow({
  workflowId,
  code,
  name,
  description
})
```

如果你只想“加一个节点”或“改一条连线”，思路也仍然是：改完整代码后整体更新。

---

## 查看与搜索

### search_workflows

适合：

- 不记得工作流 ID
- 想看有没有现成工作流
- 想按项目范围搜

### get_workflow_details

适合：

- 准备更新前先看当前状态
- 想确认触发器和节点结构
- 想知道当前是否已发布

---

## 执行与调试

### execute_workflow

```javascript
execute_workflow({
  workflowId,
  inputs,
  executionMode
})
```

常见输入类型：

- chat
- form
- webhook

### executionMode

- `manual`：偏向测试当前草稿
- `production`：偏向执行已发布版本

### get_execution

```javascript
get_execution({
  workflowId,
  executionId,
  includeData,
  nodeNames,
  truncateData
})
```

适合：

- 看节点输出
- 看失败点
- 精确定位某个节点的数据问题

---

## 生命周期管理

### publish_workflow

发布后，工作流进入可生产执行状态。

### unpublish_workflow

适合暂停生产执行，但保留工作流本体。

### archive_workflow

适合彻底归档旧工作流，不再作为活跃内容维护。

---

## 常见错误

### 错误 1：不验证直接创建

这会把问题推迟到创建阶段甚至执行阶段。

### 错误 2：还按旧思路做局部更新

新版路径是完整代码整体替换，不要再按“补丁操作”思考。

### 错误 3：创建后忘记发布

如果目标是上线运行，创建并不等于已经进入生产状态。

### 错误 4：调试时不看 execution 详情

只知道“执行失败”没有意义，要回到 `get_execution` 看节点数据。

---

## 最佳实践

- 创建和更新前都先验证
- 更新前先看当前详情
- 把“修改工作流”理解成“修改完整代码”
- 调试时把 `execute_workflow` 和 `get_execution` 配套使用
- 需要上线时明确执行 `publish_workflow`

