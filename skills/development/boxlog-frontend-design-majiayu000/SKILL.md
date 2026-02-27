---
name: boxlog-frontend-design
description: BoxLog専用のフロントエンドデザインスキル。「装飾のない基本体験」を実現するためのUI設計ガイドライン。STYLE_GUIDE.mdを補完し、フォント・アニメーション・デザイン判断基準を提供。
license: MIT
metadata:
  author: boxlog
  version: '1.0.0'
---

# BoxLog Frontend Design Skill

「GoogleカレンダーやTogglと同等の装飾のない基本体験」を実現するためのデザインスキル。

> **参照**: 詳細なトークン定義は `docs/design-system/STYLE_GUIDE.md` を確認すること。

## 設計原則: 装飾のない基本体験

### 判断基準フローチャート

```
この要素は機能を伝えるために必要か？
├─ Yes → 実装する
│   └─ 最小限のスタイルで表現できるか？
│       ├─ Yes → そのまま実装
│       └─ No → 機能を分割して再検討
└─ No → 実装しない（装飾的要素は避ける）
```

### 良い例（採用）

| パターン             | 理由                  |
| -------------------- | --------------------- |
| 控えめなホバー状態   | 機能的フィードバック  |
| セマンティックカラー | 意味を伝える色使い    |
| 一貫したスペーシング | 8pxグリッドで予測可能 |
| シンプルなアイコン   | 機能を瞬時に伝える    |

### 避けるべき例（不採用）

| パターン                 | 理由                   |
| ------------------------ | ---------------------- |
| 派手なグラデーション背景 | 装飾的、集中を妨げる   |
| 過剰なアニメーション     | 生産性ツールにはノイズ |
| 影の多用                 | フラットで十分         |
| イラスト・装飾画像       | 機能に寄与しない       |

---

## Typography（タイポグラフィ）

### フォント選定

**現行**: プロジェクトのシステムフォントを維持

```css
/* 変更不要 - システムフォントスタック */
font-family:
  system-ui,
  -apple-system,
  BlinkMacSystemFont,
  sans-serif;
```

**禁止**: フォント変更の提案（Inter, Roboto, カスタムフォント追加など）

### サイズ・ウェイト

STYLE_GUIDE.mdの8pxグリッドに従う:

| 用途     | サイズ | ウェイト | Tailwind                 |
| -------- | ------ | -------- | ------------------------ |
| 本文     | 16px   | 400      | `text-base font-normal`  |
| 強調本文 | 16px   | 500      | `text-base font-medium`  |
| 小見出し | 24px   | 600      | `text-2xl font-semibold` |
| ラベル   | 14px   | 400      | `text-sm font-normal`    |

---

## Motion（アニメーション）

### 基本原則: 機能的アニメーションのみ

```
アニメーションを追加する前に確認:
1. ユーザーの操作に対するフィードバックか？ → OK
2. 状態変化を伝えるものか？ → OK
3. 視覚的な装飾か？ → NG
```

### 許可されるアニメーション

| 種類       | 用途               | 実装                              |
| ---------- | ------------------ | --------------------------------- |
| Transition | ホバー、フォーカス | `transition-colors duration-150`  |
| Fade       | 表示/非表示        | `transition-opacity duration-200` |
| Scale      | 押下フィードバック | `active:scale-95`                 |

### 禁止されるアニメーション

| 種類                         | 理由                     |
| ---------------------------- | ------------------------ |
| ページ遷移アニメーション     | 速度が優先               |
| 背景のパーティクル           | 装飾的                   |
| ローディングの凝った演出     | シンプルなスピナーで十分 |
| バウンス、弾性アニメーション | 生産性ツールには不適切   |

### 統一パターン

```tsx
// ✅ 標準トランジション（150ms）
className = 'transition-colors duration-150';

// ✅ 表示切り替え（200ms）
className = 'transition-opacity duration-200';

// ✅ ホバー + 押下フィードバック
className = 'transition-colors hover:bg-state-hover active:scale-[0.98]';

// ❌ 過剰なアニメーション
className = 'animate-bounce animate-pulse';
```

---

## Color（カラー）

### セマンティックトークン厳守

STYLE_GUIDE.mdで定義されたトークンのみ使用。直接色指定禁止。

```tsx
// ✅ セマンティックトークン
className = 'bg-card text-foreground border-border';

// ❌ 直接色指定
className = 'bg-white text-gray-900 border-gray-200';
className = 'bg-blue-500';
```

### Primary色の制限

Primary = 「ユーザーアクションを促す要素」にのみ:

- ✅ CTAボタン、リンク、フォーカスリング
- ❌ 装飾、背景、待機状態

---

## Layout（レイアウト）

### 情報密度

**目標**: Google Calendar / Toggl と同等の情報密度

| コンポーネント | 密度                             |
| -------------- | -------------------------------- |
| カレンダーセル | 高密度（多くのイベント表示可能） |
| リスト項目     | 中密度（必要情報のみ）           |
| フォーム       | 低密度（操作性優先）             |

### 余白

8pxグリッド厳守（STYLE_GUIDE.md参照）:

```tsx
// ✅ 8の倍数
className = 'p-4 gap-4'; // 16px

// ❌ 8の倍数でない
className = 'p-3 gap-5'; // 12px, 20px
```

---

## Components（コンポーネント設計）

### shadcn/ui活用原則

1. **そのまま使う**: カスタマイズは最小限
2. **薄いラッパー**: スタイル統一のみ
3. **オーバーライド禁止**: 基本動作を変えない

### 新規コンポーネント判断

```
新しいコンポーネントが必要か？
├─ shadcn/uiにあるか？ → あれば使う
├─ 3箇所以上で使うか？ → Yes: components/common/に追加
└─ 1-2箇所のみか？ → インラインで実装
```

---

## 参照リンク

- [STYLE_GUIDE.md](docs/design-system/STYLE_GUIDE.md) - 詳細なトークン定義
- [src/CLAUDE.md](src/CLAUDE.md) - コーディング規約
- [globals.css](src/styles/globals.css) - CSS変数定義

---

**バージョン**: 1.0.0
**最終更新**: 2026-01-19
