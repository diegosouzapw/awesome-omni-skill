---
name: kanban
description: Code Kanban 本地任务管理系统。支持项目、任务、Worktree 的完整 CRUD 操作。私有数据不暴露到公开 GitHub。
---

# kanban (本地任务管理)

Code Kanban API 的完整封装，支持项目管理、任务管理、Worktree 集成。

## 环境配置

```bash
# 设置 API 地址 (CodeKanban 官方默认 3005，本项目使用 3007)
export KANBAN_URL="http://127.0.0.1:3007"
```

> **注意**: 下文所有 `${API}` 均指 `${KANBAN_URL}/api/v1`

## AI 使用建议

**优先使用 CLI 而非直接调用 API**，CLI 封装了常用操作且输出更简洁：

```bash
# 推荐：CLI 方式
node ~/.claude/skills/kanban/kanban-cli.js list --status=todo

# 不推荐：直接 curl API（输出冗长，占用上下文）
curl -s "${API}/projects/{id}/tasks" | jq ...
```

**常用场景：**
| 场景 | 命令 |
|------|------|
| 快速查看待办 | `list --status=todo` |
| 查看进行中详情 | `list --status=in_progress -v` |
| 查看单个任务 | `show <短ID>` |
| 开始任务 | `start <id>` |
| 完成任务 | `done <id>` |
| 改优先级 | `move <id> --priority=0` |

## 命令速查

| 命令 | 说明 |
|------|------|
| `/kanban` | 显示当前项目状态 |
| `/kanban list` | 列出所有任务 (简略) |
| `/kanban list -v` | 列出所有任务 (展开详情) |
| `/kanban list --status=todo` | 只看待办 |
| `/kanban list --status=in_progress` | 只看进行中 |
| `/kanban list --status=done` | 只看已完成 |
| `/kanban list --priority=0` | 只看 P0 紧急任务 |
| `/kanban show <id>` | 查看任务详情 (支持短 ID) |
| `/kanban add <title>` | 创建新任务 |
| `/kanban add <title> --description=<text>` | 创建带描述的任务 |
| `/kanban add <title> --priority=0 --tags=t1,t2` | 创建带优先级和标签的任务 |
| `/kanban edit <id> --description=<text>` | 编辑任务描述 |
| `/kanban edit <id> --tags=t1,t2 --due=2025-01-20` | 编辑标签和截止日期 |
| `/kanban done <id>` | 标记任务完成 |
| `/kanban start <id>` | 开始任务 (in_progress) |
| `/kanban move <id> --status=todo` | 改回待办 |
| `/kanban move <id> --priority=0` | 改优先级 |
| `/kanban move <id> --worktree=<wt-id>` | 绑定 Worktree |
| `/kanban batch` | 批量并行执行 |
| `/kanban worktree <id>` | 为任务创建 worktree |
| `/kanban export` | 导出 AI 友好的任务上下文 |
| `/kanban export --json` | 导出 JSON 格式 |

## CLI 直接调用

```bash
# CLI 路径
CLI="$HOME/.claude/skills/kanban/kanban-cli.js"

# 查看状态
node "$CLI"

# 列出任务
node "$CLI" list --status=todo
node "$CLI" list --status=in_progress -v

# 查看详情 (支持短 ID)
node "$CLI" show WtS0Nofb

# 创建任务 (支持全量参数)
node "$CLI" add "新任务标题" --priority=1
node "$CLI" add "带描述任务" --priority=0 --description="任务详情\n第二行" --tags=type/feature,epic/xxx --due=2025-01-20

# 编辑任务 (更新内容)
node "$CLI" edit <id> --description="新描述"
node "$CLI" edit <id> "新标题" --priority=1 --tags=tag1,tag2

# 状态操作
node "$CLI" start <id>      # 开始
node "$CLI" done <id>       # 完成
node "$CLI" move <id> --status=todo --priority=0  # 移动

# 绑定 Worktree
node "$CLI" move <id> --worktree=<wt-id>
node "$CLI" move <id> --worktree=  # 解绑 (空字符串)

# 删除任务
node "$CLI" delete <id>

# 批量导入
node "$CLI" import tasks.json

# JSON 输出
node "$CLI" list --json
node "$CLI" show <id> --json
```

## 基础配置

```bash
# Shell 中使用
KANBAN_URL="${KANBAN_URL:-http://127.0.0.1:3007}"
API="${KANBAN_URL}/api/v1"
```

```javascript
// JavaScript 中使用
const BASE_URL = process.env.KANBAN_URL || "http://127.0.0.1:3007";
const API = `${BASE_URL}/api/v1`;
```

---

## 1. 项目操作

### 1.1 列出项目

```bash
curl -s "${API}/projects" | jq '.items[] | {id, name, path}'
```

### 1.2 检测当前项目

```bash
# 根据当前目录匹配项目
CWD=$(pwd)
curl -s "${API}/projects" | jq --arg cwd "$CWD" '.items[] | select(.path == $cwd)'
```

### 1.3 创建项目

```bash
curl -X POST "${API}/projects/create" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "项目名称",
    "path": "/path/to/project",
    "description": "项目描述",
    "defaultBranch": "main"
  }'
```

### 1.4 更新项目

```bash
curl -X POST "${API}/projects/{projectId}/update" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "新名称",
    "description": "新描述"
  }'
```

---

## 2. 任务操作

### 2.1 列出任务

```bash
# 所有任务
curl -s "${API}/projects/{projectId}/tasks" | jq '.items'

# 按状态过滤
curl -s "${API}/projects/{projectId}/tasks" | jq '.items[] | select(.status == "todo")'

# 按优先级过滤
curl -s "${API}/projects/{projectId}/tasks" | jq '.items[] | select(.priority == 0)'
```

### 2.2 创建任务

```bash
curl -X POST "${API}/projects/{projectId}/tasks/create" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "任务标题",
    "description": "任务描述\n\nDepends on: [task-id]",
    "status": "todo",
    "priority": 0,
    "tags": ["type/feature", "epic/lifecycle"],
    "dueDate": null,
    "worktreeId": null
  }'
```

> **注意**: `dueDate` 和 `worktreeId` 为必填字段，可设为 `null`

**优先级定义:**
- `0`: P0 紧急
- `1`: P1 高
- `2`: P2 中
- `3`: P3 低

### 2.3 更新任务内容

```bash
# 更新标题和描述（不支持 status）
curl -X POST "${API}/tasks/{taskId}/update" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "新标题",
    "description": "新描述",
    "priority": 0,
    "tags": ["tag1", "tag2"],
    "dueDate": "2026-01-20T23:59:59+08:00"
  }'
```

**UpdateTaskBody 支持的字段:**
- `title` - 任务标题
- `description` - 任务描述
- `priority` - 优先级 (0-3)
- `tags` - 标签数组
- `dueDate` - 截止日期 (ISO 8601)

**注意:** `status` 字段不在 UpdateTaskBody 中，需使用 `/move` 端点。

### 2.4 移动任务（更新状态）

```bash
# 更新状态为 done
curl -X POST "${API}/tasks/{taskId}/move" \
  -H "Content-Type: application/json" \
  -d '{"status": "done"}'

# 更新状态为 in_progress
curl -X POST "${API}/tasks/{taskId}/move" \
  -H "Content-Type: application/json" \
  -d '{"status": "in_progress"}'

# 同时更新状态和排序
curl -X POST "${API}/tasks/{taskId}/move" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "done",
    "orderIndex": 1000,
    "worktreeId": "optional-worktree-id"
  }'
```

**MoveTaskBody 支持的字段:**
- `status` - 新状态 (todo/in_progress/done/blocked)
- `orderIndex` - 排序索引
- `worktreeId` - 关联 Worktree

### 2.5 删除任务

```bash
curl -X DELETE "${API}/tasks/{taskId}/delete"
```

### 2.6 任务状态流转

```
todo → in_progress → done
         ↓
       blocked (可选)
```

---

## 3. Worktree 操作

### 3.1 列出 Worktrees

```bash
curl -s "${API}/projects/{projectId}/worktrees" | jq '.items'
```

### 3.2 创建 Worktree（推荐方式）

**不要使用 Kanban API 创建 worktree**，它会放在项目内部。使用 git 手动创建到项目同级目录：

```bash
# 推荐路径结构：项目同级平铺，命名为 {project-name}-{branch-safe}
# 例如：/mnt/f/pb/paper-burner → /mnt/f/pb/paper-burner-fix-issue-123

PROJECT_PATH="/path/to/project"
PROJECT_NAME=$(basename "$PROJECT_PATH")
BRANCH_NAME="fix/issue-123"
BRANCH_SAFE=$(echo "$BRANCH_NAME" | tr '/' '-')
WORKTREE_PATH="$(dirname "$PROJECT_PATH")/${PROJECT_NAME}-${BRANCH_SAFE}"

# 创建 worktree
git -C "$PROJECT_PATH" worktree add "$WORKTREE_PATH" -b "$BRANCH_NAME"

# 同步到 Kanban
curl -s -X POST "${API}/projects/${PROJECT_ID}/sync-worktrees"
```

> **命名惯例**：`{project}-{branch-safe}`，如 `paper-burner-fix-test-assertions`

### 3.3 删除 Worktree

```bash
# 删除 worktree 并删除分支
curl -X POST "${API}/worktrees/{worktreeId}?deleteBranch=true"

# 仅删除 worktree，保留分支
curl -X POST "${API}/worktrees/{worktreeId}?deleteBranch=false"

# 强制删除（有未提交更改时）
curl -X POST "${API}/worktrees/{worktreeId}?force=true&deleteBranch=true"
```

### 3.4 绑定任务到 Worktree

```bash
curl -X POST "${API}/tasks/{taskId}/move" \
  -H "Content-Type: application/json" \
  -d '{
    "worktreeId": "worktree-id"
  }'
```

### 3.4 Worktree 提交

```bash
curl -X POST "${API}/worktrees/{worktreeId}/commit" \
  -H "Content-Type: application/json" \
  -d '{
    "message": "feat: implement feature X"
  }'
```

### 3.5 Worktree 合并

```bash
curl -X POST "${API}/worktrees/{worktreeId}/merge" \
  -H "Content-Type: application/json" \
  -d '{
    "targetBranch": "main"
  }'
```

---

## 4. 评论操作

### 4.1 获取任务评论

```bash
curl -s "${API}/tasks/{taskId}/comments" | jq '.items'
```

### 4.2 添加评论

```bash
curl -X POST "${API}/tasks/{taskId}/comments/create" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "评论内容"
  }'
```

---

## 5. AI 会话操作

### 5.1 获取任务关联的 AI 会话

```bash
curl -s "${API}/tasks/{taskId}/ai-sessions" | jq '.items'
```

### 5.2 关联 AI 会话到任务

```bash
curl -X POST "${API}/tasks/{taskId}/ai-sessions/link" \
  -H "Content-Type: application/json" \
  -d '{
    "sessionId": "ai-session-id"
  }'
```

---

## 6. 批量操作

### 6.1 批量执行

调用 `kanban-batch` skill:

```bash
node ~/.claude/skills/kanban-batch/kanban-planner.js [options]
```

### 6.2 批量创建任务

```javascript
const tasks = [
  { title: "Task 1", priority: 0 },
  { title: "Task 2", priority: 1, deps: "Task 1" },
  { title: "Task 3", priority: 1 },
];

for (const task of tasks) {
  await fetch(`${API}/projects/${projectId}/tasks/create`, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      title: task.title,
      description: task.deps ? `Depends on: [${task.deps}]` : "",
      status: "todo",
      priority: task.priority,
    }),
  });
}
```

---

## 7. 常用工作流

### 7.1 创建 Epic + 子任务

```bash
# 1. 创建 Epic 任务
EPIC_ID=$(curl -s -X POST "${API}/projects/${PROJECT_ID}/tasks/create" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "[Epic] 资源生命周期治理",
    "description": "## 目标\n统一实现 Disposable 模式",
    "status": "todo",
    "priority": 0,
    "tags": ["type/epic"]
  }' | jq -r '.item.id')

# 2. 创建子任务
curl -X POST "${API}/projects/${PROJECT_ID}/tasks/create" \
  -H "Content-Type: application/json" \
  -d "{
    \"title\": \"StateEngine dispose\",
    \"description\": \"Depends on: [${EPIC_ID}]\",
    \"status\": \"todo\",
    \"priority\": 0,
    \"tags\": [\"epic/lifecycle\"]
  }"
```

### 7.2 开始任务工作流

```bash
# 1. 创建 worktree（手动，不用 Kanban API）
PROJECT_PATH="/path/to/project"
PROJECT_NAME=$(basename "$PROJECT_PATH")
BRANCH_NAME="fix/task-xxx"
BRANCH_SAFE=$(echo "$BRANCH_NAME" | tr '/' '-')
WORKTREE_PATH="$(dirname "$PROJECT_PATH")/${PROJECT_NAME}-${BRANCH_SAFE}"

git -C "$PROJECT_PATH" worktree add "$WORKTREE_PATH" -b "$BRANCH_NAME"

# 2. 同步并获取 worktree ID
curl -s -X POST "${API}/projects/${PROJECT_ID}/sync-worktrees"
WORKTREE_ID=$(curl -s "${API}/projects/${PROJECT_ID}/worktrees" | node -e "
const d=JSON.parse(require('fs').readFileSync(0,'utf8'));
const wt = d.items?.find(w => w.branchName === '$BRANCH_NAME');
console.log(wt?.id || '');
")

# 3. 绑定到任务并更新状态为进行中
curl -X POST "${API}/tasks/${TASK_ID}/move" \
  -H "Content-Type: application/json" \
  -d "{\"worktreeId\": \"${WORKTREE_ID}\", \"status\": \"in_progress\"}"
```

### 7.3 完成任务工作流

```bash
# 1. 提交代码
curl -X POST "${API}/worktrees/${WORKTREE_ID}/commit" \
  -H "Content-Type: application/json" \
  -d '{"message": "fix: implement dispose"}'

# 2. 合并到主分支
curl -X POST "${API}/worktrees/${WORKTREE_ID}/merge" \
  -H "Content-Type: application/json" \
  -d '{"targetBranch": "main"}'

# 3. 标记任务完成
curl -X POST "${API}/tasks/${TASK_ID}/move" \
  -H "Content-Type: application/json" \
  -d '{"status": "done"}'
```

---

## 8. 查询示例

### 8.1 看板视图

```bash
echo "=== TODO ==="
curl -s "${API}/projects/${PROJECT_ID}/tasks" | jq -r '.items[] | select(.status == "todo") | "[\(.priority)] \(.title)"'

echo "=== IN PROGRESS ==="
curl -s "${API}/projects/${PROJECT_ID}/tasks" | jq -r '.items[] | select(.status == "in_progress") | "[\(.priority)] \(.title)"'

echo "=== DONE ==="
curl -s "${API}/projects/${PROJECT_ID}/tasks" | jq -r '.items[] | select(.status == "done") | "[\(.priority)] \(.title)"'
```

### 8.2 优先级统计

```bash
curl -s "${API}/projects/${PROJECT_ID}/tasks" | jq '
  .items | group_by(.priority) |
  map({priority: .[0].priority, count: length})
'
```

### 8.3 今日到期任务

```bash
TODAY=$(date +%Y-%m-%d)
curl -s "${API}/projects/${PROJECT_ID}/tasks" | jq --arg today "$TODAY" '
  .items[] | select(.dueDate != null and (.dueDate | startswith($today)))
'
```

---

## 9. 系统操作

### 9.1 健康检查

```bash
curl -s "${API}/health"
```

### 9.2 版本信息

```bash
curl -s "${API}/system/version"
```

### 9.3 AI 助手状态

```bash
curl -s "${API}/system/ai-assistant-status"
```

---

## 10. 执行指令

当用户调用 `/kanban` 时:

### 无参数 - 显示当前项目状态

```bash
# 检测项目
PROJECT=$(curl -s "${API}/projects" | jq --arg cwd "$(pwd)" '.items[] | select(.path == $cwd)')
PROJECT_ID=$(echo $PROJECT | jq -r '.id')
PROJECT_NAME=$(echo $PROJECT | jq -r '.name')

echo "Project: $PROJECT_NAME"

# 显示任务统计
TASKS=$(curl -s "${API}/projects/${PROJECT_ID}/tasks")
TODO=$(echo $TASKS | jq '[.items[] | select(.status == "todo")] | length')
IN_PROGRESS=$(echo $TASKS | jq '[.items[] | select(.status == "in_progress")] | length')
DONE=$(echo $TASKS | jq '[.items[] | select(.status == "done")] | length')

echo "Tasks: $TODO todo, $IN_PROGRESS in progress, $DONE done"
```

### list - 列出任务

```bash
curl -s "${API}/projects/${PROJECT_ID}/tasks" | jq -r '
  .items | sort_by(.priority) | .[] |
  "[P\(.priority)] [\(.status)] \(.title) (\(.id))"
'
```

### add <title> - 创建任务

```bash
curl -X POST "${API}/projects/${PROJECT_ID}/tasks/create" \
  -H "Content-Type: application/json" \
  -d "{\"title\": \"$TITLE\", \"status\": \"todo\", \"priority\": 2}"
```

### done <id> - 完成任务

```bash
curl -X POST "${API}/tasks/${TASK_ID}/move" \
  -H "Content-Type: application/json" \
  -d '{"status": "done"}'
```

### batch - 批量执行

```bash
node ~/.claude/skills/kanban-batch/kanban-planner.js
```

### worktree <task-id> - 创建并绑定 worktree

**重要**: 不使用 Kanban API 创建 worktree（会放在项目内部），而是手动用 git 创建到项目外部，然后绑定。

**AI 执行流程**:

1. **检查工作区状态（前置条件）**
   ```bash
   git -C "$PROJECT_PATH" status --porcelain
   ```
   - 如果工作区不干净（有未提交的更改），**停止流程**
   - 提示用户：
     ```
     ⚠️ 工作区不干净，有未提交的更改：
     [列出更改的文件]

     建议：请先提交或暂存当前更改，保持工作区干净后再创建 worktree。

     可选操作：
     - git stash（暂存更改）
     - git commit（提交更改）
     - 继续（不推荐，可能导致状态混乱）
     ```
   - 等待用户处理后再继续

2. **分析现有 worktree 位置**
   ```bash
   git -C "$PROJECT_PATH" worktree list
   ```
   查看已有 worktree 放在哪里，推断用户偏好。

3. **检查项目结构**
   - 项目路径是什么？
   - 同级目录已有哪些 worktree？
   - 命名惯例：`{project}-{branch-safe}`（同级平铺）

4. **提出建议并请求用户确认**
   ```
   === Worktree 创建计划 ===
   任务: [任务标题]
   分支名: fix/xxx
   建议路径: /path/to/project-fix-xxx

   基于: [说明为什么建议这个路径，如已有 worktree 的命名惯例]

   确认创建? 或指定其他路径?
   ```

5. **用户确认后，手动创建 worktree**
   ```bash
   git -C "$PROJECT_PATH" worktree add "$WORKTREE_PATH" -b "$BRANCH_NAME"
   ```

6. **刷新 Kanban 并绑定任务**
   ```bash
   # 刷新 Kanban worktree 列表
   curl -s -X POST "${API}/projects/${PROJECT_ID}/sync-worktrees"

   # 获取新 worktree 的 ID
   WORKTREE_ID=$(curl -s "${API}/projects/${PROJECT_ID}/worktrees" | node -e "
   const d=JSON.parse(require('fs').readFileSync(0,'utf8'));
   const wt = d.items?.find(w => w.branchName === '$BRANCH_NAME');
   console.log(wt?.id || '');
   ")

   # 绑定到任务并开始
   curl -X POST "${API}/tasks/${TASK_ID}/move" \
     -H "Content-Type: application/json" \
     -d "{\"worktreeId\": \"${WORKTREE_ID}\", \"status\": \"in_progress\"}"
   ```

7. **输出结果**
   ```
   Worktree 已创建并绑定:
     路径: $WORKTREE_PATH
     分支: $BRANCH_NAME
     任务: $TASK_TITLE
   ```

---

## 与 GitHub 对比

| 功能 | GitHub | Code Kanban |
|------|--------|-------------|
| Issues | 公开 | 私有 |
| Projects | 需创建 | 自动 |
| Worktree | 手动 | API 集成 |
| AI 会话 | 无 | 原生追踪 |
| 终端集成 | 无 | 原生支持 |
| 离线使用 | 否 | 是 |

---

## 常见坑点

### API 方法

**所有写操作都用 POST，不用 PUT/PATCH！**

| 操作 | 正确 | 错误 |
|------|------|------|
| 更新任务内容 | `POST /tasks/{id}/update` | ~~`PUT /tasks/{id}`~~ |
| 更新任务状态 | `POST /tasks/{id}/move` | ~~`PUT /tasks/{id}/update`~~ |
| 创建任务 | `POST /projects/{id}/tasks/create` | ~~`POST /projects/{id}/tasks`~~ |

### UpdateTaskBody vs MoveTaskBody

- **UpdateTaskBody** (`/update`): title, description, priority, tags, dueDate
- **MoveTaskBody** (`/move`): status, orderIndex, worktreeId

`status` 字段只在 MoveTaskBody 中，不要在 `/update` 里传！

### 端点路径

| 操作 | 端点 |
|------|------|
| 获取单个任务 | `GET /api/v1/tasks/{id}` |
| 列出项目任务 | `GET /api/v1/projects/{projectId}/tasks` |
| 创建任务 | `POST /api/v1/projects/{projectId}/tasks/create` |
| 更新任务内容 | `POST /api/v1/tasks/{id}/update` |
| 更新任务状态 | `POST /api/v1/tasks/{id}/move` |

### API 文档

- 在线文档: `${KANBAN_URL}/docs`
- OpenAPI 规范: `${KANBAN_URL}/openapi.yaml`

### Node.js 替代 jq

当系统没有 jq 时，用 node 替代:

```bash
# 替代 jq '.items'
curl -s "${API}/tasks" | node -e "
const d = JSON.parse(require('fs').readFileSync(0, 'utf8'));
console.log(JSON.stringify(d.items, null, 2));
"

# 替代 jq -r '.item.id'
curl -s "${API}/tasks/${ID}" | node -e "
const d = JSON.parse(require('fs').readFileSync(0, 'utf8'));
console.log(d.item?.id);
"
```

### Worktree 相关坑点

**1. 不要用 Kanban API 创建 worktree**

Kanban API 创建的 worktree 会放在项目配置的 `worktreeBasePath`（默认是项目内部 `.worktrees/`），且这个配置**无法通过 API 修改**（`UpdateProjectInputBody` 不支持）。

正确做法：手动用 `git worktree add` 创建到项目外部，然后调用 `sync-worktrees` 让 Kanban 检测到它。

**2. 解绑 worktreeId 用空字符串，不是 null**

```bash
# ❌ 错误 - null 不生效
curl -X POST "${API}/tasks/{id}/move" -d '{"worktreeId": null}'

# ✅ 正确 - 用空字符串
curl -X POST "${API}/tasks/{id}/move" -d '{"worktreeId": ""}'
```

**3. 删除外部 worktree 后需同步**

```bash
# 删除 worktree
git worktree remove /path/to/worktree

# 同步 Kanban（否则它还认为 worktree 存在）
curl -s -X POST "${API}/projects/${PROJECT_ID}/sync-worktrees"
```

**4. Kanban 会自动检测外部创建的 worktree**

只要 worktree 属于该项目的 git 仓库，`sync-worktrees` 或 `worktrees` 列表接口就能发现它，无论路径在哪里。

### 项目配置限制

`UpdateProjectInputBody` 只支持以下字段：
- `name` - 项目名称
- `description` - 项目描述
- `hidePath` - 是否隐藏路径

**不支持修改**：`worktreeBasePath`、`defaultBranch`、`path` 等。这些只能在创建项目时设置。
