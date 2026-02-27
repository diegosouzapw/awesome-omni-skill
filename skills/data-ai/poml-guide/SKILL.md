---
name: poml-guide
description: Microsoft POML（Prompt Orchestration Markup Language）の構文、タグリファレンス、
  プロンプト設計パターンを提供する。「POMLとは」「POMLの書き方」「roleタグ」「taskタグ」
  「POMLでプロンプトを構造化したい」「POML validate」「POML render」などPOML関連の
  質問や .poml ファイルの編集時に使用する。
globs:
  - "**/*.poml"
---

# POML (Prompt Orchestration Markup Language) Guide

Microsoft が開発した XML ベースのマークアップ言語。AI Agent への構造化した指示記述に使用する。

## Quick Reference

| タグ | 用途 | 主要属性 |
|---|---|---|
| `<poml>` | ルート要素 | `version` |
| `<role>` | エージェントのペルソナ定義 | - |
| `<task>` | 実行タスクの指定 | - |
| `<example>` | Few-shot 用サンプル | `type` |
| `<output-format>` | 出力形式の指定 | - |
| `<document>` | 参照ドキュメント埋め込み | `src`, `type` |
| `<table>` | 表形式データ | `src`, `format` |
| `<img>` | 画像参照 | `src`, `alt` |
| `<stylesheet>` | 出力スタイル制御 | - |
| `<output-schema>` | JSON Schema 定義 | - |
| `<let>` | 変数定義 | `name` |

## 最小テンプレート

```xml
<poml>
  <role>あなたは{{role_name}}です。</role>
  <task>{{task_description}}</task>
</poml>
```

## CLI コマンド

```bash
# インストール
pip install poml

# バリデーション
poml check <file.poml>

# レンダリング（プレーンテキスト変換）
poml render <file.poml>

# 変数を渡してレンダリング
poml render <file.poml> --var key=value
```

## 詳細ドキュメント

- [poml-syntax.md](./poml-syntax.md) - ファイル構造・テンプレート変数・条件分岐・ループ
- [poml-tags-reference.md](./poml-tags-reference.md) - 全タグの属性仕様と使用例
- [poml-patterns.md](./poml-patterns.md) - 用途別設計パターン集
- [poml-stylesheet.md](./poml-stylesheet.md) - stylesheet によるフォーマット制御
