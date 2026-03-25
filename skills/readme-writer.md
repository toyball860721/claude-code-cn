# 📝 README 生成 (README Writer)

## 描述

根据项目代码和结构自动生成高质量的中文 README.md，包含项目简介、功能特性、快速开始、安装说明、使用示例、API 概览、贡献指南和许可证等完整章节。支持生成徽章（badges）和目录结构图。

## 使用场景

- 新开源项目创建 README
- 已有项目的 README 重写或完善
- 从英文 README 生成中文版本
- 内部项目补充文档
- 项目发布前的文档检查

## Prompt 模板

```
请根据以下信息生成一份完整的中文 README.md。

项目信息：
- **项目名称**：{name}
- **一句话描述**：{tagline}
- **技术栈**：{tech_stack}
- **目标用户**：{target_audience}
- **项目结构**（如有）：
```
{project_tree}
```
- **主要功能**（简要列出）：{features}
- **安装方式**：{install_method}
- **开源协议**：{license}

README 要求：
1. 开头有项目 Logo 占位和徽章
2. 目录（TOC）
3. 功能特性（带 emoji）
4. 快速开始（3步以内跑起来）
5. 详细安装说明
6. 使用示例（代码块）
7. 配置说明
8. 项目结构
9. 贡献指南
10. 许可证

风格：简洁专业，中文为主，技术术语保留英文。
```

## 示例输入

```
项目名称：FastCache
一句话描述：轻量级 Python 缓存库，支持 TTL、LRU 和分布式模式
技术栈：Python 3.10+, Redis (可选)
目标用户：Python 后端开发者
主要功能：内存缓存、Redis 缓存、TTL 过期、LRU 淘汰、装饰器 API、异步支持
安装方式：pip install fastcache-py
开源协议：MIT
```

## 示例输出

````markdown
<div align="center">

<!-- Logo 占位 -->
# 🚀 FastCache

**轻量级 Python 缓存库，支持 TTL、LRU 和分布式模式**

[![PyPI version](https://badge.fury.io/py/fastcache-py.svg)](https://pypi.org/project/fastcache-py/)
[![Python](https://img.shields.io/pypi/pyversions/fastcache-py.svg)](https://pypi.org/project/fastcache-py/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Tests](https://github.com/yourname/fastcache/actions/workflows/test.yml/badge.svg)](https://github.com/yourname/fastcache/actions)

[English](./README_EN.md) | 简体中文

</div>

---

## 📋 目录

- [功能特性](#-功能特性)
- [快速开始](#-快速开始)
- [安装](#-安装)
- [使用示例](#-使用示例)
- [配置](#-配置)
- [项目结构](#-项目结构)
- [贡献指南](#-贡献指南)
- [许可证](#-许可证)

## ✨ 功能特性

- 🏎️ **极速内存缓存** — 纯 Python 实现，零外部依赖
- ⏰ **TTL 过期** — 支持按 key 设置过期时间
- 📦 **LRU 淘汰** — 内存满时自动淘汰最久未使用的数据
- 🌐 **Redis 分布式** — 可选 Redis 后端，多实例共享缓存
- 🎯 **装饰器 API** — 一行代码缓存函数结果
- ⚡ **异步支持** — 原生 async/await 支持

## 🚀 快速开始

```bash
pip install fastcache-py
```

```python
from fastcache import cache

@cache(ttl=60)
def get_user(user_id: int):
    return db.query(user_id)  # 60秒内重复调用走缓存
```

就是这么简单。

## 📦 安装

### 基础安装（仅内存缓存）

```bash
pip install fastcache-py
```

### 完整安装（含 Redis 支持）

```bash
pip install fastcache-py[redis]
```

### 开发环境

```bash
git clone https://github.com/yourname/fastcache.git
cd fastcache
pip install -e ".[dev]"
```

> 要求 Python >= 3.10

## 📖 使用示例

### 基础用法

```python
from fastcache import Cache

c = Cache(maxsize=1000, ttl=300)
c.set("key", "value")
print(c.get("key"))  # "value"
```

### 装饰器模式

```python
from fastcache import cache

@cache(ttl=120, maxsize=500)
def expensive_query(params):
    ...
```

### Redis 分布式模式

```python
from fastcache import RedisCache

c = RedisCache(url="redis://localhost:6379/0")
await c.set("shared_key", {"data": 42})
```

## ⚙️ 配置

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| maxsize | int | 1024 | 最大缓存条目数 |
| ttl | int | 0 | 过期时间（秒），0 表示不过期 |
| strategy | str | "lru" | 淘汰策略：lru / fifo / lfu |
| redis_url | str | None | Redis 连接地址 |

## 🗂 项目结构

```
fastcache/
├── __init__.py        # 公开 API
├── cache.py           # 核心缓存实现
├── decorators.py      # 装饰器
├── backends/
│   ├── memory.py      # 内存后端
│   └── redis.py       # Redis 后端
├── strategies/
│   ├── lru.py
│   └── lfu.py
└── utils.py
```

## 🤝 贡献指南

欢迎贡献！请阅读 [CONTRIBUTING.md](./CONTRIBUTING.md) 了解详情。

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/amazing`)
3. 提交更改 (`git commit -m 'feat: 添加某功能'`)
4. 推送分支 (`git push origin feature/amazing`)
5. 创建 Pull Request

## 📄 许可证

[MIT](./LICENSE) © 2025 Your Name
````
