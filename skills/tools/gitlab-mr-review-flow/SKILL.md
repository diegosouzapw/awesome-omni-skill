---
name: gitlab-mr-review-flow
description: 标准化“需求描述 → 规范检索 (QMD) → 实现改动 → GitLab MCP 创建 MR → 代码评审报告”的流程技能。
---

# GitLab MR Review Flow with QMD & Rules

## Overview

规范化执行从需求拆解到 MR 创建与代码评审的全流程。
**核心增强**：
1.  **QMD 集成**：在开发前自动检索相关的工程规范与文档。
2.  **GitLab MCP**：强制使用 MCP 工具管理 MR。
3.  **中文评审报告**：按规范输出标准化报告。

## Workflow Decision Tree

- 需要检索内部规范/文档 → 使用 `qmd` skill (search/get)。
- 需要隔离环境/并行修改 → 使用 `using-git-worktrees` skill。
- 涉及任何 git 操作 → 必须使用 `git-master` skill。
- 需要创建/更新 MR → 必须使用 GitLab MCP。
- 需要评审报告 → 使用 `requesting-code-review` skill 并输出中文报告。

## Step 1. Gather Inputs & Context

收集并确认：
- 仓库路径与目标模块范围
- 需求来源（文档/issue/日志）与验收标准
- **关键词提取**：用于 QMD 检索的关键词 (e.g., "API 规范", "数据库设计", "错误处理").

## Step 2. Retrieve Norms & Documentation (QMD)

在动手写代码前，先检索相关的业务或技术规范。

1.  **Search**: 根据提取的关键词检索文档。
    ```bash
    # 示例
    qmd search "API 规范"
    qmd search "数据库命名"
    ```
2.  **Read**: 读取最相关的 1-2 份文档，提取对本次开发有约束力的条款。
    ```bash
    qmd get "path/to/relevant-doc.md"
    ```
3.  **Context Injection**: 将提取到的规范点记录在上下文中，后续用于 Self-Check 和 Code Review。

## Step 3. Prepare Workspace & Implement

1.  读取仓库内 `AGENTS.md` 约束。
2.  如需隔离，使用 `using-git-worktrees`。
3.  实现改动，保持原子提交。
4.  执行测试并记录结果。

## Step 4. Branching & Push

### 分支命名规则
*   **Target**: `fix-{feature}-mr` 或 `feat-{feature}-mr` (空分支)
*   **Source**: `fix-{feature}` 或 `feat-{feature}` (含改动)

### 操作流程
```bash
# 自动创建 Target 和 Source 分支
./scripts/start_feature.sh "{feature-name}"
```

## Step 5. Create MR via GitLab MCP

必须使用 GitLab MCP。禁止浏览器操作。

**MR 描述模板** (需包含根因、改动、**以及遵循的规范**):
(Script `create_mr.sh` contains the template)

Automatically create MR using the script:

```bash
# Usage: ./scripts/create_mr.sh "MR Title"
export GITLAB_PERSONAL_ACCESS_TOKEN="<your-token>"
./scripts/create_mr.sh "feat: <Title>"
```

## Step 6. Code Review Report (中文)

调用 `requesting-code-review` skill，结合 Step 2 获取的规范进行评审。

**评审重点**:
1.  是否符合 QMD 检索到的规范？
2.  是否存在逻辑漏洞？
3.  代码风格与最佳实践。

**报告模板**:

```markdown
**AI Review:** {YYYYMMDD}-{BUILD_ID}
**参考规范库**: {列出参考的 QMD 文档}

对比基准： {BASE_BRANCH}

一、规范符合度检查 (Based on QMD)
- [ ] 规范A: {检查结果}
- [ ] 规范B: {检查结果}

二、严重问题
...

三、中等问题
...

四、总结
...
```

## Step 7. Inline Code Comments (Discussions)

对于规范违例，**必须**在 MR Diff 中进行行级评论。

1.  **Locate Violation**: Identify file and line number.
2.  **Post Discussion**: Run the helper script.

```bash
# Usage: ./scripts/post_comment.sh <file> <line> "<comment>"
./scripts/post_comment.sh "bad_code.py" 10 "❌ Violation: ..."
```
