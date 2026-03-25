# 🐛 Bug 定位 (Bug Hunter)

## 描述

根据错误信息、堆栈追踪或异常行为描述，定位 Bug 根因并给出修复方案。支持分析运行时错误、逻辑错误、并发问题、内存泄漏等，输出完整的排查路径和修复代码。

## 使用场景

- 线上报错的快速定位
- 难以复现的间歇性 Bug 分析
- 堆栈追踪的解读和根因分析
- 性能劣化原因排查
- 数据不一致问题追踪

## Prompt 模板

```
请帮我定位以下 Bug 并给出修复方案。

信息：
- **错误信息/现象**：{error_description}
- **错误堆栈**（如有）：
```
{stack_trace}
```
- **相关代码**：
```{language}
{code}
```
- **复现条件**：{reproduction_steps}
- **预期行为**：{expected}
- **实际行为**：{actual}

请输出：
1. **根因分析**：为什么会发生这个错误
2. **排查路径**：你是如何定位到根因的（逻辑链）
3. **修复方案**：具体的代码修改
4. **验证方法**：如何确认 Bug 已修复
5. **防御措施**：如何避免类似问题再次发生
```

## 示例输入

```
错误信息：生产环境偶发 KeyError: 'user_role'，大约每天出现 3-5 次

堆栈：
Traceback (most recent call last):
  File "middleware.py", line 45, in check_permission
    role = session['user_role']
KeyError: 'user_role'

相关代码：
```python
# auth.py
def login(username, password):
    user = db.get_user(username)
    if user and check_password(password, user.password_hash):
        session['user_id'] = user.id
        session['user_name'] = user.name
        session['user_role'] = user.role
        return True
    return False

# middleware.py
def check_permission(required_role):
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            role = session['user_role']
            if role_level(role) < role_level(required_role):
                abort(403)
            return f(*args, **kwargs)
        return wrapper
    return decorator
```

复现条件：无法稳定复现，仅在生产环境偶发
预期行为：中间件正常读取 user_role 并进行权限检查
实际行为：偶发 KeyError
```

## 示例输出

```markdown
# Bug 分析报告

## 🎯 根因分析

**根因**：Session 过期或被清除后，用户的请求仍携带旧的 Session Cookie，此时 `session` 字典中不再包含 `user_role` 键。

具体场景：
1. 用户登录成功，session 写入 `user_id`、`user_name`、`user_role`
2. Session 过期（服务器端 TTL 到期 / Redis 淘汰 / 服务重启）
3. 用户浏览器仍持有旧 Cookie，发起请求
4. `session` 是空字典或部分恢复的字典，不含 `user_role`
5. `session['user_role']` → KeyError

**为什么偶发**：只在 Session 过期到浏览器下次被重定向到登录页之间的窗口期触发。

## 🔍 排查路径

1. `KeyError: 'user_role'` → `session` 中缺少该 key
2. 登录函数确实设置了该 key → 不是写入遗漏
3. 偶发 + 生产环境 → 时间相关问题 → Session 生命周期
4. 检查 Session 存储配置 → 确认是否有 TTL
5. 结论：Session 过期后未做防御性检查

## ✅ 修复方案

```python
# middleware.py — 修复后
def check_permission(required_role):
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            role = session.get('user_role')
            if role is None:
                # Session 已过期，重定向到登录页
                return redirect(url_for('auth.login', next=request.url))
            if role_level(role) < role_level(required_role):
                abort(403)
            return f(*args, **kwargs)
        return wrapper
    return decorator
```

## 🧪 验证方法

1. 手动清除服务端 Session 存储，保留浏览器 Cookie，访问受保护页面 → 应跳转登录
2. 添加测试用例模拟空 session
3. 部署后监控该 KeyError 是否归零

## 🛡️ 防御措施

- **规则**：永远用 `session.get()` 而非 `session[]`
- **全局中间件**：在所有权限检查前统一验证 session 完整性
- **监控**：对 KeyError 添加 Sentry 告警，附带 session 内容快照
```
