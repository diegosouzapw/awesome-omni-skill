---
name: git-worktree
version: "1.0.0"
description: Git Worktree 管理命令。提供 init、list、remove 三个子命令来管理项目 worktree。
user-invocable: true
allowed-tools:
  - Bash
  - AskUserQuestion
---

# Git Worktree 管理命令 (v1.0.0)

管理项目中的 Git worktree，支持创建、列表查看和删除操作。

## 调用方式

```
/git-worktree init "任务描述"   # 创建 worktree
/git-worktree list             # 列出所有 worktree
/git-worktree remove           # 删除 worktree
```

## 命令详解

### init - 创建新的 Worktree

根据用户描述的任务，自动推断分支类型并创建 worktree。创建完成后询问用户是否要在 iTerm2 中打开新 tab 并启动 Claude。

**调用方式：**
```
/git-worktree init "任务描述"
```

**参数说明：**
- `任务描述` - 任务的简短描述（必需）

**交互流程：**
1. 执行脚本创建 worktree
2. 脚本输出 `[ASK_USER_OPEN_ITERM]` 标记和信息
3. AI 检测到标记后，使用 AskUserQuestion 询问用户是否打开 iTerm2
4. 用户同意后，执行以下命令打开 iTerm2：

```bash
cd /Users/lee0407/dev/projs/auth && python3 .claude/skills/git-worktree/scripts/open_iterm.py "<worktree_path>" "<branch_name>" "<description>"
```

**自动推断的分支类型：**

| 关键词 | 分支类型 |
|--------|----------|
| feat, 功能, 添加, 新增, 实现 | feature |
| fix, 修复, bug, 错误, 问题 | bugfix |
| refactor, 重构, 优化, 重写 | refactor |
| docs, 文档, 说明, readme | docs |
| test, 测试, 单元 | test |
| chore, 配置, 构建, 依赖 | chore |

**执行流程：**
1. 切换到 develop 分支并拉取最新代码
2. 根据描述推断分支类型并生成分支名
3. 创建新分支
4. 在 `worktrees/` 目录下创建 worktree
5. 切换回 develop 分支
6. 输出 `[ASK_USER_OPEN_ITERM]` 标记，等待 AI 询问用户

**使用示例：**
```
/git-worktree init "添加用户 Excel 导入导出功能"
# 结果：创建 feature/excel-import-export 分支和 worktree
#       AI 检测到 [ASK_USER_OPEN_ITERM] 标记
#       AI 使用 AskUserQuestion 询问用户
#       用户同意后执行 open_iterm.py 打开 iTerm2 并启动 Claude
```

## AI 响应规则

当执行 `/git-worktree init` 后，脚本会输出 `[ASK_USER_OPEN_ITERM]` 标记。AI 应：

1. **检测到 `[ASK_USER_OPEN_ITERM]` 标记**
2. **解析标记后的信息**（格式：`path|branch|description`）
3. **使用 AskUserQuestion 询问用户**：
```json
{
  "questions": [
    {
      "question": "是否要在 iTerm2 中打开新 tab 并启动 Claude？",
      "header": "iTerm2",
      "options": [
        {"label": "是，打开 iTerm2", "description": "在 iTerm2 新 tab 中启动 Claude"},
        {"label": "否，稍后手动打开", "description": "不打开，稍后自行执行"}
      ],
      "multiSelect": false
    }
  ]
}
```
4. **用户选择"是"后**，执行：
```bash
cd /Users/lee0407/dev/projs/auth && python3 .claude/skills/git-worktree/scripts/open_iterm.py "<worktree_path>" "<branch_name>" "<description>"
```

## 脚本说明

### worktree_manager.py
主脚本，处理 worktree 的创建、列表和删除操作。

### open_iterm.py
辅助脚本，用于在 iTerm2 中打开新 tab 并启动 Claude。

## 命令详解

### list - 列出所有 Worktree

显示当前仓库中所有 worktree 的列表。

**调用方式：**
```
/git-worktree list
```

**输出格式：**
```
Git Worktree 列表
============================================================
主 /Users/lee0407/dev/projs/auth                    [develop] ✓
└─ worktrees/feature-excel-import              [feature/excel-import] ✓
└─ worktrees/bugfix-login-issue                [bugfix/login-issue] ✗
============================================================
✓ = 干净工作区 | ✗ = 有未提交更改
```

**使用示例：**
```
/git-worktree list
```

### remove - 删除 Worktree

交互式删除指定的 worktree。

**调用方式：**
```
/git-worktree remove           # 交互式选择
/git-worktree remove 1         # 按索引删除
/git-worktree remove feature-xxx  # 按路径删除
```

**注意事项：**
- 不会删除主目录的 worktree
- 不会删除有未提交更改的 worktree（除非强制）

**使用示例：**
```
/git-worktree remove
# 显示可删除的 worktree 列表，供用户选择
```

## 工作原理

Git Worktree 允许在同一个仓库中同时在多个分支上工作：

```
auth/
├── .git/
├── worktrees/
│   ├── feature-excel-import/     # 功能开发 worktree
│   └── bugfix-login-issue/       # Bug 修复 worktree
├── auth-biz/
├── auth-server/
└── ...
```

## 最佳实践

1. **每个任务一个 worktree** - 保持工作区干净
2. **及时清理** - 任务完成后删除不需要的 worktree
3. **描述清晰** - 使用有意义的任务描述便于识别
4. **先更新 develop** - 创建前确保 develop 是最新的

## 注意事项

1. 所有 worktree 必须创建在 `worktrees/` 目录下
2. 创建前会自动检查 develop 分支是否为最新
3. 删除前会询问用户确认
4. 不会删除主目录的 worktree
