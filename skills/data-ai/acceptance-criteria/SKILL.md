---
name: acceptance-criteria
description: 受け入れ基準書を作成する。検収基準、完了条件の定義時に使う。
phase: planning
pmbok-area: scope
---

## 目的

プロジェクト成果物の受け入れ条件を明確に定義する。

## トリガー語

- 「受け入れ基準を作成」
- 「検収基準を定義」
- 「完了条件を策定」

---

## 入力で最初に聞くこと

| # | 質問 | 必須 |
|---|------|------|
| 1 | **プロジェクト名**は？ | ✓ |
| 2 | **対象成果物**は？ | ✓ |

---

## 手順

### Step 1: 成果物の特定

### Step 2: 各成果物の受け入れ基準定義
- 機能要件の充足
- 非機能要件の充足
- ドキュメント要件

### Step 3: 検証方法の定義

### Step 4: 保存
- `workspace/{ProjectName}/docs/AcceptanceCriteria.md`

---

## 成果物

| 成果物 | 保存先 |
|--------|--------|
| 受け入れ基準書 | `workspace/{ProjectName}/docs/AcceptanceCriteria.md` |

---

## 検証（完了条件）

- [ ] 各成果物に受け入れ基準が設定されている
- [ ] 検証方法が明記されている
- [ ] 承認者が特定されている

---

## 参照

- Command: `.claude/commands/02_aipjm_02_planning_11_acceptance.md`
