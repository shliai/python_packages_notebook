# JSON 教程
Python 内置的 `json` 模块用于处理 JSON（JavaScript Object Notation）数据，是数据交换和存储的常用工具。

目录
导入json模块
基本概念
序列化（Python → JSON）
反序列化（JSON → Python）
常用参数
完整示例：配置文件管理
总结

## 导入json模块
`json` 是 Python 标准库，无需安装，直接导入：
```python
import json
```

## 基本概念
- **序列化**：将 Python 对象转换为 JSON 字符串/文件
- **反序列化**：将 JSON 字符串/文件转换为 Python 对象

## 序列化（Python → JSON）
### json.dumps()：转为 JSON 字符串
```python
import json

data = {
    "name": "张三",
    "age": 25,
    "is_student": False,
    "hobbies": ["读书", "编程"]
}

# 转为 JSON 字符串
json_str = json.dumps(data)
print(json_str)
```

### json.dump()：写入 JSON 文件
```python
import json

data = {
    "name": "张三",
    "age": 25
}

# 写入文件
with open("data.json", "w", encoding="utf-8") as f:
    json.dump(data, f)
```

## 反序列化（JSON → Python）
### json.loads()：解析 JSON 字符串
```python
import json

json_str = '{"name": "张三", "age": 25}'

# 转为 Python 字典
data = json.loads(json_str)
print(data["name"])  # 输出：张三
```

### json.load()：读取 JSON 文件
```python
import json

# 读取文件
with open("data.json", "r", encoding="utf-8") as f:
    data = json.load(f)
print(data["age"])  # 输出：25
```

## 常用参数
### ensure_ascii：处理中文
```python
import json

data = {"name": "张三"}
# 不转义中文
json_str = json.dumps(data, ensure_ascii=False)
print(json_str)  # 输出：{"name": "张三"}
```

### indent：格式化输出
```python
import json

data = {"name": "张三", "age": 25}
# 缩进 4 空格，美化输出
json_str = json.dumps(data, indent=4, ensure_ascii=False)
print(json_str)
```

### sort_keys：按键排序
```python
import json

data = {"age": 25, "name": "张三"}
# 按 key 字母顺序排序
json_str = json.dumps(data, sort_keys=True)
print(json_str)
```

## 完整示例：配置文件管理
```python
import json

# 配置文件路径
CONFIG_FILE = "config.json"

def load_config():
    """读取配置"""
    try:
        with open(CONFIG_FILE, "r", encoding="utf-8") as f:
            return json.load(f)
    except FileNotFoundError:
        # 默认配置
        return {"theme": "light", "font_size": 14}

def save_config(config):
    """保存配置"""
    with open(CONFIG_FILE, "w", encoding="utf-8") as f:
        json.dump(config, f, indent=4, ensure_ascii=False)

# 使用示例
config = load_config()
print("当前配置:", config)

# 修改配置
config["font_size"] = 16
save_config(config)
print("配置已更新")
```

## 总结
这个 `json` 教程涵盖了以下主要内容：
- 模块导入：无需安装，直接使用
- 序列化：`dumps()` 转字符串、`dump()` 写文件
- 反序列化：`loads()` 解析字符串、`load()` 读文件
- 常用参数：`ensure_ascii` 处理中文、`indent` 格式化、`sort_keys` 排序
- 完整示例：配置文件的读写管理

通过学习这些内容，您应该能够使用 `json` 模块处理常见的 JSON 数据需求。建议继续学习 JSON Schema 验证、复杂嵌套数据处理等高级特性。