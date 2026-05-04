# Python Click 教程
Click 是 Python 最流行的**命令行工具开发库**，以"最少代码、最优雅方式"创建专业级 CLI 程序。相比原生 `argparse`，它支持自动生成帮助、任意嵌套子命令、类型自动转换、跨平台兼容等特性，是 Flask、Django 等框架官方推荐的 CLI 工具。

目录
- 安装 Click
- 第一个 Click 应用
- 位置参数（@click.argument）
- 选项参数（@click.option）
- 子命令（@click.group）
- 内置参数类型
- 回调与验证
- 帮助与版本信息
- 高级特性
- 完整示例：文件管理工具
- 总结

## 安装 Click
```bash
pip install click
```

## 第一个 Click 应用
```python
# hello.py
import click

@click.command()
def hello():
    """简单的问候程序"""
    click.echo("Hello, Click!")

if __name__ == '__main__':
    hello()
```

运行：
```bash
python hello.py
# 输出：Hello, Click!

# 自动生成帮助
python hello.py --help
```

**关键说明**：
- `@click.command()`：将函数转换为命令行命令
- `click.echo()`：替代 `print()`，自动处理跨平台编码和颜色输出
- 函数文档字符串会自动成为命令的帮助信息

## 位置参数（@click.argument）
用于定义命令后面必须跟随的参数，默认是必选的。

### 基础用法
```python
@click.command()
@click.argument('name')
def greet(name):
    """问候指定的人"""
    click.echo(f"你好, {name}!")
```

运行：
```bash
python greet.py 张三
# 输出：你好, 张三!
```

### 可选参数
```python
@click.command()
@click.argument('name', required=False, default='世界')
def greet(name):
    click.echo(f"你好, {name}!")
```

### 多个参数
```python
@click.command()
@click.argument('a', type=int)
@click.argument('b', type=int)
def add(a, b):
    """计算两个数的和"""
    click.echo(f"{a} + {b} = {a + b}")
```

### 可变长度参数
```python
@click.command()
@click.argument('files', nargs=-1)
def process_files(files):
    """处理多个文件"""
    for file in files:
        click.echo(f"处理文件: {file}")
```

运行：
```bash
python process.py file1.txt file2.txt file3.txt
```

## 选项参数（@click.option）
用于定义 `--option` 形式的可选参数，是 Click 最强大的功能之一。

### 基础用法
```python
@click.command()
@click.option('--count', default=1, help='问候次数')
@click.option('--name', default='世界', help='问候对象')
def greet(count, name):
    for _ in range(count):
        click.echo(f"你好, {name}!")
```

运行：
```bash
python greet.py --count=3 --name=张三
```

### 短选项
```python
@click.command()
@click.option('--name', '-n', default='世界', help='问候对象')
def greet(name):
    click.echo(f"你好, {name}!")
```

运行：
```bash
python greet.py -n 张三
```

### 必选选项
```python
@click.command()
@click.option('--name', required=True, help='必须指定姓名')
def greet(name):
    click.echo(f"你好, {name}!")
```

### 布尔选项
```python
@click.command()
@click.option('--shout', is_flag=True, help='是否大声问候')
def greet(shout):
    message = "你好, 世界!"
    if shout:
        message = message.upper()
    click.echo(message)
```

运行：
```bash
python greet.py --shout
# 输出：你好, 世界!
```

### 多值选项
```python
@click.command()
@click.option('--color', type=(str, int), help='颜色和透明度')
def set_color(color):
    name, alpha = color
    click.echo(f"设置颜色: {name}, 透明度: {alpha}")
```

运行：
```bash
python set_color.py --color red 255
```

### 计数选项
```python
@click.command()
@click.option('-v', '--verbose', count=True, help='详细程度')
def log(verbose):
    if verbose == 0:
        click.echo("普通信息")
    elif verbose == 1:
        click.echo("详细信息")
    else:
        click.echo("调试信息")
```

运行：
```bash
python log.py -vv
# 输出：调试信息
```

### 提示输入
```python
@click.command()
@click.option('--name', prompt='请输入你的姓名', help='你的姓名')
@click.option('--password', prompt=True, hide_input=True, confirmation_prompt=True)
def login(name, password):
    click.echo(f"欢迎, {name}!")
```

运行时会自动提示输入密码，且输入内容不可见。

### 环境变量支持
```python
@click.command()
@click.option('--api-key', envvar='API_KEY', help='API密钥')
def connect(api_key):
    click.echo(f"使用API密钥: {api_key}")
```

## 子命令（@click.group）
Click 最强大的特性，支持创建任意嵌套的子命令结构，适合复杂的 CLI 工具。

### 基础子命令
```python
# cli.py
import click

@click.group()
def cli():
    """一个简单的命令行工具"""
    pass

@cli.command()
@click.argument('name')
def greet(name):
    """问候用户"""
    click.echo(f"你好, {name}!")

@cli.command()
@click.argument('a', type=int)
@click.argument('b', type=int)
def add(a, b):
    """计算两个数的和"""
    click.echo(f"{a} + {b} = {a + b}")

if __name__ == '__main__':
    cli()
```

运行：
```bash
python cli.py greet 张三
python cli.py add 3 5

# 查看所有子命令
python cli.py --help
```

### 嵌套子命令
```python
@click.group()
def cli():
    pass

@click.group()
def database():
    """数据库操作"""
    pass

@database.command()
def init():
    """初始化数据库"""
    click.echo("数据库初始化完成")

@database.command()
def drop():
    """删除数据库"""
    click.echo("数据库已删除")

cli.add_command(database)
```

运行：
```bash
python cli.py database init
python cli.py database drop
```

## 内置参数类型
Click 提供了丰富的内置类型，自动进行类型转换和验证。

```python
# 数值类型
@click.option('--age', type=int)
@click.option('--price', type=float)
@click.option('--score', type=click.IntRange(0, 100))  # 0-100的整数
@click.option('--discount', type=click.FloatRange(0, 1))  # 0-1的浮点数

# 字符串类型
@click.option('--name', type=str)
@click.option('--choice', type=click.Choice(['red', 'green', 'blue']))  # 限定选项

# 文件和路径类型
@click.option('--input', type=click.File('r', encoding='utf-8'))  # 只读文件
@click.option('--output', type=click.File('w', encoding='utf-8'))  # 只写文件
@click.option('--path', type=click.Path(exists=True))  # 必须存在的路径
```

## 回调与验证
使用回调函数对参数进行自定义验证和处理。

```python
def validate_age(ctx, param, value):
    if value < 0 or value > 150:
        raise click.BadParameter("年龄必须在0-150之间")
    return value

@click.command()
@click.option('--age', type=int, callback=validate_age)
def set_age(age):
    click.echo(f"年龄设置为: {age}")
```

## 帮助与版本信息
### 自动生成帮助
Click 会自动根据函数文档字符串和参数说明生成帮助信息。

### 版本选项
```python
@click.command()
@click.version_option(version='1.0.0', prog_name='mycli')
def main():
    pass
```

运行：
```bash
python main.py --version
# 输出：mycli, version 1.0.0
```

## 高级特性
### 上下文对象
在子命令之间传递数据：
```python
@click.group()
@click.option('--debug', is_flag=True)
@click.pass_context
def cli(ctx, debug):
    ctx.obj = {'debug': debug}

@cli.command()
@click.pass_context
def hello(ctx):
    if ctx.obj['debug']:
        click.echo("调试模式已开启")
    click.echo("Hello!")
```

### 进度条
```python
import time

@click.command()
def process():
    with click.progressbar(range(100), label='处理中') as bar:
        for i in bar:
            time.sleep(0.01)
```

### 颜色输出
```python
@click.command()
def status():
    click.secho("成功", fg='green')
    click.secho("警告", fg='yellow')
    click.secho("错误", fg='red', bold=True)
```

### 异常处理
```python
@click.command()
def dangerous():
    try:
        # 危险操作
        1 / 0
    except Exception as e:
        click.echo(f"错误: {e}", err=True)
        raise click.Abort()
```

## 完整示例：文件管理工具
```python
# file_manager.py
import os
import click

@click.group()
@click.pass_context
def cli(ctx):
    """简单的文件管理工具"""
    ctx.obj = {}

@cli.command()
@click.argument('filename')
@click.option('--content', default='', help='文件内容')
def create(filename, content):
    """创建新文件"""
    with open(filename, 'w', encoding='utf-8') as f:
        f.write(content)
    click.echo(f"文件 {filename} 创建成功")

@cli.command()
@click.argument('filename')
def delete(filename):
    """删除文件"""
    if os.path.exists(filename):
        os.remove(filename)
        click.echo(f"文件 {filename} 已删除")
    else:
        click.echo(f"文件 {filename} 不存在", err=True)

@cli.command()
@click.argument('path', default='.')
def list(path):
    """列出目录内容"""
    for item in os.listdir(path):
        click.echo(item)

if __name__ == '__main__':
    cli()
```

运行：
```bash
python file_manager.py create test.txt --content="Hello, World!"
python file_manager.py list
python file_manager.py delete test.txt
```

## 总结
这个 Click 教程涵盖了以下主要内容：
- 基础用法：命令定义、位置参数、选项参数
- 核心特性：子命令嵌套、自动生成帮助、类型验证
- 高级功能：上下文传递、进度条、颜色输出、异常处理
- 完整示例：实用的文件管理工具

通过学习这些内容，你可以快速开发出专业级的 Python 命令行工具。Click 还有更多高级特性（如插件系统、命令别名、配置文件支持等），可以在实际项目中逐步探索。