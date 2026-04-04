---
name: n8n-prototype
description: |
  快速绘制 n8n 工作流原型图。用于在正式开发前设计流程结构，使用 noOp 节点占位、Sticky Note 标注说明。

  触发场景：
  - 用户要求"画一个n8n流程"、"设计工作流"、"搭建原型"
  - 用户描述业务流程需要转化为 n8n 工作流
  - 用户需要快速可视化一个自动化流程的结构
---

# n8n 工作流原型设计

快速绘制 n8n 工作流原型，先画结构确认流程，再逐步替换真实节点。

---

## 原型模式选择

开始前先确认用户需要哪种原型模式：

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| **骨架模式** | 全部用 noOp + Sticky Note，只画流程结构 | 快速讨论流程逻辑，还没确定技术方案 |
| **混合模式** ⭐推荐 | 核心节点用真实的，业务节点用 noOp | 流程基本确定，需要验证可行性 |
| **可执行模式** | 尽量用真实节点，可直接测试 | 流程已确定，直接搭建可运行版本 |

### 混合模式节点选择规则

| 节点类型 | 必须用真实节点 | 可用 noOp 占位 |
|----------|---------------|----------------|
| 触发器 (Trigger) | ✅ 必须 | - |
| 流程控制 (If/Switch/Merge) | ✅ 必须 | - |
| AI Agent | ✅ 推荐 | 简单场景可占位 |
| Data Table 缓存 | ✅ 推荐 | - |
| HTTP Request | 可选 | ✅ 可占位 |
| 第三方集成 (钉钉/飞书等) | - | ✅ 占位 |
| 业务逻辑处理 | - | ✅ 占位 |

---

## 核心规则

1. **noOp 占位** - 未实现的业务逻辑用 `n8n-nodes-base.noOp` 占位，通过命名表达用途
2. **必须有触发器** - 默认使用 `n8n-nodes-base.manualTrigger`，用户指定则用其他
3. **中文命名** - 格式：`动词 + 对象`，如 "解析PDF"、"发送通知"
4. **添加 Sticky Note** - 用便签标注流程说明，让原型一目了然
5. **布局规范** - 起始 [250, 300]，水平间距 200px，垂直间距 150px

---

## noOp 占位命名规范

### 命名格式
```
[类型标签] 动词 + 对象
```

### 类型标签（可选）
| 标签 | 含义 | 示例 |
|------|------|------|
| `[API]` | 需要调用外部 API | `[API] 获取天气数据` |
| `[DB]` | 数据库操作 | `[DB] 查询用户信息` |
| `[通知]` | 发送通知/消息 | `[通知] 发送钉钉消息` |
| `[文件]` | 文件处理 | `[文件] 解析Excel` |
| `[AI]` | AI/大模型处理 | `[AI] 智能分类` |
| `[人工]` | 需要人工介入 | `[人工] 审核内容` |

### 命名示例
```
✅ 好的命名：
- [API] 调用支付接口
- [通知] 发送企业微信
- [DB] 更新订单状态
- [文件] 生成PDF报告
- 计算折扣金额
- 格式化输出数据

❌ 避免的命名：
- 处理（太模糊）
- 步骤1（无意义）
- TODO（不明确）
```

---

## 从原型到正式：替换指南

### 阶段一：骨架原型
```
Manual Trigger → [API] 获取数据 → [AI] 智能分析 → If判断 → [通知] 发送结果
                    (noOp)           (noOp)                    (noOp)
```

### 阶段二：替换核心节点
```
Manual Trigger → HTTP Request → AI Agent → If判断 → [通知] 发送结果
                  (真实节点)    (真实节点)            (noOp)
```

### 阶段三：完整实现
```
Manual Trigger → HTTP Request → AI Agent → If判断 → Slack/钉钉
                  (配置完成)    (配置完成)           (配置完成)
```

### 从原型落地到工作流代码

当原型确认后，推荐进入新版 MCP 落地流程：

```text
search_nodes -> get_node_types -> 写 SDK 代码 -> validate_workflow -> create_workflow_from_code
```

原型负责确认结构，正式创建由 `create_workflow_from_code` 完成。

### 替换对照表

| noOp 占位 | 替换为真实节点 | 配置要点 |
|-----------|---------------|----------|
| `[API] 调用XXX` | HTTP Request | URL、Method、Headers、Body |
| `[DB] 查询数据` | Data Table / Postgres | 表名、查询条件 |
| `[AI] 智能处理` | AI Agent + Chat Model | Prompt、Output Parser |
| `[文件] 解析PDF` | Extract from File | 文件路径、解析格式 |
| `[文件] 生成Excel` | Convert to File | 数据映射 |
| `[通知] 发钉钉` | HTTP Request (钉钉Webhook) | Webhook URL、消息格式 |
| `[通知] 发邮件` | Send Email | SMTP配置、收件人 |
| `[人工] 审核` | Wait + Form | 等待条件、表单字段 |

---

## 节点速查表

### 触发器 (Trigger)

| 名称 | 类型 | 版本 | 说明 |
|-----|------|-----|------|
| Manual Trigger | `n8n-nodes-base.manualTrigger` | 1 | 手动点击触发，测试首选 |
| Webhook | `n8n-nodes-base.webhook` | 2 | HTTP 接口触发 |
| Schedule Trigger | `n8n-nodes-base.scheduleTrigger` | 1.2 | 定时任务 |
| Form Trigger | `n8n-nodes-base.formTrigger` | 2.2 | 网页表单触发 |
| Chat Trigger | `@n8n/n8n-nodes-langchain.chatTrigger` | 1.4 | AI 聊天入口 |
| Manual Chat Trigger | `@n8n/n8n-nodes-langchain.manualChatTrigger` | 1 | 手动测试聊天 |
| Error Trigger | `n8n-nodes-base.errorTrigger` | 1 | 其他工作流出错时触发 |
| Execute Workflow Trigger | `n8n-nodes-base.executeWorkflowTrigger` | 1 | 被其他工作流调用时触发 |

### 流程控制

| 名称 | 类型 | 版本 | 说明 |
|-----|------|-----|------|
| If | `n8n-nodes-base.if` | 2.2 | 二选一分支 (true/false) |
| Switch | `n8n-nodes-base.switch` | 3.2 | 多条件分支 |
| Filter | `n8n-nodes-base.filter` | 2.2 | 过滤数据 |
| Merge | `n8n-nodes-base.merge` | 3.1 | 合并多个分支 |
| Split In Batches | `n8n-nodes-base.splitInBatches` | 3.1 | 分批循环处理 |
| Loop Over Items | `n8n-nodes-base.splitInBatches` | 3.1 | 逐条处理（同上） |
| Wait | `n8n-nodes-base.wait` | 1.1 | 等待/延迟 |
| Stop and Error | `n8n-nodes-base.stopAndError` | 1 | 抛出错误并停止 |

### 数据处理

| 名称 | 类型 | 版本 | 说明 |
|-----|------|-----|------|
| Set (Edit Fields) | `n8n-nodes-base.set` | 3.4 | 设置/编辑字段，模拟数据 |
| Code | `n8n-nodes-base.code` | 2 | 运行 JS/Python 代码 |
| Split Out | `n8n-nodes-base.splitOut` | 1 | 拆分数组为多条 |
| Aggregate | `n8n-nodes-base.aggregate` | 1 | 多条合并为一条 |
| Summarize | `n8n-nodes-base.summarize` | 1 | 统计求和/计数/最大值等 |
| Sort | `n8n-nodes-base.sort` | 1 | 排序 |
| Limit | `n8n-nodes-base.limit` | 1 | 限制条数 |
| Remove Duplicates | `n8n-nodes-base.removeDuplicates` | 1 | 去重 |
| Compare Datasets | `n8n-nodes-base.compareDatasets` | 2.2 | 比较两组数据 |
| Item Lists | `n8n-nodes-base.itemLists` | 3.1 | 列表操作工具 |

### 数据转换

| 名称 | 类型 | 版本 | 说明 |
|-----|------|-----|------|
| Date & Time | `n8n-nodes-base.dateTime` | 2 | 日期时间处理 |
| Crypto | `n8n-nodes-base.crypto` | 1 | 加密/哈希/签名 |
| HTML | `n8n-nodes-base.html` | 1.2 | HTML 解析/提取 |
| Markdown | `n8n-nodes-base.markdown` | 1 | Markdown ↔ HTML |
| XML | `n8n-nodes-base.xml` | 1.1 | XML 解析/生成 |
| JSON | n8n 内置表达式 | - | 用 `JSON.parse()` / `JSON.stringify()` |

### 文件处理

| 名称 | 类型 | 版本 | 说明 |
|-----|------|-----|------|
| Read Binary File | `n8n-nodes-base.readBinaryFile` | 1 | 读取本地文件 |
| Write Binary File | `n8n-nodes-base.writeBinaryFile` | 1 | 写入本地文件 |
| Extract from File | `n8n-nodes-base.extractFromFile` | 1 | 解析 PDF/CSV/Excel 等 |
| Convert to File | `n8n-nodes-base.convertToFile` | 1.1 | 生成 CSV/Excel/PDF 等 |
| Compression | `n8n-nodes-base.compression` | 1 | 压缩/解压 ZIP |

### 网络请求

| 名称 | 类型 | 版本 | 说明 |
|-----|------|-----|------|
| HTTP Request | `n8n-nodes-base.httpRequest` | 4.2 | 通用 HTTP 请求 |
| Respond to Webhook | `n8n-nodes-base.respondToWebhook` | 1.1 | 返回 Webhook 响应 |
| GraphQL | `n8n-nodes-base.graphql` | 1 | GraphQL 请求 |
| FTP | `n8n-nodes-base.ftp` | 1 | FTP 文件传输 |
| SSH | `n8n-nodes-base.ssh` | 1 | SSH 远程命令 |

### 数据存储

| 名称 | 类型 | 版本 | 说明 |
|-----|------|-----|------|
| Data Table | `n8n-nodes-base.dataTable` | 1.1 | n8n 内置数据表（原型首选） |
| Postgres | `n8n-nodes-base.postgres` | 2.5 | PostgreSQL 数据库 |
| MySQL | `n8n-nodes-base.mySql` | 2.4 | MySQL 数据库 |
| MongoDB | `n8n-nodes-base.mongoDb` | 1.2 | MongoDB 数据库 |
| Redis | `n8n-nodes-base.redis` | 1 | Redis 缓存 |

> ⚠️ **原型阶段推荐用 Data Table**，无需外部数据库配置

### AI 节点

| 名称 | 类型 | 版本 | 说明 |
|-----|------|-----|------|
| AI Agent | `@n8n/n8n-nodes-langchain.agent` | 3.1 | AI 代理，核心节点 |
| Basic LLM Chain | `@n8n/n8n-nodes-langchain.chainLlm` | 1.5 | 简单 LLM 调用 |
| Structured Output Parser | `@n8n/n8n-nodes-langchain.outputParserStructured` | 1.3 | 结构化输出 |
| OpenAI Chat Model | `@n8n/n8n-nodes-langchain.lmChatOpenAi` | 1.3 | OpenAI/兼容模型 |
| Anthropic Chat Model | `@n8n/n8n-nodes-langchain.lmChatAnthropic` | 1.3 | Claude 模型 |
| Window Buffer Memory | `@n8n/n8n-nodes-langchain.memoryBufferWindow` | 1.3 | 对话记忆（窗口） |
| Chat Memory Manager | `@n8n/n8n-nodes-langchain.memoryManager` | 1.1 | 管理聊天记忆 |
| Respond to Chat | `@n8n/n8n-nodes-langchain.chat` | 1 | 对话中间回复 |

> ⚠️ **重要**: AI Agent 必须连接 Chat Model，不能留空！

### 原型工具

| 名称 | 类型 | 版本 | 说明 |
|-----|------|-----|------|
| No Operation | `n8n-nodes-base.noOp` | 1 | 占位节点，什么都不做 |
| Sticky Note | `n8n-nodes-base.stickyNote` | 1 | 便签说明 |
| Debug Helper | `n8n-nodes-base.debugHelper` | 1 | 生成测试数据/模拟错误 |
| Execute Sub-workflow | `n8n-nodes-base.executeWorkflow` | 1.2 | 调用子工作流 |

### 常用通知（需用 HTTP Request 实现）

| 平台 | 实现方式 | 说明 |
|------|---------|------|
| 钉钉 | HTTP Request → Webhook URL | 机器人 Webhook |
| 企业微信 | HTTP Request → Webhook URL | 机器人 Webhook |
| 飞书 | HTTP Request → Webhook URL | 机器人 Webhook |
| Slack | `n8n-nodes-base.slack` | 有官方节点 |
| Telegram | `n8n-nodes-base.telegram` | 有官方节点 |
| Email | `n8n-nodes-base.emailSend` | SMTP 发送 |

---

## AI 节点配置要点

### AI Agent 必须三件套
```
AI Agent (hasOutputParser: true)
    │
    ├── [ai_languageModel] → Chat Model (必须连接!)
    │
    └── [ai_outputParser] → Structured Output Parser (需要JSON输出时)
```

### 结构化输出触发关键词
- "提取XXX字段"
- "返回JSON格式"
- "输出结构化数据"
- "按格式返回"

### 常见错误
```
❌ 错误: JSON.stringify({ field1: $json.field1, ... })
✅ 正确: JSON.stringify($json.output)

❌ 错误: {{ $('节点名').item.json.字段 }}  (引用上一节点)
✅ 正确: {{ $json.字段 }}
```

---

## 流程模式

### 线性流程
```
Trigger → A → B → C
```

### 条件分支
```
Trigger → If
           ├── True  → A
           └── False → B
```

### 多分支
```
Trigger → Switch
           ├── Case1 → A
           ├── Case2 → B
           └── Default → C
```

### 分支合并
```
Trigger → If
           ├── True  → A ─┐
           └── False → B ─┴→ Merge → C
```

### 缓存模式
```
Trigger → Data Table(rowNotExists) → 调用API → 保存缓存
        → Data Table(rowExists) → 获取缓存
                                        ↓
                              Merge → 后续处理
```

### Webhook 异步响应
```
Webhook → 提取参数 → Respond to Webhook → 后台处理(noOp)
```

### 循环处理（Loop Over Items）
```
Trigger → Loop Over Items ←───────────────┐
              │                           │
              ├── [done] → 循环完成后处理   │
              │                           │
              └── [loop] → 处理单条 → Wait ─┘
```

---

## Loop Over Items 详解

### 重要提示
> ⚠️ **你可能不需要这个节点！** n8n 节点默认会自动对每个输入项执行一次。
> 只有当你需要**控制批次大小**或**在循环中添加延迟**时才需要使用。

### 节点信息
| 属性 | 值 |
|------|-----|
| 类型 | `n8n-nodes-base.splitInBatches` |
| 版本 | 3.1 |
| 别名 | Split In Batches / Loop Over Items |

### 两个输出端口（关键！）

| 端口 | 名称 | 索引 | 说明 |
|------|------|------|------|
| 第一个 | `done` | 0 | 所有批次处理**完成后**输出，连接到循环后的节点 |
| 第二个 | `loop` | 1 | 当前批次数据，连接到循环**内部**的处理节点 |

### 参数配置

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `batchSize` | number | 1 | 每批处理的数量 |
| `options.reset` | boolean | false | 是否从头开始（用于新数据集） |

### 正确用法示意图

```
                    ┌─────────────────────────────┐
                    │                             │
                    ▼                             │
输入数据 → Loop Over Items                         │
              │                                   │
              ├── [done:0] → 循环结束 → 后续处理    │
              │                                   │
              └── [loop:1] → HTTP请求 → Wait(1秒) ─┘
                              ▲
                              │
                         循环体内的节点
```

### 常见错误

```
❌ 错误1: 只连接一个输出
Loop Over Items → 处理节点 → 结束
（缺少循环回连，只执行一次）

✅ 正确: 两个输出都要连接
Loop Over Items
    ├── [done] → 结束处理
    └── [loop] → 处理节点 → 回连到 Loop Over Items

❌ 错误2: 把 done 和 loop 接反了
Loop Over Items
    ├── [done] → 处理节点 → 回连  （错！done是循环结束后）
    └── [loop] → 结束处理         （错！loop是循环体）

✅ 正确: done=循环后, loop=循环体
```

### 什么时候需要 Loop Over Items？

| 场景 | 是否需要 | 原因 |
|------|---------|------|
| 批量调用API | ✅ 需要 | 控制并发，避免限流 |
| 逐条处理+延迟 | ✅ 需要 | 在每条之间加 Wait |
| 普通数据处理 | ❌ 不需要 | n8n 自动逐条执行 |
| 简单转换 | ❌ 不需要 | 直接用 Set/Code |

### 典型场景：批量API调用+限流

```
Trigger → Loop Over Items (batchSize=1)
              │
              ├── [done] → 发送完成通知
              │
              └── [loop] → HTTP Request → Wait(1秒) → 回连Loop
```

**连接配置**:
```json
{
  "Loop Over Items": {
    "main": [
      [{ "node": "发送完成通知", "type": "main", "index": 0 }],
      [{ "node": "HTTP Request", "type": "main", "index": 0 }]
    ]
  },
  "Wait": {
    "main": [
      [{ "node": "Loop Over Items", "type": "main", "index": 0 }]
    ]
  }
}
```

---

## 最佳实践

### 1. 数据去重/防重复
```
Webhook → Data Table(rowExists)
           ├── 存在 → noOp(跳过)
           └── 不存在 → 处理 → Data Table(insert)
```

### 2. API缓存
```
Trigger → Data Table(get) → If(有缓存?)
           ├── True  → 返回缓存
           └── False → HTTP Request → Data Table(upsert)
```

### 3. 批量限流
```
Trigger → Split In Batches → HTTP Request → Wait(1秒) → 循环
```

### 4. AI + 人工复核
```
Trigger → AI Agent → If(置信度>90%?)
           ├── True  → 自动执行
           └── False → [人工] 审核
```

### 5. 错误处理
```
Trigger → 主处理
           ├── 成功 → 完成
           └── 失败 → Wait → 重试 → 超限通知
```

---

## n8n 表达式速查

| 表达式 | 说明 |
|--------|------|
| `{{ $json.字段 }}` | 上一节点的字段 |
| `{{ $json["字段名"] }}` | 字段名有特殊字符时 |
| `{{ $('节点名').item.json.字段 }}` | 指定节点的字段 |
| `{{ $input.all() }}` | 上一节点所有数据 |
| `{{ $now }}` | 当前时间 |
| `{{ $today }}` | 今天日期 |
| `{{ $runIndex }}` | 当前执行索引 |
| `{{ $itemIndex }}` | 当前数据项索引 |

---

## 示例

**用户**: "帮我画一个简历筛选流程：解析PDF，AI评估，根据评分通知HR或归档"

**骨架模式**:
```
[Sticky Note: 简历筛选流程 - 评分>=80通知HR，否则归档]

Manual Trigger
    ↓
[文件] 解析简历PDF (noOp)
    ↓
[AI] 评估简历 (noOp)
    ↓
If: 评分>=80?
    ├── True  → [通知] 通知HR (noOp)
    └── False → [DB] 归档 (noOp)
```

**混合模式**:
```
Manual Trigger
    ↓
[文件] 解析简历PDF (noOp)
    ↓
AI Agent (评估简历)
    ↓
If: 评分>=80?
    ├── True  → [通知] 通知HR (noOp)
    └── False → Data Table (归档)
```
