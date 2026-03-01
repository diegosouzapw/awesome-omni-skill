---
name: project-index
description: Use this skill for large project maintenance with layered CLAUDE.md index system. Triggers when users need to (1) analyze and document existing codebases, (2) generate hierarchical CLAUDE.md files for modules, (3) set up incremental update hooks after code changes, or (4) navigate large projects efficiently. Supports legacy project onboarding and automatic context management.
version: 1.1.0
triggers:
  - 分析项目
  - 生成文档
  - 代码审计
  - 测试修复
  - 测试生成
  - 检查过期
  - project index
  - CLAUDE.md
  - AUDIT.md
dependencies:
  required:
    - codeagent-wrapper
  optional:
    - kanban
    - pi-cli  # 推荐使用 pi-cli 作为统一入口
ports:
  dashboard: 3008
  kanban: 3007
tags:
  - documentation
  - testing
  - audit
  - maintenance
---

# Project Index - Layered CLAUDE.md System

自动生成和维护大型项目的层次化 CLAUDE.md 索引系统。

## 📌 推荐使用 pi-cli

`project-index` 和 `pi-cli` 功能已统一。推荐使用 `pi` CLI 作为统一入口：

```bash
# 等效命令对照
node scripts/module-analyzer.js --stale  →  pi module analyze --llm --stale
node scripts/test-fix.js                 →  pi test fix --llm
node scripts/dashboard.js --open         →  pi ui
```

详见 [pi-cli SKILL.md](/home/wing/.claude/skills/pi-cli/SKILL.md)

---

## ⛔ 严禁操作 (CRITICAL - 必读)

> **2026-01-22 事故记录**：执行 `git checkout HEAD -- tests/` 和 `git checkout HEAD -- js/`，导致用户一整天的手动工作（100+ 文件）永久丢失，无法恢复。

**以下操作必须先询问用户确认，否则绝对禁止执行：**

1. `git checkout HEAD --` / `git checkout -- <path>` — 会永久丢弃未提交修改
2. `git reset --hard` — 会永久丢弃所有未提交修改
3. `git clean -fd` — 会永久删除未跟踪文件
4. `git stash drop` / `git stash clear` — 会永久删除 stash
5. 任何批量删除文件的操作 (`rm -rf`, `find -delete` 等)

**批量任务进度保护**：每完成一批任务后，必须提交或提醒用户提交。

---

## 功能清单

| 功能 | 脚本 | 说明 |
|------|------|------|
| **文档生成** | `generate.js` | 生成/更新 CLAUDE.md |
| **代码审计** | `code-audit.js` | 生成 AUDIT.md，检测安全问题 |
| **过期检测** | `check-stale.js` | 检测文档/审计/测试是否过期 |
| **依赖图构建** | `dependency-graph.js` | 构建文件级依赖图，检测循环依赖 |
| **影响分析** | `impact-analyzer.js` | 分析变更文件的下游影响范围 |
| **stale 传播** | `stale-propagate.js` | 依赖传播 stale 状态 + 测试重跑列表 |
| **测试优先级** | `test-prioritize.js` | 智能排序：root cause 优先 + 并行批次 |
| **智能测试** | `test-affected.js` | 只运行受变更影响的测试 |
| **测试映射** | `test-mapper.js` | 源码↔测试映射，生成 .test-map.json |
| **测试生成** | `test-generator.js` | 批量生成缺失测试 |
| **测试修复** | `test-fix.js` | 并发修复测试错误 |
| **审计修复** | `audit-fix.js` | 并发修复审计问题 |
| **验收门禁** | `acceptance-gate.js` | DAG 分组验证 git 变更，仅提交通过文件 |
| **Dashboard** | `dashboard.js` | Web UI (http://localhost:3008) |

## AI 交互指引

### 主动询问

完成批量任务后，应主动询问用户：

```
任务完成。是否打开 Dashboard 查看详细状态？
→ 运行: node scripts/dashboard.js --open
→ 访问: http://localhost:3008
```

### 任务前检查

执行文档/测试任务前，先检查状态：

```bash
node scripts/check-stale.js --stale-only --json
```

根据输出决定处理范围。

## 触发场景

1. **新项目入驻** — 用户说"帮我分析这个项目"、"生成文档"
2. **遗留项目理解** — 用户说"这个代码库怎么组织的"
3. **代码修改后** — 检测到核心模块变更，提醒更新文档
4. **测试维护** — 用户说"生成测试"、"修复测试"
5. **审计需求** — 用户说"检查安全问题"、"代码审计"
6. **提交前验收** — 用户说"验收门禁"、"检查变更"、"提交验证"

## 前置依赖

| 依赖 | 类型 | 用途 |
|------|------|------|
| **codeagent-wrapper** | 必需 | LLM 调用 |
| **kanban** | 可选 | 审计任务自动创建 |

## 核心概念

### 三层架构

```
project/CLAUDE.md           # Layer 1: 概览 + 模块索引
    ↓
src/modules/auth/CLAUDE.md  # Layer 2: 模块详情 + 子模块索引
    ↓
src/modules/auth/jwt/CLAUDE.md  # Layer 3: 实现细节
```

### 智能覆盖策略

不是每个目录都需要独立 CLAUDE.md：

- **大目录** (≥5 文件或 ≥200 行) → 必须有独立 CLAUDE.md
- **小目录** + 父目录有 CLAUDE.md → 由父目录覆盖
- **小目录** + 父目录无 CLAUDE.md → 孤儿，需关注

详见 [docs/coverage.md](docs/coverage.md)

### 层级依赖排序

批量生成时按目录深度从深到浅处理：

```
js/agents/core/sandbox/system  → 先生成
js/agents/core/sandbox         → 后生成
js/agents/core                 → 再后
js/agents                      → 最后
```

确保父目录生成时可引用子目录的 CLAUDE.md。

### AUDIT.md 同策略

代码审计 (`code-audit.js`) 采用相同策略：
- 大目录生成独立 AUDIT.md
- 小目录由父目录审计覆盖
- 审计提示词包含未独立审计的小子目录代码

## 快速开始

```bash
# 1. 扫描项目
node scripts/scan.js

# 2. 生成 CLAUDE.md
node scripts/generate.js --auto

# 3. 安装 hooks
node scripts/hook.js init

# 4. 启动 Dashboard
node scripts/dashboard.js --open
```

## 常用命令

详见 [docs/commands.md](docs/commands.md)

快速参考：

```bash
node scripts/module-analyzer.js          # 日常维护
node scripts/module-analyzer.js --all    # 全量处理
node scripts/check-stale.js --stale-only # 过期检测
node scripts/test-fix.js                 # 修复测试错误
node scripts/dashboard.js --open         # Web UI
```

## 工作流

### 新项目

1. `hook.js init` - 安装 hooks + 创建配置
2. `scan.js` - 分析结构
3. `generate.js --auto` - 生成文档
4. 开始开发

### 遗留项目

1. `scan.js` - 理解结构
2. `hook.js init` - 安装配置
3. `generate.js --auto` - 生成文档
4. `module-analyzer.js` - 初始审计
5. 按需调整

### 代码修改后

修改模块代码后，**必须**检查并更新对应的 `CLAUDE.md` 和 `AUDIT.md`：

1. **检查 CLAUDE.md**
   - 内容需要更新 → 修改文档内容
   - 内容仍然准确 → `touch CLAUDE.md` 更新时间戳

2. **检查 AUDIT.md**
   - 新增安全问题 → 补充到 Issues 列表
   - 问题已修复 → 使用 `audit-archive.js` 归档
   - 内容仍然准确 → `touch AUDIT.md` 更新时间戳

3. **验证状态**
   ```bash
   node scripts/check-stale.js <module-path> --stale-only
   node scripts/audit-status.js <module-path>
   ```

> **重要**：即使没有实质性改动，也必须 touch 文件以更新时间戳，否则 stale 检测会持续报告该模块过期。

## 详细文档

| 文档 | 内容 |
|------|------|
| [docs/commands.md](docs/commands.md) | 常用命令详解 |
| [docs/coverage.md](docs/coverage.md) | 智能覆盖率策略、批量任务发现 |
| [docs/dashboard.md](docs/dashboard.md) | Web UI 仪表盘详解 |
| [docs/scripts.md](docs/scripts.md) | 所有脚本参考 |
| [docs/config.md](docs/config.md) | 配置文件说明 |
| [docs/testing.md](docs/testing.md) | 测试工具链详解 |
| [docs/anti-patterns.md](docs/anti-patterns.md) | ⚠️ 禁止的错误模式 |
| [docs/dag-scheduling.md](docs/dag-scheduling.md) | DAG 依赖调度（目录层级处理） |
| [docs/batch-best-practices.md](docs/batch-best-practices.md) | 批量任务最佳实践 |

## 硬约束（MUST READ）

执行以下功能前，**必须**先读取对应的最佳实践文档：

| 功能 | 必读文档 | 说明 |
|------|----------|------|
| test-fix, test-generator | [docs/testing.md](docs/testing.md), [docs/anti-patterns.md](docs/anti-patterns.md) | 测试修复和生成的最佳实践与禁止模式 |
| audit-fix, code-audit | [docs/audit-best-practices.md](docs/audit-best-practices.md) | 审计修复流程、问题分类、归档规范 |
| module-analyzer --llm | [docs/batch-best-practices.md](docs/batch-best-practices.md) | 批量 LLM 任务的并发设置、checkpoint、rate limit 处理 |
| dependency-graph, impact-analyzer, stale-propagate, test-prioritize | [docs/dependency-best-practices.md](docs/dependency-best-practices.md) | 依赖分析、影响范围、stale 传播、测试优先级排序 |

**约束级别：**
- ✅ **MUST READ** - 执行功能前必须读取对应文档
- ✅ **MUST FOLLOW** - 必须遵循文档中的硬约束规则（MUST/MUST NOT）
- ⚠️ **违反后果** - 可能导致数据丢失、测试失败、审计问题遗漏

**AI 执行检查清单：**
1. 识别用户请求的功能类型
2. 检查是否有对应的必读文档
3. 读取并理解文档中的硬约束规则
4. 执行功能时严格遵循约束
5. 完成后验证是否符合最佳实践

## 配置

创建 `.stale-config.json`：

```json
{
  "include": ["js/agents/**"],
  "ignore": ["tests/**", "docs/**"],
  "features": { "doc": true, "audit": true, "kanban": true, "testAnalysis": true },
  "concurrency": 6
}
```

## 测试追踪

测试状态与 CLAUDE.md/AUDIT.md 统一追踪：

```bash
# 检查测试覆盖状态
node scripts/check-stale.js --type=test --stale-only

# 检查所有类型（doc + audit + test）
node scripts/check-stale.js --type=all --stale-only

# 刷新测试映射
node scripts/test-mapper.js

# 查看模块测试详情
node scripts/test-view.js js/agents/core
```

测试状态类型：
- **missing** - 模块无测试覆盖
- **stale** - 源码已修改但测试未更新
- **fresh** - 测试与源码同步

详见 [docs/testing.md](docs/testing.md)

## 与 Codex 集成

更新父目录时注入小子目录上下文：

```bash
context=$(node generate.js --prompt-context --module js/agents/ingest)

codex-wrapper - <<EOF
更新 js/agents/ingest/CLAUDE.md

$context
EOF
```

## Kanban API

审计任务自动创建到 Kanban：

```bash
export KANBAN_URL=http://127.0.0.1:3007/api/v1
```

未运行 Kanban 服务时静默跳过。

## 依赖分析系统

构建文件级依赖图，支持影响分析和 stale 传播。

### 依赖图构建

```bash
# 扫描 js/agents 构建依赖图
node scripts/dependency-graph.js --module js/agents

# 查询单文件依赖
node scripts/dependency-graph.js --check shared/index.js

# JSON 输出
node scripts/dependency-graph.js --json
```

输出文件：`.dep-graph.json`

### 影响分析

分析变更文件的下游影响范围：

```bash
# 分析指定文件
node scripts/impact-analyzer.js shared/utils/logger.js core/event-bus.js

# 分析 git 变更
node scripts/impact-analyzer.js --since HEAD~5
node scripts/impact-analyzer.js --staged

# JSON 输出
node scripts/impact-analyzer.js --since HEAD~3 --json
```

### Stale 传播

将 stale 状态沿依赖图向下游传播：

```bash
# 自动检测 + 传播
node scripts/stale-propagate.js

# 指定变更文件
node scripts/stale-propagate.js --changed core/event-bus.js

# 包含测试重跑列表
node scripts/stale-propagate.js --changed core/event-bus.js --tests

# 调整传播深度（默认 2）
node scripts/stale-propagate.js --depth 3
```

输出示例：
```
Direct stale: 2 files
Propagated stale: 45 files (L1: 12, L2: 33)
Tests to re-run: 40 files
```

### 典型工作流

```bash
# 1. 构建/更新依赖图
node scripts/dependency-graph.js --module js/agents

# 2. 代码修改后，分析影响
node scripts/impact-analyzer.js --staged

# 3. 检查 stale 传播
node scripts/stale-propagate.js --tests

# 4. 运行受影响的测试
node scripts/test-affected.js --staged
```

### 智能测试修复（100+ 错误场景）

当有大量测试失败时，智能排序修复顺序：

```bash
# 分析失败测试的优先级
node scripts/test-prioritize.js --from-file test-results.json

# 输出：
# Phase 1 - Root causes: shared/utils/logger.js (261 依赖)
# Phase 2 - Independent: 可并行修复的批次
# Phase 3 - Leaf nodes: 最后修复
```

修复策略：
1. **先修 root cause** — 被依赖最多的文件，一个修复解决多个错误
2. **并行修独立集** — 无依赖关系的文件可以 60 开并发
3. **最后修叶子节点** — 依赖链末端的文件

```bash
# 按优先级运行测试
node scripts/test-affected.js --staged --prioritized
```

## 验收门禁

提交前验证 git 变更，确保代码质量。

### 验证流程

1. **扫描变更** — 获取 git diff 中的 JS/TS 文件
2. **DAG 分组** — 按依赖图分组，相关文件一起验证
3. **并发验证** — 四维检查：lint、测试覆盖、功能测试、安全审计
4. **结果汇总** — 标记通过/失败文件
5. **选择性提交** — 仅提交通过验证的文件

### 使用方式

```bash
# 预览变更文件分组
node scripts/acceptance-gate.js --dry-run

# 运行验证
node scripts/acceptance-gate.js

# 验证并自动提交通过的文件
node scripts/acceptance-gate.js --commit

# 跳过部分检查
node scripts/acceptance-gate.js --skip-lint --skip-security

# 调整并发
node scripts/acceptance-gate.js --concurrency=8
```

### 安全约束

- 禁止 `git checkout`、`git reset` 等危险操作
- 仅提交通过验证的文件
- 失败文件只标记，不自动修改

## ⚠️ 禁止的错误模式

测试修复和代码生成时必须避免的反模式。

详见 [docs/anti-patterns.md](docs/anti-patterns.md)
