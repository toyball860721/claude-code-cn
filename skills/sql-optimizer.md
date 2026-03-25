# ⚡ SQL 优化 (SQL Optimizer)

## 描述

分析 SQL 查询语句的性能问题，给出优化方案。涵盖索引建议、查询改写、执行计划解读、分页优化、慢查询治理等，支持 MySQL、PostgreSQL、SQLite。

## 使用场景

- 慢查询日志中的 SQL 优化
- EXPLAIN 执行计划解读
- 索引设计和优化建议
- 大表查询的分页优化
- ORM 生成的低效 SQL 改写
- 数据库迁移前的性能预评估

## Prompt 模板

```
请优化以下 SQL 查询。

环境信息：
- **数据库**：{database}（MySQL 8.0 / PostgreSQL 15 / SQLite）
- **表结构**：
```sql
{table_schema}
```
- **数据量**：{data_volume}（如：users 500万行，orders 2000万行）
- **现有索引**：{existing_indexes}
- **当前执行时间**：{current_time}

需要优化的 SQL：
```sql
{sql}
```

EXPLAIN 结果（如有）：
```
{explain_output}
```

请输出：
1. **问题诊断**：当前 SQL 的性能瓶颈
2. **优化后 SQL**：改写后的查询
3. **索引建议**：需要创建/修改的索引（含 CREATE INDEX 语句）
4. **优化原理**：为什么这样改更快
5. **预期提升**：优化后的预估执行时间
```

## 示例输入

```
数据库：MySQL 8.0
数据量：orders 2000万行，users 500万行

表结构：
CREATE TABLE orders (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT,
    status VARCHAR(20),
    amount DECIMAL(10,2),
    created_at DATETIME,
    INDEX idx_user_id (user_id)
);

当前执行时间：4.7秒

SQL：
SELECT u.name, COUNT(o.id) as order_count, SUM(o.amount) as total_amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed'
  AND o.created_at >= '2024-01-01'
  AND o.created_at < '2025-01-01'
GROUP BY u.id, u.name
HAVING total_amount > 1000
ORDER BY total_amount DESC
LIMIT 20 OFFSET 10000;
```

## 示例输出

```markdown
# SQL 优化报告

## 🔍 问题诊断

| # | 问题 | 影响 |
|---|------|------|
| 1 | `LEFT JOIN` + `WHERE o.status` 等价于 `INNER JOIN`，但优化器可能选错执行计划 | 扫描多余行 |
| 2 | 缺少 `(status, created_at)` 复合索引 | 全表扫描 orders |
| 3 | `OFFSET 10000` 深分页问题 | 需要扫描并丢弃前 10000 行 |
| 4 | `GROUP BY u.id, u.name` 不必要 | `u.name` 由 `u.id` 函数决定 |

## ✅ 优化后 SQL

```sql
-- 步骤1：复合索引覆盖查询条件
CREATE INDEX idx_orders_status_date ON orders(status, created_at, user_id, amount);

-- 步骤2：改写查询
SELECT u.name, t.order_count, t.total_amount
FROM (
    SELECT user_id,
           COUNT(*) as order_count,
           SUM(amount) as total_amount
    FROM orders
    WHERE status = 'completed'
      AND created_at >= '2024-01-01'
      AND created_at < '2025-01-01'
    GROUP BY user_id
    HAVING SUM(amount) > 1000
    ORDER BY total_amount DESC
    LIMIT 20 OFFSET 10000
) t
JOIN users u ON u.id = t.user_id
ORDER BY t.total_amount DESC;
```

## 💡 优化原理

1. **复合索引**：`(status, created_at)` 让 WHERE 条件走索引范围扫描，附带 `user_id, amount` 实现索引覆盖，避免回表
2. **子查询先聚合**：先在 orders 表完成聚合和分页，再 JOIN users 表只查 20 行，避免大表 JOIN 后再排序
3. **去掉 LEFT JOIN**：WHERE 条件已经过滤了 NULL 行，改为 INNER JOIN / 子查询语义更清晰
4. **GROUP BY 精简**：子查询中只按 `user_id` 分组

## 📊 预期提升

| 指标 | 优化前 | 优化后 |
|------|--------|--------|
| 执行时间 | 4.7s | ~0.2s |
| 扫描行数 | ~2000万 | ~50万（索引范围扫描）|
| 临时表 | 大临时表排序 | 小结果集排序 |
```
