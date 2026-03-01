---
name: commit-quick
description: codemapの更新をスキップして素早くgit commitする。変更内容を分析し、適切なコミットメッセージを自動生成してコミットを作成する。
---

# Commit Quick Skill

codemap更新をスキップして素早くコミットするスキル。

## 使用タイミング

- 小さな変更を素早くコミットしたい時
- codemap更新が不要な変更の時
- ユーザーから「クイックコミット」と依頼された時

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

### Step 3: ファイルをステージング

```bash
# 関連ファイルを個別に追加（機密ファイルを避ける）
git add <file1> <file2> ...
```

注意:
- `.env`、`credentials.json` などの機密ファイルは絶対に追加しない
- `git add -A` や `git add .` は避け、ファイルを明示的に指定する

### Step 4: コミット作成

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

### Step 5: 結果確認

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
- **codemapの更新は行わない**（素早いコミットを優先）
