---
name: start
description: Execute a task autonomously with real-time Telegram progress updates, automatic deployment, commit, and push. No user interaction required - makes all decisions independently.
allowed-tools:
  - Bash(git *)
  - Bash(cargo *)
  - Bash(bun *)
  - Bash(/home/alan/code/start/.claude/scripts/*.sh)
  - Bash(curl *)
  - Bash(chmod *)
  - Bash(mkdir *)
  - Bash(rm *)
  - Bash(cp *)
  - Bash(mv *)
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Task
---

# Start Skill

自动执行任务，全程通过 Telegram 实时反馈进度，完成后自动部署、提交并推送代码。

## 核心原则

**完全自主执行**：
- ❌ 不使用 `AskUserQuestion` - 自己做决策
- ✅ 遇到选择时，使用最佳实践方案
- ✅ 遇到不确定的情况，选择保守/安全的方案
- ✅ 多个关键节点发送 Telegram 通知保持用户知情

## Arguments

`$ARGUMENTS` - 任务描述

示例:
- `/start 添加一个新的 outbound 配置管理模块`
- `/start 修复空状态下无法创建配置的 bug`
- `/start 优化前端性能，减少不必要的 re-render`

## 完整工作流

```
1. 📨 发送"开始执行"通知
2. 🔍 分析任务需求
3. 📖 阅读相关代码
4. 💡 制定执行方案（不询问用户）
5. 🔧 执行任务
   - 在关键节点发送进度通知
   - 创建/修改文件
   - 运行必要的命令
6. ✅ 代码质量检查（biome + TypeScript + Rust fmt）
7. 🚀 部署（deploy script）
8. ✅ 如果部署成功：commit + push
9. 📤 发送完成通知
10. ❌ 如果失败：发送错误通知
```

## 使用的工具和脚本

- **Telegram 通知**: `/home/alan/code/start/.claude/scripts/telegram-notify.sh`
- **部署脚本**: `/home/alan/code/start/.claude/scripts/deploy.sh`
- **Git 操作**: 直接使用 git 命令

## 实施细节

### 1. 发送开始通知

```bash
/home/alan/code/start/.claude/scripts/telegram-notify.sh "🚀 开始执行任务

📋 任务: ${ARGUMENTS}

⏳ 正在分析需求..."
```

### 2. 分析和规划

- 使用 Glob/Grep 查找相关代码
- 使用 Read 阅读关键文件
- **不使用 AskUserQuestion** - 自主决策
- 遵循项目 CLAUDE.md 中的最佳实践

**决策原则**:
- 参考现有代码模式
- 遵循项目架构约定
- 选择最简单可行的方案
- 保持代码一致性

### 3. 发送进度通知

在关键节点发送更新（每 2-3 个重要步骤）:

```bash
/home/alan/code/start/.claude/scripts/telegram-notify.sh "⚙️ 进度更新

当前步骤: ${current_step}

已完成:
- ${completed_item_1}
- ${completed_item_2}

正在进行: ${current_action}"
```

**建议的通知节点**:
- ✅ 开始执行任务
- ✅ 完成代码分析/规划
- ✅ 完成后端代码（如适用）
- ✅ 完成前端代码（如适用）
- ✅ 代码质量检查通过/失败
- ✅ 开始部署
- ✅ 部署成功/失败
- ✅ 任务完成

### 4. 执行任务

根据任务类型执行相应操作：

**新功能 (feat)**:
- 创建必要的文件
- 遵循现有模块模式
- 更新相关配置/路由

**Bug 修复 (fix)**:
- 定位问题代码
- 实施修复
- 验证修复效果

**重构 (refactor)**:
- 提取公共逻辑
- 改进代码结构
- 保持功能不变

**优化 (perf)**:
- 识别性能瓶颈
- 实施优���方案
- 测量改进效果

### 5. 代码质量检查

**在部署之前必须执行所有适用的检查**:

```bash
# 发送检查开始通知
/home/alan/code/sing-box-config-generator/.claude/scripts/telegram-notify.sh "🔍 开始代码质量检查

运行 Biome 和 TypeScript 检查..."

# 1. Biome 检查 (必须)
echo "Running Biome check..."
if ! bun run check; then
    /home/alan/code/sing-box-config-generator/.claude/scripts/telegram-notify.sh "❌ Biome 检查失败

请修复 lint 和格式问题后重试"
    exit 1
fi

# 2. TypeScript 类型检查 (必须)
echo "Running TypeScript type check..."
if ! bun run type-check; then
    /home/alan/code/sing-box-config-generator/.claude/scripts/telegram-notify.sh "❌ TypeScript 类型检查失败

请修复类型错误后重试"
    exit 1
fi

# 3. Rust 格式检查 (如果修改了后端代码)
# 检查是否有 Rust 文件被修改
if git diff --name-only HEAD | grep -q '\.rs$'; then
    echo "Rust files modified, running cargo fmt check..."
    if ! cargo fmt --check; then
        /home/alan/code/sing-box-config-generator/.claude/scripts/telegram-notify.sh "❌ Rust 格式检查失败

运行 'cargo fmt' 修复格式问题"
        exit 1
    fi
fi

# 所有检查通过
/home/alan/code/sing-box-config-generator/.claude/scripts/telegram-notify.sh "✅ 代码质量检查通过

准备部署..."
```

### 6. 部署

```bash
# 发送部署开始通知
/home/alan/code/start/.claude/scripts/telegram-notify.sh "🚀 开始部署

构建前端和后端...
重启服务..."

# 执行部署
DEPLOY_OUTPUT=$(/home/alan/code/start/.claude/scripts/deploy.sh 2>&1)
DEPLOY_STATUS=$?

if [ $DEPLOY_STATUS -eq 0 ]; then
    /home/alan/code/start/.claude/scripts/telegram-notify.sh "✅ 部署成功

服务已重启并通过健康检查
准备提交代码..."
else
    /home/alan/code/start/.claude/scripts/telegram-notify.sh "❌ 部署失败

错误信息:
${DEPLOY_OUTPUT}

任务中止，未提交代码"
    exit 1
fi
```

### 7. Commit 和 Push

**只在部署成功后执行**:

```bash
# 检查是否有改动
if ! git diff --quiet || ! git diff --cached --quiet || [ -n "$(git ls-files --others --exclude-standard)" ]; then
    # 分析改动，确定 commit 类型和消息
    git status
    git diff

    # 自主决定 commit message
    # 类型: feat, fix, refactor, perf, docs, style, test, build, chore
    # 格式: <type>(<scope>): <subject>

    git add .

    git commit -m "$(cat <<'EOF'
<type>(<scope>): <中文描述>

<详细说明>

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>
EOF
)"

    # Push 到远程
    git push

    COMMIT_MSG=$(git log -1 --pretty=%B)
else
    COMMIT_MSG="无文件改动"
fi
```

### 8. 发送完成通知

```bash
# 获取 commit 信息和 GitHub URL
COMMIT_HASH=$(git rev-parse HEAD 2>/dev/null || echo "N/A")
REMOTE_URL=$(git remote get-url origin 2>/dev/null || echo "")

# 从 git remote URL 提取 owner/repo
# 支持格式: git@github.com:owner/repo.git 或 https://github.com/owner/repo.git
if [ -n "$REMOTE_URL" ]; then
    # 先移除 .git 后缀，再提取 owner/repo
    REPO_PATH=$(echo "$REMOTE_URL" | sed -E 's/\.git$//' | sed -E 's/.*[:/]([^/]+\/[^/]+)$/\1/')
    COMMIT_URL="https://github.com/${REPO_PATH}/commit/${COMMIT_HASH}"
else
    COMMIT_URL=""
fi

# 构建完成通知消息
FINAL_MESSAGE="✅ 任务完成

📋 任务: ${ARGUMENTS}

🔧 主要改动:
- ${change_1}
- ${change_2}
- ${change_3}

✨ 结果:
${outcomes}

📝 Commit:
${COMMIT_MSG}"

# 添加 GitHub 链接（如果可用）
if [ -n "$COMMIT_URL" ]; then
    FINAL_MESSAGE="${FINAL_MESSAGE}

🔗 查看更改: ${COMMIT_URL}"
fi

# 添加状态信息
FINAL_MESSAGE="${FINAL_MESSAGE}

🚀 部署状态: 成功
📤 推送状态: 完成"

# Telegram 消息长度限制：4096 字符
# 如果超过限制，截断详细信息部分
MAX_LENGTH=4000  # 留一些安全余地
if [ ${#FINAL_MESSAGE} -gt $MAX_LENGTH ]; then
    # 构建简化版消息
    FINAL_MESSAGE="✅ 任务完成

📋 任务: ${ARGUMENTS}

🔧 主要改动: (内容较长，详见 GitHub)

📝 Commit: $(echo "$COMMIT_MSG" | head -n 1)

🔗 查看更改: ${COMMIT_URL}

🚀 部署状态: 成功
📤 推送状态: 完成"
fi

# 发送通知
/home/alan/code/sing-box-config-generator/.claude/scripts/telegram-notify.sh "$FINAL_MESSAGE"
```

## 自主决策指南

### 架构决策

**场景**: 需要选择实现方式
- ✅ 参考项目中现有类似功能的实现
- ✅ 遵循 CLAUDE.md 中的架构约定
- ✅ 选择最符合项目模式的方案

**示例**:
- 添加新模块 → 参考 log/ruleset/inbound 模块的结构
- 添加 API 端点 → 遵循 `/api/{module}` 约定
- 前端状态管理 → 使用 TanStack Query

### 命名决策

**场景**: 需要命名文件/组件/变量
- ✅ 遵循项目现有命名规范
- ✅ 使用清晰描述性的名称
- ✅ 保持与周围代码一致

**示例**:
- 模块名 → 使用小写单数形式（log, ruleset, inbound）
- 组件名 → 使用 PascalCase（FocusEditor, ConfigCard）
- API 路由 → 使用复数形式 REST 约定

### 范围决策

**场景**: 不确定改动范围
- ✅ 优先最小化改动
- ✅ 只修改必要的文件
- ✅ 避免过度工程化

**示例**:
- Bug 修复 → 只修复问题本身，不重构周围代码
- 新功能 → 实现核心功能，不添加"可能有用"的额外功能

### 技术选择

**场景**: 需要选择技术方案
- ✅ 使用项目已有的技术栈
- ✅ 不引入新的依赖（除非必须）
- ✅ 遵循项目的技术决策

**项目技术栈**:
- Backend: Rust + Axum + Tokio
- Frontend: React + Rsbuild + TanStack Router + TanStack Query
- UI: Radix UI + Tailwind CSS
- Package Manager: Bun（不用 npm/pnpm）

## 错误处理

### 执行失败

任何步骤失败时:

```bash
/home/alan/code/start/.claude/scripts/telegram-notify.sh "❌ 任务失败

📋 任务: ${ARGUMENTS}

❌ 失败步骤: ${failed_step}

错误信息:
${error_message}

⚠️ 状态:
- 代码改动: ${changes_made}
- 部署状态: 未执行
- Commit 状态: 未执行"
```

### 部署失败

部署失败时不提交代码:

```bash
# 构建错误消息
ERROR_MESSAGE="⚠️ 部署失败，代码未提交

📋 任务: ${ARGUMENTS}

✅ 代码改动已完成:
${changes}

❌ 部署失败:
${deploy_error}

💡 建议:
检查日志: sudo journalctl -u sing-box-config-generator -n 50
代码改动已保留但未提交"

# Telegram 消息长度限制检查
MAX_LENGTH=4000
if [ ${#ERROR_MESSAGE} -gt $MAX_LENGTH ]; then
    ERROR_MESSAGE="⚠️ 部署失败，代码未提交

📋 任务: ${ARGUMENTS}

✅ 代码改动已完成（详见本地文件）

❌ 部署失败: (错误信息过长，详见本地日志)

💡 建议:
检查日志: sudo journalctl -u sing-box-config-generator -n 50
代码改动已保留但未提交"
fi

/home/alan/code/sing-box-config-generator/.claude/scripts/telegram-notify.sh "$ERROR_MESSAGE"
```

## 最佳实践

### 1. 充分沟通
- 开始时说明执行计划
- 关键节点及时反馈
- 完成时总结改动

### 2. 遵循约定
- 严格遵循 CLAUDE.md 指南
- 参考现有代码模式
- 保持代码一致性

### 3. 保证质量
- 执行完整的代码质量检查
  - Biome 检查（必须）
  - TypeScript 类型检查（必须）
  - Rust fmt 检查（修改后端时）
- 部署前验证代码
- 通过健康检查再提交
- 写清晰的 commit message

### 4. 错误恢复
- 失败时清晰报告
- 不隐藏错误信息
- 提供恢复建议

## 示例场景

### 场景 1: 添加新模块

```
用户: /start 添加 outbound 配置管理模块

流程:
1. 📨 通知: "开始执行任务：添加 outbound 配置管理模块"
2. 🔍 阅读现有模块代码（log, ruleset, inbound）
3. 📨 通知: "已分析现有模块结构，开始实现后端 API"
4. 🔧 创建 src/backend/api/outbound.rs
5. 🔧 更新 src/main.rs 添加路由
6. 📨 通知: "后端完成，开始实现前端"
7. 🔧 创建 src/frontend/routes/outbound/
8. 🔧 创建 src/frontend/api/outbound.ts
9. 🔧 更新 Sidebar 添加导航
10. 📨 通知: "代码完成，开始质量检查"
11. ✅ 执行代码质量检查（biome + TypeScript + Rust fmt）
12. 📨 通知: "检查通过，开始部署"
13. 🚀 执行 deploy.sh
14. 📨 通知: "部署成功，提交代码"
15. ✅ Commit + Push
16. 📨 通知: "任务完成（包含 GitHub commit 链接）"
```

### 场景 2: Bug 修复

```
用户: /start 修复空状态下无法创建配置的问题

流程:
1. 📨 通知: "开始执行任务：修复空状态创建问题"
2. 🔍 分析问题原因
3. 📨 通知: "定位到问题：FocusEditor 在条件渲染内部，开始修复"
4. 🔧 修改 4 个配置模块的 index.tsx
5. 📨 通知: "修复完成，开始质量检查"
6. ✅ 执行代码质量检查（biome + TypeScript）
7. 📨 通知: "检查通过，开始部署"
8. 🚀 执行 deploy.sh
9. 📨 通知: "部署成功，提交代码"
10. ✅ Commit + Push
11. 📨 通知: "Bug 修复完成（包含 GitHub commit 链接）"
```

### 场景 3: 性能优化

```
用户: /start 优化前端性能，减少不必要的 re-render

流程:
1. 📨 通知: "开始执行任务：优化前端性能"
2. 🔍 分析性能瓶颈
3. �� 通知: "发现问题：组件过度渲染，使用 React.memo 优化"
4. 🔧 优化关键组件
5. 📨 通知: "优化完成，开始质量检查"
6. ✅ 执行代码质量检查（biome + TypeScript）
7. 📨 通知: "检查通过，开始部署"
8. 🚀 执行 deploy.sh
9. 📨 通知: "部署成功，提交代码"
10. ✅ Commit + Push
11. 📨 通知: "性能优化完成（包含 GitHub commit 链接）"
```

## 注意事项

1. **不要使用 AskUserQuestion** - 这是核心要求
2. **频繁发送 Telegram 通知** - 让用户知道进度
3. **必须执行代码质量检查** - 部署前运行所有适用的检查
4. **检查失败立即停止** - 不要在检查失败时继续部署
5. **部署失败不提交代码** - 保证代码质量
6. **自主决策** - 遵循最佳实践和项目约定
7. **保持简洁** - 只做必要的改动

## 配置要求

- Telegram 通知已配置（见 `.claude/telegram-config.md`）
- 部署脚本可用（`.claude/scripts/deploy.sh`）
- Git 仓库已初始化并配置远程仓库

## 相关文档

- **项目指南**: `/home/alan/code/start/CLAUDE.md`
- **Telegram 配置**: `/home/alan/code/start/.claude/telegram-config.md`
- **部署脚本**: `/home/alan/code/start/.claude/scripts/deploy.sh`
- **通知脚本**: `/home/alan/code/start/.claude/scripts/telegram-notify.sh`
