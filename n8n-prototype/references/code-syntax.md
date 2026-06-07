# Code 节点语法 (n8n 2.0+)

## ⚠️ 重要变更

n8n 2.0 已弃用 `_input` 方式，必须使用新的数据访问方式！

## 运行模式

| 模式 | 说明 | 数据访问方式 |
|------|------|-------------|
| Run Once for All Items | 一次处理所有数据 | `_items[0]["json"]` |
| Run Once for Each Item | 逐条处理数据 | `_item["json"]` |

## Python 预装库

- `requests` - HTTP 请求
- `pandas` - 数据处理
- `numpy` - 数值计算
- 支持自定义安装第三方库

## Python 代码示例

### Run Once for All Items 模式

```python
# 获取所有输入数据
all_data = _items  # 列表，每项是 {"json": {...}, "binary": {...}}

# 获取第一条数据的 json
first_item = _items[0]["json"]

# 遍历处理所有数据
results = []
for item in _items:
    data = item["json"]
    # 处理逻辑...
    results.append({"processed": data["name"]})

return results
```

### Run Once for Each Item 模式

```python
# 直接获取当前项的数据
data = _item["json"]

# 处理逻辑
result = {
    "name": data["name"],
    "processed": True
}

return result
```

### 使用 requests 调用 API

```python
import requests

data = _items[0]["json"]
response = requests.post(
    "https://api.example.com/process",
    json={"input": data["content"]}
)
result = response.json()

return [{"result": result}]
```

### 使用 pandas 处理数据

```python
import pandas as pd

# 将输入转为 DataFrame
data_list = [item["json"] for item in _items]
df = pd.DataFrame(data_list)

# 数据处理
df["score"] = df["score"].astype(float)
df_filtered = df[df["score"] >= 80]

# 转回 n8n 格式
return df_filtered.to_dict("records")
```

### 错误处理

```python
try:
    data = _items[0]["json"]
    # 处理逻辑...
    return [{"success": True, "data": result}]
except Exception as e:
    return [{"success": False, "error": str(e)}]
```

## JavaScript 代码示例

### Run Once for All Items 模式

```javascript
// 获取所有输入数据
const items = $input.all();

// 处理数据
const results = items.map(item => {
  return {
    json: {
      processed: true,
      originalName: item.json.name
    }
  };
});

return results;
```

### Run Once for Each Item 模式

```javascript
// 获取当前项
const item = $input.item;

// 处理逻辑
return {
  json: {
    name: item.json.name,
    processed: true
  }
};
```

### JS 沙箱里发 HTTP 请求 ⚠️

n8n Code 节点 JS 沙箱**没有** `fetch` / `$helpers` / `$http`，调它们直接报 `xxx is not defined`。
唯一靠谱的方式是 `this.helpers.httpRequest`：

```javascript
// ✅ 正确：带 query 参数 + 原始字符串响应
const res = await this.helpers.httpRequest({
  method: 'GET',
  url: 'https://api.example.com/path',
  qs: { foo: 'bar' },           // 自动 URL-encode 拼接到 ?foo=bar
  json: false,                   // false=原始字符串；true=自动解析 JSON
  returnFullResponse: false,     // false=只要 body；true=拿 {body, headers, statusCode}
});

// POST + JSON body
const res2 = await this.helpers.httpRequest({
  method: 'POST',
  url: 'https://api.example.com/submit',
  body: { name: 'foo', count: 1 },
  json: true,                    // body 自动 JSON.stringify，响应自动解析
});
```

**重试模板**（响应 body 包含错误标记时重试 3 次，每次间隔 1 秒）：

```javascript
const sleep = (ms) => new Promise(r => setTimeout(r, ms));
const MAX_ATTEMPTS = 4;
const ERROR_MARKER = 'curl出错';

let body = '';
for (let a = 0; a < MAX_ATTEMPTS; a++) {
  try {
    const res = await this.helpers.httpRequest({ method: 'GET', url, qs, json: false });
    body = typeof res === 'string' ? res : JSON.stringify(res);
  } catch (e) {
    body = `[error] ${e?.message ?? String(e)}`;
  }
  if (!body.includes(ERROR_MARKER)) break;
  if (a < MAX_ATTEMPTS - 1) await sleep(1000);
}
```

**禁忌速查**（沙箱内全部 `undefined`）：`fetch`、`$helpers`、`$http`、`process`、`window`、`document`。
**兜底可用**：`require('https')`、`require('http')`、`require('crypto')`、`require('url')` 等内置模块。
