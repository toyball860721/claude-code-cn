# 🔍 中文代码审查 (Code Review CN)

## 描述

用中文对代码进行全面审查，涵盖代码质量、安全漏洞、性能瓶颈、可读性和最佳实践。输出结构化的审查报告，按严重程度分级，附带修改建议。

## 使用场景

- PR 提交前的自查
- 团队 Code Review 辅助
- 遗留代码质量评估
- 新人代码指导
- 安全审计前的预检

## Prompt 模板

```
请对以下代码进行全面审查，用中文输出结构化报告。

审查维度：
1. **严重问题**（Bug、安全漏洞、数据丢失风险）
2. **性能问题**（时间/空间复杂度、N+1查询、内存泄漏）
3. **代码规范**（命名、格式、注释、函数长度）
4. **架构建议**（设计模式、职责分离、可扩展性）
5. **测试覆盖**（缺失的测试场景、边界条件）

对每个问题：
- 指出具体行号
- 说明问题原因
- 给出修改后的代码

代码如下：
```{language}
{code}
```

项目上下文：{context}
```

## 示例输入

```python
def get_user_data(user_id):
    conn = sqlite3.connect('app.db')
    cursor = conn.cursor()
    result = cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
    data = result.fetchone()
    return {"name": data[0], "email": data[1], "password": data[2]}
```

## 示例输出

```markdown
# 代码审查报告

## 🔴 严重问题（2个）

### 1. SQL 注入漏洞 [第4行]
**问题**：使用 f-string 拼接 SQL 语句，攻击者可注入任意 SQL。
**修复**：使用参数化查询。
```python
result = cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))
```

### 2. 密码泄露 [第6行]
**问题**：返回值包含 password 字段，可能通过 API 暴露用户密码。
**修复**：排除敏感字段，或只查询需要的列。

## 🟡 性能问题（1个）

### 3. 数据库连接未关闭 [第2行]
**问题**：每次调用都创建新连接且未关闭，会导致连接池耗尽。
**修复**：使用 `with` 语句或连接池。
```python
with sqlite3.connect('app.db') as conn:
    cursor = conn.cursor()
    ...
```

## 🔵 代码规范（1个）

### 4. 使用索引访问元组 [第5-6行]
**问题**：`data[0]`、`data[1]` 可读性差，后续字段变更容易出错。
**修复**：使用 `Row` 工厂或 `namedtuple`。

## 重构后完整代码

```python
def get_user_data(user_id: int) -> dict | None:
    with sqlite3.connect('app.db') as conn:
        conn.row_factory = sqlite3.Row
        cursor = conn.cursor()
        cursor.execute("SELECT name, email FROM users WHERE id = ?", (user_id,))
        row = cursor.fetchone()
        if row is None:
            return None
        return {"name": row["name"], "email": row["email"]}
```
```
