# 📖 API 文档生成 (API Doc Gen)

## 描述

根据代码自动生成中文 API 文档，支持 REST API、GraphQL、Python/TypeScript 函数库等。输出符合 OpenAPI 规范或自定义 Markdown 格式的完整文档，包含参数说明、返回值、错误码和调用示例。

## 使用场景

- 新项目快速生成 API 文档
- 已有代码补充缺失文档
- 从代码生成 OpenAPI/Swagger YAML
- 为内部 SDK 生成中文使用手册
- 接口变更后批量更新文档

## Prompt 模板

```
请根据以下代码生成完整的中文 API 文档。

输出格式：{format}  （可选：Markdown / OpenAPI YAML / 表格）

对每个接口/函数，包含：
1. **接口路径/函数签名**
2. **功能描述**（一句话 + 详细说明）
3. **请求参数**（名称、类型、是否必填、说明、默认值）
4. **返回值**（结构、字段说明）
5. **错误码/异常**（状态码、错误信息、触发条件）
6. **调用示例**（curl/代码示例）
7. **注意事项**

代码如下：
```{language}
{code}
```
```

## 示例输入

```python
@app.route('/api/v1/projects', methods=['POST'])
@login_required
def create_project():
    data = request.get_json()
    name = data.get('name')
    desc = data.get('description', '')
    if not name:
        return jsonify({"error": "项目名称不能为空"}), 400
    if len(name) > 100:
        return jsonify({"error": "项目名称不能超过100字符"}), 400
    project = Project(name=name, description=desc, owner_id=current_user.id)
    db.session.add(project)
    db.session.commit()
    return jsonify(project.to_dict()), 201
```

## 示例输出

```markdown
## 创建项目

`POST /api/v1/projects`

创建一个新项目。需要登录认证。

### 请求头

| 名称 | 必填 | 说明 |
|------|------|------|
| Authorization | 是 | Bearer Token |
| Content-Type | 是 | application/json |

### 请求参数（JSON Body）

| 参数 | 类型 | 必填 | 说明 | 默认值 |
|------|------|------|------|--------|
| name | string | 是 | 项目名称，最长100字符 | — |
| description | string | 否 | 项目描述 | 空字符串 |

### 返回值

**成功（201 Created）**
```json
{
  "id": 42,
  "name": "我的项目",
  "description": "这是一个示例项目",
  "owner_id": 1,
  "created_at": "2025-01-15T10:30:00Z"
}
```

### 错误码

| 状态码 | 错误信息 | 说明 |
|--------|----------|------|
| 400 | 项目名称不能为空 | 未提供 name 参数 |
| 400 | 项目名称不能超过100字符 | name 超过长度限制 |
| 401 | Unauthorized | 未登录或 Token 过期 |

### 调用示例

```bash
curl -X POST https://api.example.com/api/v1/projects \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "新项目", "description": "项目描述"}'
```
```
