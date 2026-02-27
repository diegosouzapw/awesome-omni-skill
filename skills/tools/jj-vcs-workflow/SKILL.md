---
name: jj-vcs-workflow
description: "Jujutsu (jj) VCS の総合ガイド。基本コマンド・Git移行・並列開発・履歴操作・PRレビュー・安全な push ワークフローをカバー。以下の場合に使用: (1) jj コマンドの使い方を確認したいとき (2) Git から jj への移行時 (3) 並列開発・履歴書き換え・コンフリクト解消を行うとき (4) PR レビュー対応時 (5) push を実行したいとき"
globs:
  - "**/.jj/**"
---

# Jujutsu (jj) VCS Workflow

Jujutsu (jj) を使った開発のための総合ガイド。基本操作から高度なワークフローまでをカバーする。

## 基本概念

- **Change**: jj の作業単位。Git の「コミット」に相当するが、常に編集可能
- **Working Copy (@)**: 現在編集中の Change。自動的にスナップショットされる（`git add` 不要）
- **Bookmark**: Git のブランチに相当。主にリモートとの同期ポイントとして使用
- **Operation Log**: すべての操作履歴を追跡。`jj undo` で取り消し可能

## クイックリファレンス

| 操作 | コマンド |
|-----|---------|
| 状態確認 | `jj status` / `jj st` |
| 差分表示 | `jj diff` |
| 履歴表示 | `jj log` |
| コミット | `jj commit -m "メッセージ"` |
| 説明編集 | `jj describe -m "メッセージ"` |
| 新規 Change | `jj new` |
| 取り消し | `jj undo` |
| リモート取得 | `jj git fetch` |

## push は必ず safe-push 経由で行う

`jj git push` の直接実行は禁止されている。push は必ず以下のワークフローで行う:

1. `jj git fetch` でリモートの最新状態を取得
2. `jj bookmark list --conflicted` で diverge を確認
3. `jj git push --dry-run` で push 内容を確認（hook で強制）
4. ユーザー確認後に `jj git push` を実行

hook とシェルガードにより、dry-run を経由しない push は自動的にブロックされる。
詳細は [safe-push.md](./guides/safe-push.md) を参照。

## 詳細ドキュメント

### 基本操作

- **[commands.md](./commands.md)**: 主要コマンドの詳細な使い方とオプション
- **[git-to-jj.md](./git-to-jj.md)**: Git コマンドと jj コマンドの対応表
- **[revisions.md](./revisions.md)**: リビジョン指定方法（@, @-, revset 式）

### ワークフロー

- **[workflows.md](./workflows.md)**: 新規機能開発・不具合修正のワークフロー
- **[best-practices.md](./best-practices.md)**: ベストプラクティスとトラブルシューティング

### 高度な操作

- **[guides/parallel-work.md](./guides/parallel-work.md)**: 並列開発ワークフロー（`jj new` / `jj edit`）
- **[guides/workspace.md](./guides/workspace.md)**: Workspace を使った並列開発（ファイルシステム分離）
- **[guides/history-maintenance.md](./guides/history-maintenance.md)**: 履歴の書き換え（squash, split, rebase）
- **[guides/conflict-collab.md](./guides/conflict-collab.md)**: コンフリクト解消と Git 連携
- **[guides/pr-review-workflow.md](./guides/pr-review-workflow.md)**: Bookmark を活用した PR レビュー対応
- **[guides/safe-push.md](./guides/safe-push.md)**: 安全な push ワークフローと hook 設定

## 参考リンク

- 公式ドキュメント: https://www.jj-vcs.dev/
- CLI リファレンス: https://www.jj-vcs.dev/latest/cli-reference/
- Git コマンド対応表: https://www.jj-vcs.dev/latest/git-command-table/
