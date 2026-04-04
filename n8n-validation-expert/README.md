# n8n Validation Expert

用于解释 `validate_workflow` 返回的结果，帮助决定先修什么、后修什么。

---

## 现在的主路线

```text
写工作流代码 -> validate_workflow -> 分级读错误 -> 修代码 -> 再验证
```

---

## 文件说明

- [SKILL.md](SKILL.md): 验证总流程和优先级
- [ERROR_CATALOG.md](ERROR_CATALOG.md): 高频错误类型
- [FALSE_POSITIVES.md](FALSE_POSITIVES.md): 常见误判场景

---

## 适用问题

- 为什么 `validate_workflow` 没过
- 哪个报错应该先修
- 这是语法问题、参数问题还是连接问题
- 为什么看起来像误报

