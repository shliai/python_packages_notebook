# Python logging 模块教程

`logging` 是 Python 标准库中用于记录日志的核心模块，相比 `print` 语句，它提供了分级记录、多输出目标、格式化控制、日志轮转和线程安全等企业级功能，是生产环境代码的必备组件。

---

## 目录
- [日志级别](#日志级别)
- [基本使用：basicConfig](#基本使用basicconfig)
- [核心组件：Logger、Handler、Formatter、Filter](#核心组件loggerhandlerformatterfilter)
- [配置方式：dictConfig](#配置方式dictconfig)
- [日志轮转](#日志轮转)
- [异常日志记录](#异常日志记录)
- [最佳实践](#最佳实践)
- [常见问题解决](#常见问题解决)

---

## 日志级别

logging 定义了5个标准日志级别，用于区分不同严重程度的信息：

| 级别         | 数值 | 适用场景                                   |
|--------------|------|--------------------------------------------|
| `DEBUG`      | 10   | 开发调试阶段的详细信息，如变量值、执行流程 |
| `INFO`       | 20   | 程序正常运行的确认信息                     |
| `WARNING`    | 30   | 不影响程序运行但需要注意的潜在问题         |
| `ERROR`      | 40   | 某个功能模块执行失败的错误信息             |
| `CRITICAL`   | 50   | 导致程序无法继续运行的严重错误             |

> **重要提示**：logging 的**默认级别是 WARNING**，这意味着默认情况下只会记录 WARNING 及以上级别的日志，DEBUG 和 INFO 级别的日志会被忽略。

---

## 基本使用：basicConfig

`basicConfig()` 是最简单的日志配置方式，适合小型脚本和快速原型开发。

### 输出到控制台

```python
import logging

# 全局日志配置
logging.basicConfig(
    level=logging.DEBUG,  # 设置最低记录级别
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',  # 日志格式
    datefmt='%Y-%m-%d %H:%M:%S'  # 时间格式
)

# 记录不同级别的日志
logging.debug('数据库连接成功')
logging.info('用户登录成功')
logging.warning('磁盘空间不足')
logging.error('文件读取失败')
logging.critical('系统崩溃')
```

### 输出到文件

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    filename='app.log',  # 日志文件路径
    filemode='a',  # 'a' 追加模式（默认），'w' 覆盖模式
    encoding='utf-8'  # 解决中文乱码问题
)

logging.info('程序启动')
logging.warning('配置文件使用默认值')
```

---

## 核心组件：Logger、Handler、Formatter、Filter

对于中大型项目，需要使用 logging 的四大核心组件进行灵活配置：

- **Logger**：记录器，应用程序直接调用的接口
- **Handler**：处理器，决定日志发送到何处（文件、控制台、网络等）
- **Formatter**：格式化器，定义日志的输出格式
- **Filter**：过滤器，提供更细粒度的日志过滤

### 完整组件配置示例

```python
import logging

# 1. 创建 Logger 实例（按模块命名，如 'myapp.database'）
logger = logging.getLogger('myapp')
logger.setLevel(logging.DEBUG)  # Logger 全局级别
logger.propagate = False  # 禁止日志向上传递到根 Logger

# 2. 创建 Formatter
detailed_formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(filename)s:%(lineno)d - %(funcName)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)

simple_formatter = logging.Formatter(
    '%(asctime)s - %(levelname)s - %(message)s'
)

# 3. 创建 Handler 并设置级别和格式
# 控制台处理器（只输出 INFO 及以上级别）
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)
console_handler.setFormatter(simple_formatter)

# 文件处理器（输出所有级别）
file_handler = logging.FileHandler('app.log', encoding='utf-8')
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(detailed_formatter)

# 4. 将 Handler 添加到 Logger
logger.addHandler(console_handler)
logger.addHandler(file_handler)

# 5. 使用 Logger 记录日志
logger.debug('这是调试信息，只写入文件')
logger.info('这是普通信息，控制台和文件都显示')
logger.error('这是错误信息')
```

### 常用格式符说明

| 格式符          | 说明                                   |
|-----------------|----------------------------------------|
| `%(asctime)s`   | 日志记录时间                           |
| `%(name)s`      | Logger 名称                            |
| `%(levelname)s` | 日志级别名称                           |
| `%(levelno)s`   | 日志级别数值                           |
| `%(message)s`   | 日志消息内容                           |
| `%(filename)s`  | 调用日志的文件名                       |
| `%(lineno)d`    | 调用日志的行号                         |
| `%(funcName)s`  | 调用日志的函数名                       |
| `%(module)s`    | 调用日志的模块名                       |
| `%(process)d`   | 进程ID                                 |
| `%(thread)d`    | 线程ID                                 |

---

## 配置方式：dictConfig

使用字典配置是最灵活、最强大的配置方式，适合复杂项目和生产环境。

```python
import logging
import logging.config

LOGGING_CONFIG = {
    'version': 1,
    'disable_existing_loggers': False,  # 保留已创建的 Logger
    
    'formatters': {
        'standard': {
            'format': '%(asctime)s - %(name)s - %(levelname)s - %(message)s',
            'datefmt': '%Y-%m-%d %H:%M:%S'
        },
        'detailed': {
            'format': '%(asctime)s - %(name)s - %(levelname)s - %(filename)s:%(lineno)d - %(funcName)s - %(message)s',
            'datefmt': '%Y-%m-%d %H:%M:%S'
        }
    },
    
    'filters': {},
    
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'level': 'INFO',
            'formatter': 'standard',
            'stream': 'ext://sys.stdout'
        },
        'file': {
            'class': 'logging.handlers.RotatingFileHandler',
            'level': 'DEBUG',
            'formatter': 'detailed',
            'filename': 'logs/app.log',
            'maxBytes': 10 * 1024 * 1024,  # 10MB
            'backupCount': 5,
            'encoding': 'utf-8'
        }
    },
    
    'loggers': {
        'myapp': {
            'handlers': ['console', 'file'],
            'level': 'DEBUG',
            'propagate': False
        },
        # 第三方库日志级别控制
        'requests': {
            'handlers': ['console'],
            'level': 'WARNING',
            'propagate': False
        }
    },
    
    'root': {
        'handlers': ['console'],
        'level': 'WARNING'
    }
}

# 加载配置
logging.config.dictConfig(LOGGING_CONFIG)

# 使用
logger = logging.getLogger('myapp')
logger.info('日志系统初始化完成')
```

---

## 日志轮转

生产环境必须使用日志轮转，避免单个日志文件无限增长。

### 按大小轮转（RotatingFileHandler）

```python
import logging
from logging.handlers import RotatingFileHandler

logger = logging.getLogger('myapp')
logger.setLevel(logging.DEBUG)

# 每个文件最大 10MB，保留 5 个备份
handler = RotatingFileHandler(
    'app.log',
    maxBytes=10 * 1024 * 1024,  # 10MB
    backupCount=5,
    encoding='utf-8'
)

formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
logger.addHandler(handler)
```

### 按时间轮转（TimedRotatingFileHandler）

```python
import logging
from logging.handlers import TimedRotatingFileHandler

logger = logging.getLogger('myapp')
logger.setLevel(logging.DEBUG)

# 每天轮转一次，保留 30 天的日志
handler = TimedRotatingFileHandler(
    'app.log',
    when='midnight',  # 每天午夜轮转
    interval=1,
    backupCount=30,
    encoding='utf-8'
)

# when 参数可选值：
# 'S' - 秒
# 'M' - 分钟
# 'H' - 小时
# 'D' - 天
# 'W0'-'W6' - 每周（W0=周一）
# 'midnight' - 每天午夜

formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)
logger.addHandler(handler)
```

---

## 异常日志记录

使用 `logger.exception()` 可以自动记录完整的异常堆栈信息，这是调试错误的最佳方式。

```python
import logging

logging.basicConfig(level=logging.ERROR)

try:
    result = 10 / 0
except ZeroDivisionError as e:
    # 自动包含异常类型、消息和堆栈跟踪
    logging.exception('计算过程中发生除零错误')
```

输出示例：
```
2024-05-20 10:30:00,123 - root - ERROR - 计算过程中发生除零错误
Traceback (most recent call last):
  File "example.py", line 6, in <module>
    result = 10 / 0
ZeroDivisionError: division by zero
```

---

## 最佳实践

1. **按模块创建 Logger**：使用 `logging.getLogger(__name__)` 为每个模块创建独立的 Logger
2. **避免使用根 Logger**：根 Logger 会接收所有未配置的日志，容易产生混乱
3. **合理设置日志级别**：开发环境用 DEBUG，测试环境用 INFO，生产环境用 WARNING/ERROR
4. **使用日志轮转**：生产环境必须配置日志轮转，防止磁盘占满
5. **记录足够的上下文**：在日志中包含用户ID、请求ID等信息，方便问题排查
6. **不要在日志中记录敏感信息**：如密码、密钥、身份证号等
7. **使用结构化日志**：对于大型系统，考虑使用 JSON 格式的日志，方便后续分析

---

## 常见问题解决

- **中文乱码**：在 FileHandler 中添加 `encoding='utf-8'` 参数
- **日志重复输出**：设置 `logger.propagate = False` 禁止向上传递
- **第三方库日志过多**：单独配置第三方库的日志级别，如 `logging.getLogger('requests').setLevel(logging.WARNING)`
- **多进程日志问题**：使用 `ConcurrentLogHandler` 或专门的日志收集系统（如 ELK）

---

## 总结

这个 logging 教程涵盖了以下主要内容：

- 基础概念：日志级别、基本配置方法
- 核心组件：Logger、Handler、Formatter、Filter 的使用
- 配置方式：dictConfig 字典配置
- 日志轮转：按大小和时间轮转的实现
- 异常记录：自动捕获并记录异常堆栈
- 最佳实践：生产环境的使用建议
- 常见问题：常见问题的解决方法

通过学习这些内容，您应该能够在项目中合理使用 logging 模块记录日志。建议继续学习 logging 的更多高级特性，如结构化日志、日志收集与分析等。