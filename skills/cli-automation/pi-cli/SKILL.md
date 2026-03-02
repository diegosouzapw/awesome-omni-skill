---
name: pi-cli
description: 统一项目分析 CLI 工具。支持 DAG 调度、LLM 批量任务、依赖图构建、测试分析/修复、文档生成、代码审计、Web Dashboard。与 project-index 功能完全一致，推荐作为统一入口。
version: 2.1.0
triggers:
  - pi init
  - pi deps
  - pi test
  - pi doc
  - pi audit
  - pi module
  - pi ui
  - pi task
  - pi stale
  - pi update
  - 分析项目
  - 生成文档
  - 代码审计
  - 测试修复
  - 测试生成
  - 检查过期
  - CLAUDE.md
  - AUDIT.md
dependencies:
  required: []
  optional:
    - codeagent-wrapper  # LLM 批量任务
    - kanban             # 审计任务自动创建
tags:
  - cli
  - analysis
  - testing
  - documentation
  - audit
  - llm
  - maintenance
ports:
  dashboard: 3008
  kanban: 3007
---

# pi-cli - 统一项目分析 CLI

统一的项目分析和维护工具，支持静态分析和 LLM 驱动的智能分析。与 project-index 功能完全一致。

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

## 核心能力

| 能力 | 说明 |
|------|------|
| **DAG 调度** | 依赖感知的并发执行，子目录先于父目录 |
| **LLM 批量** | codeagent-wrapper 集成，并发 + 重试 + checkpoint |
| **依赖图** | 文件级依赖分析、影响范围、stale 传播 |
| **测试分析** | 映射、优先级排序、受影响测试、LLM 修复 |
| **文档生成** | CLAUDE.md 生成（静态 + LLM 模式） |
| **代码审计** | AUDIT.md 生成、安全/质量问题检测 |
| **Dashboard** | Web UI，SSE 实时更新，任务管理，配置编辑 |

## 命令对照

| pi-cli 命令 | project-index 脚本 | 说明 |
|-------------|-------------------|------|
| `pi init` | `hook.js init` | 初始化配置 |
| `pi deps build` | `dependency-graph.js` | 构建依赖图 |
| `pi deps impact` | `impact-analyzer.js` | 影响分析 |
| `pi deps propagate` | `stale-propagate.js` | Stale 传播 |
| `pi test map` | `test-mapper.js` | 测试映射 |
| `pi test fix --llm` | `test-fix.js` | 测试修复 |
| `pi test prioritize` | `test-prioritize.js` | 优先级排序 |
| `pi test affected` | `test-affected.js` | 受影响测试 |
| `pi test generate` | `test-generator.js` | 生成测试 |
| `pi doc generate` | `generate.js` | 生成 CLAUDE.md |
| `pi doc check` | `check-stale.js` | 过期检测 |
| `pi audit scan` | `code-audit.js` | 代码审计 |
| `pi audit fix` | `audit-fix.js` | 审计修复 |
| `pi module analyze --llm` | `module-analyzer.js` | LLM 模块分析 |
| `pi ui` | `dashboard.js` | Web Dashboard |

## 快速开始

```bash
# 初始化配置
pi init

# 构建依赖图
pi deps build

# LLM 驱动的模块分析（生成 CLAUDE.md + AUDIT.md）
pi module analyze --llm --concurrency=20

# 测试分析和修复
pi test map
pi test fix --llm --concurrency=10

# 启动 Dashboard
pi ui --port=3008
```

## 命令

| 命令 | 子命令 | 说明 |
|------|--------|------|
| `init` | - | 初始化 .pi-config.json |
| `deps` | build, impact, propagate, query | 依赖图分析 |
| `test` | map, run, plan, fix, affected, prioritize, generate | 测试操作 |
| `doc` | generate, check, scan | 文档生成 |
| `audit` | scan, fix, status, archive | 代码审计 |
| `module` | analyze | LLM 模块分析 |
| `task` | list, start, cancel, types | 任务管理 |
| `stale` | notify, status | Stale 通知 |
| `update` | - | 增量更新 |
| `hook` | init, install, uninstall | Claude Code hooks |
| `ui` | - | Web Dashboard |

## AI 交互指引

### 主动询问

完成批量任务后，应主动询问用户：

```
任务完成。是否打开 Dashboard 查看详细状态？
→ 运行: pi ui
→ 访问: http://localhost:3008
```

### 任务前检查

执行文档/测试任务前，先检查状态：

```bash
pi doc check --stale-only --json
```

根据输出决定处理范围。

## 触发场景

1. **新项目入驻** — 用户说"帮我分析这个项目"、"生成文档"
2. **遗留项目理解** — 用户说"这个代码库怎么组织的"
3. **代码修改后** — 检测到核心模块变更，提醒更新文档
4. **测试维护** — 用户说"生成测试"、"修复测试"
5. **审计需求** — 用户说"检查安全问题"、"代码审计"

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

### 层级依赖排序 (DAG)

批量生成时按目录深度从深到浅处理：

```
js/agents/core/sandbox/system  → 先生成
js/agents/core/sandbox         → 后生成
js/agents/core                 → 再后
js/agents                      → 最后
```

确保父目录生成时可引用子目录的 CLAUDE.md。

## LLM 模式

通过 `--llm` 启用 LLM 驱动的智能分析：

```bash
# 模块分析（含 Kanban 集成）
KANBAN_URL=http://localhost:3007/api/v1 pi module analyze --llm

# 测试修复（安全约束：只改测试，不改实现）
pi test fix --llm --concurrency=20
```

## DAG 调度

任务带 `dependencies` 字段时自动启用 DAG 调度：

```javascript
const tasks = [
  { id: 'parent', dependencies: ['child1', 'child2'], prompt: '...' },
  { id: 'child1', prompt: '...' },
  { id: 'child2', prompt: '...' }
];
// child1, child2 并发执行，完成后 parent 才开始
```

## 依赖分析系统

构建文件级依赖图，支持影响分析和 stale 传播。

### 依赖图构建

```bash
# 构建依赖图
pi deps build

# 查询单文件依赖
pi deps query shared/index.js
```

输出文件：`.dep-graph.json`

### 影响分析

分析变更文件的下游影响范围：

```bash
# 分析指定文件
pi deps impact shared/utils/logger.js core/event-bus.js

# 分析 git 变更
pi deps impact --since HEAD~5
pi deps impact --staged
```

### Stale 传播

将 stale 状态沿依赖图向下游传播：

```bash
# 自动检测 + 传播
pi deps propagate

# 指定变更文件
pi deps propagate --changed core/event-bus.js

# 包含测试重跑列表
pi deps propagate --changed core/event-bus.js --tests

# 调整传播深度（默认 2）
pi deps propagate --depth 3
```

### 典型工作流

```bash
# 1. 构建/更新依赖图
pi deps build

# 2. 代码修改后，分析影响
pi deps impact --staged

# 3. 检查 stale 传播
pi deps propagate --tests

# 4. 运行受影响的测试
pi test affected --staged
```

## 智能测试修复（100+ 错误场景）

当有大量测试失败时，智能排序修复顺序：

```bash
# 分析失败测试的优先级
pi test prioritize --from-file test-results.json
```

修复策略：
1. **先修 root cause** — 被依赖最多的文件，一个修复解决多个错误
2. **并行修独立集** — 无依赖关系的文件可以 60 开并发
3. **最后修叶子节点** — 依赖链末端的文件

## 工作流

### 新项目

1. `pi init` - 初始化配置
2. `pi deps build` - 构建依赖图
3. `pi doc generate` - 生成文档
4. 开始开发

### 遗留项目

1. `pi init` - 初始化配置
2. `pi deps build` - 构建依赖图
3. `pi doc generate` - 生成文档
4. `pi module analyze --llm` - 初始审计
5. 按需调整

### 代码修改后

修改模块代码后，**必须**检查并更新对应的 `CLAUDE.md` 和 `AUDIT.md`：

1. **检查 CLAUDE.md**
   - 内容需要更新 → 修改文档内容
   - 内容仍然准确 → `touch CLAUDE.md` 更新时间戳

2. **检查 AUDIT.md**
   - 新增安全问题 → 补充到 Issues 列表
   - 问题已修复 → 使用 `pi audit archive` 归档
   - 内容仍然准确 → `touch AUDIT.md` 更新时间戳

3. **验证状态**
   ```bash
   pi doc check <module-path> --stale-only
   pi audit status <module-path>
   ```

> **重要**：即使没有实质性改动，也必须 touch 文件以更新时间戳，否则 stale 检测会持续报告该模块过期。

## 配置文件

### .pi-config.json

```json
{
  "name": "my-project",
  "language": "javascript",
  "src": {
    "dirs": ["src"],
    "pattern": "**/*.js",
    "ignore": ["node_modules", "dist"]
  },
  "test": {
    "dirs": ["tests"],
    "pattern": "**/*.test.js",
    "cmd": "npm test",
    "framework": "vitest"
  },
  "cache": ".project-index",
  "llm": {
    "provider": "codex",
    "timeout": 600000
  }
}
```

### .stale-config.json

```json
{
  "include": ["js/agents/**"],
  "ignore": ["tests/**", "docs/**"],
  "features": { "doc": true, "audit": true, "kanban": true, "testAnalysis": true },
  "concurrency": 6
}
```

## Dashboard

访问 `http://localhost:3008`，功能包括：

- **总览**：过期状态、缓存来源
- **模块**：模块列表、过滤、状态筛选
- **任务**：启动/取消/删除/重试任务
- **依赖图**：D3 可视化
- **配置**：编辑 .stale-config.json
- **启动**：任务类型选择、参数配置

## 目录结构

```
pi-cli/
├── cli.js              # 统一入口
├── lib/
│   ├── shared.js       # 工具函数
│   ├── context.js      # 配置加载
│   ├── types.js        # 类型定义
│   ├── deps/           # 依赖图分析
│   │   └── graph.js    # 依赖图 + 影响分析
│   ├── test/           # 测试操作
│   │   ├── mapper.js   # 源码↔测试映射
│   │   ├── runner.js   # 测试运行
│   │   ├── prioritize.js # 优先级排序
│   │   ├── fix.js      # LLM 测试修复
│   │   └── generator.js # 测试生成
│   ├── doc/            # 文档生成
│   │   └── generate.js # CLAUDE.md 生成 + stale 检测
│   ├── audit/          # 代码审计
│   │   └── scan.js     # AUDIT.md 生成
│   ├── module/         # LLM 模块分析
│   │   └── analyzer.js # 批量文档/审计生成
│   ├── llm/            # LLM 批量执行
│   │   └── batch.js    # DAG 调度 + codeagent-wrapper
│   ├── task/           # 任务管理
│   │   └── manager.js  # PID 跟踪 + 状态管理
│   ├── stale/          # Stale 通知
│   └── update/         # 增量更新
└── ui/
    └── server.js       # Dashboard (SSE + API)
```

## Kanban 集成

审计任务自动创建到 Kanban：

```bash
export KANBAN_URL=http://127.0.0.1:3007/api/v1
```

未运行 Kanban 服务时静默跳过。

## 安全约束

LLM 任务自动注入安全前缀，禁止：
- 执行 git checkout / reset / clean
- 删除文件
- 修改实现代码（仅测试修复时）

## ⚠️ 禁止的错误模式

测试修复和代码生成时必须避免的反模式：

1. **不要跳过测试** — 每次修复后必须运行测试验证
2. **不要批量删除** — 单个文件逐一处理
3. **不要硬编码** — 使用配置文件
4. **不要忽略错误** — 记录并上报

## 经验教训

参见 [LESSONS.md](./LESSONS.md)
