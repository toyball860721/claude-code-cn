# Claude Code 中文速查表 🚀

> A4 可打印 | 版本 1.0 | 2025年6月

---

## ⌨️ 快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+C` | 中断当前操作 |
| `Ctrl+D` | 退出 Claude Code |
| `Tab` | 接受自动补全建议 |
| `Esc` | 取消当前输入 |
| `↑` / `↓` | 浏览历史命令 |
| `Ctrl+L` | 清屏 |
| `Ctrl+R` | 搜索历史命令 |

---

## 🔧 常用命令

### 启动与配置

```bash
claude                          # 启动交互模式
claude "你的问题"                # 单次提问模式
claude -p "问题" --output-format json  # 管道模式，JSON输出
claude config                   # 打开配置
claude config set theme dark    # 设置暗色主题
claude update                   # 更新到最新版
```

### 交互模式内置命令

```
/help                  # 查看帮助
/clear                 # 清除对话历史
/compact               # 压缩上下文（长对话必备）
/cost                  # 查看当前会话费用
/doctor                # 诊断环境问题
/init                  # 初始化项目配置（生成 CLAUDE.md）
/model                 # 切换模型
/permissions           # 查看/修改权限
/status                # 查看当前状态
/vim                   # 切换 Vim 编辑模式
```

### 文件操作

```
读取文件：直接说 "读一下 src/main.py"
编辑文件：直接说 "把第15行的变量名改成 userName"
创建文件：直接说 "创建一个 utils.py 文件"
搜索代码：直接说 "在项目里搜索所有用到 getUserById 的地方"
```

---

## 🚩 常用 Flags

| Flag | 说明 | 示例 |
|------|------|------|
| `-p` / `--print` | 非交互模式，输出后退出 | `claude -p "解释这段代码"` |
| `--output-format` | 输出格式：text/json/stream-json | `claude -p "..." --output-format json` |
| `--model` | 指定模型 | `claude --model sonnet` |
| `--max-turns` | 限制对话轮次 | `claude -p "..." --max-turns 3` |
| `--permission-mode` | 权限模式 | `claude --permission-mode plan` |
| `--verbose` | 详细日志 | `claude --verbose` |
| `--dangerously-skip-permissions` | 跳过所有权限检查（慎用） | 仅限CI/脚本环境 |
| `--add-dir` | 添加额外工作目录 | `claude --add-dir ../shared` |

---

## 📚 Skills 列表

### 核心 Skills（9个）

| Skill | 用途 | 文件 |
|-------|------|------|
| 🔍 中文代码审查 | PR 审查、代码质量 | `code-review-cn.md` |
| 📖 API 文档生成 | 从代码生成中文 API 文档 | `api-doc-gen.md` |
| 🧪 自动写单元测试 | 生成 pytest/Jest 测试 | `test-writer.md` |
| 🔧 重构建议 | 代码坏味道分析+重构 | `refactor-advisor.md` |
| 🐛 Bug 定位 | 错误堆栈分析+修复 | `bug-hunter.md` |
| ⚡ SQL 优化 | 慢查询优化+索引建议 | `sql-optimizer.md` |
| 🐳 Docker 配置 | Dockerfile+Compose 生成 | `docker-helper.md` |
| 📝 README 生成 | 中文 README 自动生成 | `readme-writer.md` |
| 📋 CHANGELOG 生成 | 版本变更日志 | `changelog-gen.md` |
| ✍️ Git 提交信息 | 中文 Conventional Commits | `git-commit-cn.md` |

### 扩展 Skills（20个）

环境变量管理 · i18n 国际化 · 正则生成器 · 数据库迁移 · Git Hooks · TypeScript 类型生成 · 架构图生成 · 安全审计 · CI/CD 配置 · Lint 配置 · 错误处理模板 · API Mock · Monorepo 配置 · 提交信息润色 · 复杂度分析 · 数据格式转换 · 项目脚手架 · 性能分析 · Nginx 配置 · 代码注释生成

---

## 🔄 Workflows（10个）

| 工作流 | 说明 |
|--------|------|
| `pr-review.json` | PR 自动审查 → 评论 |
| `release.json` | 版本发布全流程 |
| `refactor.json` | 安全重构：测试→改→验 |
| `bug-fix.json` | 从 Issue 到修复 PR |
| `api-doc-sync.json` | API 文档自动同步 |
| `test-coverage.json` | 测试覆盖率补全 |
| `project-init.json` | 新项目一键初始化 |
| `db-optimize.json` | 慢查询批量优化 |
| `security-audit.json` | 全面安全审计 |
| `code-migration.json` | 技术栈迁移辅助 |

---

## 🐞 调试技巧

### 上下文溢出

```
/compact               # 压缩对话历史，释放上下文空间
/clear                 # 彻底清除，重新开始
```

提示：长任务每 10-15 轮对话用一次 `/compact`

### 回答不准确

```
# 提供更多上下文
"请先阅读 src/models/ 目录下所有文件，然后回答..."

# 限定范围
"只看 user.py 的第 50-100 行"

# 要求引用来源
"请引用具体代码行号"
```

### 权限问题

```
/permissions                    # 查看当前权限
claude --permission-mode plan   # 只规划不执行
# .claude/settings.json 中配置允许的命令
```

### 费用控制

```
/cost                          # 查看当前花费
claude -p "..." --max-turns 3  # 限制轮次控制费用
```

### CLAUDE.md 最佳实践

```markdown
# CLAUDE.md — 放在项目根目录

## 项目约定
- 使用 pnpm，不用 npm/yarn
- 测试框架：vitest
- 代码风格：遵循 .eslintrc

## 常用命令
- 运行测试：pnpm test
- 构建：pnpm build
- Lint：pnpm lint

## 架构说明
- src/api/ → API 路由
- src/services/ → 业务逻辑
- src/models/ → 数据模型
```

---

## 💡 高效提示词模板

```
# 让 Claude 像高级工程师一样思考
"作为资深 {language} 工程师，请审查这段代码..."

# 分步骤执行复杂任务
"请按以下步骤操作：1. 先读取... 2. 然后分析... 3. 最后修改..."

# 限定输出格式
"用表格对比方案 A 和方案 B 的优劣"

# 批量操作
"对 src/ 下所有 .py 文件执行：添加类型注解"
```

---

<div align="center">

**Claude Code 中文实战包** · 更多内容见 `ebook/` 目录

`v1.0 · 2025`

</div>
