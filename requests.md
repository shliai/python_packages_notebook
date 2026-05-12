# Python requests 模块 完整教程
`requests` 是 Python 最流行、最简单的 HTTP 请求库，用于发送网络请求、爬取网页、调用接口、上传下载文件等。

## 目录
1. 安装与导入
2. 发送请求（GET / POST / PUT / DELETE 等）
3. 响应对象属性与方法
4. 请求参数（headers / params / data / json / cookies）
5. 文件上传与下载
6. 会话保持（Session）
7. 超时、代理、SSL 验证
8. 异常处理
9. 实战案例合集

---

# 1. 安装与导入
```bash
pip install requests
```

```python
import requests
```

---

# 2. 发送请求
## GET 请求（最常用）
```python
import requests

response = requests.get('https://www.baidu.com')
print(response.text)
```

带参数 GET：
```python
params = {'wd': 'python'}
response = requests.get('https://www.baidu.com/s', params=params)
```

## POST 请求
表单格式：
```python
data = {'username': 'admin', 'password': '123456'}
response = requests.post('https://httpbin.org/post', data=data)
```

JSON 格式：
```python
json_data = {'name': 'test', 'age': 18}
response = requests.post('https://httpbin.org/post', json=json_data)
```

## 其他 HTTP 方法
```python
requests.put('https://httpbin.org/put', data={'key':'a'})
requests.delete('https://httpbin.org/delete')
requests.head('https://httpbin.org/get')
requests.patch('https://httpbin.org/patch', data={'key':'b'})
requests.options('https://httpbin.org/get')
```

通用写法：
```python
response = requests.request('GET', 'https://www.baidu.com')
```

---

# 3. 响应对象属性与方法（完整版）
```python
response = requests.get('https://httpbin.org/get')
```

| 属性/方法 | 说明 |
|---------|------|
| `response.status_code` | 状态码（200=成功） |
| `response.reason` | 状态描述（OK / Not Found） |
| `response.text` | 字符串内容 |
| `response.content` | 字节内容（图片/文件） |
| `response.json()` | 解析 JSON（返回字典） |
| `response.headers` | 响应头 |
| `response.cookies` | 响应 Cookie |
| `response.url` | 最终请求 URL |
| `response.encoding` | 编码 |
| `response.apparent_encoding` | 自动识别编码 |
| `response.ok` | 状态码 <400 返回 True |
| `response.raise_for_status()` | 状态码错误抛出异常 |
| `response.elapsed` | 请求耗时 |
| `response.history` | 重定向历史 |
| `response.is_redirect` | 是否重定向 |

示例：
```python
print(response.status_code)
print(response.reason)
print(response.url)
print(response.json())
```

---

# 4. 请求参数（完整版）
## 4.1 请求头 headers
```python
headers = {
    'User-Agent': 'Mozilla/5.0',
    'Referer': 'https://www.baidu.com',
    'Content-Type': 'application/json'
}

response = requests.get('https://httpbin.org/get', headers=headers)
```

## 4.2 URL 参数 params
```python
params = {'page': 1, 'size': 10}
response = requests.get('https://httpbin.org/get', params=params)
```

## 4.3 表单数据 data
```python
data = {'user': 'test', 'pwd': '123'}
response = requests.post(url, data=data)
```

## 4.4 JSON 数据 json
```python
json_data = {'name': 'test'}
response = requests.post(url, json=json_data)
```

## 4.5 携带 Cookies
```python
cookies = {'token': '123456'}
response = requests.get(url, cookies=cookies)
```

---

# 5. 文件上传与下载
## 5.1 文件上传
```python
files = {
    'file': ('test.jpg', open('test.jpg', 'rb'), 'image/jpeg')
}
response = requests.post('https://httpbin.org/post', files=files)
```

## 5.2 文件下载
```python
response = requests.get('https://www.baidu.com/logo.jpg', stream=True)
with open('logo.jpg', 'wb') as f:
    for chunk in response.iter_content(chunk_size=1024):
        f.write(chunk)
```

---

# 6. 会话保持 Session
```python
session = requests.Session()

# 登录
session.post('https://xxx.com/login', data={'user':'admin','pwd':'123'})

# 自动携带 Cookie
response = session.get('https://xxx.com/user')
```

---

# 7. 超时、代理、SSL
## 超时
```python
response = requests.get(url, timeout=5)
```

## 代理
```python
proxies = {
    'http': 'http://127.0.0.1:7890',
    'https': 'http://127.0.0.1:7890'
}
response = requests.get(url, proxies=proxies)
```

## 关闭 SSL 验证
```python
response = requests.get(url, verify=False)
```

---

# 8. 异常处理
```python
from requests.exceptions import Timeout, ConnectionError, RequestException

try:
    response = requests.get(url, timeout=3)
    response.raise_for_status()
except Timeout:
    print("超时")
except ConnectionError:
    print("网络错误")
except RequestException as e:
    print("请求失败", e)
```

---

# 9. 实战案例合集
## 案例1：爬取网页
```python
import requests
headers = {'User-Agent': 'Mozilla/5.0'}
res = requests.get('https://www.baidu.com', headers=headers)
print(res.text)
```

## 案例2：调用 JSON API
```python
res = requests.get('https://httpbin.org/get')
data = res.json()
print(data)
```

## 案例3：模拟登录
```python
session = requests.Session()
login_data = {'username': 'admin', 'password': '123456'}
session.post('https://xxx.com/login', data=login_data)
res = session.get('https://xxx.com/profile')
```

## 案例4：上传文件
```python
files = {'file': open('a.txt','rb')}
res = requests.post('https://httpbin.org/post', files=files)
```

## 案例5：下载图片
```python
res = requests.get('https://xxx.com/a.jpg', stream=True)
with open('a.jpg','wb') as f:
    f.write(res.content)
```

---

这份教程包含：
- 安装导入
- GET/POST/PUT/DELETE 所有请求
- 响应对象全部属性
- headers / params / data / json / cookies
- 文件上传下载
- Session 会话
- 超时、代理、SSL
- 异常处理
- 实战案例