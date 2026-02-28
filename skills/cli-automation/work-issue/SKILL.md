---
name: work-issue
description: issues/ディレクトリのタスクを1つ実装し、ブランチ作成・コミット・PR作成を行なうワークフロー
globs:
  - "issues/**/*.md"
---

# Issue Workflow Skill

issues/ディレクトリのタスクを処理し、1タスク = 1ブランチ、1コミット、1PRの形式で完了する。

## ワークフロー（ループ処理）

issueがなくなるまで以下を繰り返す:

### 1. タスク取得
- `issues/`ディレクトリの`.md`ファイル（.gitkeep除く）を検索
- 最初に見つかったファイルを自動選択
- **issueがない場合はループ終了**

### 2. ブランチ作成
- mainブランチを最新化
- issue名ベースでブランチを作成（`issue/{issue-name}`）

### 3. 実装
- issueファイルの内容を読み込み
- 指示に従って実装を行う
- CLAUDE.mdのルールに従う
- pnpm check:fixを行なう

### 4. コミット
- issueファイルを削除
- 変更をステージング
- 1コミットで完了

### 5. PR作成
- リモートにプッシュ
- `gh pr create`でPR作成

### 6. mainに戻る
- mainブランチに戻る
- **ステップ1に戻り、次のissueを処理**

## 制約
- **`gh pr create`のみ許可**: 他のghコマンドは禁止
- **ブランチ作成・切り替えを許可**: issue単位でブランチを作成
- **1issue = 1ブランチ、1コミット、1PR**

## コマンド例

```bash
# 1. ブランチ作成
git checkout main
git pull origin main
git checkout -b issue/{issue-name}

# 2. 実装後
git add {changed-files}
rm issues/{issue-name}.md
git add issues/{issue-name}.md
git commit -m "..."

# 3. PR作成
git push -u origin issue/{issue-name}
gh pr create --base main --title "..." --body "..."

# 4. mainに戻る
git checkout main
# ループ継続...
```

## 注意事項
- issueファイルは実装完了後に削除する
- 削除はコミットに含める
- **すべてのissueを処理するまで継続する**
- **mainブランチへのpushやその他操作はいっさい行わないこと**
