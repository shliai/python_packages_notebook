# SQLAlchemy 完整教程
SQLAlchemy 是 Python 生态中最强大的数据库工具包，提供 **Core（核心层，原生 SQL 抽象）** 与 **ORM（对象关系映射层，类与表映射）** 两种模式，兼顾灵活性与开发效率，支持 SQLite、MySQL、PostgreSQL 等主流数据库。

---

## 目录
- [安装 SQLAlchemy](#安装-sqlalchemy)
- [核心概念（Engine/Base/Session）](#核心概念enginebasesession)
- [ORM 模型定义（表映射）](#orm-模型定义表映射)
- [基础 CRUD 操作](#基础-crud-操作)
- [高级查询（过滤/排序/分页/聚合）](#高级查询过滤排序分页聚合)
- [表关系映射（一对多/多对多）](#表关系映射一对多多对多)
- [事务管理](#事务管理)
- [Flask-SQLAlchemy 集成](#flask-sqlalchemy-集成)
- [数据库迁移（Alembic）](#数据库迁移alembic)
- [最佳实践与常见问题](#最佳实践与常见问题)

---

## 安装 SQLAlchemy
```bash
# 基础安装（默认支持 SQLite）
pip install sqlalchemy

# 安装 MySQL 依赖
pip install sqlalchemy pymysql

# 安装 PostgreSQL 依赖
pip install sqlalchemy psycopg2-binary
```

---

## 核心概念（Engine/Base/Session）
### 1. Engine（引擎）
数据库连接核心，管理连接池与 SQL 执行，全局唯一。
```python
from sqlalchemy import create_engine

# SQLite（文件型，无需服务）
engine = create_engine("sqlite:///test.db", echo=True)  # echo=True 打印SQL日志

# MySQL（格式：mysql+pymysql://用户:密码@主机:端口/库名）
# engine = create_engine("mysql+pymysql://root:123456@localhost:3306/test_db")

# PostgreSQL
# engine = create_engine("postgresql://user:pass@localhost:5432/test_db")
```

### 2. Base（模型基类）
所有 ORM 模型的父类，用于关联表结构与 Python 类。
```python
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()
```

### 3. Session（会话）
数据库操作的接口，管理事务、缓存与查询，每次操作需实例化。
```python
from sqlalchemy.orm import sessionmaker

Session = sessionmaker(bind=engine)
session = Session()  # 每次操作创建一个会话
```

---

## ORM 模型定义（表映射）
将 Python 类映射为数据库表，类属性映射为表字段。

### 完整模型示例
```python
from sqlalchemy import Column, Integer, String, DateTime, Text, ForeignKey
from sqlalchemy.orm import relationship
from datetime import datetime

# 用户表
class User(Base):
    __tablename__ = "users"  # 表名

    id = Column(Integer, primary_key=True, autoincrement=True)  # 主键
    username = Column(String(50), unique=True, nullable=False)  # 唯一、非空
    email = Column(String(100), unique=True)
    password = Column(String(255), nullable=False)
    created_at = Column(DateTime, default=datetime.now)  # 默认当前时间
    bio = Column(Text)  # 长文本

    # 关系：一个用户对应多篇文章（一对多）
    posts = relationship("Post", back_populates="author")

# 文章表
class Post(Base):
    __tablename__ = "posts"

    id = Column(Integer, primary_key=True)
    title = Column(String(200), nullable=False)
    content = Column(Text, nullable=False)
    created_at = Column(DateTime, default=datetime.now)
    
    # 外键：关联用户表
    author_id = Column(Integer, ForeignKey("users.id"))
    
    # 关系：一篇文章对应一个作者
    author = relationship("User", back_populates="posts")

# 创建所有表（仅需执行一次）
Base.metadata.create_all(engine)
```

### 常用字段类型
| 类型          | 说明                     |
|---------------|--------------------------|
| `Integer`     | 整数                     |
| `String(n)`   | 字符串，n 为最大长度     |
| `Text`        | 长文本                   |
| `DateTime`    | 日期时间                 |
| `Float`       | 浮点数                   |
| `Boolean`     | 布尔值                   |
| `ForeignKey`  | 外键                     |

---

## 基础 CRUD 操作
### 1. 增（Create）
```python
# 单条添加
user1 = User(username="zhangsan", email="zhangsan@example.com", password="123456")
session.add(user1)
session.commit()  # 提交事务

# 批量添加
users = [
    User(username="lisi", email="lisi@example.com", password="123456"),
    User(username="wangwu", email="wangwu@example.com", password="123456")
]
session.add_all(users)
session.commit()
```

### 2. 查（Read）
```python
# 按主键查询
user = session.get(User, 1)  # 查询 id=1 的用户

# 查询所有
users = session.query(User).all()

# 条件查询（filter）
user = session.query(User).filter(User.username == "zhangsan").first()

# 多条件查询
user = session.query(User).filter(
    User.username == "zhangsan",
    User.email == "zhangsan@example.com"
).first()
```

### 3. 改（Update）
```python
# 先查询再修改
user = session.get(User, 1)
if user:
    user.email = "new_email@example.com"
    session.commit()

# 批量更新
session.query(User).filter(User.username.like("zhang%")).update(
    {User.password: "new_password"}
)
session.commit()
```

### 4. 删（Delete）
```python
# 先查询再删除
user = session.get(User, 1)
if user:
    session.delete(user)
    session.commit()

# 批量删除
session.query(User).filter(User.username.like("test%")).delete()
session.commit()
```

---

## 高级查询（过滤/排序/分页/聚合）
### 1. 复杂过滤
```python
from sqlalchemy import or_, and_, not_

# OR 条件
users = session.query(User).filter(
    or_(User.username == "zhangsan", User.username == "lisi")
).all()

# LIKE 模糊查询
users = session.query(User).filter(User.username.like("zhang%")).all()

# IN 范围查询
users = session.query(User).filter(User.id.in_([1, 2, 3])).all()

# IS NULL / IS NOT NULL
users = session.query(User).filter(User.bio.is_(None)).all()
```

### 2. 排序与分页
```python
# 排序（order_by）
users = session.query(User).order_by(User.created_at.desc()).all()  # 降序
users = session.query(User).order_by(User.username.asc()).all()  # 升序

# 分页（limit/offset）
# 第1页，每页10条
users = session.query(User).limit(10).offset(0).all()
# 第2页，每页10条
users = session.query(User).limit(10).offset(10).all()
```

### 3. 聚合查询
```python
from sqlalchemy import func

# 计数
count = session.query(func.count(User.id)).scalar()

# 求和、平均值、最大值、最小值
total = session.query(func.sum(Post.id)).scalar()
avg_id = session.query(func.avg(Post.id)).scalar()
max_id = session.query(func.max(Post.id)).scalar()

# 分组统计
# 统计每个用户的文章数
result = session.query(
    User.username,
    func.count(Post.id).label("post_count")
).join(Post).group_by(User.id).all()
```

---

## 表关系映射（一对多/多对多）
### 1. 一对多关系（User -> Post）
```python
# 模型定义见前文，使用 relationship 关联

# 添加关联数据
user = User(username="author1", email="author1@example.com", password="123456")
post1 = Post(title="文章1", content="内容1", author=user)
post2 = Post(title="文章2", content="内容2", author=user)

session.add(user)
session.add_all([post1, post2])
session.commit()

# 查询关联数据
user = session.get(User, 1)
print(user.posts)  # 输出该用户的所有文章

post = session.get(Post, 1)
print(post.author)  # 输出该文章的作者
```

### 2. 多对多关系（Post <-> Tag）
```python
# 中间表（关联表）
post_tag = Table(
    "post_tag",
    Base.metadata,
    Column("post_id", Integer, ForeignKey("posts.id"), primary_key=True),
    Column("tag_id", Integer, ForeignKey("tags.id"), primary_key=True)
)

# 标签表
class Tag(Base):
    __tablename__ = "tags"
    id = Column(Integer, primary_key=True)
    name = Column(String(50), unique=True)
    posts = relationship("Post", secondary=post_tag, back_populates="tags")

# 修改 Post 模型，添加关系
class Post(Base):
    # ... 其他字段 ...
    tags = relationship("Tag", secondary=post_tag, back_populates="posts")

# 创建表
Base.metadata.create_all(engine)

# 添加多对多数据
post = Post(title="多对多测试", content="内容")
tag1 = Tag(name="Python")
tag2 = Tag(name="SQLAlchemy")

post.tags.append(tag1)
post.tags.append(tag2)

session.add(post)
session.commit()

# 查询多对多数据
post = session.get(Post, 1)
print(post.tags)  # 输出文章的所有标签

tag = session.get(Tag, 1)
print(tag.posts)  # 输出标签下的所有文章
```

---

## 事务管理
```python
try:
    # 执行多个操作
    user1 = User(username="user1", password="123456")
    user2 = User(username="user2", password="123456")
    session.add_all([user1, user2])
    
    # 模拟错误
    # 1 / 0
    
    session.commit()  # 提交事务
except Exception as e:
    session.rollback()  # 回滚事务
    print(f"操作失败：{e}")
finally:
    session.close()  # 关闭会话
```

---

## Flask-SQLAlchemy 集成
在 Flask 中使用 SQLAlchemy 的简化版本。

### 安装与配置
```bash
pip install flask-sqlalchemy
```

```python
from flask import Flask
from flask_sqlalchemy import SQLAlchemy

app = Flask(__name__)
app.config["SQLALCHEMY_DATABASE_URI"] = "sqlite:///test.db"
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False
db = SQLAlchemy(app)

# 模型定义（继承 db.Model）
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(50), unique=True)

# 创建表
with app.app_context():
    db.create_all()

# 使用
@app.route("/")
def index():
    user = User(username="flask_user")
    db.session.add(user)
    db.session.commit()
    return "用户添加成功"

if __name__ == "__main__":
    app.run(debug=True)
```

---

## 数据库迁移（Alembic）
管理数据库 schema 变更（添加字段、修改表结构等）。

### 安装与初始化
```bash
pip install alembic
alembic init alembic  # 初始化迁移目录
```

### 配置 alembic.ini
```ini
# 修改数据库连接
sqlalchemy.url = sqlite:///test.db
```

### 配置 alembic/env.py
```python
# 导入 Base 模型
from your_module import Base
target_metadata = Base.metadata
```

### 生成与执行迁移
```bash
# 生成迁移脚本（自动检测模型变更）
alembic revision --autogenerate -m "create users table"

# 执行迁移
alembic upgrade head

# 回滚迁移
alembic downgrade -1
```

---

## 最佳实践与常见问题
1. **会话管理**：每次请求创建一个会话，操作完成后关闭，避免资源泄漏。
2. **查询优化**：使用 `join` 替代多次查询，使用 `selectinload` 预加载关系数据避免 N+1 问题。
3. **避免全局 session**：不要使用全局 session，应按需创建。
4. **中文乱码**：MySQL 连接字符串添加 `?charset=utf8mb4`。
5. **连接池配置**：生产环境配置 `pool_size` 和 `max_overflow` 优化连接池。

---

## 总结
本教程涵盖了 SQLAlchemy 的核心内容：
- 基础：安装、核心概念、模型定义
- 操作：CRUD、高级查询、事务管理
- 进阶：表关系、Flask 集成、数据库迁移
- 实践：最佳实践与常见问题

通过学习这些内容，你可以使用 SQLAlchemy 高效地操作数据库。建议继续学习 SQLAlchemy Core、性能优化等高级特性。

需要我帮你**生成一个完整的 SQLAlchemy 项目模板**吗？