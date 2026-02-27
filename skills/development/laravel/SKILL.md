---
name: laravel
description: Use when implementing or debugging this Laravel v12 app; leverage Laravel Boost MCP (search-docs, artisan, schema, logs, tinker) and follow project conventions.
alwaysApply: true
---

# Laravel開発スキル（Laravel 12 + Boost）

このスキルは、本リポジトリ（Laravel 12 / PHP 8.5.2）での実装・調査・デバッグを、Laravel Boost MCPを中心に進めるためのガイドです。

## 前提（このリポジトリ）

- **Laravel**: v12
- **PHP**: 8.5.2
- **MCPサーバー**: `project-0-booking2-laravel-boost`（Boost）

## Boostツール（よく使う）

- **アプリ情報**: `application-info`
- **ドキュメント検索**: `search-docs`（クエリは英語）
- **DBスキーマ確認**: `database-schema`
- **DBデータ確認（読み取り専用）**: `database-query`
- **ルート確認**: `list-routes`
- **Artisanコマンド一覧**: `list-artisan-commands`
- **Configキー一覧**: `list-available-config-keys`
- **Config取得**: `get-config`
- **直近エラー**: `last-error`
- **ログ確認**: `read-log-entries`
- **ブラウザログ**: `browser-logs`
- **URLの確定**: `get-absolute-url`
- **Tinker**: `tinker`

---

## 実装前の必須チェック

Laravel関連の実装・修正を始める前に、必要に応じて次を実行します。

1. **アプリ情報の確認**: `application-info`
2. **ドキュメント検索**: `search-docs`（英語クエリで短く複数回）
3. **DBが絡む場合**: `database-schema`（テーブル/カラム/インデックス/外部キー）
4. **ルートが絡む場合**: `list-routes`（既存の命名・責務・ミドルウェアを把握）

---

## 実装の基本方針（Laravel way）

- **ルーティング**: named route と `route()` を優先する
- **バリデーション**: Controller直書きではなく Form Request を作成して集約する
- **DBアクセス**: Eloquent と Relationship を優先し、N+1は eager load で防ぐ
- **設定値**: `config()` を使用し、`env()` は config ファイル以外で使わない
- **Laravel 12の構造**: ミドルウェア登録は `bootstrap/app.php` 側で行う（`app/Http/Kernel.php` 前提にしない）

---

## デバッグ時の標準フロー

1. **エラーを取る**: `last-error`
2. **ログを追う**: `read-log-entries`
3. **フロント絡み**: `browser-logs`
4. **状態を確認**: `database-query`（読み取り専用） / `tinker`
5. **必要ならDocs再確認**: `search-docs`（英語クエリ）

---

## 禁止事項

- **日本語でのDocs検索**: `search-docs` は日本語クエリだと結果が弱い/出ないため、英語で検索する
- **DBへの書き込み**: `database-query` は読み取り用途に限定し、破壊的操作をしない
- **秘密情報の露出**: `.env` やトークン、鍵などの機密を表示・貼り付けしない

---

## 仕上げ（推奨）

- **整形**: `vendor/bin/pint --dirty`
- **テスト**: 関連テストを最小セットで実行（例: `php artisan test --compact --filter=...`）
