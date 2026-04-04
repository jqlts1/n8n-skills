---
name: n8n-node-pitfalls
description: |
  n8n 节点易错点和注意事项速查。当遇到以下情况时使用：
  - 使用 Merge、Loop Over Items、IF、Switch 等流程控制节点
  - 并行分支合并时出现问题
  - 节点输出不符合预期
  - 需要了解某个节点的常见陷阱
---

# n8n 节点易错点速查

常见节点的使用陷阱和正确做法。

---

## Merge 合并节点

### 核心问题
> ⚠️ **Number of Inputs 默认只有 2！** 合并超过 2 个分支必须手动调整。

### 参数说明

| 参数 | 默认值 | 说明 |
|------|--------|------|
| Mode | Append | 合并方式，Append 最常用 |
| Number of Inputs | 2 | **必须与实际分支数一致** |

### 常见错误

```
❌ 5 个分支连到 Merge，但 Number of Inputs = 2
   → 只有 2 个输入端口，其他 3 个无法连接

❌ 不用 Merge，直接把多个节点连到同一个目标节点
   → 每个源节点完成后单独触发目标节点
   → 目标节点被执行多次，而不是等待所有输入

✅ 正确做法：
   1. 添加 Merge 节点
   2. Settings → Number of Inputs = 实际分支数
   3. 各分支连接到 Merge 的不同输入端口
   4. Merge 等待所有输入到齐后一次性输出
```

### 配置位置
- n8n UI → Merge 节点 → Settings → Number of Inputs

---

## Loop Over Items 循环节点

### 核心问题
> ⚠️ **你可能不需要这个节点！** n8n 节点默认自动对每个 item 执行一次。
> 只有需要**控制批次大小**或**添加延迟**时才使用。

### 两个输出端口

| 端口 | 索引 | 说明 |
|------|------|------|
| done | 0 (第一个) | 循环**完成后**输出 |
| loop | 1 (第二个) | 当前批次，需要**回连**形成循环 |

### 常见错误

```
❌ 只连接 done 端口，忘记 loop 回连
   → 只执行一次，不会循环

❌ done 和 loop 接反了
   → 逻辑完全错乱

❌ 回连到错误的输入端口
   → 死循环或跳过处理

✅ 正确连接方式：
   Loop Over Items
       ├── [done:0] → 循环结束后的处理
       └── [loop:1] → 循环体处理 → Wait(可选) → 回连到 Loop 输入
```

---

## IF 条件节点

### 核心问题
> ⚠️ **两个输出端口顺序！** true 在前，false 在后。

### 两个输出端口

| 端口 | 索引 | 说明 |
|------|------|------|
| true | 0 (第一个) | 条件为真时输出 |
| false | 1 (第二个) | 条件为假时输出 |

### 常见错误

```
❌ 用 sourceIndex 连接时搞混 true/false
   → sourceIndex=0 是 true，sourceIndex=1 是 false

❌ 只连接一个输出端口
   → 另一个分支的数据被丢弃

✅ 使用 MCP 时用语义参数：
   branch="true" 或 branch="false"
   而不是 sourceIndex=0 或 sourceIndex=1
```

---

## Switch 分支节点

### 核心问题
> ⚠️ **输出端口数量与规则数量一致！** 每个规则对应一个输出。

### 常见错误

```
❌ 添加了 3 个规则，但只连接了 2 个输出
   → 第 3 个规则匹配的数据无处可去

❌ 用 sourceIndex 连接时数错了
   → 数据路由到错误的分支

✅ 使用 MCP 时用语义参数：
   case=0, case=1, case=2...
   而不是手动计算 sourceIndex
```

---

## HTTP Request 节点

### 核心问题
> ⚠️ **下载文件时注意输出属性名！** 默认是 `data`，可自定义。
> ⚠️ **分页优先用 HTTP Request 内置分页！** 不要先上 Loop。

### 下载文件配置

| 参数 | 路径 | 说明 |
|------|------|------|
| Response Format | options.response.response.responseFormat | 设为 `file` |
| Output Property Name | options.response.response.outputPropertyName | 二进制属性名 |

### 常见错误

```
❌ 下载多个文件用相同的 outputPropertyName
   → 后面的文件覆盖前面的

❌ 忘记设置 responseFormat: file
   → 返回的是文本而不是二进制

✅ 正确做法：
   - 每个下载节点用不同的 outputPropertyName
   - 例如：image1, image2, image3...
```

### 分页补充

- 需要拉多页接口时，优先用 HTTP Request 的内置分页配置
- 只有要做人为分批、等待、节流时，才考虑 Loop Over Items
- 详细说明看 [docs/节点操作类/HTTP分页.md](/docs/节点操作类/HTTP分页.md)

---

## Code 节点

### 核心问题
> ⚠️ **mode 参数决定执行方式！**

### 两种模式

| Mode | 说明 |
|------|------|
| runOnceForEachItem | 对每个 item 执行一次（默认） |
| runOnceForAllItems | 只执行一次，处理所有 items |

### 常见错误

```
❌ 想合并多个 items 但用了 runOnceForEachItem
   → 每个 item 单独处理，无法合并

❌ 想单独处理每个 item 但用了 runOnceForAllItems
   → 需要自己写循环逻辑

✅ 合并多个 items 时：
   mode: "runOnceForAllItems"
   使用 $input.all() 获取所有 items
```

### 合并二进制数据示例

```javascript
// mode: runOnceForAllItems
const allItems = $input.all();
const mergedBinary = {};

for (const item of allItems) {
  if (item.binary) {
    Object.assign(mergedBinary, item.binary);
  }
}

return [{
  json: { message: `已合并 ${Object.keys(mergedBinary).length} 个文件` },
  binary: mergedBinary
}];
```

---

## AI Agent 节点

### 核心问题
> ⚠️ **必须连接 Chat Model！** 没有语言模型，Agent 无法工作。

### 必须的连接

| 连接类型 | 说明 |
|----------|------|
| ai_languageModel | **必须** - 连接 Chat Model 节点 |
| ai_tool | 可选 - 连接工具节点 |
| ai_memory | 可选 - 连接记忆节点 |
| ai_outputParser | 可选 - 连接输出解析器 |

### 常见错误

```
❌ 创建 AI Agent 但没连接 Chat Model
   → 验证失败，无法运行

❌ 用 main 类型连接 Chat Model
   → 应该用 ai_languageModel 类型

✅ 正确连接：
   sourceOutput: "ai_languageModel"
   而不是 sourceOutput: "main"
```

---

## 并行分支 → 合并的完整模式

当需要并行执行多个操作，然后合并结果时：

```
触发器
    ├── 操作1 ──┐
    ├── 操作2 ──┤
    ├── 操作3 ──┼──→ Merge (Number of Inputs = N) ──→ 后续处理
    ├── ...    ──┤
    └── 操作N ──┘
```

### 关键配置

1. **Merge 节点**：Number of Inputs = 并行分支数
2. **各分支**：连接到 Merge 的不同输入端口（targetIndex 0, 1, 2...）
3. **后续节点**：只连接 Merge 的输出

---

## 快速检查清单

使用流程控制节点时，检查：

- [ ] Merge：Number of Inputs = 实际分支数？
- [ ] Loop：loop 端口有回连？
- [ ] IF：true/false 分支都连接了？
- [ ] Switch：每个 case 都有对应输出连接？
- [ ] Code：mode 设置正确？
- [ ] AI Agent：连接了 Chat Model？
- [ ] HTTP 下载：outputPropertyName 唯一？
