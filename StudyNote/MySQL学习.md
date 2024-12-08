<div style="text-align: center;"><font size="6">MySQL学习</font></div>

## 一、基本操作

#### 1、连接到MySQL

```bash
mysql -u 用户名 -p
```

#### 2、数据库操作

```sql
-- 显示数据库
SHOW DATABASES;

-- 创建数据库
CREATE DATABASE 数据库名;

-- 删除数据库
DROP DATABASE 数据库名;

-- 选择数据库
USE 数据库名
```

#### 3、表的基本操作

```sql
-- 显示表
SHOW TABLES;

-- 创建表
CREATE TABLE 表明(
    列名1 数据类型 [约束],
    列名2 数据类型 [约束],
    ...
);
例子:
CREATE TABLE users(
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    age INT,
    email VARCHAR(100)
);

-- 查看表结构
DESCRIBE 表名;

-- 删除表
DROP TABLE 表名;

-- 插入数据
INSERT INTO 表名 (列名1, 列名2, ...) VALUES (值1, 值2, ...);
例子:
INSERT INTO users (name, age, email) VALUES ('Alice', 25, 'alice@example.com');


-- 查询数据
SELECT 列名1, 列名2, ... FROM 表名 WHERE 条件;
例子:
SELECT * FROM users WHERE age > 20;

-- 更新数据
UPDATE 表名 SET 列名1 = 新值1, 列名2 = 新值2, ... WHERE 条件
例子:
UPDATE users SET age = 26 WHERE name = 'Alice';

-- 删除数据
DELETE FROM 表名 WHERE 条件;

-- 添加列
ALTER TABLE 表名 ADD 列名 数据类型;

-- 删除列
ALTER TABLE 表名 DROP COLUMN 列名;

-- 修改列
ALTER TABLE 表名 MODIFY COLUMN 列名 新数据类型;

-- 添加索引
CREATE INDEX 索引名 ON 表名(列名);

-- 删除索引
DROP INDEX 索引名 ON 表名;

```

## 二、事务

