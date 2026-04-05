# SQL 学习笔记

## 目录
1. [SQL 基础语法](#sql-基础语法)
2. [索引](#索引)
3. [数据库性能](#数据库性能)
4. [SQL 优化](#sql-优化)
5. [SQL 性能分析](#sql-性能分析)

---

## SQL 基础语法

### SELECT 查询
```sql
-- 基本查询
SELECT * FROM users;

-- 查询指定列
SELECT id, name, email FROM users;

-- 别名
SELECT name AS "用户名", email AS "邮箱" FROM users;

-- 去重
SELECT DISTINCT status FROM orders;

-- 限制结果
SELECT * FROM users LIMIT 10 OFFSET 20;

-- 条件查询
SELECT * FROM users WHERE age > 18;

-- 多条件
SELECT * FROM users 
WHERE age > 18 
AND status = 'active'
OR country = 'USA';

-- 排序
SELECT * FROM users ORDER BY created_at DESC;

-- 聚合函数
SELECT 
    COUNT(*) AS total_users,
    AVG(age) AS avg_age,
    MAX(age) AS max_age,
    MIN(age) AS min_age,
    SUM(salary) AS total_salary
FROM users;

-- 分组
SELECT country, COUNT(*) AS count 
FROM users 
GROUP BY country 
HAVING COUNT(*) > 100;
```

### INSERT 插入
```sql
-- 单条插入
INSERT INTO users (name, email, age) 
VALUES ('Alice', 'alice@example.com', 25);

-- 多条插入
INSERT INTO users (name, email, age) VALUES 
    ('Bob', 'bob@example.com', 30),
    ('Charlie', 'charlie@example.com', 28);

-- 从其他表插入
INSERT INTO users_archive (id, name, email)
SELECT id, name, email FROM users WHERE status = 'archived';
```

### UPDATE 更新
```sql
-- 更新单列
UPDATE users SET email = 'new@example.com' WHERE id = 1;

-- 更新多列
UPDATE users 
SET email = 'new@example.com',
    age = 26,
    updated_at = NOW()
WHERE id = 1;

-- 批量更新
UPDATE products 
SET price = price * 0.9 
WHERE category = 'electronics' 
AND stock > 100;
```

### DELETE 删除
```sql
-- 删除数据
DELETE FROM users WHERE id = 1;

-- 批量删除
DELETE FROM users 
WHERE status = 'inactive' 
AND created_at < '2023-01-01';

-- 清空表
DELETE FROM users;  -- 可回滚
TRUNCATE TABLE users;  -- 不可回滚，速度快
```

### JOIN 连接
```sql
-- INNER JOIN
SELECT u.name, o.order_id, o.total
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN
SELECT u.name, o.order_id
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- RIGHT JOIN
SELECT u.name, o.order_id
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- FULL OUTER JOIN
SELECT u.name, o.order_id
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;

-- 多表连接
SELECT 
    u.name,
    o.order_id,
    p.product_name
FROM users u
INNER JOIN orders o ON u.id = o.user_id
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products p ON oi.product_id = p.id;
```

### 子查询
```sql
-- 在WHERE中使用
SELECT * FROM users 
WHERE age > (SELECT AVG(age) FROM users);

-- 在FROM中使用
SELECT avg_age FROM (
    SELECT country, AVG(age) AS avg_age
    FROM users
    GROUP BY country
) AS country_stats;

-- 使用IN
SELECT * FROM products 
WHERE category_id IN (
    SELECT id FROM categories WHERE name LIKE 'Electronics%'
);

-- 使用EXISTS
SELECT * FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id
);

-- 相关子查询
SELECT p.name, p.price
FROM products p
WHERE p.price > (SELECT AVG(price) FROM products WHERE category_id = p.category_id);
```

### UNION 和 INTERSECT
```sql
-- UNION (合并，去重)
SELECT name FROM customers
UNION
SELECT name FROM suppliers;

-- UNION ALL (合并，保留重复)
SELECT name FROM customers
UNION ALL
SELECT name FROM suppliers;

-- INTERSECT (交集)
SELECT name FROM customers
INTERSECT
SELECT name FROM suppliers;
```

### 常用函数

#### 字符串函数
```sql
SELECT 
    CONCAT(first_name, ' ', last_name) AS full_name,
    UPPER(name) AS uppercase,
    LOWER(email) AS lowercase,
    LENGTH(name) AS name_length,
    SUBSTRING(name, 1, 3) AS first_3_chars,
    TRIM('  hello  ') AS trimmed,
    REPLACE(name, 'Mr.', '') AS cleaned_name,
    LIKE '%test%',  -- 模糊匹配
    LEFT(name, 5),
    RIGHT(name, 5),
    LPAD(id, 6, '0') AS padded_id;
```

#### 日期函数
```sql
SELECT 
    NOW() AS current_datetime,
    CURDATE() AS current_date,
    CURTIME() AS current_time,
    YEAR(created_at) AS year,
    MONTH(created_at) AS month,
    DAY(created_at) AS day,
    DATE_FORMAT(created_at, '%Y-%m-%d') AS formatted_date,
    DATE_ADD(created_at, INTERVAL 7 DAY) AS week_later,
    DATE_SUB(created_at, INTERVAL 1 MONTH) AS month_before,
    DATEDIFF(NOW(), created_at) AS days_since_created,
    DAYNAME(created_at) AS day_name,
    MONTHNAME(created_at) AS month_name,
    WEEK(created_at) AS week_number,
    EXTRACT(YEAR FROM created_at) AS year_part;
```

#### 数值函数
```sql
SELECT 
    ABS(-10) AS absolute_value,
    CEIL(4.3) AS ceiling,
    FLOOR(4.7) AS floor,
    ROUND(3.14159, 2) AS rounded,
    MOD(10, 3) AS modulus,
    POWER(2, 3) AS power,
    SQRT(16) AS square_root,
    RAND() AS random_number,
    GREATEST(3, 1, 5, 2) AS greatest,
    LEAST(3, 1, 5, 2) AS least;
```

#### 条件函数
```sql
-- CASE WHEN
SELECT 
    name,
    age,
    CASE 
        WHEN age < 18 THEN 'Minor'
        WHEN age < 65 THEN 'Adult'
        ELSE 'Senior'
    END AS age_group,
    CASE status
        WHEN 'active' THEN 1
        WHEN 'inactive' THEN 0
        ELSE -1
    END AS status_code
FROM users;

-- COALESCE
SELECT COALESCE(email, phone, 'No contact') AS contact_info;

-- NULLIF
SELECT NULLIF(value, 0);  -- 如果value=0则返回NULL

-- IF
SELECT IF(age > 18, 'Adult', 'Minor') AS age_category;
```

---

## 索引

### 创建索引
```sql
-- 单列索引
CREATE INDEX idx_users_email ON users(email);

-- 多列索引 (复合索引)
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);

-- 唯一索引
CREATE UNIQUE INDEX idx_products_sku ON products(sku);

-- 全文索引
CREATE FULLTEXT INDEX idx_articles_content ON articles(content);

-- 降序索引 (MySQL 8.0+)
CREATE INDEX idx_users_created ON users(created_at DESC);
```

### 索引类型

#### B-Tree 索引 (默认)
```sql
-- 适用于: =, >, <, BETWEEN, LIKE 'prefix%'
CREATE INDEX idx_users_age ON users(age);

-- 可以优化以下查询
SELECT * FROM users WHERE age = 25;
SELECT * FROM users WHERE age > 18 AND age < 65;
SELECT * FROM users WHERE name LIKE 'John%';
```

#### Hash 索引
```sql
-- 适用于: =, IN
CREATE INDEX idx_sessions_token USING HASH ON sessions(token);
```

#### 复合索引最佳实践
```sql
-- 创建复合索引
CREATE INDEX idx_orders_user_status ON orders(user_id, status, created_at);

-- 可以优化以下查询
SELECT * FROM orders WHERE user_id = 1;
SELECT * FROM orders WHERE user_id = 1 AND status = 'pending';
SELECT * FROM orders WHERE user_id = 1 AND status = 'pending' AND created_at > '2024-01-01';

-- 注意: 最左前缀原则
-- 这个索引不能优化
SELECT * FROM orders WHERE status = 'pending';
```

### 查看和删除索引
```sql
-- 查看表的所有索引
SHOW INDEX FROM users;

-- MySQL
SELECT 
    TABLE_NAME,
    INDEX_NAME,
    COLUMN_NAME,
    NON_UNIQUE
FROM information_schema.STATISTICS
WHERE TABLE_NAME = 'users';

-- 删除索引
DROP INDEX idx_users_email ON users;

-- 或
ALTER TABLE users DROP INDEX idx_users_email;
```

### 索引使用技巧
```sql
-- 避免在索引列上使用函数
-- 不好:
SELECT * FROM users WHERE YEAR(created_at) = 2024;

-- 好:
SELECT * FROM users WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';

-- 避免在索引列上进行计算
-- 不好:
SELECT * FROM orders WHERE total / 2 > 100;

-- 好:
SELECT * FROM orders WHERE total > 200;

-- 使用覆盖索引
CREATE INDEX idx_users_name_age ON users(name, age);

-- 这个查询不需要回表
SELECT name, age FROM users WHERE name = 'John';

-- 前导列要在WHERE中使用
CREATE INDEX idx_orders_status_date ON orders(status, created_at);

-- 好:
SELECT * FROM orders WHERE status = 'pending';
SELECT * FROM orders WHERE status = 'pending' AND created_at > '2024-01-01';
```

---

## 数据库性能

### 连接池配置
```sql
-- MySQL 连接池设置
-- my.cnf
max_connections = 200
wait_timeout = 28800
interactive_timeout = 28800

-- PostgreSQL 连接池
-- postgresql.conf
max_connections = 100
superuser_reserved_connections = 3
```

### 缓存配置
```sql
-- MySQL 查询缓存 (MySQL 8.0已移除)
-- 查询缓存在MySQL 5.7中
query_cache_type = 1
query_cache_size = 64M
query_cache_limit = 2M

-- 启用查询缓存
SELECT SQL_CACHE * FROM users WHERE id = 1;

-- 不使用缓存
SELECT SQL_NO_CACHE * FROM users WHERE id = 1;
```

### 内存配置
```sql
-- MySQL InnoDB 缓冲池
-- my.cnf
innodb_buffer_pool_size = 4G
innodb_buffer_pool_instances = 4
innodb_log_file_size = 1G
innodb_flush_log_at_trx_commit = 1

-- PostgreSQL 内存配置
-- postgresql.conf
shared_buffers = 4GB
work_mem = 64MB
maintenance_work_mem = 512MB
effective_cache_size = 12GB
```

### 并发控制
```sql
-- 设置隔离级别
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- 设置锁等待超时
-- MySQL
SET innodb_lock_wait_timeout = 50;

-- PostgreSQL
SET lock_timeout = '1s';

-- 设置死锁检测
-- PostgreSQL
SET deadlock_timeout = '1s';
```

---

## SQL 优化

### 查询优化

#### 避免 SELECT *
```sql
-- 不好:
SELECT * FROM users WHERE id = 1;

-- 好:
SELECT id, name, email FROM users WHERE id = 1;
```

#### 使用 LIMIT 限制结果
```sql
-- 添加 LIMIT
SELECT id, name FROM users LIMIT 100;

-- 分页查询
SELECT id, name FROM users LIMIT 10 OFFSET 20;
```

#### 批量操作替代循环
```sql
-- 不好: 循环插入
INSERT INTO users (name) VALUES ('A');
INSERT INTO users (name) VALUES ('B');

-- 好: 批量插入
INSERT INTO users (name) VALUES 
    ('A'), ('B'), ('C'), ('D');
```

#### 使用 UNION ALL 替代 OR
```sql
-- 优化前
SELECT * FROM orders WHERE user_id = 1 OR status = 'pending';

-- 优化后
SELECT * FROM orders WHERE user_id = 1
UNION ALL
SELECT * FROM orders WHERE status = 'pending' AND user_id != 1;
```

#### 使用 JOIN 替代子查询
```sql
-- 优化前 (子查询)
SELECT * FROM users 
WHERE id IN (SELECT user_id FROM orders WHERE total > 100);

-- 优化后 (JOIN)
SELECT DISTINCT u.* 
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE o.total > 100;
```

### 写操作优化

#### 批量更新
```sql
-- 优化前
UPDATE products SET price = price * 0.9 WHERE id = 1;
UPDATE products SET price = price * 0.9 WHERE id = 2;

-- 优化后
UPDATE products SET price = price * 0.9 WHERE id IN (1, 2);
```

#### 使用 EXISTS 替代 COUNT
```sql
-- 优化前
SELECT COUNT(*) FROM users WHERE status = 'active';

-- 优化后
SELECT EXISTS(SELECT 1 FROM users WHERE status = 'active') AS has_active;
```

### 索引优化

#### 创建合适的索引
```sql
-- 根据查询创建索引
CREATE INDEX idx_orders_user_status_date 
ON orders(user_id, status, created_at);

-- 前导列原则
-- 如果查询是 WHERE user_id = ? AND status = ?
-- 索引顺序应该是 (user_id, status)
```

#### 使用覆盖索引
```sql
-- 覆盖索引: 包含所有SELECT和WHERE需要的列
CREATE INDEX idx_users_name_status ON users(name, status);

-- 这个查询可以直接从索引返回，不需要回表
SELECT name, status FROM users WHERE name = 'John' AND status = 'active';
```

### 表结构优化

#### 适当的数据类型
```sql
-- 使用适当的数据类型
-- 不好:
CREATE TABLE users (
    age VARCHAR(10),  -- 存储数字应该用整数类型
    phone_number TEXT  -- 手机号应该用固定长度字符串
);

-- 好:
CREATE TABLE users (
    age INT,
    phone_number CHAR(11)
);
```

#### 垂直分割
```sql
-- 将不常用字段分离到另一张表
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE user_profiles (
    user_id INT PRIMARY KEY,
    bio TEXT,
    avatar_url VARCHAR(255),
    preferences JSON
);
```

#### 适当冗余
```sql
-- 反规范化以减少JOIN
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    user_name VARCHAR(100),  -- 冗余但减少JOIN
    product_name VARCHAR(100),  -- 冗余但提高性能
    total DECIMAL(10, 2)
);
```

---

## SQL 性能分析

### EXPLAIN 分析查询
```sql
-- 基本分析
EXPLAIN SELECT * FROM users WHERE email = 'test@example.com';

-- 详细分析
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@example.com';

-- JSON格式输出 (MySQL)
EXPLAIN FORMAT=JSON SELECT * FROM users WHERE email = 'test@example.com';
```

### EXPLAIN 输出解读

#### MySQL EXPLAIN
```
id: 1                           -- 查询编号
select_type: SIMPLE             -- 查询类型
table: users                    -- 表名
type: ref                       -- 访问类型 (const, ref, range, all)
possible_keys: idx_email        -- 可能使用的索引
key: idx_email                  -- 实际使用的索引
key_len: 102                    -- 索引长度
ref: const                      -- 使用常量比较
rows: 1                         -- 预估扫描行数
filtered: 100.00                -- 过滤百分比
Extra: Using index condition    -- 额外信息
```

#### 访问类型说明
| type | 说明 | 性能 |
|------|------|------|
| const | 主键或唯一索引等值查询 | 最快 |
| eq_ref | 使用主键或唯一索引连接 | 快 |
| ref | 使用非唯一索引等值查询 | 快 |
| range | 索引范围查询 | 中 |
| index | 全索引扫描 | 慢 |
| ALL | 全表扫描 | 最慢 |

### 慢查询日志

#### MySQL 慢查询
```sql
-- 开启慢查询日志
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;  -- 超过2秒记录
SET GLOBAL slow_query_log_file = '/var/log/mysql/slow.log';

-- 查看慢查询配置
SHOW VARIABLES LIKE 'slow_query%';
SHOW VARIABLES LIKE 'long_query_time%';
```

#### PostgreSQL 慢查询
```sql
-- 开启慢查询日志
ALTER SYSTEM SET log_min_duration_statement = 2000;  -- 毫秒

-- 查看日志
SELECT * FROM pg_stat_activity WHERE state = 'active';
```

### 性能监控

#### MySQL 监控
```sql
-- 查看当前连接
SHOW PROCESSLIST;
SHOW FULL PROCESSLIST;

-- 查看状态
SHOW STATUS LIKE 'Threads%';
-- Threads_connected: 当前连接数
-- Threads_running: 正在运行的查询数

-- 查看InnoDB状态
SHOW ENGINE INNODB STATUS;

-- 查看查询统计
SHOW GLOBAL STATUS LIKE 'Com_select';
SHOW GLOBAL STATUS LIKE 'Com_insert';
SHOW GLOBAL STATUS LIKE 'Slow_queries';
```

#### PostgreSQL 监控
```sql
-- 查看活动连接
SELECT * FROM pg_stat_activity WHERE state = 'active';

-- 查看查询统计
SELECT * FROM pg_stat_statements 
ORDER BY total_time DESC 
LIMIT 10;

-- 查看表统计
SELECT * FROM pg_stat_user_tables;
```

### 常用诊断查询

#### MySQL
```sql
-- 查找全表扫描的查询
SELECT * FROM mysql.general_log 
WHERE command_type = 'Query' 
AND argument LIKE '%SELECT%';

-- 查找最慢的查询
SELECT 
    digest_text AS query,
    count_star AS execution_count,
    avg_timer_wait / 1000000000000 AS avg_duration
FROM performance_schema.events_statements_summary_by_digest
ORDER BY avg_timer_wait DESC
LIMIT 10;
```

#### PostgreSQL
```sql
-- 查找最慢的查询
SELECT 
    query,
    calls,
    total_time,
    mean_time,
    rows
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;

-- 查找缺失索引的表
SELECT 
    schemaname,
    tablename,
    seq_scan,
    idx_scan
FROM pg_stat_user_tables
WHERE seq_scan > idx_scan
ORDER BY seq_scan DESC;
```

### PROFILE 分析 (MySQL)
```sql
-- 开启PROFILING
SET profiling = 1;

-- 执行查询
SELECT COUNT(*) FROM orders WHERE status = 'pending';

-- 查看PROFILE
SHOW PROFILES;
SHOW PROFILE FOR QUERY 1;

-- 关闭PROFILING
SET profiling = 0;
```
