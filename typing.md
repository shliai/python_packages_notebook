# Python typing 教程
`typing` 是 Python 3.5+ 引入的标准库，用于提供**类型提示（Type Hints）**，让动态语言拥有静态类型检查能力，大幅提升代码可读性、IDE 智能提示和可维护性。

目录
- 导入方式（Python 3.9+ 新写法）
- 基础类型注解
- 容器类型注解
- 函数类型注解
- 常用特殊类型
- 类与实例类型
- 类型别名
- TypedDict（结构化字典）
- 泛型基础
- 静态类型检查工具
- 完整示例：用户管理系统
- 总结

## 导入方式
Python 3.9+ 推荐直接使用**内置集合类型**进行泛型标注，无需从 `typing` 导入大写版本：
```python
# Python 3.9+ 推荐写法
from typing import Optional, Union, Callable, Literal, TypedDict, TypeVar, Generic

# Python 3.8 及以下旧写法（兼容）
# from typing import List, Dict, Tuple, Set, Optional
```

## 基础类型注解
直接使用 Python 内置类型标注变量、函数参数和返回值：
```python
# 变量类型注解
name: str = "张三"
age: int = 25
height: float = 1.75
is_student: bool = True

# 只声明类型，稍后赋值
email: str

# 函数参数和返回值注解
def greet(user: str) -> str:
    return f"你好, {user}!"

# 无返回值函数
def print_message(msg: str) -> None:
    print(msg)
```

## 容器类型注解
### 列表（list）
```python
# 所有元素都是整数
numbers: list[int] = [1, 2, 3, 4, 5]

# 元素可以是字符串或整数
mixed: list[str | int] = ["a", 1, "b", 2]
```

### 字典（dict）
```python
# 键是字符串，值是整数
scores: dict[str, int] = {"语文": 90, "数学": 95}

# 键是字符串，值可以是字符串或 None
user_info: dict[str, Optional[str]] = {"name": "张三", "phone": None}
```

### 元组（tuple）
```python
# 固定长度，每个元素类型明确
point: tuple[int, int] = (10, 20)
person: tuple[str, int, bool] = ("张三", 25, True)

# 可变长度，所有元素类型相同
numbers: tuple[int, ...] = (1, 2, 3, 4, 5)
```

### 集合（set）
```python
unique_ids: set[int] = {1, 2, 3, 4, 5}
```

## 函数类型注解
### 普通函数
```python
def add(a: int, b: int) -> int:
    return a + b

# 可选参数
def say_hello(name: str, greeting: str = "你好") -> str:
    return f"{greeting}, {name}!"
```

### Callable（函数作为参数/返回值）
```python
from typing import Callable

# 接收两个 int，返回 int 的函数
def apply_func(func: Callable[[int, int], int], x: int, y: int) -> int:
    return func(x, y)

# 使用示例
result = apply_func(add, 3, 4)  # 结果：7
```

## 常用特殊类型
### Optional（可以是 X 或 None）
```python
from typing import Optional

# username 可以是字符串或 None
def get_user(username: Optional[str]) -> Optional[dict]:
    if username is None:
        return None
    return {"name": username}
```

### Union（多种类型）
Python 3.10+ 可以用 `|` 代替 `Union`：
```python
from typing import Union

# 旧写法
def process_data(data: Union[str, int, float]) -> str:
    return str(data)

# Python 3.10+ 新写法
def process_data(data: str | int | float) -> str:
    return str(data)
```

### Literal（限定字面量）
限制变量只能是指定的几个值：
```python
from typing import Literal

# 日志级别只能是这四个值
LogLevel = Literal["DEBUG", "INFO", "WARNING", "ERROR"]

def set_log_level(level: LogLevel) -> None:
    print(f"设置日志级别为: {level}")

set_log_level("INFO")  # 正确
set_log_level("FATAL") # 类型检查报错
```

### Any（任意类型）
关闭类型检查，慎用！仅用于无法确定类型的场景：
```python
from typing import Any

def print_anything(value: Any) -> None:
    print(value)
```

### NoReturn（永不返回）
表示函数永远不会正常返回（如抛出异常或死循环）：
```python
from typing import NoReturn

def raise_error(msg: str) -> NoReturn:
    raise ValueError(msg)
```

## 类与实例类型
### 类作为类型
```python
class User:
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age

# 标注参数为 User 实例
def print_user(user: User) -> None:
    print(f"姓名: {user.name}, 年龄: {user.age}")
```

### Self 类型（Python 3.11+）
用于标注类方法返回自身实例：
```python
from typing import Self

class User:
    def set_name(self, name: str) -> Self:
        self.name = name
        return self  # 链式调用

user = User().set_name("张三")
```

## 类型别名
简化复杂的类型定义，提高代码可读性：
```python
# Python 3.10+ 推荐写法
from typing import TypeAlias

UserDict: TypeAlias = dict[str, str | int | bool]

# 旧写法（兼容所有版本）
UserDict = dict[str, str | int | bool]

# 使用
def create_user(data: UserDict) -> UserDict:
    return data
```

## TypedDict（结构化字典）
精确标注字典的结构，指定每个键的类型，IDE 会自动提示字段：
```python
from typing import TypedDict

class User(TypedDict):
    name: str       # 必填字段
    age: int        # 必填字段
    email: Optional[str]  # 可选字段

# 使用
user: User = {
    "name": "张三",
    "age": 25,
    "email": "zhangsan@example.com"
}
```

Python 3.11+ 支持显式标记必填/非必填字段：
```python
from typing import TypedDict, Required, NotRequired

class User(TypedDict):
    name: Required[str]    # 必填
    age: Required[int]     # 必填
    email: NotRequired[str] # 非必填
```

## 泛型基础
让函数/类支持多种类型，同时保持类型安全：
```python
from typing import TypeVar, Generic

# 定义类型变量
T = TypeVar("T")

# 泛型函数
def get_first_item(items: list[T]) -> Optional[T]:
    if items:
        return items[0]
    return None

# 使用
first_int = get_first_item([1, 2, 3])  # 类型：Optional[int]
first_str = get_first_item(["a", "b"]) # 类型：Optional[str]
```

## 静态类型检查工具
类型提示本身不会在运行时报错，需要配合静态检查工具使用：
```bash
# 安装 mypy（最常用）
pip install mypy

# 检查文件
mypy your_script.py

# 安装 pyright（VS Code 默认使用）
pip install pyright

# 检查文件
pyright your_script.py
```

## 完整示例：用户管理系统
```python
from typing import TypedDict, Optional, Literal

# 定义用户结构
class User(TypedDict):
    id: int
    name: str
    age: int
    role: Literal["admin", "user", "guest"]
    email: Optional[str]

# 模拟数据库
users: list[User] = []
next_id: int = 1

def add_user(name: str, age: int, role: Literal["admin", "user", "guest"], email: Optional[str] = None) -> User:
    """添加用户"""
    global next_id
    user: User = {
        "id": next_id,
        "name": name,
        "age": age,
        "role": role,
        "email": email
    }
    users.append(user)
    next_id += 1
    return user

def get_user_by_id(user_id: int) -> Optional[User]:
    """根据ID查询用户"""
    for user in users:
        if user["id"] == user_id:
            return user
    return None

def list_users_by_role(role: Literal["admin", "user", "guest"]) -> list[User]:
    """根据角色列出用户"""
    return [user for user in users if user["role"] == role]

# 使用示例
add_user("张三", 25, "admin", "zhangsan@example.com")
add_user("李四", 22, "user")

user = get_user_by_id(1)
if user:
    print(f"找到用户: {user['name']}")

admin_users = list_users_by_role("admin")
print(f"管理员数量: {len(admin_users)}")
```

## 总结
这个 `typing` 教程涵盖了以下主要内容：
- 基础用法：变量、函数、容器的类型注解
- 常用特殊类型：`Optional`、`Union`、`Literal`、`Any`
- 高级特性：`TypedDict`、类型别名、泛型
- 工具使用：`mypy`/`pyright` 静态类型检查
- 完整示例：结合实际场景的用户管理系统

通过学习这些内容，您可以写出更清晰、更健壮的 Python 代码。建议在项目中逐步引入类型提示，先从核心模块和公共接口开始。