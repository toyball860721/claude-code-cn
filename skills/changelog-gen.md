# 📋 CHANGELOG 生成 (Changelog Gen)

## 描述

根据 Git 提交历史或版本变更描述，自动生成符合 [Keep a Changelog](https://keepachangelog.com/zh-CN/) 规范的中文 CHANGELOG.md。支持 Conventional Commits 解析、版本分组、变更分类，输出可直接使用的发版日志。

## 使用场景

- 版本发布前生成 CHANGELOG
- 从 Git log 批量生成历史 CHANGELOG
- PR 合并后自动追加变更记录
- 将英文 commit 转为中文变更日志
- 定期整理项目变更记录

## Prompt 模板

```
请根据以下信息生成 CHANGELOG。

版本信息：
- **新版本号**：{version}
- **发布日期**：{date}
- **上个版本号**：{previous_version}

Git 提交记录：
```
{git_log}
```

要求：
1. 遵循 Keep a Changelog 格式
2. 变更分类：新增(Added)、变更(Changed)、废弃(Deprecated)、移除(Removed)、修复(Fixed)、安全(Security)
3. 中文描述，技术术语保留英文
4. 每条变更一句话说清楚，附关联的 Issue/PR 编号
5. 合并同类变更，去除无意义的 commit（如 "fix typo"、"merge branch"）
6. 按重要程度排序
```

## 示例输入

```
新版本号：v2.3.0
发布日期：2025-06-15
上个版本号：v2.2.0

Git log：
a1b2c3d feat(auth): add OAuth2 PKCE flow support (#234)
d4e5f6a feat(api): add batch export endpoint (#228)
b7c8d9e fix(db): connection pool exhaustion under high concurrency (#231)
f0a1b2c fix(upload): file size validation bypass via chunked encoding (#229)
e3d4c5b refactor(middleware): replace custom logger with structlog
a6b7c8d chore(deps): bump fastapi from 0.104 to 0.109
f9e0d1c feat(ui): dark mode support (#225)
b2c3d4e fix(search): incorrect pagination when filter applied (#230)
a5b6c7d docs: update API reference for v2.3
e8f9a0b test: add integration tests for OAuth2 flow
c1d2e3f perf(query): optimize user search with trigram index (#232)
d4e5f6g deprecated(api): mark /v1/export endpoint for removal
f7a8b9c fix: typo in error message
a0b1c2d Merge branch 'develop' into main
```

## 示例输出

```markdown
# Changelog

本文档记录项目的所有重要变更，格式遵循 [Keep a Changelog](https://keepachangelog.com/zh-CN/)。

## [2.3.0] - 2025-06-15

### 新增 (Added)
- 支持 OAuth2 PKCE 授权流程，提升公开客户端的安全性 (#234)
- 新增批量导出 API 接口 `/v2/export/batch` (#228)
- 界面支持深色模式 (#225)

### 变更 (Changed)
- 用户搜索改用 trigram 索引，搜索速度提升约 5 倍 (#232)
- 中间件日志从自定义实现迁移到 structlog，结构化日志输出
- FastAPI 升级至 0.109，支持 Pydantic v2 特性

### 废弃 (Deprecated)
- `/v1/export` 接口标记为废弃，将在 v3.0 移除，请迁移至 `/v2/export/batch`

### 修复 (Fixed)
- 修复高并发场景下数据库连接池耗尽的问题 (#231)
- 修复通过 chunked encoding 绕过文件大小校验的安全漏洞 (#229)
- 修复搜索结果应用过滤器后分页数据错误的问题 (#230)

---

### 对比链接
[2.3.0]: https://github.com/org/repo/compare/v2.2.0...v2.3.0
```
