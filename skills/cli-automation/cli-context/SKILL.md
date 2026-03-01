---
name: cli-context
description: Explains the CLI app architecture and capabilities. Use when working on CLI features, debugging CLI issues, or needing to understand the TTS pipeline, emotion annotation, or command flags.
user-invocable: false
---

# Cartesia Download CLI - Architecture Reference

## Overview

テキストから Cartesia TTS API で音声を生成する CLI ツール。Claude LLM による感情アノテーション付き。

## Package Structure

```
apps/cli/src/
├── cli.ts                    # gunshi エントリポイント
├── types.ts                  # RawCliArgs, RcConfig, AppError
├── commands/
│   └── download.ts           # download コマンド実装
└── core/
    ├── config.ts             # 設定解決 (args → env → rc → defaults)
    ├── io.ts                 # ファイル I/O 抽象化 (neverthrow)
    └── format-error.ts       # ユーザー向けエラーメッセージ

packages/core/src/
├── types.ts                  # TtsConfig, TtsClient, TextAnnotator
├── pipeline.ts               # TTS ストリーミングパイプライン
├── tts-client.ts             # Cartesia API ラッパー
├── annotator.ts              # アノテーターファクトリ
├── marker-parser.ts          # [SEP] マーカーパーサー
├── wav.ts                    # WAV ヘッダービルダー
└── providers/
    └── claude-annotator.ts   # Claude 感情アノテーション
```

## Command: download

```bash
cartesia-download [flags]
```

| Flag                 | Short | Default                    | Description              |
| -------------------- | ----- | -------------------------- | ------------------------ |
| `--text`             | `-t`  | —                          | 読み上げテキスト         |
| `--input`            | `-i`  | —                          | テキストファイルパス     |
| `--voice-id`         |       | —                          | Cartesia Voice ID (必須) |
| `--output`           | `-o`  | —                          | WAV 出力パス             |
| `--model`            | `-m`  | `sonic-3`                  | TTS モデル ID            |
| `--sample-rate`      |       | `44100`                    | サンプルレート (Hz)      |
| `--provider`         |       | `claude`                   | LLM プロバイダー         |
| `--provider-model`   |       | `claude-sonnet-4-20250514` | LLM モデル               |
| `--provider-api-key` |       | —                          | LLM API キー             |
| `--no-annotate`      |       | `false`                    | 感情アノテーション無効化 |

## Data Flow

```
テキスト入力 (--text / --input ファイル)
    ↓
設定解決 (CLI args > env vars > .cartesiarc.json > defaults)
    ↓
[感情アノテーション] Claude が SSML タグ + [SEP] マーカーを挿入
    ↓
チャンクごとに Cartesia TTS API → PCM ストリーム → stdout
    ↓
[--output 指定時] WAV ファイル + アノテーション済みテキスト (.txt) 保存
```

## Emotion Annotation

Claude が以下の SSML タグをテキストに挿入する:

- `<emotion value="happy|sad|angry|excited|..."/>` — 感情
- `<speed ratio="0.5-1.0"/>` — 速度 (1.0 以下)
- `<volume ratio="0.5-2.0"/>` — 音量

使用可能な感情値: neutral, angry, excited, content, sad, scared, happy, curious, sarcastic, hesitant, confident, calm, surprised

`[SEP]` マーカーでチャンク分割し、チャンクごとに TTS API を呼ぶことでリアルタイムストリーミングを実現。

## Config Resolution Priority

1. **CLI args** (最優先)
2. **環境変数** (`CARTESIA_API_KEY`, `CARTESIA_VOICE_ID`, `ANTHROPIC_API_KEY`)
3. **RC ファイル** (`.cartesiarc.json` — ディレクトリを上方探索)
4. **デフォルト値** (model: sonic-3, sampleRate: 44100)

## Error Types

| Error                 | Cause                       |
| --------------------- | --------------------------- |
| `MissingApiKey`       | CARTESIA_API_KEY 未設定     |
| `MissingVoiceId`      | Voice ID 未指定             |
| `MissingText`         | --text も --input も未指定  |
| `FileReadError`       | 入力ファイル読み取り失敗    |
| `FileWriteError`      | 出力ファイル書き込み失敗    |
| `TtsApiError`         | Cartesia API エラー         |
| `AnnotationError`     | Claude API エラー           |
| `UnsupportedProvider` | "claude" 以外のプロバイダー |

## Key Design Patterns

- **neverthrow**: 全ての I/O は `ResultAsync` でエラーハンドリング
- **DI**: `io`, `ttsClient`, `annotator` は外部注入 (テスタビリティ)
- **Streaming**: PCM チャンクを逐次 stdout に書き出し、リアルタイム再生可能
- **Immutable**: const only, 関数型パターン
