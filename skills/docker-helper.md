# 🐳 Dockerfile 生成 (Docker Helper)

## 描述

根据项目类型和需求自动生成优化的 Dockerfile、docker-compose.yml 和 .dockerignore。支持多阶段构建、镜像瘦身、安全加固、国内镜像源配置，输出生产可用的容器化配置。

## 使用场景

- 新项目容器化
- 优化现有 Dockerfile（减小镜像体积）
- 生成 docker-compose 多服务编排
- 配置国内镜像加速（适合中国大陆开发环境）
- CI/CD 流水线中的构建优化
- 从开发环境到生产环境的 Docker 配置

## Prompt 模板

```
请为以下项目生成 Docker 配置文件。

项目信息：
- **语言/框架**：{tech_stack}（如 Python/FastAPI、Node.js/Next.js、Go/Gin）
- **项目结构**：
```
{project_tree}
```
- **依赖管理**：{package_manager}（如 pip/poetry、npm/pnpm、go mod）
- **环境**：{environment}（开发 / 生产 / 两者都要）
- **额外服务**：{services}（如 PostgreSQL、Redis、Nginx）
- **特殊需求**：{requirements}（如 国内镜像源、GPU 支持、特定端口）

请生成：
1. `Dockerfile`（多阶段构建，生产优化）
2. `docker-compose.yml`（如需要多服务）
3. `.dockerignore`
4. 关键配置说明
```

## 示例输入

```
语言/框架：Python 3.12 / FastAPI
依赖管理：poetry
环境：生产
额外服务：PostgreSQL 16, Redis 7
特殊需求：国内镜像源加速，非 root 用户运行
```

## 示例输出

```dockerfile
# ===== Dockerfile =====

# 阶段1：依赖安装
FROM python:3.12-slim AS builder

# 国内镜像源
RUN pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/ \
    && pip config set install.trusted-host mirrors.aliyun.com

RUN pip install poetry==1.8.3 \
    && poetry config virtualenvs.in-project true

WORKDIR /app
COPY pyproject.toml poetry.lock ./
RUN poetry install --only main --no-interaction --no-ansi

# 阶段2：生产镜像
FROM python:3.12-slim AS runtime

# 安全：创建非 root 用户
RUN groupadd -r appuser && useradd -r -g appuser -d /app appuser

WORKDIR /app

# 只复制虚拟环境和源代码
COPY --from=builder /app/.venv ./.venv
COPY ./src ./src

# 环境变量
ENV PATH="/app/.venv/bin:$PATH" \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

USER appuser
EXPOSE 8000

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')"

CMD ["uvicorn", "src.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```yaml
# ===== docker-compose.yml =====

version: "3.9"

services:
  app:
    build: .
    ports:
      - "8000:8000"
    env_file: .env
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    volumes:
      - pg_data:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: ${DB_NAME:-myapp}
      POSTGRES_USER: ${DB_USER:-postgres}
      POSTGRES_PASSWORD: ${DB_PASS:?请设置数据库密码}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASS:-changeme}
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

volumes:
  pg_data:
  redis_data:
```

```gitignore
# ===== .dockerignore =====
.git
.venv
__pycache__
*.pyc
.env
.mypy_cache
.pytest_cache
tests/
docs/
*.md
docker-compose*.yml
```
