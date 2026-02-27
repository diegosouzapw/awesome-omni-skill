---
name: ui-designer
description: UI/UX デザインインテリジェンス。Webアプリの画面構成、レイアウト、コンポーネント、スタイル方針をまとめる。50以上のスタイル、97カラーパレット、57フォントペアリング、99 UXガイドライン、25チャートタイプ、9テクノロジースタック対応の検索可能なデザインデータベース付き。
---

# ui-designer

Webアプリ・モバイルアプリ向けの包括的デザインガイド。50以上のスタイル、97カラーパレット、57フォントペアリング、99 UXガイドライン、25チャートタイプを9テクノロジースタックにわたって網羅。優先度ベースの推奨を備えた検索可能なデータベース。

## 適用タイミング

以下の場面でこのガイドラインを参照する:
- 新しいUIコンポーネントやページを設計するとき
- カラーパレットやタイポグラフィを選定するとき
- コードのUX問題をレビューするとき
- ランディングページやダッシュボードを構築するとき
- アクセシビリティ要件を実装するとき

## ルールカテゴリ（優先度順）

| 優先度 | カテゴリ | 影響度 | ドメイン |
|--------|----------|--------|----------|
| 1 | アクセシビリティ | 最重要 | `ux` |
| 2 | タッチ＆インタラクション | 最重要 | `ux` |
| 3 | パフォーマンス | 高 | `ux` |
| 4 | レイアウト＆レスポンシブ | 高 | `ux` |
| 5 | タイポグラフィ＆カラー | 中 | `typography`, `color` |
| 6 | アニメーション | 中 | `ux` |
| 7 | スタイル選定 | 中 | `style`, `product` |
| 8 | チャート＆データ | 低 | `chart` |

## クイックリファレンス

### 1. アクセシビリティ（最重要）

- `color-contrast` - 通常テキストで最低4.5:1のコントラスト比
- `focus-states` - インタラクティブ要素に可視フォーカスリング
- `alt-text` - 意味のある画像に説明的なaltテキスト
- `aria-labels` - アイコンのみのボタンにaria-label
- `keyboard-nav` - タブ順序がビジュアル順序と一致
- `form-labels` - for属性付きlabelを使用

### 2. タッチ＆インタラクション（最重要）

- `touch-target-size` - 最小44x44pxのタッチターゲット
- `hover-vs-tap` - メインの操作にはclick/tapを使用
- `loading-buttons` - 非同期操作中はボタンを無効化
- `error-feedback` - 問題箇所の近くに明確なエラーメッセージ
- `cursor-pointer` - クリック可能な要素にcursor-pointerを追加

### 3. パフォーマンス（高）

- `image-optimization` - WebP、srcset、遅延読み込みを使用
- `reduced-motion` - prefers-reduced-motionを確認
- `content-jumping` - 非同期コンテンツ用のスペースを確保

### 4. レイアウト＆レスポンシブ（高）

- `viewport-meta` - width=device-width initial-scale=1
- `readable-font-size` - モバイルで最小16pxの本文テキスト
- `horizontal-scroll` - コンテンツがビューポート幅に収まること
- `z-index-management` - z-indexスケールを定義（10, 20, 30, 50）

### 5. タイポグラフィ＆カラー（中）

- `line-height` - 本文テキストに1.5〜1.75を使用
- `line-length` - 1行あたり65〜75文字に制限
- `font-pairing` - 見出しと本文のフォントの個性を合わせる

### 6. アニメーション（中）

- `duration-timing` - マイクロインタラクションに150〜300msを使用
- `transform-performance` - width/heightではなくtransform/opacityを使用
- `loading-states` - スケルトンスクリーンまたはスピナー

### 7. スタイル選定（中）

- `style-match` - プロダクトタイプに合ったスタイルを選ぶ
- `consistency` - 全ページで同じスタイルを使用
- `no-emoji-icons` - 絵文字ではなくSVGアイコンを使用

### 8. チャート＆データ（低）

- `chart-type` - データタイプに合ったチャートを選ぶ
- `color-guidance` - アクセシブルなカラーパレットを使用
- `data-table` - アクセシビリティのためにテーブル代替を提供

## 使い方

以下のCLIツールを使って特定のドメインを検索する。

---

## 前提条件

Pythonがインストールされているか確認:

```bash
python3 --version || python --version
```

インストールされていない場合、OSに応じてインストール:

**macOS:**
```bash
brew install python3
```

**Ubuntu/Debian:**
```bash
sudo apt update && sudo apt install python3
```

**Windows:**
```powershell
winget install Python.Python.3.12
```

---

## このスキルの使い方

ユーザーがUI/UX関連の作業（design、build、create、implement、review、fix、improve）を依頼した場合、以下のワークフローに従う:

### Step 1: ユーザー要件を分析する

ユーザーのリクエストからキー情報を抽出:
- **プロダクトタイプ**: SaaS、EC、ポートフォリオ、ダッシュボード、ランディングページなど
- **スタイルキーワード**: ミニマル、ポップ、プロフェッショナル、エレガント、ダークモードなど
- **業界**: ヘルスケア、フィンテック、ゲーム、教育など
- **スタック**: React、Vue、Next.jsなど、指定がなければ `html-tailwind` をデフォルト

### Step 2: デザインシステムを生成する（必須）

**まず `--design-system` で包括的な推奨を取得**:

```bash
python3 skills/ui-designer/scripts/search.py "<プロダクトタイプ> <業界> <キーワード>" --design-system [-p "プロジェクト名"]
```

このコマンドは:
1. 5つのドメインを並列検索（product、style、color、landing、typography）
2. `ui-reasoning.csv` のルールを適用して最適なマッチを選定
3. 完全なデザインシステムを返却: パターン、スタイル、カラー、タイポグラフィ、エフェクト
4. アンチパターンも含む

**例:**
```bash
python3 skills/ui-designer/scripts/search.py "beauty spa wellness service" --design-system -p "Serenity Spa"
```

### Step 2b: デザインシステムの永続化（マスター＋オーバーライドパターン）

デザインシステムを**セッション横断で階層的に参照**するために `--persist` を追加:

```bash
python3 skills/ui-designer/scripts/search.py "<クエリ>" --design-system --persist -p "プロジェクト名"
```

作成されるファイル:
- `design-system/MASTER.md` — グローバルなデザインルールの真のソース
- `design-system/pages/` — ページ固有のオーバーライド用フォルダ

**ページ固有のオーバーライド付き:**
```bash
python3 skills/ui-designer/scripts/search.py "<クエリ>" --design-system --persist -p "プロジェクト名" --page "dashboard"
```

追加で作成:
- `design-system/pages/dashboard.md` — マスターからの逸脱ルール

**階層的参照の仕組み:**
1. 特定ページ（例: 「Checkout」）構築時、まず `design-system/pages/checkout.md` を確認
2. ページファイルが存在すればマスターファイルを**オーバーライド**
3. 存在しなければ `design-system/MASTER.md` のみを使用

**コンテキスト対応の参照プロンプト:**
```
[ページ名] ページを構築中です。design-system/MASTER.md を読んでください。
また design-system/pages/[page-name].md が存在するか確認してください。
ページファイルが存在すれば、そのルールを優先してください。
存在しなければ、マスタールールのみを使用してください。
では、コードを生成してください...
```

### Step 3: 詳細検索で補完する（必要に応じて）

デザインシステム取得後、ドメイン検索で詳細を補完:

```bash
python3 skills/ui-designer/scripts/search.py "<キーワード>" --domain <ドメイン> [-n <最大結果数>]
```

**詳細検索が必要な場面:**

| ニーズ | ドメイン | 例 |
|--------|----------|-----|
| スタイルの追加候補 | `style` | `--domain style "glassmorphism dark"` |
| チャートの推奨 | `chart` | `--domain chart "real-time dashboard"` |
| UXベストプラクティス | `ux` | `--domain ux "animation accessibility"` |
| フォントの代替案 | `typography` | `--domain typography "elegant luxury"` |
| ランディング構成 | `landing` | `--domain landing "hero social-proof"` |

### Step 4: スタック別ガイドライン（デフォルト: html-tailwind）

スタック固有のベストプラクティスを取得。ユーザーが指定しない場合は **`html-tailwind` をデフォルト**に:

```bash
python3 skills/ui-designer/scripts/search.py "<キーワード>" --stack html-tailwind
```

利用可能なスタック: `html-tailwind`, `react`, `nextjs`, `vue`, `svelte`, `swiftui`, `react-native`, `flutter`, `shadcn`, `jetpack-compose`

---

## 検索リファレンス

### 利用可能なドメイン

| ドメイン | 用途 | キーワード例 |
|----------|------|-------------|
| `product` | プロダクトタイプ別推奨 | SaaS, e-commerce, portfolio, healthcare, beauty, service |
| `style` | UIスタイル、カラー、エフェクト | glassmorphism, minimalism, dark mode, brutalism |
| `typography` | フォントペアリング、Google Fonts | elegant, playful, professional, modern |
| `color` | プロダクトタイプ別カラーパレット | saas, ecommerce, healthcare, beauty, fintech, service |
| `landing` | ページ構成、CTA戦略 | hero, hero-centric, testimonial, pricing, social-proof |
| `chart` | チャートタイプ、ライブラリ推奨 | trend, comparison, timeline, funnel, pie |
| `ux` | ベストプラクティス、アンチパターン | animation, accessibility, z-index, loading |
| `react` | React/Next.jsパフォーマンス | waterfall, bundle, suspense, memo, rerender, cache |
| `web` | Webインターフェースガイドライン | aria, focus, keyboard, semantic, virtualize |
| `prompt` | AIプロンプト、CSSキーワード | （スタイル名） |

### 利用可能なスタック

| スタック | フォーカス |
|----------|-----------|
| `html-tailwind` | Tailwindユーティリティ、レスポンシブ、a11y（デフォルト） |
| `react` | State、hooks、パフォーマンス、パターン |
| `nextjs` | SSR、ルーティング、画像、APIルート |
| `vue` | Composition API、Pinia、Vue Router |
| `svelte` | Runes、stores、SvelteKit |
| `swiftui` | Views、State、Navigation、Animation |
| `react-native` | Components、Navigation、Lists |
| `flutter` | Widgets、State、Layout、Theming |
| `shadcn` | shadcn/uiコンポーネント、テーマ、フォーム、パターン |
| `jetpack-compose` | Composables、Modifiers、State Hoisting、Recomposition |

---

## ワークフロー例

**ユーザーリクエスト:** 「美容サロンのランディングページを作って」

### Step 1: 要件分析
- プロダクトタイプ: 美容/スパサービス
- スタイルキーワード: エレガント、プロフェッショナル、ソフト
- 業界: 美容/ウェルネス
- スタック: html-tailwind（デフォルト）

### Step 2: デザインシステム生成（必須）

```bash
python3 skills/ui-designer/scripts/search.py "beauty spa wellness service elegant" --design-system -p "Serenity Spa"
```

**出力:** パターン、スタイル、カラー、タイポグラフィ、エフェクト、アンチパターンを含む完全なデザインシステム。

### Step 3: 詳細検索で補完（必要に応じて）

```bash
# アニメーションとアクセシビリティのUXガイドラインを取得
python3 skills/ui-designer/scripts/search.py "animation accessibility" --domain ux

# 必要に応じて代替タイポグラフィを取得
python3 skills/ui-designer/scripts/search.py "elegant luxury serif" --domain typography
```

### Step 4: スタック別ガイドライン

```bash
python3 skills/ui-designer/scripts/search.py "layout responsive form" --stack html-tailwind
```

**その後:** デザインシステム + 詳細検索結果を統合してデザインを実装する。

---

## 出力フォーマット

`--design-system` フラグは2つの出力形式をサポート:

```bash
# ASCIIボックス（デフォルト）- ターミナル表示に最適
python3 skills/ui-designer/scripts/search.py "fintech crypto" --design-system

# Markdown - ドキュメント化に最適
python3 skills/ui-designer/scripts/search.py "fintech crypto" --design-system -f markdown
```

---

## 検索のコツ

1. **キーワードは具体的に** — 「app」より「healthcare SaaS dashboard」
2. **複数回検索** — 異なるキーワードで異なるインサイトが得られる
3. **ドメインを組み合わせる** — Style + Typography + Color = 完全なデザインシステム
4. **常にUXを確認** — 「animation」「z-index」「accessibility」で頻出問題をチェック
5. **stackフラグを使う** — スタック固有のベストプラクティスを取得
6. **反復する** — 最初の検索がマッチしなければ別のキーワードを試す

---

## プロフェッショナルUIのための共通ルール

UIがアマチュアに見える原因となる、見落としがちな問題:

### アイコン＆ビジュアル要素

| ルール | やるべき | やってはいけない |
|--------|----------|-----------------|
| **絵文字アイコン禁止** | SVGアイコンを使用（Heroicons、Lucide、Simple Icons） | UIアイコンとして絵文字を使用 |
| **安定したホバー状態** | hover時にcolor/opacityトランジション | レイアウトがずれるscaleトランスフォーム |
| **正確なブランドロゴ** | Simple Iconsから公式SVGを調査 | ロゴパスを推測したり不正確なものを使用 |
| **一貫したアイコンサイズ** | viewBox(24x24)固定でw-6 h-6を使用 | アイコンサイズをランダムに混在 |

### インタラクション＆カーソル

| ルール | やるべき | やってはいけない |
|--------|----------|-----------------|
| **カーソルポインター** | クリック/ホバー可能なカードに `cursor-pointer` 追加 | インタラクティブ要素でデフォルトカーソルのまま |
| **ホバーフィードバック** | ビジュアルフィードバック（色、影、ボーダー）を提供 | 要素がインタラクティブだと分からない状態 |
| **スムーズなトランジション** | `transition-colors duration-200` を使用 | 即時切り替えや500ms超の遅すぎるトランジション |

### ライト/ダークモードのコントラスト

| ルール | やるべき | やってはいけない |
|--------|----------|-----------------|
| **ライトモードのglassカード** | `bg-white/80` 以上の不透明度を使用 | `bg-white/10`（透明すぎる）を使用 |
| **ライトモードのテキストコントラスト** | テキストに `#0F172A`（slate-900）を使用 | 本文に `#94A3B8`（slate-400）を使用 |
| **ミュートテキスト（ライト）** | 最低 `#475569`（slate-600）を使用 | gray-400以下を使用 |
| **ボーダーの可視性** | ライトモードで `border-gray-200` を使用 | `border-white/10`（見えない）を使用 |

### レイアウト＆スペーシング

| ルール | やるべき | やってはいけない |
|--------|----------|-----------------|
| **フローティングナビバー** | `top-4 left-4 right-4` のスペーシング追加 | `top-0 left-0 right-0` にナビバーを固定 |
| **コンテンツパディング** | 固定ナビバーの高さを考慮 | 固定要素の後ろにコンテンツが隠れる |
| **一貫したmax-width** | 同じ `max-w-6xl` か `max-w-7xl` を使用 | コンテナ幅をバラバラに混在 |

---

## 納品前チェックリスト

UIコードを納品する前に以下を確認:

### ビジュアル品質
- [ ] アイコンに絵文字を使用していない（SVGを使用）
- [ ] 全アイコンが統一されたアイコンセット（Heroicons/Lucide）から
- [ ] ブランドロゴが正確（Simple Iconsで検証済み）
- [ ] ホバー状態でレイアウトシフトが発生しない
- [ ] テーマカラーを直接使用（bg-primary）、var()ラッパーではない

### インタラクション
- [ ] クリック可能な全要素に `cursor-pointer`
- [ ] ホバー状態で明確なビジュアルフィードバック
- [ ] トランジションがスムーズ（150〜300ms）
- [ ] キーボードナビゲーション用のフォーカス状態が可視

### ライト/ダークモード
- [ ] ライトモードのテキストが十分なコントラスト（最低4.5:1）
- [ ] Glass/透明要素がライトモードで可視
- [ ] ボーダーが両モードで可視
- [ ] 両モードでテスト済み

### レイアウト
- [ ] フローティング要素がエッジから適切なスペーシング
- [ ] 固定ナビバーの後ろにコンテンツが隠れていない
- [ ] 375px、768px、1024px、1440pxでレスポンシブ
- [ ] モバイルで水平スクロールなし

### アクセシビリティ
- [ ] 全画像にaltテキスト
- [ ] フォーム入力にラベル
- [ ] 色だけで情報を伝えていない
- [ ] `prefers-reduced-motion` を尊重

