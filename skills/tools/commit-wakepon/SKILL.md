---
name: commit
description: 現在の変更をgit commitする。変更内容を分析し、適切なコミットメッセージを自動生成してコミットを作成する。
---

# Commit Skill

現在の作業内容をgit commitするスキル。

## 使用タイミング

- 機能実装が完了した時
- バグ修正が完了した時
- リファクタリングが完了した時
- ユーザーから「コミットして」と依頼された時

## 実行手順

### Step 1: 状態確認（並列実行）

```bash
# 変更されたファイルを確認
git status

# 変更内容を確認
git diff

# 最近のコミットスタイルを確認
git log --oneline -5
```

### Step 2: 変更内容の分析

変更されたファイルとdiffから以下を判断:
- 変更の種類（feat, fix, refactor, docs, test, chore, perf, ci）
- 変更の概要
- 主要な変更点のリスト

### Step 3: Codemap 更新（src/ に変更がある場合）

Step 1 の `git status` の結果に `src/` 以下のファイル変更が含まれている場合:
1. `/update-codemaps` スキルを実行して codemap を再生成する
2. 生成された `codemaps/` 以下の変更もコミット対象に含める

`src/` 以下に変更がない場合（docs, chore 等）はこのステップをスキップする。

### Step 4: ファイルをステージング

```bash
# 関連ファイルを個別に追加（機密ファイルを避ける）
# codemap が更新された場合は codemaps/ 以下も含める
git add <file1> <file2> ... codemaps/architecture.md codemaps/data.md
```

注意:
- `.env`、`credentials.json` などの機密ファイルは絶対に追加しない
- `git add -A` や `git add .` は避け、ファイルを明示的に指定する

### Step 5: コミット作成

```bash
git commit -m "$(cat <<'EOF'
<type>: <簡潔な説明>

- 変更点1
- 変更点2
- 変更点3

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
EOF
)"
```

### Step 6: 結果確認

```bash
git status
```

## コミットメッセージ形式

```
<type>: <description>

<optional body>

Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>
```

### Type の種類

| Type | 説明 |
|------|------|
| feat | 新機能 |
| fix | バグ修正 |
| refactor | リファクタリング（機能変更なし） |
| docs | ドキュメントのみの変更 |
| test | テストの追加・修正 |
| chore | ビルドプロセスやツールの変更 |
| perf | パフォーマンス改善 |
| ci | CI設定の変更 |

## 出力形式

コミット完了後、以下を報告:

```
コミットが完了しました。

**コミット内容:**
- **ハッシュ:** <commit-hash>
- **ブランチ:** <branch-name>
- **変更ファイル:** X ファイル（+Y行、-Z行）
- **新規ファイル:** （あれば）
```

## 注意事項

- ユーザーから明示的に依頼されない限りコミットしない
- `--no-verify` や `--force` は使用しない
- コミット前にビルドやテストが通ることを確認済みであることが望ましい
- pushはユーザーから明示的に依頼されない限り行わない
