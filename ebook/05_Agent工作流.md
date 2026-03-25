# 第 5 章：Agent 工作流 — 从单任务到多 Agent 协作

## 超越单次对话

前面几章讲的都是"你给 Claude Code 一个指令，它执行一个任务"的模式。这种模式处理日常开发任务绰绰有余，但当你面对复杂的工程任务时——比如"从零搭建一个完整的微服务"或者"对整个代码库做安全审计"——单次对话就不够用了。

这时候你需要的是 **Agent 工作流（Agent Workflow）**：把一个大任务分解成多个步骤，每个步骤由一个专门的 Agent 负责，Agent 之间通过明确的输入输出接口协作。

## 工作流的基本概念

### 什么是 Agent 工作流？

一个 Agent 工作流就是一个有序的任务链：

```
[分析需求] → [设计方案] → [编写代码] → [编写测试] → [代码审查] → [生成文档]
```

每个节点都是一个 Agent 调用，上一步的输出自动成为下一步的输入。

### 为什么不直接一个 Agent 做完？

三个原因：

1. **上下文限制**：复杂任务的上下文可能超过单次 Session 的窗口限制
2. **专注度**：每个 Agent 只关注一件事，输出质量更高
3. **可重试性**：某一步失败了，只需重试那一步，不用从头开始

## 工作流的实现方式

### 方式一：Shell 脚本串联

最简单的方式——用 bash 脚本把多个 `claude --print` 调用串起来：

```bash
#!/bin/bash
# workflow: 全栈功能开发

FEATURE="用户认证模块"

# Step 1: 需求分析
claude --print "分析我们的项目结构，为'${FEATURE}'写一份技术方案，保存到 /tmp/plan.md" \
  --permission-mode bypassPermissions

# Step 2: 后端实现
claude --print "根据 /tmp/plan.md 的方案，实现后端部分。完成后运行测试。" \
  --permission-mode bypassPermissions

# Step 3: 前端实现
claude --print "根据 /tmp/plan.md 的方案，实现前端部分。确保和后端 API 对接正确。" \
  --permission-mode bypassPermissions

# Step 4: 测试补充
claude --print "检查测试覆盖率，为覆盖率不足的部分补充测试。目标覆盖率 80%。" \
  --permission-mode bypassPermissions

# Step 5: 文档生成
claude --print "为这次新增的功能生成文档，更新 README 和 API 文档。" \
  --permission-mode bypassPermissions

echo "✅ 工作流完成"
```

### 方式二：OpenClaw 子 Agent 模式

如果你使用 OpenClaw，可以利用它的子 Agent 系统实现更强大的工作流：

```bash
# 主 Agent 可以 spawn 子 Agent 来并行处理任务
# 子 Agent 完成后自动向主 Agent 报告结果
```

OpenClaw 的优势在于：
- 子 Agent 之间可以并行执行
- 自动管理 Agent 的生命周期
- 内置结果汇总和错误处理

### 方式三：JSON 工作流模板

本产品的 `workflows/` 目录提供了 10 个 JSON 格式的工作流模板。格式如下：

```json
{
  "name": "全栈功能开发",
  "description": "从需求到上线的完整流程",
  "steps": [
    {
      "id": "analyze",
      "name": "需求分析",
      "prompt": "分析项目结构，为{{feature}}写技术方案",
      "output": "/tmp/plan.md"
    },
    {
      "id": "implement",
      "name": "代码实现",
      "depends_on": ["analyze"],
      "prompt": "根据{{analyze.output}}实现代码"
    }
  ]
}
```

你可以用一个简单的 runner 脚本来执行这些模板（runner 脚本包含在产品中）。

## 十大工作流场景

### 1. 全栈功能开发
分析需求 → 设计 API → 实现后端 → 实现前端 → 集成测试 → 更新文档

### 2. 代码审查自动化
获取 PR diff → 安全检查 → 性能分析 → 代码风格检查 → 生成审查报告

### 3. Bug 修复流水线
复现 Bug → 定位根因 → 修复代码 → 回归测试 → 更新 CHANGELOG

### 4. 项目初始化
选择技术栈 → 生成项目结构 → 配置 CI/CD → 创建 README → 初始化 Git

### 5. 数据库迁移
分析 Schema 变更 → 生成迁移文件 → 验证迁移 → 更新 ORM 模型 → 更新测试数据

### 6. API 开发全流程
设计接口 → 生成路由 → 实现逻辑 → 写测试 → 生成文档 → Mock 数据

### 7. 性能优化
Profiling → 识别瓶颈 → 优化代码 → 基准测试对比 → 生成优化报告

### 8. 安全审计
依赖扫描 → 代码扫描 → 配置检查 → 渗透测试建议 → 生成安全报告

### 9. 文档生成
扫描代码 → 提取注释 → 生成 API 文档 → 生成 README → 生成 CHANGELOG

### 10. CI/CD 集成
分析项目类型 → 生成 Dockerfile → 生成 CI 配置 → 生成部署脚本 → 验证流水线

## 工作流设计原则

### 原则 1：单一职责
每个 Agent 步骤只做一件事。"分析并实现"应该拆成"分析"和"实现"两步。

### 原则 2：明确边界
每一步的输入和输出要明确定义。用文件作为中间产物（如 `/tmp/plan.md`），而不是依赖上下文传递。

### 原则 3：可重试
任何一步失败了，应该能单独重试而不影响其他步骤。这要求步骤之间是松耦合的。

### 原则 4：人在回路
关键步骤要设置检查点，等人工确认后再继续。例如技术方案确认、代码审查通过、数据库迁移批准。

### 原则 5：渐进式自动化
不要一开始就追求全自动。先手动跑几次工作流，确认每一步的质量后，再逐步自动化。

## 实战：构建你的第一个工作流

让我们用一个真实场景演示：**为现有项目添加 Docker 支持**。

```bash
#!/bin/bash
# workflow: Docker 化

echo "🐳 Step 1: 分析项目"
claude --print "分析项目的技术栈、依赖和运行方式，输出到 /tmp/docker-analysis.md"

echo "🐳 Step 2: 生成 Dockerfile"
claude --print "根据 /tmp/docker-analysis.md，生成最优的 Dockerfile（多阶段构建、最小镜像）"

echo "🐳 Step 3: 生成 docker-compose.yml"
claude --print "生成 docker-compose.yml，包含应用、数据库和 Redis 服务"

echo "🐳 Step 4: 生成 .dockerignore"
claude --print "生成 .dockerignore 文件，排除不需要的文件"

echo "🐳 Step 5: 验证"
claude --print "运行 docker build 和 docker-compose up 验证配置是否正确"

echo "✅ Docker 化完成！"
```

## 小结

Agent 工作流是 Claude Code 从"工具"进化为"协作者"的关键。通过工作流：

- 复杂任务被分解为可管理的步骤
- 每个步骤都有明确的质量标准
- 失败可以精确定位和重试
- 人机协作有了清晰的边界

下一章，我们聚焦中国开发者最关心的话题——国内网络环境下的最佳实践。
