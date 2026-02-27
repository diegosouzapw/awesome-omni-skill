---
name: designing-databases
description: データベーススキーマ設計と最適化を支援します。正規化戦略、インデックス設計、パフォーマンス最適化を提供します。データモデル設計、データベース構造の最適化が必要な場合に使用してください。
---

# データベース設計と最適化

## 概要

データベースのスキーマ設計、クエリ最適化、パフォーマンスチューニングを包括的に支援するスキルです。

## 実行フロー

### Step 1: 要件分析

#### ビジネス要件の整理

- ユースケースとデータフロー
- トランザクション特性（OLTP vs OLAP）
- データボリューム予測
- 成長予測とスケーラビリティ要件

#### 非機能要件

- **パフォーマンス**: 目標レスポンス時間、スループット
- **可用性**: SLA、ダウンタイム許容度
- **整合性**: ACID要件、結果整合性の許容度
- **セキュリティ**: 暗号化、アクセス制御

### Step 2: データモデリング（新規設計時）

#### 概念モデル（ER図）

エンティティと関係を定義：

```
User (ユーザー)
  - user_id (PK)
  - email
  - name
  - created_at

Order (注文)
  - order_id (PK)
  - user_id (FK)
  - total_amount
  - status
  - created_at
```

#### 論理モデル

- 正規化（第3正規形まで）
- 非正規化のトレードオフ判断
- データ型とサイズの決定
- 制約の定義（NOT NULL、UNIQUE、CHECK）

### Step 3: スキーマ設計

#### テーブル定義

```sql
CREATE TABLE users (
  user_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) NOT NULL UNIQUE,
  name VARCHAR(100) NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_users_email ON users(email);
```

#### インデックス戦略

**インデックスが効果的な場合:**

- WHERE句で頻繁に検索される列
- JOIN条件で使用される外部キー
- ORDER BY、GROUP BYで使用される列

**複合インデックス:**

```sql
CREATE INDEX idx_orders_user_status
ON orders(user_id, status, created_at);
```

#### パーティショニング

```sql
CREATE TABLE orders (
  order_id UUID,
  user_id UUID,
  total_amount DECIMAL(10, 2),
  created_at TIMESTAMP
) PARTITION BY RANGE (created_at);
```

### Step 4: クエリ最適化（既存システム）

#### パフォーマンス分析

```sql
-- 実行計画の確認
EXPLAIN ANALYZE
SELECT u.name, COUNT(o.order_id) as order_count
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
GROUP BY u.user_id, u.name;

-- スロークエリTop 10
SELECT query, calls, mean_time, max_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;
```

#### N+1問題の解消

```javascript
// 改善: Eager Loading
const users = await User.findAll({
  include: [{ model: Order }]
});
```

### Step 5: パフォーマンスチューニング

#### データベース設定の最適化

**PostgreSQL:**

```sql
shared_buffers = 4GB
work_mem = 16MB
maintenance_work_mem = 512MB
effective_cache_size = 12GB
```

#### コネクションプール

```javascript
const pool = new Pool({
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});
```

### Step 6: スケーラビリティ戦略

- 読み取りレプリカ
- シャーディング
- キャッシング戦略（Redis Cache-aside パターン）

### Step 7: 移行計画（必要に応じて）

**ゼロダウンタイム移行:**

```sql
-- 1. 新カラム追加
ALTER TABLE users ADD COLUMN new_email VARCHAR(255);

-- 2. データ移行
UPDATE users SET new_email = email WHERE new_email IS NULL;

-- 3. 旧カラム削除
ALTER TABLE users DROP COLUMN email;
ALTER TABLE users RENAME COLUMN new_email TO email;
```

## パフォーマンス目標

| 指標             | 目標値               |
| ---------------- | -------------------- |
| クエリレイテンシ | p95 < 100ms          |
| API応答時間      | p95 < 200ms          |
| キャッシュヒット率| 90%以上             |
| 稼働率           | 99.99%               |

## データベース選定ガイド

### SQL Databases

| データベース | 適したユースケース             |
| ------------ | ------------------------------ |
| PostgreSQL   | 汎用、複雑なクエリ、GIS        |
| MySQL        | Webアプリ、シンプルなクエリ    |

### NoSQL Databases

| データベース | 適したユースケース         |
| ------------ | -------------------------- |
| MongoDB      | ドキュメント、柔軟なスキーマ |
| Redis        | キャッシュ、セッション     |
| DynamoDB     | サーバーレス、AWS環境      |

## ベストプラクティス

1. **測定に基づく最適化**: 推測ではなくデータに基づいて最適化
2. **将来を見据えた設計**: スケーラビリティを考慮
3. **保守性の確保**: ドキュメント化、命名規則の統一
4. **セキュリティ**: 最小権限の原則、暗号化
5. **バックアップと復旧**: 定期的なバックアップ、DR計画

## 関連ファイル

- [TEMPLATES.md](./TEMPLATES.md) - スキーマテンプレート
- [QUERIES.md](./QUERIES.md) - 便利なクエリ集
