---
name: n8n-execution-testing
description: |
  Use when testing n8n workflows, deciding between mock testing and real execution, debugging failed runs, or figuring out what to inspect after execute_workflow, test_workflow, or get_execution results.
---

# n8n 执行与测试

这份 Skill 只解决一件事：工作流已经画出来了，下一步该怎么测、怎么定位失败。

---

## 什么时候用

- 你已经有工作流，想先安全验证
- 你不确定该用 `test_workflow` 还是 `execute_workflow`
- 运行失败了，但不知道先看哪里
- 你拿到了 `get_execution` 结果，不确定下一步该改节点、表达式还是连接

---

## 先做哪个测试

| 场景 | 推荐工具 | 原因 |
|------|----------|------|
| 还不想打真实外部服务 | `prepare_test_pin_data` + `test_workflow` | 先用模拟数据跑主链路 |
| 要验证真实触发和真实集成 | `execute_workflow` | 看真实输入和真实执行结果 |
| 已经失败，想定位哪一段坏了 | `get_execution` | 直接看节点级结果 |

---

## 推荐顺序

```text
1. validate_workflow
2. 能 mock 就先 mock：prepare_test_pin_data -> test_workflow
3. 需要真实链路时再 execute_workflow
4. 失败后立刻 get_execution
5. 根据失败位置回到节点配置、表达式或连接关系
```

---

## 如何判断用模拟还是真跑

### 优先用模拟测试

- 工作流里有外部 API
- 有凭证节点，不想真的发消息、写库、扣额度
- 你现在主要想确认主链路、分支、表达式、字段映射

### 直接真实执行

- 你要验证 webhook / chat / form 的真实入口
- 你要确认上线前的真实行为
- 模拟数据已经通过，现在只差最终联调

---

## 失败后先看什么

### 1. 如果 `validate_workflow` 就报错

- 先修代码结构、节点类型、参数结构
- 不要跳到执行阶段

### 2. 如果 `test_workflow` 失败

优先怀疑：

- Pin 数据结构不对
- 表达式引用错字段
- 分支条件和预期输入不匹配
- 某个逻辑节点对空值、数组、对象处理不对

### 3. 如果 `execute_workflow` 失败

优先怀疑：

- 真实输入结构和你想的不一样
- 凭证、权限、网络、外部服务响应异常
- 上游节点输出和测试数据不一致

### 4. 如果只知道“执行失败”

这还不够。必须继续：

```text
execute_workflow -> get_execution -> 找失败节点 -> 回到对应 Skill 修
```

---

## 失败类型到下一步动作

| 看到的问题 | 下一步 |
|------------|--------|
| 字段取不到、表达式报错 | 转到 `n8n-expression-syntax` |
| 某节点参数不对 | 转到 `n8n-node-configuration` |
| 分支、循环、合并异常 | 转到 `n8n-node-pitfalls` |
| 整体代码验证不过 | 转到 `n8n-validation-expert` |

---

## 常见误区

### ❌ 一上来就真实执行

外部服务一多，失败来源会混在一起，排查很慢。

### ❌ 模拟通过就当上线没问题

模拟只说明主链路和映射大体没问题，不代表真实凭证、权限、外部响应一定对。

### ❌ 失败后只盯最终报错

真正要看的不是一句报错，而是**哪个节点先坏了、它收到的输入是什么、它吐出了什么**。

---

## 相关入口

- `n8n-mcp-tools-expert`：各 MCP 工具总览
- `n8n-validation-expert`：验证错误怎么拆
- `n8n-node-configuration`：节点参数怎么补
- `n8n-expression-syntax`：表达式怎么修
