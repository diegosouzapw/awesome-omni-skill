---
name: adr-knowledge-base
description: ADR知見の体系的参照・適用。主要ADR抜粋（ADR_010, 013, 016, 019, 020, 021）・ADR検索・参照方法・技術決定パターン集・ADR作成判断基準。Phase C以降の技術決定時に使用。
allowed-tools: Read, Grep
---

# ADR Knowledge Base Skill

## 概要

このSkillは、プロジェクトで蓄積されたADR（Architecture Decision Record）知見を体系的に参照・適用するための手順・パターン・判断基準を提供します。

## 使用タイミング

Claudeは以下の状況でこのSkillを自律的に使用すべきです：

1. **技術決定時**
   - 新しい技術選定時
   - アーキテクチャ設計時
   - ライブラリ選定時

2. **問題発生時**
   - 既存ADRに解決策がないか確認
   - 過去の類似問題を参照

3. **ADR作成判断時**
   - 新規ADR作成が必要か判断
   - Agent Skills作成が適切か判断

4. **実装時**
   - ADR準拠の実装方法確認
   - 制約条件・ベストプラクティス確認

## 主要ADR抜粋

### ADR_010: 実装規約

**詳細**: [`adr-excerpts/ADR_010_実装規約.md`](./adr-excerpts/ADR_010_実装規約.md)

**重要ポイント**:
- ✅ Blazor Server・F#初学者対応（詳細コメント必須）
- ✅ MainAgent責務分担原則（実装コード直接修正禁止）
- ✅ Fix-Mode活用（SubAgentによる修正）

**適用シーン**:
- 新規コード作成時
- コメント記載判断時
- エラー修正時のSubAgent選択時

---

### ADR_013: SubAgentプール方式採用

**重要ポイント**:
- ✅ 13種類のSubAgent定義
- ✅ 並列実行による効率化
- ✅ Step開始時のSubAgent組み合わせ選択

**適用シーン**:
- Step開始時のSubAgent選択
- 並列実行判断時
- SubAgent責務分担判断時

---

### ADR_016: プロセス遵守違反防止策

**詳細**: [`adr-excerpts/ADR_016_プロセス遵守.md`](./adr-excerpts/ADR_016_プロセス遵守.md)

**重要ポイント**:
- 🔴 **コマンド = 契約**: 一字一句を法的契約として遵守
- 🔴 **承認 = 必須**: 「ユーザー承認」表記は例外なく取得
- 🔴 **手順 = 聖域**: 定められた順序の変更禁止

**適用シーン**:
- Command実行時
- ユーザー承認取得時
- プロセス手順確認時

---

### ADR_019: namespace設計規約

**詳細**: [`adr-excerpts/ADR_019_namespace設計.md`](./adr-excerpts/ADR_019_namespace設計.md)

**重要ポイント**:
- ✅ Bounded Context別サブnamespace使用
- ✅ 階層制限（3階層推奨、4階層許容）
- ✅ F# Compilation Order制約対応

**適用シーン**:
- 新規クラス・モジュール作成時
- namespace設計時
- F#ファイル配置時

---

### ADR_020: テストアーキテクチャ決定

**詳細**: [`adr-excerpts/ADR_020_テストアーキテクチャ.md`](./adr-excerpts/ADR_020_テストアーキテクチャ.md)

**重要ポイント**:
- ✅ レイヤー×テストタイプ分離方式
- ✅ 命名規則: `UbiquitousLanguageManager.{Layer}.{TestType}.Tests`
- ✅ 参照関係原則（Unit Tests: テスト対象層のみ）

**適用シーン**:
- 新規テストプロジェクト作成時
- テストプロジェクト命名時
- テストプロジェクト参照関係設定時

---

### ADR_021: Playwright統合戦略

**重要ポイント**:
- ✅ Playwright MCP + Agents統合（推奨度10/10点）
- ✅ 93.3%E2Eテスト作成効率向上
- ✅ data-testid属性戦略

**適用シーン**:
- E2Eテスト実装時
- Playwright MCP活用時
- data-testid属性設計時

---

## ADR検索・参照方法

### ADR配置場所

```
Doc/07_Decisions/
├── ADR_001～ADR_023.md  # 承認済みADR
├── backup/               # Skills化済みADR（ADR_010, 019）
└── template.md          # ADR作成テンプレート
```

### ADR検索方法

**キーワード検索**:
```bash
# Grepツール使用
grep -r "キーワード" Doc/07_Decisions/

# 例: テスト関連ADR検索
grep -r "テスト" Doc/07_Decisions/
```

**ADR番号から検索**:
```bash
# ADR_020を参照
Read Doc/07_Decisions/ADR_020_テストアーキテクチャ決定.md
```

### ADR参照のベストプラクティス

1. **実装前にADR確認**
   - 関連するADRがないか検索
   - 既存の技術決定を尊重

2. **ADR準拠の実装**
   - ADR番号をコメントで記録
   - ADRの制約条件を遵守

3. **ADR更新判断**
   - ADRと矛盾する実装が必要な場合
   - ADRの更新または新規ADR作成を検討

---

## 技術決定パターン集

### パターン1: 技術選定（ライブラリ・フレームワーク）

**判断フロー**:
1. ✅ **既存ADR確認**: 類似の技術選定ADRがないか確認
2. ✅ **代替案評価**: 複数の代替案を比較
3. ✅ **リスク評価**: 技術的制約・学習コスト・保守性
4. ✅ **ADR作成**: 技術選定理由を文書化

**例**: ADR_021 Playwright統合戦略

### パターン2: アーキテクチャ設計

**判断フロー**:
1. ✅ **Clean Architecture準拠確認**: レイヤー分離・依存関係方向
2. ✅ **既存ADR参照**: ADR_019 namespace設計規約等
3. ✅ **設計決定記録**: 新規ADR作成（重要な設計決定の場合）

**例**: ADR_019 namespace設計規約、ADR_020 テストアーキテクチャ決定

### パターン3: プロセス改善

**判断フロー**:
1. ✅ **問題特定**: プロセス違反・効率低下の原因特定
2. ✅ **改善策検討**: 複数の改善策を比較
3. ✅ **ADR作成**: プロセス改善の根拠・手順を文書化

**例**: ADR_016 プロセス遵守違反防止策、ADR_018 SubAgent指示改善とFix-Mode活用

---

## ADR作成判断基準（vs Agent Skills）

### ADR作成が適切な場合

1. **歴史的記録が必要**
   - なぜこの決定をしたか（背景・理由）
   - 代替案との比較・リスク評価

2. **技術選定の根拠**
   - ライブラリ・フレームワーク選定
   - アーキテクチャ設計決定

3. **一度限りの決定**
   - プロジェクト固有の技術的制約
   - 特定の状況下での判断

**例**: ADR_021 Playwright統合戦略（技術選定根拠・代替案比較）

### Agent Skills作成が適切な場合

1. **Claudeが自律的に適用すべき**
   - 実装時に自動適用（パターン・チェックリスト）
   - 繰り返し使うルール

2. **実装パターン・チェックリスト**
   - 具体的な実装手順
   - 判断基準・チェックリスト

3. **継続的な改善対象**
   - Phase毎に更新・拡充
   - 実践知の蓄積

**例**: tdd-red-green-refactor Skill（TDDサイクル実践パターン）

### 両方作成すべき場合

**ADR**: 技術選定の根拠・背景を記録
**Agent Skills**: 実装パターン・チェックリストを提供

**例**:
- ADR_020（テストアーキテクチャ決定） → なぜレイヤー×テストタイプ分離方式を採用したか
- test-architecture Skill → 新規テストプロジェクト作成チェックリスト

---

## ADR参照チェックリスト

### 実装前

- [ ] 関連するADRを検索した
- [ ] 既存ADRの制約条件を確認した
- [ ] ADR準拠の実装方針を決定した

### 実装中

- [ ] ADR番号をコメントで記録した
- [ ] ADRの制約条件を遵守している
- [ ] ADRと矛盾する実装をしていない

### 技術決定時

- [ ] ADR作成が必要か判断した
- [ ] Agent Skills作成が適切か判断した
- [ ] 既存ADRの更新が必要か判断した

---

## 参照元ADR

- **ADR_010**: 実装規約（Blazor Server・F#初学者対応）
- **ADR_013**: SubAgentプール方式採用
- **ADR_016**: プロセス遵守違反防止策
- **ADR_019**: namespace設計規約
- **ADR_020**: テストアーキテクチャ決定
- **ADR_021**: Playwright統合戦略

---

## 関連Skills・Rules

- **ADRとAgent_Skills判断ガイドライン.md**: ADR vs Skills判断フロー（30秒チェック）
- **clean-architecture-guardian Skill**: Clean Architecture準拠性チェック

---

**作成日**: 2025-11-01
**Phase B-F2 Step2**: Agent Skills Phase 2展開
**参照**: ADR_010, 013, 016, 019, 020, 021
