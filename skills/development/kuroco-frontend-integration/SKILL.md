---
name: kuroco-frontend-integration
description: Kurocoとフロントエンドフレームワークの統合パターンおよびAI自動デプロイメント。使用キーワード：「Kuroco Nuxt」「Kuroco Next.js」「フロントエンド連携」「Nuxt3」「Nuxt2」「App Router」「Pages Router」「SSG」「SSR」「静的生成」「コンテンツ表示」「ログイン実装」「会員登録」「signup」「KurocoPages」「フロントエンド環境構築」「Vue」「React」「useAsyncData」「$fetch」「asyncData」「composable」「useAuth」「認証状態」「プロフィール取得」「profile」「generateStaticParams」「動的ルート」「v-html」「dangerouslySetInnerHTML」「XSS対策」「サードパーティCookie」「credentials include」「AI自動デプロイ」「Kuroco自動化」「サイト登録API」「自動ビルド」「自動デプロイ」「temp-upload」「presigned URL」「S3アップロード」「artifact_url」「KurocoFront」「プレビューデプロイ」「stage_url」「add_site」「site_key」「kuroco_front/deploy」「CI/CD」「フロントエンドビルド」「ZIP配備」「自動公開」「nuxt generate」「next build」「vite build」「デプロイAPI」「一時アップロード」「署名付きURL」。Nuxt/Next.jsでのKuroco連携、認証実装、SSG/SSR設定、AI自動デプロイに関する質問で使用。
---

# Kuroco フロントエンド統合パターン

Kuroco HeadlessCMSとNuxt.js/Next.jsなどのフロントエンドフレームワークの統合パターン、およびAI自動デプロイメント。

**ドキュメント参照**: `/kuroco-docs` スキルを使用してKuroco公式ドキュメントを検索・参照できます。

**チュートリアル**: フロントエンドのデプロイ手順やサンプルサイトの構築方法は [Kurocoサンプルサイトチュートリアル](https://kuroco.app/ja/docs/tutorials/kuroco-sample-site/) を参照してください。

## 目次

- [サポートフレームワーク](#サポートフレームワーク)
- [環境設定](#環境設定)
- [API設定の前提条件](#api設定の前提条件)
- [認証実装](#認証実装)
- [Nuxt.js統合](#nuxtjs統合) → 詳細は [references/nuxt.md](references/nuxt.md)
- [Next.js統合](#nextjs統合) → 詳細は [references/nextjs.md](references/nextjs.md)
- [AI自動デプロイ](#ai自動デプロイ) → 詳細は [references/ai-deployment.md](references/ai-deployment.md)

## サポートフレームワーク

| フレームワーク | バージョン | 推奨ユースケース |
|--------------|-----------|----------------|
| Nuxt.js 3.x | Vue 3系 | 新規プロジェクト（推奨） |
| Nuxt.js 2.x | Vue 2系 | 既存プロジェクト |
| Next.js 13+ | React (App Router) | 新規Reactプロジェクト |
| Next.js (Pages) | React (Pages Router) | 既存Reactプロジェクト |

## 環境設定

### 環境変数

```bash
# .env.local
NUXT_PUBLIC_API_BASE=https://example.g.kuroco.app
NEXT_PUBLIC_API_BASE=https://example.g.kuroco.app
API_ID=1
```

### プロジェクト構成

**Nuxt.js:**
```
pages/
├── news/
│   ├── index.vue      # 一覧
│   └── [slug].vue     # 詳細 (Nuxt3)
├── login.vue
└── profile.vue
composables/
├── useAuth.ts
└── useApi.ts
```

**Next.js (App Router):**
```
app/
├── news/
│   ├── page.tsx       # 一覧
│   └── [slug]/page.tsx
├── login/page.tsx
└── profile/page.tsx
lib/
├── auth.ts
└── api.ts
```

## API設定の前提条件

### 1. セキュリティ設定（Cookie認証）

1. 管理画面 → API → セキュリティ → **Cookie**を選択
2. フロントエンドとAPIドメインをサブドメイン違いに設定
   - 例: `www.example.com` と `api.example.com`

### 2. CORS設定

管理画面: [API] → [セキュリティ] → [CORS設定]

```
CORS_ALLOW_ORIGINS:
  - http://localhost:3000
  - https://your-frontend-domain.com

CORS_ALLOW_CREDENTIALS: true

CORS_ALLOW_METHODS:
  - GET
  - POST
```

## 認証実装

### ログイン

```typescript
interface LoginResponse {
  grant_token: string
  status: number
  member_id: number
}

async function login(email: string, password: string): Promise<LoginResponse> {
  const response = await fetch(
    'https://example.g.kuroco.app/rcms-api/1/login',
    {
      method: 'POST',
      credentials: 'include',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password })
    }
  )

  if (!response.ok) {
    const error = await response.json()
    throw new Error(error.errors?.[0]?.message || 'ログインに失敗しました')
  }

  return response.json()
}
```

### ログアウト

```typescript
async function logout(): Promise<void> {
  await fetch('https://example.g.kuroco.app/rcms-api/1/logout', {
    method: 'POST',
    credentials: 'include'
  })
}
```

### ログイン状態の確認

```typescript
async function checkAuth(): Promise<ProfileResponse | null> {
  try {
    const response = await fetch(
      'https://example.g.kuroco.app/rcms-api/1/profile',
      { credentials: 'include' }
    )
    if (!response.ok) return null
    return response.json()
  } catch {
    return null
  }
}
```

### 会員登録

```typescript
async function signup(memberData: SignupData): Promise<void> {
  const response = await fetch(
    'https://example.g.kuroco.app/rcms-api/1/member/insert',
    {
      method: 'POST',
      credentials: 'include',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(memberData)
    }
  )

  if (!response.ok) {
    const error = await response.json()
    throw new Error(error.errors?.[0]?.message || '登録に失敗しました')
  }
}
```

## Nuxt.js統合

**詳細な実装例**: [references/nuxt.md](references/nuxt.md) を参照

クイックスタート（Nuxt 3）:

```typescript
// composables/useKurocoApi.ts
export function useKurocoApi() {
  const config = useRuntimeConfig()

  async function get<T>(endpoint: string, params?: Record<string, any>): Promise<T> {
    const query = params ? `?${new URLSearchParams(params)}` : ''
    return await $fetch<T>(
      `${config.public.apiBase}/rcms-api/${config.public.apiId}/${endpoint}${query}`,
      { credentials: 'include' }
    )
  }

  return { get }
}
```

## Next.js統合

**詳細な実装例**: [references/nextjs.md](references/nextjs.md) を参照

クイックスタート（App Router）:

```typescript
// lib/api.ts
export async function apiGet<T>(endpoint: string): Promise<T> {
  const response = await fetch(
    `${process.env.NEXT_PUBLIC_API_BASE}/rcms-api/1/${endpoint}`,
    { credentials: 'include', cache: 'no-store' }
  )
  if (!response.ok) throw new Error(`API Error: ${response.status}`)
  return response.json()
}
```

## KurocoPages統合

KurocoPagesはKurocoが提供するフロントエンドホスティングサービス。

```json
// kuroco_front.json
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}
```

デプロイ: 管理画面 → フロントエンド → KurocoPages → GitHubリポジトリ連携

## 注意事項

### サードパーティCookie問題

SafariなどではサードパーティCookieがブロックされます。

**解決策**: APIドメインとフロントエンドドメインを同一ドメイン（サブドメイン違い）に設定

### HTMLサニタイズ

`v-html` や `dangerouslySetInnerHTML` を使用する際はXSSに注意:

```typescript
import DOMPurify from 'dompurify'
const sanitizedHtml = DOMPurify.sanitize(htmlContent)
```

## AI自動デプロイ

AIがKurocoサイトの登録からフロントエンドのビルド・デプロイまでを自動実行します。

**詳細なワークフロー**: [references/ai-deployment.md](references/ai-deployment.md) を参照

### デプロイの流れ

1. 認証確認（whoami）
2. ユーザー確認（デプロイ先、モード）
3. サイト登録（新規の場合）
4. フロントエンドビルド（nuxt generate / next build / vite build）
5. アーティファクトアップロード（署名付きURL → S3）
6. デプロイ実行（KurocoFront）

### 関連リファレンス

- [references/ai-deployment.md](references/ai-deployment.md) - デプロイワークフロー全体
- [references/schemas.md](references/schemas.md) - パラメータリファレンス
- [references/site-registration.md](references/site-registration.md) - サイト登録API詳細
- [references/temp-upload.md](references/temp-upload.md) - 一時アップロードAPI詳細
- [references/deploy.md](references/deploy.md) - デプロイAPI詳細

---

## 関連スキル

- `/kuroco-api-content` - API設計・認証パターン、コンテンツCRUD操作
- `/kuroco-admin-api-browser` - 管理API（admin_api）の操作

## 関連ドキュメント

- `../kuroco-docs/docs/tutorials/integrate-kuroco-with-nuxt.md` - Nuxt.js統合
- `../kuroco-docs/docs/tutorials/integrate-login.md` - ログイン実装
- `../kuroco-docs/docs/tutorials/signup.md` - 会員登録
- `../kuroco-docs/docs/tutorials/beginners-guide.md` - ビギナーズガイド
- `../kuroco-docs/docs/tutorials/corporate-sample-site-to-ssg.md` - SSG対応
- [Kurocoサンプルサイトチュートリアル](https://kuroco.app/ja/docs/tutorials/kuroco-sample-site/) - サンプルサイトの構築・デプロイ手順
