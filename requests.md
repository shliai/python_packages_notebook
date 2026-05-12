# Python Requests 完整教程
## 一、安装模块
```bash
pip install requests
```

## 二、快速入门
```python
import requests

# 发送GET请求
response = requests.get("https://httpbin.org/get")
# 打印响应内容
print(response.text)
```

## 三、七大 HTTP 请求方法
### 1. GET 请求
用于**获取数据**，使用 `params` 传递 URL 参数。
```python
import requests

url = "https://httpbin.org/get"
# 拼接url参数 ?name=张三&age=20
params = {
    "name": "张三",
    "age": 20
}
response = requests.get(url, params=params)

print("完整请求地址：", response.url)
print("响应内容：", response.text)
```

### 2. POST 请求
用于**提交数据**，支持表单、JSON 两种传参方式。

#### 表单传参 data
```python
url = "https://httpbin.org/post"
data = {
    "username": "admin",
    "password": "123456"
}
response = requests.post(url, data=data)
print(response.json())
```

#### JSON 传参 json
```python
url = "https://httpbin.org/post"
json_data = {
    "title": "Python教程",
    "price": 99
}
response = requests.post(url, json=json_data)
print(response.json())
```

### 3. 其他请求方式
```python
# PUT：全量更新数据
requests.put("https://httpbin.org/put", data={"k1":"v1"})

# PATCH：局部更新数据
requests.patch("https://httpbin.org/patch", json={"k1":100})

# DELETE：删除资源
requests.delete("https://httpbin.org/delete")

# HEAD：只获取响应头，不获取响应体
requests.head("https://httpbin.org/get")

# OPTIONS：查询服务器支持的请求方法
requests.options("https://httpbin.org/get")
```

### 4. 通用请求方法
`requests.request()` 可以适配任意请求方式
```python
import requests

# GET
res1 = requests.request("GET", "https://httpbin.org/get")
# POST
res2 = requests.request("POST", "https://httpbin.org/post", json={"a": 1})
```

## 四、Response 响应对象详解
```python
response = requests.get("https://httpbin.org/get")
```

| 属性/方法 | 作用说明 |
| ---- | ---- |
| `response.status_code` | HTTP 状态码 200/404/500 |
| `response.reason` | 状态文字描述 OK / Not Found |
| `response.text` | 字符串格式响应内容 |
| `response.content` | 二进制字节内容（图片、文件专用） |
| `response.json()` | 解析 JSON 数据，返回字典/列表 |
| `response.headers` | 响应头信息 |
| `response.cookies` | 服务器返回的 Cookie |
| `response.url` | 最终请求的完整 URL |
| `response.encoding` | 当前编码格式 |
| `response.apparent_encoding` | 自动检测真实编码 |
| `response.ok` | 状态码小于400返回True，否则False |
| `response.raise_for_status()` | 状态码异常自动抛出异常 |

示例：
```python
import requests

res = requests.get("https://httpbin.org/get")
print("状态码：", res.status_code)
print("状态描述：", res.reason)
print("编码：", res.apparent_encoding)
print("JSON解析：", res.json())
```

## 五、常用请求参数
### 1. 请求头 headers
伪装浏览器，绕过基础反爬
```python
import requests

url = "https://httpbin.org/headers"
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
    "Referer": "https://www.baidu.com"
}
res = requests.get(url, headers=headers)
print(res.json())
```

### 2. 携带 Cookies
访问需要登录态的接口
```python
import requests

url = "https://httpbin.org/cookies"
cookies = {
    "token": "abc123456",
    "user": "admin"
}
res = requests.get(url, cookies=cookies)
print(res.json())
```

### 3. 超时设置 timeout
防止请求卡死，单位秒
```python
# 整体超时5秒
res = requests.get("https://httpbin.org/delay/3", timeout=5)

# 连接超时3秒，读取超时7秒
res = requests.get("https://httpbin.org/delay/5", timeout=(3,7))
```

### 4. 代理 proxies
爬虫切换 IP 使用
```python
proxies = {
    "http": "http://127.0.0.1:7890",
    "https": "http://127.0.0.1:7890"
}
res = requests.get("https://httpbin.org/ip", proxies=proxies)
print(res.json())
```

### 5. 关闭 SSL 证书验证
访问自签名 HTTPS 网站
```python
res = requests.get("https://xxx.com", verify=False)
```

## 六、Session 会话保持
自动**保存、携带 Cookie**，适合模拟登录后连续请求
```python
import requests

# 创建会话对象
session = requests.Session()

# 登录（会话自动记录cookie）
session.post("https://httpbin.org/cookies/set/user/admin")

# 后续请求自动携带cookie
res = session.get("https://httpbin.org/cookies")
print(res.json())

# 关闭会话
session.close()
```

## 七、文件上传与下载
### 1. 文件上传
```python
import requests

url = "https://httpbin.org/post"
files = {
    "file": ("test.txt", open("test.txt", "rb"), "text/plain")
}
res = requests.post(url, files=files)
print(res.json())
```

### 2. 文件下载
流式下载，避免大文件占用内存
```python
import requests

url = "https://httpbin.org/image/png"
res = requests.get(url, stream=True)

with open("test.png", "wb") as f:
    # 分块写入
    for chunk in res.iter_content(chunk_size=1024):
        if chunk:
            f.write(chunk)
print("下载完成")
```

## 八、异常处理
捕获网络超时、连接失败、请求异常
```python
import requests
from requests.exceptions import Timeout, ConnectionError, RequestException

try:
    res = requests.get("https://httpbin.org/get", timeout=3)
    # 状态码非2xx直接抛异常
    res.raise_for_status()
except Timeout:
    print("请求超时")
except ConnectionError:
    print("网络连接失败")
except RequestException as e:
    print("请求异常：", e)
else:
    print("请求成功")
```

## 九、最佳实践
1. 频繁请求同一域名，优先使用 `Session` 会话；
2. 所有请求必须加 `timeout`，防止程序卡死；
3. 爬虫必须配置 `User-Agent` 请求头伪装浏览器；
4. 图片、文件下载使用 `stream=True` 流式读取；
5. 接口请求优先用 `raise_for_status()` 校验状态码；
6. 传递 JSON 参数直接用 `json=`，不要手动拼接字典。
