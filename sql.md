# SQL 完整教程
SQL（Structured Query Language，结构化查询语言）是关系型数据库的标准操作语言，用于数据的增删改查、数据库结构管理和权限控制。本教程以 **MySQL 8.0** 为主，兼顾 PostgreSQL、SQLite 等主流数据库的差异，从基础到进阶全面讲解 SQL 核心知识。

---

## 目录
- [SQL 基础概念与环境搭建](#sql-基础概念与环境搭建)
- [数据库与表的基本操作](#数据库与表的基本操作)
- [基础 CRUD 操作](#基础-crud-操作)
- [高级查询（连接/子查询/分组）](#高级查询连接子查询分组)
- [SQL 内置函数](#sql-内置函数)
- [事务管理](#事务管理)
- [索引与性能优化](#索引与性能优化)
- [视图与存储过程](#视图与存储过程)
- [触发器与事件](#触发器与事件)
- [用户管理与权限控制](#用户管理与权限控制)
- [最佳实践与常见问题](#最佳实践与常见问题)

---

## SQL 基础概念与环境搭建
### 1. 核心概念
- **数据库（Database）**：存储表的容器，一个数据库包含多张表
- **表（Table）**：由行（记录）和列（字段）组成的二维数据结构
- **字段（Column）**：表中的一列，存储特定类型的数据
- **记录（Row）**：表中的一行，代表一个完整的数据实体
- **主键（Primary Key）**：唯一标识表中每条记录的字段，非空且唯一
- **外键（Foreign Key）**：关联两张表的字段，保证数据的一致性

### 2. MySQL 安装与启动
```bash
# Ubuntu/Debian 安装
sudo apt update
sudo apt install mysql-server

# CentOS/RHEL 安装
sudo dnf install mysql-server

# Windows/Mac 下载安装包：https://dev.mysql.com/downloads/mysql/
```

```bash
# 启动服务
sudo systemctl start mysql
sudo systemctl enable mysql  # 设置开机自启

# 连接数据库（本地）
mysql -u root -p
# 输入密码后进入 MySQL 命令行
```

### 3. 基本命令
```sql
-- 查看所有数据库
SHOW DATABASES;

-- 选择数据库
USE database_name;

-- 查看当前数据库中的所有表
SHOW TABLES;

-- 查看表结构
DESC table_name;

-- 退出 MySQL
EXIT;
```

---

## 数据库与表的基本操作
### 1. 数据库操作
```sql
-- 创建数据库
CREATE DATABASE IF NOT EXISTS test_db 
DEFAULT CHARACTER SET utf8mb4 
DEFAULT COLLATE utf8mb4_unicode_ci;

-- 删除数据库
DROP DATABASE IF EXISTS test_db;

-- 修改数据库字符集
ALTER DATABASE test_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

### 2. 表操作
#### 创建表
```sql
CREATE TABLE IF NOT EXISTS users (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '用户ID',
    username VARCHAR(50) NOT NULL UNIQUE COMMENT '用户名',
    email VARCHAR(100) NOT NULL UNIQUE COMMENT '邮箱',
    password VARCHAR(255) NOT NULL COMMENT '密码',
    age TINYINT UNSIGNED COMMENT '年龄',
    gender ENUM('male', 'female', 'other') DEFAULT 'other' COMMENT '性别',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

#### 常用字段类型
| 类型分类 | 常用类型 | 说明 |
|----------|----------|------|
| 整数 | TINYINT | 1字节，范围 -128~127 |
| | INT | 4字节，范围 -2147483648~2147483647 |
| | BIGINT | 8字节，大整数 |
| 字符串 | CHAR(n) | 定长字符串，最多255字符 |
| | VARCHAR(n) | 变长字符串，最多65535字符 |
| | TEXT | 长文本，最多65535字符 |
| | LONGTEXT | 超长文本，最多4GB |
| 日期时间 | DATE | 日期，格式 YYYY-MM-DD |
| | TIME | 时间，格式 HH:MM:SS |
| | DATETIME | 日期时间，格式 YYYY-MM-DD HH:MM:SS |
| | TIMESTAMP | 时间戳，范围 1970-01-01~2038-01-19 |
| 其他 | BOOLEAN | 布尔值，等价于 TINYINT(1) |
| | ENUM | 枚举类型，只能从指定值中选择 |

#### 约束条件
- `PRIMARY KEY`：主键约束
- `FOREIGN KEY`：外键约束
- `NOT NULL`：非空约束
- `UNIQUE`：唯一约束
- `DEFAULT`：默认值约束
- `CHECK`：检查约束（MySQL 8.0+ 支持）

#### 修改表
```sql
-- 添加字段
ALTER TABLE users ADD COLUMN phone VARCHAR(20) AFTER email;

-- 修改字段类型
ALTER TABLE users MODIFY COLUMN age SMALLINT UNSIGNED;

-- 修改字段名
ALTER TABLE users CHANGE COLUMN phone mobile VARCHAR(20);

-- 删除字段
ALTER TABLE users DROP COLUMN mobile;

-- 添加主键
ALTER TABLE users ADD PRIMARY KEY (id);

-- 添加外键
ALTER TABLE posts ADD FOREIGN KEY (author_id) REFERENCES users(id);

-- 删除表
DROP TABLE IF EXISTS users;
```

---

## 基础 CRUD 操作
### 1. 增（INSERT）
```sql
-- 插入单条记录
INSERT INTO users (username, email, password, age, gender)
VALUES ('zhangsan', 'zhangsan@example.com', '123456', 25, 'male');

-- 插入多条记录
INSERT INTO users (username, email, password, age, gender)
VALUES 
('lisi', 'lisi@example.com', '123456', 22, 'female'),
('wangwu', 'wangwu@example.com', '123456', 30, 'male');

-- 插入完整字段（可省略字段名）
INSERT INTO users VALUES (4, 'zhaoliu', 'zhaoliu@example.com', '123456', 28, 'male', NOW(), NOW());
```

### 2. 查（SELECT）
```sql
-- 查询所有字段
SELECT * FROM users;

-- 查询指定字段
SELECT username, email, age FROM users;

-- 别名查询
SELECT username AS '用户名', email AS '邮箱' FROM users;

-- 去重查询
SELECT DISTINCT gender FROM users;

-- 条件查询（WHERE）
SELECT * FROM users WHERE age > 25;
SELECT * FROM users WHERE gender = 'male' AND age < 30;
SELECT * FROM users WHERE age BETWEEN 20 AND 30;
SELECT * FROM users WHERE username IN ('zhangsan', 'lisi');
SELECT * FROM users WHERE email LIKE '%@example.com';  -- % 匹配任意字符
SELECT * FROM users WHERE username LIKE 'zhang_';     -- _ 匹配单个字符
SELECT * FROM users WHERE bio IS NULL;  -- 判断空值

-- 排序查询（ORDER BY）
SELECT * FROM users ORDER BY age ASC;   -- 升序（默认）
SELECT * FROM users ORDER BY created_at DESC;  -- 降序
SELECT * FROM users ORDER BY age DESC, created_at ASC;  -- 多字段排序

-- 分页查询（LIMIT）
SELECT * FROM users LIMIT 10;  -- 前10条
SELECT * FROM users LIMIT 10 OFFSET 10;  -- 第11-20条（第2页）
-- 简写：LIMIT 偏移量, 条数
SELECT * FROM users LIMIT 10, 10;
```

### 3. 改（UPDATE）
```sql
-- 更新单条记录
UPDATE users SET email = 'new_zhangsan@example.com' WHERE id = 1;

-- 更新多条记录
UPDATE users SET age = age + 1 WHERE gender = 'male';

-- 更新多个字段
UPDATE users SET password = 'new_password', updated_at = NOW() WHERE id = 2;

-- 注意：UPDATE 必须加 WHERE 条件，否则会更新整张表！
```

### 4. 删（DELETE）
```sql
-- 删除单条记录
DELETE FROM users WHERE id = 1;

-- 删除多条记录
DELETE FROM users WHERE age < 18;

-- 清空表（保留表结构）
DELETE FROM users;

-- 清空表（更快，不可回滚）
TRUNCATE TABLE users;

-- 注意：DELETE 必须加 WHERE 条件，否则会删除整张表！
```

---

## 高级查询（连接/子查询/分组）
### 1. 连接查询（JOIN）
用于从多张表中查询关联数据。

#### 准备数据
```sql
CREATE TABLE posts (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(200) NOT NULL,
    content TEXT NOT NULL,
    author_id INT NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (author_id) REFERENCES users(id)
);

INSERT INTO posts (title, content, author_id) VALUES
('第一篇文章', '内容1', 2),
('第二篇文章', '内容2', 2),
('第三篇文章', '内容3', 3);
```

#### 内连接（INNER JOIN）
只返回两张表中匹配的记录。
```sql
SELECT 
    u.username,
    p.title,
    p.created_at
FROM users u
INNER JOIN posts p ON u.id = p.author_id;
```

#### 左连接（LEFT JOIN）
返回左表的所有记录，右表中没有匹配的记录显示 NULL。
```sql
SELECT 
    u.username,
    p.title
FROM users u
LEFT JOIN posts p ON u.id = p.author_id;
-- 会显示所有用户，没有文章的用户 title 为 NULL
```

#### 右连接（RIGHT JOIN）
返回右表的所有记录，左表中没有匹配的记录显示 NULL。
```sql
SELECT 
    u.username,
    p.title
FROM users u
RIGHT JOIN posts p ON u.id = p.author_id;
```

#### 全连接（FULL JOIN）
返回两张表的所有记录，MySQL 不直接支持，可通过 `UNION` 实现。
```sql
SELECT * FROM users u LEFT JOIN posts p ON u.id = p.author_id
UNION
SELECT * FROM users u RIGHT JOIN posts p ON u.id = p.author_id;
```

### 2. 子查询
嵌套在其他查询中的查询，也叫嵌套查询。

#### 标量子查询（返回单个值）
```sql
-- 查询年龄最大的用户
SELECT * FROM users WHERE age = (SELECT MAX(age) FROM users);
```

#### 列子查询（返回一列值）
```sql
-- 查询有文章的用户
SELECT * FROM users WHERE id IN (SELECT DISTINCT author_id FROM posts);
```

#### 行子查询（返回一行值）
```sql
-- 查询与张三年龄和性别都相同的用户
SELECT * FROM users WHERE (age, gender) = (SELECT age, gender FROM users WHERE username = 'zhangsan');
```

#### 表子查询（返回一张表）
```sql
-- 查询用户及其文章数量
SELECT u.username, p_count.post_count
FROM users u
JOIN (
    SELECT author_id, COUNT(*) AS post_count 
    FROM posts 
    GROUP BY author_id
) p_count ON u.id = p_count.author_id;
```

### 3. 分组查询（GROUP BY）
将数据按指定字段分组，通常与聚合函数一起使用。
```sql
-- 统计每个性别的用户数量
SELECT gender, COUNT(*) AS user_count
FROM users
GROUP BY gender;

-- 统计每个用户的文章数量
SELECT u.username, COUNT(p.id) AS post_count
FROM users u
LEFT JOIN posts p ON u.id = p.author_id
GROUP BY u.id;

-- 分组后过滤（HAVING）
-- 查询文章数大于1的用户
SELECT u.username, COUNT(p.id) AS post_count
FROM users u
JOIN posts p ON u.id = p.author_id
GROUP BY u.id
HAVING post_count > 1;
```

> **注意**：`WHERE` 过滤行，`HAVING` 过滤分组；`WHERE` 在分组前执行，`HAVING` 在分组后执行。

### 4. 联合查询（UNION）
合并多个 SELECT 语句的结果集。
```sql
-- 合并两个查询结果（自动去重）
SELECT username, email FROM users WHERE gender = 'male'
UNION
SELECT username, email FROM users WHERE age > 25;

-- 合并所有结果（不去重，更快）
SELECT username, email FROM users WHERE gender = 'male'
UNION ALL
SELECT username, email FROM users WHERE age > 25;
```

---

## SQL 内置函数
### 1. 聚合函数
| 函数 | 说明 |
|------|------|
| `COUNT(*)` | 统计记录数 |
| `SUM(column)` | 求和 |
| `AVG(column)` | 求平均值 |
| `MAX(column)` | 求最大值 |
| `MIN(column)` | 求最小值 |

### 2. 字符串函数
```sql
-- 字符串长度
SELECT LENGTH('hello');  -- 5

-- 拼接字符串
SELECT CONCAT(username, ' - ', email) FROM users;

-- 截取字符串
SELECT SUBSTRING(username, 1, 3) FROM users;  -- 从第1个字符开始截取3个

-- 转换大小写
SELECT UPPER(username), LOWER(email) FROM users;

-- 替换字符串
SELECT REPLACE(email, '@example.com', '@test.com') FROM users;

-- 去除空格
SELECT TRIM('  hello  ');  -- 'hello'
```

### 3. 数值函数
```sql
SELECT ABS(-10);  -- 10
SELECT CEIL(3.14);  -- 4
SELECT FLOOR(3.14);  -- 3
SELECT ROUND(3.14159, 2);  -- 3.14
SELECT RAND();  -- 0~1之间的随机数
SELECT MOD(10, 3);  -- 1（取余）
```

### 4. 日期函数
```sql
-- 当前日期时间
SELECT NOW();  -- 2024-05-20 10:30:00
SELECT CURDATE();  -- 2024-05-20
SELECT CURTIME();  -- 10:30:00

-- 提取日期部分
SELECT YEAR(created_at), MONTH(created_at), DAY(created_at) FROM users;

-- 日期计算
SELECT DATE_ADD(NOW(), INTERVAL 1 DAY);  -- 加1天
SELECT DATE_SUB(NOW(), INTERVAL 1 MONTH);  -- 减1个月
SELECT DATEDIFF(NOW(), created_at) FROM users;  -- 计算天数差
```

### 5. 条件函数
```sql
-- IF 函数
SELECT username, IF(age >= 18, '成年', '未成年') AS age_status FROM users;

-- CASE WHEN 函数
SELECT 
    username,
    CASE 
        WHEN age < 18 THEN '未成年'
        WHEN age < 30 THEN '青年'
        WHEN age < 60 THEN '中年'
        ELSE '老年'
    END AS age_group
FROM users;
```

---

## 事务管理
事务是一组原子性的 SQL 操作，要么全部执行成功，要么全部执行失败。

### 1. ACID 特性
- **原子性（Atomicity）**：事务是不可分割的最小单位
- **一致性（Consistency）**：事务执行前后数据保持一致
- **隔离性（Isolation）**：多个事务之间互不干扰
- **持久性（Durability）**：事务提交后数据永久保存

### 2. 事务操作
```sql
-- 开启事务
START TRANSACTION;
-- 或
BEGIN;

-- 执行 SQL 操作
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- 提交事务（所有操作生效）
COMMIT;

-- 回滚事务（所有操作撤销）
ROLLBACK;
```

### 3. 事务隔离级别
MySQL 支持 4 种隔离级别，从低到高：
1. **读未提交（Read Uncommitted）**：可能出现脏读、不可重复读、幻读
2. **读已提交（Read Committed）**：避免脏读，可能出现不可重复读、幻读（MySQL 默认）
3. **可重复读（Repeatable Read）**：避免脏读、不可重复读，可能出现幻读
4. **串行化（Serializable）**：避免所有问题，性能最差

```sql
-- 查看当前隔离级别
SELECT @@transaction_isolation;

-- 设置隔离级别
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

---

## 索引与性能优化
索引是提高数据库查询速度的关键数据结构。

### 1. 索引类型
- **主键索引（PRIMARY KEY）**：主键自动创建，唯一且非空
- **唯一索引（UNIQUE）**：字段值唯一，允许空值
- **普通索引（INDEX）**：最基本的索引
- **复合索引**：多个字段组成的索引
- **全文索引（FULLTEXT）**：用于全文搜索

### 2. 索引操作
```sql
-- 创建普通索引
CREATE INDEX idx_username ON users(username);

-- 创建唯一索引
CREATE UNIQUE INDEX idx_email ON users(email);

-- 创建复合索引
CREATE INDEX idx_gender_age ON users(gender, age);

-- 查看表的索引
SHOW INDEX FROM users;

-- 删除索引
DROP INDEX idx_username ON users;
```

### 3. 索引使用原则
- 为经常用于 `WHERE`、`JOIN`、`ORDER BY` 的字段创建索引
- 复合索引遵循最左前缀原则
- 避免在低基数字段（如性别）上创建索引
- 避免在频繁更新的字段上创建索引
- 不要创建过多索引，会影响插入和更新性能

### 4. 查看执行计划
使用 `EXPLAIN` 分析 SQL 执行计划，判断是否使用了索引。
```sql
EXPLAIN SELECT * FROM users WHERE username = 'zhangsan';
```

---

## 视图与存储过程
### 1. 视图
视图是基于 SQL 查询结果的虚拟表，简化复杂查询，提高安全性。
```sql
-- 创建视图
CREATE VIEW user_posts_view AS
SELECT 
    u.username,
    p.title,
    p.created_at
FROM users u
JOIN posts p ON u.id = p.author_id;

-- 查询视图
SELECT * FROM user_posts_view WHERE username = 'lisi';

-- 修改视图
ALTER VIEW user_posts_view AS
SELECT 
    u.id AS user_id,
    u.username,
    p.id AS post_id,
    p.title
FROM users u
JOIN posts p ON u.id = p.author_id;

-- 删除视图
DROP VIEW IF EXISTS user_posts_view;
```

### 2. 存储过程
存储过程是一组预编译的 SQL 语句，封装复杂逻辑，提高执行效率。
```sql
-- 创建存储过程
DELIMITER //  -- 修改语句结束符为 //
CREATE PROCEDURE GetUserPosts(IN user_id INT)
BEGIN
    SELECT 
        u.username,
        p.title,
        p.content
    FROM users u
    JOIN posts p ON u.id = p.author_id
    WHERE u.id = user_id;
END //
DELIMITER ;  -- 改回默认结束符

-- 调用存储过程
CALL GetUserPosts(2);

-- 删除存储过程
DROP PROCEDURE IF EXISTS GetUserPosts;
```

---

## 触发器与事件
### 1. 触发器
触发器是在表发生插入、更新、删除操作时自动执行的 SQL 语句。
```sql
-- 创建触发器：在用户更新时记录日志
CREATE TABLE user_logs (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    action VARCHAR(20) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

DELIMITER //
CREATE TRIGGER after_user_update
AFTER UPDATE ON users
FOR EACH ROW
BEGIN
    INSERT INTO user_logs (user_id, action) VALUES (OLD.id, 'update');
END //
DELIMITER ;

-- 删除触发器
DROP TRIGGER IF EXISTS after_user_update;
```

### 2. 事件
事件是定时执行的 SQL 语句，用于定时任务。
```sql
-- 开启事件调度器
SET GLOBAL event_scheduler = ON;

-- 创建事件：每天凌晨删除7天前的日志
CREATE EVENT delete_old_logs
ON SCHEDULE EVERY 1 DAY
STARTS '2024-05-21 00:00:00'
DO
DELETE FROM user_logs WHERE created_at < DATE_SUB(NOW(), INTERVAL 7 DAY);

-- 删除事件
DROP EVENT IF EXISTS delete_old_logs;
```

---

## 用户管理与权限控制
### 1. 用户管理
```sql
-- 创建用户
CREATE USER 'new_user'@'localhost' IDENTIFIED BY 'password123';
-- 允许远程连接
CREATE USER 'new_user'@'%' IDENTIFIED BY 'password123';

-- 修改用户密码
ALTER USER 'new_user'@'localhost' IDENTIFIED BY 'new_password';

-- 删除用户
DROP USER 'new_user'@'localhost';
```

### 2. 权限控制
```sql
-- 授予权限
-- 授予 test_db 所有表的查询、插入权限
GRANT SELECT, INSERT ON test_db.* TO 'new_user'@'localhost';
-- 授予所有数据库的所有权限（超级用户）
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' WITH GRANT OPTION;

-- 刷新权限
FLUSH PRIVILEGES;

-- 查看用户权限
SHOW GRANTS FOR 'new_user'@'localhost';

-- 撤销权限
REVOKE INSERT ON test_db.* FROM 'new_user'@'localhost';
```

---

## 最佳实践与常见问题
### 1. 命名规范
- 数据库、表、字段使用小写字母，单词之间用下划线分隔
- 表名使用复数形式（如 users、posts）
- 主键统一命名为 id
- 外键命名为 表名_id（如 author_id）
- 索引命名为 idx_字段名（如 idx_username）

### 2. SQL 注入防护
- 永远不要拼接 SQL 字符串
- 使用参数化查询（Prepared Statement）
- 对用户输入进行验证和转义

### 3. 性能优化
- 避免使用 `SELECT *`，只查询需要的字段
- 合理使用索引，避免全表扫描
- 避免在 WHERE 子句中对字段进行函数操作
- 大表查询使用 LIMIT 分页
- 复杂查询使用视图或存储过程

### 4. 常见问题
- **中文乱码**：确保数据库、表、连接都使用 utf8mb4 字符集
- **死锁**：避免长时间事务，按相同顺序访问表
- **连接数过多**：优化连接池配置，及时关闭连接
- **慢查询**：开启慢查询日志，分析并优化慢 SQL

---

## 总结
本教程涵盖了 SQL 的核心内容：
- 基础：环境搭建、数据库与表操作、CRUD
- 进阶：连接查询、子查询、分组聚合、函数
- 高级：事务、索引、视图、存储过程、触发器
- 管理：用户权限、性能优化、最佳实践

通过学习这些内容，你可以熟练使用 SQL 操作关系型数据库。建议结合实际项目练习，深入理解数据库设计和性能优化。