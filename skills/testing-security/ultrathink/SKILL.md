---
name: ultrathink
description: Use /ultrathink <TASK_DESCRIPTION> to launch a Coordinator Agent that directs four specialist sub-agents—Architect, Research, Coder, and Tester—to analyze, design, implement, and validate your codi...
version: 1.0.0
author: Jeronim Morina
source: Claude Code Marketplace
keywords: plan, explore, refactoring, testing, code-review
---

# ultrathink

Use /ultrathink <TASK_DESCRIPTION> to launch a Coordinator Agent that directs four specialist sub-agents—Architect, Research, Coder, and Tester—to analyze, design, implement, and validate your codi...

## 来源信息

- **原始平台**: Claude Code
- **市场来源**: Claude Code Marketplace
- **原始名称**: ultrathink
- **版本**: 1.0.0
- **作者**: Jeronim Morina
- **关键词**: plan, explore, refactoring, testing, code-review, workflow, debugging

## 功能描述

## Usage

`/ultrathink <TASK_DESCRIPTION>`

## Context

- Task description: $ARGUMENTS
- Relevant code or files will be referenced ad-hoc using @ file syntax.

## Your Role

You are the Coordinator Agent orchestrating four specialist sub-agents:
1. Architect Agent – designs high-level approach.
2. Research Agent – gathers external knowledge and precedent.
3. Coder Agent – writes or edits code.
4. Tester Agent – proposes tests and validation strategy.

## Process

1. Think step-by-step, laying out assumptions and unknowns.
2. For each sub-agent, clearly delegate its task, capture its output, and summarise insights.
3. Perform an "ultrathink" reflection phase where you combine all insights to form a cohesive solution.
4. If gaps remain, iterate (spawn sub-agents again) until confident.

## Output Format

1. **Reasoning Transcript** (optional but encouraged) – show major decision points.
2. **Final Answer** – actionable steps, code edits or commands presented in Markdown.
3. **Next Actions** – bullet list of follow-up items for the team (if any).

## 使用方法

1. **自动触发**: Codex 会根据任务描述自动选择并使用此技能
2. **手动指定**: 在提示中提及技能名称或相关关键词
3. **斜杠命令**: 使用 `/skills` 命令查看并选择可用技能

## 兼容性

- ✅ Codex CLI
- ✅ Codex IDE 扩展
- ✅ 基于 Agent Skills 开放标准

---
*此技能由 Claude Code 插件自动转换，已适配 Codex 官方技能系统*
