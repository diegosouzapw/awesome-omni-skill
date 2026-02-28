---
name: qcc-git
description: Use for Git workflow, branch strategy, and commit conventions in qcc_plus project
---

# QCC Plus Git 工作流

## 分支策略（强制）

**强制规则**: 所有开发工作必须在 `test` 分支进行，编写代码前必须确认当前分支。

| 分支 | 用途 | 说明 |
|------|------|------|
| `test` | 日常开发 | ✅ 在这里开发，推送后自动部署到测试环境（端口 8001） |
| `main` | 正式发布 | 合并测试通过的代码，用于打 tag 发布版本 |
| `prod` | 生产部署 | 部署到生产服务器（端口 8000） |

## 编写代码前检查

```bash
# 确认在 test 分支
git branch --show-current

# 如不在 test 分支
git checkout test
```

## 工作流程

```bash
# 1. 开发
git checkout test
# 编写代码
git push origin test

# 2. 发布
git checkout main
git merge test
git tag vX.Y.Z
git push origin vX.Y.Z

# 3. 部署
git checkout prod
git merge main
git push origin prod
```

## Commit 格式

使用 Conventional Commits 格式：`type: description`

| 类型 | 说明 | 版本影响 |
|------|------|----------|
| `feat` | 新功能 | minor 升级 |
| `fix` | Bug 修复 | patch 升级 |
| `feat!` / `fix!` | 重大变更 | major 升级 |
| `docs` | 文档更新 | 不触发 |
| `refactor` | 代码重构 | 不触发 |
| `test` | 测试相关 | 不触发 |
| `chore` | 构建/工具 | 不包含在 CHANGELOG |

### 示例
```
feat: 添加健康检查 API 端点
fix: 修复节点切换延迟问题
docs: 更新 CLAUDE.md 文档结构
feat!: 重构 API 接口，移除 v1 兼容性
```

## 提交规范

### Git 安全协议
- **永不**更新 git config
- **永不**运行破坏性命令（如 push --force, hard reset）除非用户明确请求
- **永不**跳过 hooks（--no-verify, --no-gpg-sign）除非用户明确请求
- **永不**强制推送到 main/master
- 避免 `git commit --amend`，仅在用户明确请求或添加 pre-commit hook 的编辑时使用
- 修改前总是检查作者：`git log -1 --format='%an %ae'`
- **永不**提交更改除非用户明确要求

### 提交消息格式

使用 HEREDOC 确保格式正确：
```bash
git commit -m "$(cat <<'EOF'
Commit message here.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
EOF
)"
```

## 质量保证

### 测试要求
- 核心业务逻辑必须有单元测试
- 使用真实数据测试，避免过度 mock
- 测试边界条件和错误场景
- 使用 `go test -race` 检测竞态条件

### 代码审查
- 所有合并到 main 的代码必须经过审查
- 检查错误处理是否完善
- 检查是否有资源泄漏（goroutine、文件句柄）
- 检查并发安全性
