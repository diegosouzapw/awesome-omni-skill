---
name: agent-skill-mapping
description: Skillの割当表（agent -> skills）を作るときのルール集。skill-assigner が mapping を更新するときに参照する。
---

# Agent-Skill Mapping Rules

## Inputs
- `.claude/agents/*.md`
- `.claude/skills/*/SKILL.md`

## Outputs
- `.claude/agent-skill-map.yaml`
- `.claude/agent-activation-rules.md`

## Rules
- 重複を減らす：同じ手順系Skillを複数agentへ持たせない
- 依存関係を明示：impl-* は検証（tests/lint）とセットにする
- 危険操作は隔離：必要なら専用agent + human review 必須

## Verification
- `agent-skill-map.yaml` がスキル名のタイポなしで参照していること
- `activation rules` に「どの入力でどのagentを起動するか」が書かれていること

## Evidence
- N/A (bootstrap skill)
