---
name: create-pr
description: JIRAチケットブランチからPRを自動作成するスキル。親ブランチ自動検出、JIRAリンク自動挿入、テンプレートに沿ったPR本文生成を行う。「PR作成」「PRを作って」「/create-pr」などのリクエストで使用。
---

# Create PR Skill

JIRAチケット番号のブランチからPull Requestを自動作成する。親ブランチの自動検出、JIRAリンクの自動挿入、テンプレートに沿った本文生成を行う。

## 使用方法

```
/create-pr
```

引数なしで実行。現在のブランチ名とgit履歴から全て自動判定する。

## 実行手順

### Step 1: 事前チェック

1. `git branch --show-current` でブランチ名を取得
2. ブランチ名が `main` または `develop` の場合はエラー終了:
   - 「main/developブランチ上ではPRを作成できません。フィーチャーブランチに切り替えてください。」
3. `gh auth status` で認証状態を確認。未認証ならエラー:
   - 「gh CLIが未認証です。`gh auth login` を実行してください。」
4. `gh pr list --head {ブランチ名} --json url --jq '.[0].url'` で既存PRを確認。存在する場合:
   - 「既存のPRが見つかりました: {URL}」と表示して終了
5. `git status --porcelain` で未コミット変更を確認。ある場合:
   - 「未コミットの変更があります。先にコミットすることを推奨します。」と警告（続行は可能）

### Step 2: 親ブランチ検出

以下の優先順で親ブランチを特定する:

#### 方法A: merge-base距離比較（推奨）

```bash
git fetch origin
CURRENT=$(git branch --show-current)
git branch -r --list 'origin/*' | grep -v "origin/$CURRENT" | grep -v 'origin/HEAD' | while read branch; do
  echo "$(git rev-list --count $(git merge-base HEAD $branch)..HEAD) $branch"
done | sort -n | head -1 | awk '{print $2}' | sed 's|origin/||'
```

最もmerge-baseが近い（= 分岐元に近い）リモートブランチを親とする。

#### 方法B: 直前のブランチ

方法Aで結果が得られない場合:

```bash
git rev-parse --abbrev-ref @{-1}
```

#### フォールバック

いずれも失敗した場合、AskUserQuestionでユーザーに確認する:
- 「親ブランチ（PRのマージ先）を指定してください」
- 選択肢: `main`, `develop`, その他

### Step 3: 差分収集

```bash
git diff origin/{parent}...HEAD --stat
git log origin/{parent}..HEAD --oneline
```

差分の内容とコミットメッセージを確認し、変更の概要を把握する。

### Step 4: PR本文生成

テンプレート（`templates/pr-template.md`）の構造に従い、PR本文を構築する:

- **概要**: 差分とコミットログから日本語で要約を自動生成
- **関連リンク**: `https://kokopelli-inc.atlassian.net/browse/{ブランチ名}` を自動挿入
- **動作確認**: 空欄のまま
- **特に見て欲しいところ**: 空欄のまま

### Step 5: プッシュ

リモートにブランチが存在しない場合:

```bash
git push -u origin {ブランチ名}
```

### Step 6: PR作成

```bash
gh pr create --base {parent} --title "{ブランチ名}: {変更の要約}" --body "$(cat <<'EOF'
# 📝 概要

{自動生成した要約}

# 🔗 関連リンク

- https://kokopelli-inc.atlassian.net/browse/{ブランチ名}

# 🧪 動作確認



# 👀 特に見て欲しいところ

EOF
)"
```

### Step 7: 結果表示

作成されたPRのURLを表示する。

## PRタイトル

- フォーマット: `{ブランチ名}: {変更の要約}`
- 例: `BAX-12345: チャットメンバー表示の不整合を修正`
- 要約は日本語で簡潔に（70文字以内）

## エラーハンドリング

| 状況 | 対応 |
|------|------|
| `main`/`develop`上で実行 | エラーメッセージを表示して終了 |
| 未コミット変更あり | 警告を表示（続行可能） |
| 既存PRが存在 | 既存PRのURLを表示して終了 |
| `gh` CLI未認証 | エラーメッセージを表示して終了 |
| 親ブランチ検出失敗 | ユーザーに確認 |
