---
name: codex-web-search
description: Codex CLI 環境内で Gemini CLI を使用した Web 検索を実行。技術情報、最新ニュース、一般的なリサーチに対応。Use when user asks to search the web, look up information, find recent news, or research a topic within Codex CLI. Also use when user says 調べて, 検索して, 最新情報, ニュース, リサーチ.
allowed-tools: Bash
context: fork
---

# Codex Web Search スキル

Codex CLI 実行環境内で Gemini CLI の `google_web_search` ツールを活用した Web 検索機能を提供。

## Instructions

### 1. 検索クエリの準備

ユーザーのリクエストから適切な検索クエリを構築する。

**クエリ最適化のポイント**:

- **技術調査**: 具体的な技術名、バージョン、「documentation」「official」を含める
  - 例: `"React 19 new features official documentation"`
- **エラー解決**: エラーメッセージをそのまま含め、「fix」「solution」を追加
  - 例: `"TypeError: Cannot read property 'map' of undefined fix React"`
- **最新情報**: 「latest」「2025」などの時間指定を含める
  - 例: `"TypeScript latest version features 2025"`
- **比較調査**: 比較対象を明確に、「vs」「comparison」を使用
  - 例: `"React vs Vue performance comparison 2025"`

### 2. Gemini CLI で検索実行

```bash
gemini --yolo "Use the google_web_search tool to search for: {検索クエリ}. You MUST perform a web search and return results with sources."
```

**フラグ説明**:
- `--yolo`: ツール実行の許可プロンプトをスキップ（自動実行）

**重要**: 「Use the google_web_search tool to search for:」という指示を必ず含める。
これにより Gemini が確実に Web 検索を実行する。

**動作**:
1. Gemini が google_web_search ツールを使用
2. Google 検索を実行
3. 検索結果を要約してソース付きで返却

### 3. 結果のフォーマット

Gemini の応答を以下の形式で報告:

```markdown
## 検索結果: {クエリ}

### 要約
{主要なポイントを2-3文で}

### 詳細
{関連情報の詳細}

### ソース
- [タイトル1](URL1)
- [タイトル2](URL2)
- ...
```

### 4. エラーハンドリング

| エラー | 対処 |
|--------|------|
| Gemini CLI が見つからない | `gemini` コマンドのインストールを案内 |
| API エラー | 再試行、またはユーザーに報告 |
| 検索結果なし | クエリを変更して再検索を提案 |

## Codex CLI 統合

このスキルは Codex CLI の `--full-auto` モードと組み合わせて使用可能:

```bash
codex --full-auto "React 19 の新機能を調べて、実装サンプルを作成"
```

Codex は自動的にこのスキルを使用して Web 検索を実行し、結果を活用してタスクを完了する。

## Examples

### 技術調査

**ユーザー**: 「Next.js 15 の新機能を調べて」

**実行**:
```bash
gemini --yolo "Use the google_web_search tool to search for: Next.js 15 new features official documentation. You MUST perform a web search and return results with sources."
```

**結果報告**:
```markdown
## 検索結果: Next.js 15 新機能

### 要約
Next.js 15 では、Turbopack がデフォルトの開発サーバーとして採用され...

### 詳細
- Turbopack: 開発時のビルド速度が大幅に向上
- React 19 対応: Server Components の強化
- ...

### ソース
- [Next.js 15 Blog](https://nextjs.org/blog/next-15)
- [Next.js Documentation](https://nextjs.org/docs)
```

### エラー解決

**ユーザー**: 「React で "Cannot read property 'map' of undefined" エラーが出る」

**実行**:
```bash
gemini --yolo "Use the google_web_search tool to search for: Cannot read property map of undefined React fix solution. You MUST perform a web search and return results with sources."
```

### 最新ニュース

**ユーザー**: 「AI 関連の最新ニュースを検索して」

**実行**:
```bash
gemini --yolo "Use the google_web_search tool to search for: AI artificial intelligence latest news today. You MUST perform a web search and return results with sources."
```

## 情報鮮度フィルタリング

検索結果を報告する前に鮮度を評価し、古い情報は適切に処理する。

### 自動判断基準

| ドメイン | 推奨鮮度 | 古い情報の扱い |
|---------|---------|---------------|
| **AI/LLM** | 6ヶ月以内 | 旧世代モデル情報は**破棄** |
| **フロントエンド** | 1年以内 | `[古い情報]` マーク付与 |
| **クラウドサービス** | 1年以内 | `[古い情報]` マーク付与 |
| **セキュリティ** | 3ヶ月以内 | 最新情報のみ採用 |
| **プログラミング言語** | 2年以内 | 参考として残す |
| **アルゴリズム・設計** | 制限なし | そのまま採用 |

### AI モデル関連の特別ルール

検索クエリに AI/LLM キーワードが含まれる場合、以下は**自動破棄**:
- 旧世代モデル: GPT-3.5以前、Claude 2以前、Gemini 1.0以前、Llama 2以前
- 2023年以前のベンチマーク・モデル比較記事
- 廃止された API（Codex API 等）

**保持対象**（古くても有効）:
- Transformer、Attention 等の基礎理論
- 学術論文（arxiv 等）
- 歴史的経緯の説明

### 出力フォーマット

ソース一覧に鮮度情報を付与:
- 2024-2025年: `✓` 最新
- 1-2年前: `[参考]` マーク
- それ以前: `[古い情報]` マーク、または破棄

### 判断困難時

目的が不明確な場合のみ、ユーザーに確認:
「最新情報のみで良いですか? 古い情報（2023年）も含めますか?」

## 注意事項

- 機密情報（API キー、パスワード等）を検索クエリに含めない
- 検索結果は要約されるため、詳細は元ソースを確認
- 連続した大量の検索は API レート制限に注意
