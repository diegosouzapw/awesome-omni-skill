---
name: skill-creator-from-logs
description: VS Code / GitHub Copilot Chat の「Chat: Export Chat...」で出力した JSON を解析し、繰り返しワークフロー（WHAT/HOW/FLOW）を発見して Agent Skills を自動生成する。"analyze my chat history", "find repetitive tasks", "create skills from logs", "Copilot Chat の履歴からスキルを作って", "VS Code Chat Export を分析して"などのリクエスト時に使用される。
license: Apache-2.0
---

# GitHub Copilot Chat のエクスポート履歴からスキルを生成

このスキルは **VS Code の公式機能（Chat: Export Chat...）で書き出した JSON** を一次ソースとして扱い、
繰り返しタスクのパターンを抽出して新しい Agent Skills を自動生成します。

## 安全性とプライバシー

- エクスポート JSON には、ソースコード断片・ログ・秘密情報が含まれる可能性があります。**生ログをそのままスキルに貼り付けない**
- 読み取りを開始する前に、必ず「読み取るファイル（Export JSON のパス）」を明示し、ユーザーの明確な承認を得る
- スキル本文・使用例に、トークン/パスワード/APIキー/秘密のURL/個人情報/ユーザー固有の絶対パスを含めない

## ワークフロー概要

以下の 6 フェーズを順番に実行します。

```
スキル生成進捗:
- [ ] Phase 1: データ収集（同意取得→Export JSON 縮約→Read）
- [ ] Phase 2: タスクパターン抽出（WHAT/HOW/FLOW の三軸抽出）
- [ ] Phase 3: スキル適性評価（スコアリング→ランク付け→候補提示）
- [ ] Phase 4: ユーザー選択（スキル化対象の決定→スコープ確認）
- [ ] Phase 5: スキル生成（SKILL.md + 補助ファイルの作成）
- [ ] Phase 6: 品質検証（チェックリスト確認）
```

## Phase 1: データ収集

### 同意ゲート（必須）

読み取りを開始する前に、ユーザーに以下を伝えて承認を得ます。

- 対象: VS Code Chat Export JSON（単体ファイル or ディレクトリ）
- 分析スコープ: どのワークスペース/プロジェクトの履歴か（`--workspace` の値）
- 時間範囲: 過去 N 日（`--days`）
- 読み取るファイル: 具体的なパス（例: `exports/*.json`）

### 収集と縮約（推奨）

Export JSON はサイズが大きくなりやすい（tool 実行結果やコード断片が大量に入ることがある）ため、
ファイル読み込みの制限を避ける目的で **縮約（フィルタ）→ JSONL 化** してから読みます。

1) ユーザーに、Export した JSON の場所を用意してもらう（例: `exports/` に集約）

2) 同梱スクリプトで縮約します:

```bash
python3 {skill-dir}/scripts/filter-vscode-chat-export.py \
  --input "{export-path-or-dir}" \
  --workspace "{project-path}" \
  --days {N} \
  --noise-filter \
  > /tmp/vscode-chat-filtered.jsonl
```

- 出力が大きい場合は `--max-turns` / `--max-assistant-chars` を下げて再実行する
- 失敗した場合は「どのファイルが大きいか」を特定し、対象セッションを絞って Export し直す

3) `/tmp/vscode-chat-filtered.jsonl` を Read して Phase 2 に渡す

### データモデル（抽象）

縮約後は「1行=1 turn（概ね user/assistant のペア）」の JSONL として扱います。
最低限、以下が取れればよいです。

- `timestamp`（あれば）
- `workspace`（または推定できるプロジェクト識別子）
- `user_text`
- `assistant_text`
- `source_file`（どの Export JSON 由来か）

> フィールド名は Export 形式で揺れるため、スクリプト側で best-effort に正規化しています。

## Phase 2: タスクパターン抽出

縮約した会話（user/assistant）から、具体的な繰り返しタスクパターンを抽出します。

### 三軸抽出（重要）

- **WHAT**: 何を達成したいか（例: 「PR レビュー指摘を一括で直す」）
- **HOW**: どう進めるか（例: 「差分だけ出す」「テスト駆動で修正する」「チェックリストで検証する」）
- **FLOW**: セッション内の作業連鎖（例: 「調査→修正→テスト→コミット」）

**ルール:** HOW を WHAT に丸めない。FLOW は複数ステップの連鎖として独立に抽出します。

### ノイズ除外（推奨）

- 形式だけの相づち（例: 「OK」「ありがとう」）
- 変化が少ない繰り返し（例: 同じ質問の言い換えのみ）
- 大量の貼り付け（コード/ログ）が主体で、指示（WHAT/HOW）が読み取れない turn

### FLOW 抽出

1. session 相当（Export ファイル単位、または timestamp の近接で）で会話をまとめる
2. そのまとまりごとに「作業の流れ」を 1 行で要約（矢印表現）
3. 似た流れが 3 回以上出るものを FLOW パターンとして記録

詳細な粒度例は [references/pattern-extraction-examples.md](references/pattern-extraction-examples.md) を参照してください。

## Phase 3: スキル適性評価

### ステップ 0: 既存スキルとの照合

ユーザー環境にインストール済みのスキルを列挙し、抽出パターンとの重複を確認します。

1. `.github/skills/` 配下のスキルを Glob で列挙
2. 各スキルの SKILL.md の `name` / `description` を読み取り
3. 抽出パターンを以下に分類:
   - **完全カバー済み** → 候補から除外
   - **部分カバー** → 未カバー部分のみ候補として残す
   - **未カバー** → そのまま候補

### ステップ 1〜3: スコアリング→ランク付け→候補提示

内部的に「頻度」「一貫性」「自動化可能なステップ数」で優先度を付けます。
ただし **内部スコアの詳細はユーザーに見せません**。

提示フォーマット:

```
スキル化推奨:
1. [パターン名] (N回) — [1行説明]

スキル化の価値あり:
2. [パターン名] (N回) — [1行説明]
```

## Phase 4: ユーザー選択

1. ユーザーにスキル化対象のパターンを選んでもらう（複数可）
2. 選択パターンに該当する代表 turn（サニタイズ前提）を 2〜3 件提示してスコープを確認
3. 以下を確定:
   - スキルのスコープ（どこからどこまで）
   - バリエーションの扱い（オプション化 or 別スキル）
   - 配置先: `.github/skills/` 配下
   - トリガーフレーズ

## Phase 5: スキル生成

### 重要ルール

- 生の会話ログをそのまま貼り付けない
- シークレット/個人情報/絶対パスを含めない（必要ならテンプレート化）
- YAML フロントマターに山括弧文字を含めない
- GitHub Copilot のコンテキスト制限を考慮し、簡潔で明確な記述を心がける

### 生成物の構造

```
{skill-name}/
  SKILL.md
  scripts/
  references/
```

SKILL.md の要件:

- `name`（kebab-case）と `description` を含む
- 番号付きの実行手順（再現可能）
- よくあるエラーと対処
- 少なくとも 2 つの使用例（サニタイズ済み）
- 200〜300行を目標（必要に応じて400行まで許容）

## Phase 6: 品質検証

1. [references/quality-checklist.md](references/quality-checklist.md) でチェックリストを確認
2. 既存の skill-creator スキルの設計原則と照合
3. 不合格項目があれば修正して再検証

## よくある問題

### Export JSON が用意できない

- このスキルは **Export JSON のみ**を入力として扱います。VS Code 側で `Chat: Export Chat...` を実行して JSON を用意してもらいます。

### Export JSON が大きすぎる

- 必ず `scripts/filter-vscode-chat-export.py` で縮約してから読み込みます
- それでも大きい場合は、対象セッションを絞って Export し直す（分析対象を減らす）

### JSON パースエラー

- 対象ファイルを 1 つに絞り、縮約スクリプトを再実行
- それでも失敗する場合は、Export の再取得（別名で保存）を依頼
