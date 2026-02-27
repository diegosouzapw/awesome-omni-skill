---
name: agentic-issue-assistant
description: Install common docs/backlog skeleton plus an AGENTS template, and wrap issue/finalization operations for an agentic workflow.
---

## When to use
- When the user wants to install the agentic workflow templates into a directory.
- When the user asks to add a TODO item into backlog.
- When the user asks to create Issues from a Milestone.
- When the user asks to create Issues from TODO items.
- When the user asks to create an Issue instantly from direct instructions (without Milestone inference).
- When the user asks to execute a specific Issue with implementation/test-repair loops until completion.
- When the user asks to create or execute a Milestone Finalization Issue.

## Defaults
- repo: `.`
- m0: `repo-integration`（互換エイリアス: `integration`）

## Preconditions
- `docs/`, `backlog/`, `AGENTS.md` が未配置なら先に `install` を実行する。
- 既存ファイルがある場合は既存内容を優先し、追記・更新は運用ルールに従って最小差分で行う。

## Action selection (Wrap)
- ユーザーが「TODOの追加」を要求したら `todo-add`。
- ユーザーが「ISSUEの起票」を要求したら `issue-create-from-milestone`。
- ユーザーが「TODOからISSUE起票」を要求したら `issue-create-from-todo`。
- ユーザーが「ISSUEのインスタント起票」を要求したら `issue-create-instant`。
- ユーザーが「ISSUEの着手/実装」を要求したら `issue-implement`。
- ユーザーが「FINALIZATIONの起票」を要求したら `finalization-create`。
- ユーザーが「FINALIZATIONの実行」を要求したら `finalization-execute`。
- 複数要求が同時にある場合は、`todo-add` -> `issue-create-from-milestone` or `issue-create-from-todo` or `issue-create-instant` -> `issue-implement` -> `finalization-create` -> `finalization-execute` の順で処理する。

## Actions
### install (default)
Run:
- `python3 <skill_dir>/scripts/apply.py --repo <repo_root> --m0 <repo-integration|existing-code-analysis>`

Then remind (install後のみ):
- 選択した M0 テンプレート（`M0-Integration` / `M0-Analysis`）の初期Issue（`CHORE-001`〜`CHORE-003`）に従って、`docs/`、`AGENTS.md`、CI の整備を進める。

### todo-add（TODOの追加）
Read first:
- `backlog/TODO.md`
- `docs/ISSUE.md`

Workflow:
1. `backlog/TODO.md` の `未起票` / `ISSUE起票済み` を確認し、同一または重複の疑いがある項目を把握する。
2. 対話で必要情報を収集する。最低限 `short-title` は必須とし、`Context` / `Why now / Why later` / `Next check timing` / `Related docs/issues` は不足分を質問して埋める。
3. 情報が不足している間は、追加質問で確定してから書き込む。
4. 収集した内容を `backlog/TODO.md` の `未起票` リストへ追加する。
5. 追加後、必要なら重複候補の扱い（統合・維持）をユーザーに確認して反映する。

Constraints:
- TODO追加時は Issue を起票しない（`backlog/issues/` と `backlog/INDEX.md` は更新しない）。
- 追加先は必ず `未起票` リストとし、`ISSUE起票済み` には追加しない。
- 既存項目はユーザー合意なく削除しない。

### issue-milestone-binding（全Issue起票メソッド共通）
Read first:
- `backlog/INDEX.md`
- `docs/10_IMPLEMENTATION_PLAN.md`
- `docs/DEVFLOW.md`

Workflow:
1. `backlog/INDEX.md` から、未完了（`[ ]`）Issueを持つ最小の `M{N}` を進行中Milestoneとして確定する。
2. 未完了Issueが1件もない場合は、`docs/10_IMPLEMENTATION_PLAN.md` の定義順で次に着手するMilestoneを1つ選び、進行中Milestoneとする。
3. 新規Issueは必ず起票時点の進行中Milestone（`backlog/issues/M{N}/`）に所属させる。
4. ユーザー指示が別Milestoneを指定していても、進行中Milestoneに補正して起票する。

### issue-create-from-milestone（Milestone根拠の起票）
Read first:
- `docs/10_IMPLEMENTATION_PLAN.md`
- `docs/DEVFLOW.md`
- `docs/ISSUE.md`
- `backlog/ISSUE-template.md`
- `backlog/INDEX.md`

Workflow:
1. `issue-milestone-binding` を実行して、起票先Milestoneを確定する。
2. 確定したMilestoneの `Purpose` / `Exit criteria` / `Risks` / `Notes` から実装単位を推測し、通常Issueに分解する。
3. `ISSUE-ID` を重複なく採番し、`backlog/issues/M{進行中N}/<ID>_<slug>.md` を作成する。
4. 各Issueは `Goal` / `Scope` / `前提ISSUE` / `Acceptance Criteria` / `Tests` / `DoD` を埋める。
5. `NFR影響` が必要な場合のみ記載し、追加対応が必要なら追加ISSUEと依存を定義する。
6. `backlog/INDEX.md` に未完了（`[ ]`）で追記する。

Constraints:
- 既存Issue本文は原則変更しない（新規作成とINDEX更新を優先）。
- 進行中Milestone以外には起票しない。

### issue-create-from-todo（TODO根拠の起票）
Read first:
- `backlog/TODO.md`
- `docs/ISSUE.md`
- `docs/DEVFLOW.md`
- `backlog/ISSUE-template.md`
- `backlog/INDEX.md`

Workflow:
1. `issue-milestone-binding` を実行して、起票先Milestoneを確定する。
2. `backlog/TODO.md` から「今やる」と決めた項目を選ぶ（ユーザー指定があればそれを優先）。
3. TODO項目の `Context` / `Why now / Why later` / `Related docs/issues` を根拠に、`Goal` / `Scope` / `Acceptance Criteria` / `Tests` を定義する。
4. `ISSUE-ID` を重複なく採番し、`backlog/issues/M{進行中N}/<ID>_<slug>.md` を作成する。
5. 起票した項目を `backlog/TODO.md` の `未起票` から `ISSUE起票済み` へ移動し、`backlog/INDEX.md` に未完了（`[ ]`）で追記する。

Constraints:
- 進行中Milestone以外には起票しない。
- 起票したTODO項目は必ず `未起票` から `ISSUE起票済み` へ移動する。

### issue-create-instant (ISSUEのインスタント起票)
Read first:
- `docs/ISSUE.md`
- `docs/DEVFLOW.md`
- `backlog/ISSUE-template.md`
- `backlog/INDEX.md`

Workflow:
1. `issue-milestone-binding` を実行して、起票先Milestoneを確定する。
2. ユーザーのその場指示から `Goal` / `Scope` / `Acceptance Criteria` / `Tests` を直接抽出する。
3. `ISSUE-ID` を重複なく採番し、`backlog/issues/M{進行中N}/<ID>_<slug>.md` を作成する。
4. テンプレートに沿って本文を埋め、必要なら `前提ISSUE` と `NFR影響` を設定する。
5. `backlog/INDEX.md` に未完了（`[ ]`）で追記する。

Constraints:
- `docs/10_IMPLEMENTATION_PLAN.md` からIssue内容を推測しない（その場指示を優先）。
- `docs/ISSUE.md` の命名・依存・完了判定ルールは必ず遵守する。
- 進行中Milestone以外には起票しない。

### issue-implement (ISSUEの着手と実装)
Read first:
- 指定ISSUE-IDのファイル
- Issue本文の `Docs` に列挙された文書
- `AGENTS.md`
- `docs/06_TESTING.md` / `docs/08_RUNBOOK.md` / `docs/07_SECURITY.md`

Workflow:
1. `前提ISSUE` の完了を確認する（未完了なら先に前提Issueを処理）。
2. Issueの `Scope` に限定して実装する。
3. テストを実行する（Issue `Tests`、CI DoD、必要な手動確認）。
4. 失敗時は原因を修復して再実行し、全Passまで `実装 -> テスト -> 修復` を反復する。
5. `Acceptance Criteria` と `DoD` を満たしたら、必要なDocs更新と `backlog/INDEX.md` の完了反映（`[x]`）を行う。

Constraints:
- 仕様変更が必要なら別Issueを起票して範囲外作業を分離する。
- `NFR影響` で追加対応が必要なら、追加ISSUEを起票し依存関係を定義する。

### finalization-create (FINALIZATIONの起票)
Read first:
- `docs/10_IMPLEMENTATION_PLAN.md`
- `docs/DEVFLOW.md`
- `docs/ISSUE.md`
- `backlog/FINALIZATION-template.md`
- `backlog/INDEX.md`

Workflow:
1. 対象Milestoneを決める（M1以降のみ）。
2. 当該Milestoneに既存 `FINALIZATION-###` がないことを確認する。
3. 重複しない `FINALIZATION-###` を採番する。
4. `backlog/issues/M{N}/FINALIZATION-###_<slug>.md` をテンプレートから作成する。
5. `backlog/INDEX.md` に未完了（`[ ]`）で追記する。

Constraints:
- M0-Integration / M0-Analysis では起票しない。
- 判定対象は「当該Milestoneの `FINALIZATION-###` 以外の全Issue（`FIX-###` 含む）」で固定する。

### finalization-execute (FINALIZATIONの実行)
Read first:
- 指定 `FINALIZATION-###` Issue
- `docs/ISSUE.md`
- `docs/DEVFLOW.md`
- `docs/06_TESTING.md`
- `docs/07_SECURITY.md`
- `docs/08_RUNBOOK.md`
- `backlog/INDEX.md`

Workflow:
1. 判定対象Issue（同Milestoneの `FINALIZATION-###` 以外）を確定する。
2. 未完了Issueがあれば先に完了させる（必要なら `FIX-###` を追加起票して解消）。
3. セキュリティ/テスト/運用手順の最終確認を行う。
4. 不備があれば修復し、検証を再実行する。
5. 判定対象が全完了であることを確認した直後に `FINALIZATION-###` を完了扱いにする。
6. `backlog/INDEX.md` と必要Docsを更新し、完了記録を残す。

Constraints:
- 判定対象Issueが1件でも未完了なら `FINALIZATION-###` を完了にしない。
- Finalization完了後の同Milestone新規Issueは原則禁止。例外時は `docs/09_DECISIONS.md` に理由を記録する。
