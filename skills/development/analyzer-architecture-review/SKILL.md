---
name: analyzer-architecture-review
description: analyzerアプリケーションのアーキテクチャレビュー。Port&Adapterアーキテクチャ（ヘキサゴナルアーキテクチャ）のルールに従っているかをチェックします。新しいPort/Adapter/Usecase/Model追加時、PRレビュー時、またはアーキテクチャ違反の検出が必要な時に使用します。Port層の関数型定義、依存関係の方向、New*関数パターン、レイヤー分離などを検証します。
---

# Analyzer Architecture Review

## Overview

このスキルは、analyzerアプリケーションのPort&Adapterアーキテクチャ（ヘキサゴナルアーキテクチャ）への準拠をレビューします。

**アーキテクチャの詳細**: [references/analyzer-architecture.md](references/analyzer-architecture.md)を参照してください。

## Review Workflow

### 1. 変更ファイルの特定

```bash
git diff --name-only
```

以下のディレクトリの変更を確認：
- `app/analyzer/internal/port/`
- `app/analyzer/internal/adapter/`
- `app/analyzer/internal/usecase/`
- `app/analyzer/internal/model/`
- `app/analyzer/internal/registory/`

### 2. レイヤー別レビュー

変更されたファイルのレイヤーに応じて、該当するチェック項目を確認します。

#### Port層の変更

- ✅ 関数型(`type Name = func(...)`)で定義されているか
- ✅ 第一引数が`context.Context`か（必要な場合）
- ✅ 実装コードが含まれていないか（型定義のみ）
- ✅ 必要に応じて入力/出力用の構造体が定義されているか
- ❌ Port層に具体的な実装が含まれていないか

詳細: [analyzer-architecture.md#Port層](references/analyzer-architecture.md#port層-internalport)

#### Adapter層の変更

- ✅ `New*`という命名規則のコンストラクタがあるか
- ✅ Portインターフェースを実装しているか
- ✅ 外部依存(HTTP、DB、ファイルなど)がこの層に閉じ込められているか
- ✅ 実装関数は非公開(小文字始まり)か
- ❌ ビジネスロジックが含まれていないか

詳細: [analyzer-architecture.md#Adapter層](references/analyzer-architecture.md#adapter層-internaladapter)

#### Usecase層の変更

- ✅ `New*`コンストラクタで全ての依存性を注入しているか
- ✅ Portインターフェースのみに依存しているか
- ✅ 実装関数は非公開(小文字始まり)か
- ❌ Adapter層の具体的な実装を直接参照していないか
- ❌ 外部システムに直接アクセスしていないか

**importチェック**: Usecaseは`internal/port/*`と`internal/model`のみをimportすべき

詳細: [analyzer-architecture.md#Usecase層](references/analyzer-architecture.md#usecase層-internalusecase)

#### Model層の変更

- ✅ ドメイン知識を表現する型を定義しているか
- ✅ ドメインロジックがここに集約されているか
- ❌ 外部依存(DB、APIなど)を持っていないか
- ❌ Port/Adapter/Usecaseに依存していないか

詳細: [analyzer-architecture.md#Model層](references/analyzer-architecture.md#model層-internalmodel)

#### Registry層の変更

- ✅ 全ての依存性の組み立てがここで行われているか
- ✅ 具体的な実装の選択がここで行われているか
- ✅ Entry point(`cmd/`)から呼び出されているか
- ❌ ビジネスロジックが含まれていないか

詳細: [analyzer-architecture.md#Registry層](references/analyzer-architecture.md#registry層-internalregistory)

### 3. 依存関係の検証

**許可される依存**:
- Usecase → Port
- Usecase → Model
- Adapter → Port
- Adapter → Model
- Port → Model（必要に応じて）

**禁止される依存**:
- ❌ Port → Usecase
- ❌ Port → Adapter
- ❌ Usecase → Adapter
- ❌ Model → Port/Usecase/Adapter

**検証方法**: ファイルのimport文を確認

```go
// ✅ 正しい Usecase の import
import (
    "github.com/ss49919201/keeput/app/analyzer/internal/port/fetcher"
    "github.com/ss49919201/keeput/app/analyzer/internal/model"
)

// ❌ 間違った Usecase の import
import (
    "github.com/ss49919201/keeput/app/analyzer/internal/adapter/fetcher/hatena"  // NG!
)
```

詳細: [analyzer-architecture.md#依存関係のルール](references/analyzer-architecture.md#依存関係のルール)

### 4. 命名規則の確認

- ✅ `New*`関数がPort型を返しているか
- ✅ 実装関数は非公開（小文字始まり）か
- ✅ Port型の命名が適切か

詳細: [analyzer-architecture.md#命名規則](references/analyzer-architecture.md#命名規則)

## Quick Reference

### アーキテクチャ違反の典型例

1. **UsecaseがAdapterに依存**
   ```go
   // ❌ NG
   import "github.com/.../adapter/fetcher/hatena"
   ```

2. **Port層に実装を含める**
   ```go
   // ❌ NG: Port層に実装があってはいけない
   func DefaultFetchLatestEntry() FetchLatestEntry { /* 実装 */ }
   ```

3. **Modelが外部依存を持つ**
   ```go
   // ❌ NG
   import "github.com/aws/aws-sdk-go-v2/service/s3"
   ```

詳細な例: [analyzer-architecture.md#アーキテクチャ違反の例](references/analyzer-architecture.md#アーキテクチャ違反の例)

## Output Format

レビュー結果は以下の形式で出力：

```markdown
# Analyzer Architecture Review

## Summary
- 変更ファイル数: X
- アーキテクチャ違反: Y件
- 警告: Z件

## Details

### ✅ Good: Port層の定義（port/fetcher/entry.go）
- 関数型で正しく定義されている
- 実装コードが含まれていない

### ❌ Issue: Usecase層の依存関係（usecase/analyze.go）
- Adapter層を直接importしている
- 修正方法: Registry層で依存性を注入する

### ⚠️ Warning: 命名規則（adapter/fetcher/hatena.go）
- 実装関数が公開されている（小文字始まりにすべき）
```

## Resources

このスキルには以下のリファレンスが含まれています：

- **references/analyzer-architecture.md**: analyzerアプリケーションの包括的なアーキテクチャガイド
  - ディレクトリ構造
  - 各層の役割と責務
  - 依存関係のルール
  - 命名規則
  - アーキテクチャ違反の例
  - レビュー時の確認事項
