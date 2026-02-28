---
name: claude-updates
description: Fetch and analyze Claude/Anthropic RSS feeds to summarize recent updates and propose improvements to CLAUDE.md, skills, and workflows.
disable-model-invocation: true
user-invocable: true
allowed-tools: Bash, Read, Write, Edit, Glob, Grep, WebFetch
---

# RSS フィードチェック

Claude / Anthropic の最新情報を RSS フィードから取得・分析し、活用提案を行う。

## 実行フロー

### 1. フィードの取得

まずフィード取得スクリプトを実行する:

```bash
bash ~/.dotfiles/scripts/fetch-rss.sh
```

### 2. フィードの読み込みと新着抽出

`~/.claude/rss/` 配下の XML ファイルを読み込み、直近7日間の新着エントリを抽出する。

各フィードから以下の情報を抽出:
- タイトル
- 公開日
- 概要またはコンテンツ
- リンク

### 3. 新着情報の要約

抽出した新着エントリを以下のカテゴリに分類して要約する:

| カテゴリ | 対象フィード |
|---------|------------|
| 新機能・モデル更新 | Anthropic News, Claude Blog |
| Claude Code の変更 | Claude Code Changelog, Claude Code Releases |
| 技術・研究 | Anthropic Engineering, Anthropic Research |
| 障害・メンテナンス | Claude Status |

要約は以下の形式で提示する:

```
## 直近の更新情報（YYYY-MM-DD 時点）

### 新機能・モデル更新
- [タイトル](リンク) - 概要1〜2文

### Claude Code の変更
- [タイトル](リンク) - 概要1〜2文

（該当なしのカテゴリは省略）
```

### 4. 影響分析と改善提案

新着情報をもとに、以下の観点で改善提案を行う:

1. **CLAUDE.md の更新** — 新機能や非推奨化により共通指示書を更新すべき箇所
2. **スキルの追加・更新** — 新しい Claude Code 機能を活用するスキルの提案
3. **ワークフローの改善** — 作業効率を上げる新しい使い方の提案
4. **settings.json の更新** — 新しいツールや権限設定の反映

提案は以下の形式で提示する:

```
## 改善提案

| # | 種類 | 提案内容 | 根拠となる更新 |
|---|------|---------|--------------|
| 1 | CLAUDE.md | ... | ... |
| 2 | スキル | ... | ... |

適用しますか？（番号で選択、複数可。0 = スキップ）
```

### 5. 提案の適用

ユーザーが選択した提案を実際に適用する:
- CLAUDE.md やスキルファイルの編集
- 変更後はユーザーに差分を確認してもらう

### 6. 確認履歴の記録

チェック結果を `~/.claude/rss/history.log` に追記する:

```
YYYY-MM-DD: チェック完了 / 新着N件 / 適用した提案の概要
```

## 新着がない場合

直近7日間に新着エントリがない場合は、その旨を簡潔に伝えて終了する。

## 注意事項

- フィード取得に失敗した場合は、失敗したフィードを報告して続行する
- 提案はあくまで提案であり、適用はユーザーの承認後に行う
- 大きな変更（CLAUDE.md の構造変更など）は差分を見せてから適用する
