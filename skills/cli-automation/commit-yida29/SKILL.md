---
name: commit
description: diff分析 → 論理的に分割コミット。conventional commitメッセージを自動生成。
disable-model-invocation: true
user-invocable: true
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git log:*), Bash(git add:*), Bash(git reset:*), Bash(git commit:*)
argument-hint: "[message or instructions]"
---

# Smart Commit — diff分析 → 分割コミット

変更内容を分析し、論理的にまとまった単位で分割コミットする。

## コンテキスト

<git_status>
!`git status`
</git_status>

<git_diff>
!`git diff HEAD`
</git_diff>

<recent_commits>
!`git log --oneline -15`
</recent_commits>

<current_branch>
!`git branch --show-current`
</current_branch>

## ワークフロー

### Step 1: 変更の分析

上記の diff を読み、変更を **論理的なグループ** に分類する。

分類の基準:
- **機能単位**: 1つの機能追加/変更に関するファイル群
- **レイヤー単位**: ドメイン層、インフラ層、API層、テストなど
- **目的単位**: リファクタ、バグ修正、新機能、ドキュメント、設定変更

### Step 2: 分割プランの提示

以下のフォーマットでユーザーに分割プランを提示する:

```
## コミット分割プラン

### Commit 1: <type>(<scope>): <summary>
- `path/to/file1.ts`
- `path/to/file2.ts`

### Commit 2: <type>(<scope>): <summary>
- `path/to/file3.ts`
- `path/to/file4.ts`

（変更が1グループのみなら分割せず1コミットで良い）
```

**重要**: プランを提示したら、ユーザーの承認を待つ。勝手にコミットしない。

### Step 3: ユーザー承認後、順次コミット

承認されたら、各コミットを順番に実行する:

1. `git add <files>` で対象ファイルのみステージ
2. `git commit` でコミット作成
3. 次のコミットへ

### Step 4: 完了報告

全コミット完了後、`git log --oneline -N` で作成したコミットを表示する。

## コミットメッセージ規約

### フォーマット

```
<type>(<scope>): <summary>
```

- **type**: `feat`, `fix`, `docs`, `refactor`, `chore`, `test`, `perf`, `style`, `ci`
- **scope**: 影響範囲（省略可）。例: `ui`, `auth`, `api`
- **summary**: 英語、命令形、72文字以内、末尾ピリオドなし

### このリポジトリのscope例

| scope | 対象 |
|-------|------|
| `pet` | デスクトップペット (`pet/`) |
| `daemon` | デーモン (`daemons/`) |
| `protocol` | 通信プロトコル (`daemons/socket_protocol.py`) |
| `scripts` | スクリプト (`scripts/`, `start.sh`, `stop.sh`) |
| `docs` | ドキュメント (`CLAUDE.md`, `instructions/`, `docs/`) |
| `config` | 設定 (`config/`, `pet/config.py`) |

### 日本語の場合

このリポジトリの `git log` に日本語コミットが多い場合は、日本語で書いても良い。
直近のコミット履歴のスタイルに合わせること。

## 制約

- **pushしない**: コミットのみ。pushはユーザーが明示的に指示した場合のみ。
- **Co-Authored-By**: コミットメッセージ末尾に以下を付与:
  ```
  Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
  ```
- **HEREDOC**: コミットメッセージは必ず HEREDOC で渡す:
  ```bash
  git commit -m "$(cat <<'EOF'
  <type>(<scope>): <summary>

  Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
  EOF
  )"
  ```
- **秘密ファイル**: `.env`, `credentials.json` 等を検知したら警告し、コミットしない。
- **空コミット**: 変更がない場合は空コミットを作成しない。
- **`--amend` 禁止**: 常に新規コミットを作成する（ユーザーが明示的に指示した場合を除く）。
- **引数の尊重**: `$ARGUMENTS` が指定された場合:
  - ファイルパス/globならそのファイルのみ対象
  - テキストならコミットメッセージのヒントとして使用
  - 両方なら両方を尊重

## 分割の判断基準

### 分割すべきケース
- 新機能の追加 + 既存コードのリファクタが混在
- 複数の独立した機能変更
- 型定義の変更 + それを使う実装の変更（レイヤー単位で分けたい場合）
- テストの追加 + 実装の変更

### 分割しなくてよいケース
- 1つの機能に関する一連の変更（型 + 実装 + テスト）
- 小さなリファクタ（ファイル数が少ない）
- 設定ファイルの単独変更
