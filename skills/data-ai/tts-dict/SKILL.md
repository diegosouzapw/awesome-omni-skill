---
name: tts-dict
description: TTS発音辞書管理。英単語のカタカナ読み登録・適用・確認。LLM自動読み取得にも対応。「辞書登録して」「英単語の読みを登録」「発音確認」で発動。
---

# TTS Dictionary Manager

英単語の読み方をTTSエンジン（COEIROINK等）に登録する辞書管理プラグイン。

## 使用タイミング

- TTS生成前に英単語の発音を登録するとき
- 「辞書登録して」「英単語の読みを登録」と依頼されたとき
- TTS出力の発音を確認・修正するとき

## クイックスタート

```bash
# 1. 辞書の健全性チェック（TTS前に必須）
node ${CLAUDE_PLUGIN_ROOT}/scripts/dict.js healthcheck

# 2. 英単語を自動登録（LLM経由で読み取得）
node ${CLAUDE_PLUGIN_ROOT}/scripts/dict.js auto-add Claude OpenAI ChatGPT --apply

# 3. 発音を確認
node ${CLAUDE_PLUGIN_ROOT}/scripts/dict.js verify Claude OpenAI ChatGPT
```

## コマンド

| コマンド | 説明 |
|---------|------|
| `healthcheck` | 辞書の健全性チェック（**TTS前に必須**） |
| `auto-add <words...> --apply` | LLMで読み取得→登録→適用 |
| `scan --input <file> [--apply] [--with-case-variants]` | テキスト/JSONから英単語を自動抽出→登録 |
| `add <word> <yomi>` | 手動登録 |
| `check <words...>` | 登録確認 |
| `verify <words...>` | エンジンの実際の発音を確認 |
| `apply` | エンジンに辞書を適用 |
| `list` | 一覧表示 |
| `reset` | 辞書をリセット |

## グローバルオプション

| オプション | 説明 | デフォルト |
|------------|------|------------|
| `--api-base` | TTS API基本URL | http://127.0.0.1:50032 |

## チェックリスト（TTS生成前）

- [ ] `dict.js healthcheck` が成功
- [ ] TTSエンジンが起動している
- [ ] 主要な英単語（Git, GitHub等）が**大文字・小文字両方で登録済み**
- [ ] `dict.js verify <主要単語>` で発音を確認済み

## 使用例

### テキストから英単語を自動抽出→登録（scan）

```bash
# dialogue.json からスキャン → プレビュー
node ${CLAUDE_PLUGIN_ROOT}/scripts/dict.js scan --input dialogue.json --dry-run

# テキスト直接指定 → 未登録単語をLLM登録 → エンジン適用
node ${CLAUDE_PLUGIN_ROOT}/scripts/dict.js scan --text "Claude CodeのPlan ModeでTypeScriptを書く" --apply

# dialogue.json からスキャン → 自動登録 + 適用
node ${CLAUDE_PLUGIN_ROOT}/scripts/dict.js scan --input dialogue.json --apply

# ケースバリアント（大文字・小文字・Pascal）も自動登録
node ${CLAUDE_PLUGIN_ROOT}/scripts/dict.js scan --text "Claude Code" --with-case-variants --apply
# → Claude, claude, CLAUDE, Code, code, CODE を一括登録
```

#### scan オプション

| オプション | 説明 |
|-----------|------|
| `--input <file>` | 入力ファイル（JSONダイアログまたはテキスト） |
| `--text <string>` | テキスト直接入力 |
| `--dry-run` | プレビューのみ（変更なし） |
| `--apply` | 登録＋エンジン適用まで実行 |
| `--with-case-variants` | ケースバリアント自動登録（lowercase, UPPERCASE, Pascal） |

### 自動登録（LLM経由・単語指定）

```bash
node ${CLAUDE_PLUGIN_ROOT}/scripts/dict.js \
  auto-add Claude OpenAI ChatGPT Kubernetes --apply
```

### 手動登録

```bash
node ${CLAUDE_PLUGIN_ROOT}/scripts/dict.js add "Claude" "クロード"
node ${CLAUDE_PLUGIN_ROOT}/scripts/dict.js add "claude" "クロード"
node ${CLAUDE_PLUGIN_ROOT}/scripts/dict.js apply
```

### 英単語の読み取得のみ（辞書登録なし）

```bash
node ${CLAUDE_PLUGIN_ROOT}/scripts/get-english-reading.js Claude OpenAI ChatGPT
```

## よくある問題

### 大文字の単語が正しく読まれない

COEIROINKは大文字小文字を区別する。"git" と "Git" は別エントリとして両方登録が必要。

### ハイフン付き単語

`obsidian-skills` のようなハイフン付き単語は辞書マッチングが不安定。
TTS用テキスト側でカタカナに置換することを推奨。

## OpenAI TTS使用時の辞書について

OpenAI TTSは多言語対応のため、英単語の辞書登録は基本的に不要です。

特殊な読み方が必要な場合は、ttsプラグインの `instructions` パラメータで直接指示できます:

```json
{
  "instructions": "Read 'LLM' as 'エルエルエム', 'API' as 'エーピーアイ', 'Claude' as 'クロード'"
}
```

このプラグイン（tts-dict）はCOEIROINK/VOICEVOX等のローカルTTSエンジン専用です。

## 前提条件

- Node.js 18+
- COEIROINK: localhost:50032（またはVOICEVOX互換エンジン）
- LLM API: `ZAI_API_KEY` または `OPENROUTER_API_KEY`
