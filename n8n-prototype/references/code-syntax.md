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
