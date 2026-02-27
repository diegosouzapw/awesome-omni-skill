---
name: frontend-coding
trigger: /frontend-coding
description: Next.js App Routerベースのフロントエンド実装スキル。UIコンポーネント、ページ、レイアウト、フォーム、React Queryフック、i18n対応の実装時に使用。backend/配下は除外。Radix UI + Tailwind CSS v4 + TypeScript + next-intl + React Query v5 + Better-Auth のパターンに従う。
---

# Frontend Coding

Next.js 16 + React 19 + TypeScript のフロントエンド実装ガイド。

## 技術スタック

- **Next.js 16** - App Router、Server Components
- **React 19** - React Compiler による自動メモ化
- **TypeScript** - 型安全性
- **Tailwind CSS v4** - @theme inline、OKLch カラースペース
- **React Query v5** - データフェッチ、キャッシング
- **next-intl** - 国際化（ja/en）
- **Better-Auth** - 認証（Google OAuth、One-Tap）
- **Radix UI + shadcn/ui** - UIコンポーネント
- **CVA (class-variance-authority)** - バリアント管理

## ディレクトリ構造

```
src/
├── app/                    # Next.js App Router
│   ├── layout.tsx          # ルートレイアウト
│   ├── globals.css         # グローバルスタイル（Tailwind v4）
│   ├── (user)/[locale]/    # ユーザー向けページ
│   │   ├── layout.tsx      # ロケールレイアウト（NextIntlClientProvider）
│   │   ├── error.tsx       # エラーバウンダリ
│   │   ├── not-found.tsx   # 404ページ
│   │   ├── (authenticated)/ # 認証後ページ
│   │   │   ├── layout.tsx  # 認証チェック + HydrationBoundary
│   │   │   └── {page}/
│   │   │       ├── page.tsx
│   │   │       └── _components/  # ページ固有コンポーネント
│   │   │           ├── container.tsx      # ロジック層
│   │   │           └── presentational.tsx # 表示層
│   │   └── (public)/        # 公開ページ
│   │       └── layout.tsx
│   └── (admin)/admin/      # 管理者向けページ
├── components/             # 共通UIコンポーネント
│   ├── ui/                 # Radix UI + shadcn/ui ベース
│   └── layout/
│       └── wrapper/        # ラッパーコンポーネント
│           └── RootLayoutWrapper/  # グローバルプロバイダー
├── features/               # 機能別フォルダ（複数ページで共有）
│   └── {feature}/
│       ├── types/          # 型定義
│       ├── queries/        # React Query クエリ定義
│       ├── mutations/      # React Query ミューテーション定義
│       ├── hooks/
│       │   ├── queries/    # useQuery カスタムフック
│       │   └── mutations/  # useMutation カスタムフック
│       └── components/
│           ├── ui/         # プレゼンテーション
│           └── layout/     # レイアウト
├── i18n/                   # 国際化設定
│   ├── routing.ts          # ルーティング定義
│   ├── request.ts          # リクエスト設定
│   └── navigation.ts       # useRouter, Link エクスポート
├── lib/                    # ユーティリティ・設定
│   ├── react-query/
│   │   └── query-client.ts # QueryClient 設定
│   ├── better-auth/
│   │   ├── auth-client.ts  # ユーザー認証クライアント
│   │   └── auth-admin-client.ts
│   └── shadcn/
│       └── utils.ts        # cn() ユーティリティ
├── providers/              # Reactプロバイダ
│   └── QueryProvider.tsx   # React Query Provider
├── messages/               # i18n メッセージ
│   ├── ja.json
│   └── en.json
├── utils/error/            # エラーユーティリティ
│   └── server-error.ts
└── env.ts                  # 環境変数（@t3-oss/env-nextjs）
```

## コンポーネント実装パターン

### shadcn/ui コンポーネントの追加

新しいshadcn/uiコンポーネントを追加する場合は、以下のコマンドを使用する:

```bash
pnpm dlx shadcn@latest add <component-name>
```

例:
```bash
pnpm dlx shadcn@latest add button
pnpm dlx shadcn@latest add dialog
pnpm dlx shadcn@latest add form
```

### UIコンポーネント (components/ui/)

```tsx
"use client"

import * as React from "react"
import { type VariantProps, cva } from "class-variance-authority"
import { cn } from "@/lib/shadcn/utils"

const componentVariants = cva("base-styles", {
  variants: {
    variant: {
      default: "...",
      destructive: "..."
    },
    size: {
      default: "h-9 px-4",
      sm: "h-8 px-3",
      lg: "h-10 px-6"
    }
  },
  defaultVariants: {
    variant: "default",
    size: "default"
  }
})

type Props = React.ComponentProps<"div"> &
  VariantProps<typeof componentVariants>

function Component({ className, variant, size, ...props }: Props) {
  return (
    <div
      data-slot="component-name"
      className={cn(componentVariants({ variant, size }), className)}
      {...props}
    />
  )
}

export { Component, componentVariants }
```

### 機能コンポーネント (features/{feature}/components/)

**UI層（プレゼンテーション）:**

```tsx
// features/{feature}/components/ui/FeatureButton/index.tsx
"use client"

import { Button } from "@/components/ui/button"

type Props = {
  onClick: () => void
  disabled?: boolean
  loading?: boolean
}

export const FeatureButton = ({ onClick, disabled, loading }: Props) => {
  return (
    <Button onClick={onClick} disabled={disabled || loading}>
      {loading ? "処理中..." : "実行"}
    </Button>
  )
}
```

**Container層（ロジック）:**

```tsx
// features/{feature}/components/layout/FeatureContainer/index.tsx
"use client"

import { useState } from "react"
import { useMutation } from "@tanstack/react-query"
import { useTranslations, useLocale } from "next-intl"
import { toast } from "sonner"
import { useRouter } from "@/i18n/navigation"
import { getQueryClient } from "@/lib/react-query/query-client"
import { FeatureButton } from "../../ui/FeatureButton"

export const FeatureContainer = () => {
  const t = useTranslations("feature")
  const locale = useLocale()
  const router = useRouter()
  const queryClient = getQueryClient()

  const mutation = useMutation({
    mutationFn: async () => { /* API呼び出し */ },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["feature-key"] })
    }
  })

  // ✅ mutateAsync + try-catch パターン（推奨）
  const handleAction = async () => {
    try {
      await mutation.mutateAsync()
      toast.success("処理が完了しました")
      router.push(`/${locale}/success`)
    } catch {
      toast.error("処理に失敗しました")
    }
  }

  return (
    <FeatureButton
      onClick={handleAction}
      loading={mutation.isPending}
    />
  )
}
```

## ページ固有コンポーネント（コンテナ・プレゼンテーショナルパターン）

ページ固有のコンポーネントは `_components` フォルダに配置し、**コンテナ・プレゼンテーショナルパターン**を使用する。

```
app/(user)/[locale]/(authenticated)/settings/
├── page.tsx                    # サーバーコンポーネント
└── _components/
    ├── container.tsx           # ロジック層（状態管理、API呼び出し）
    └── presentational.tsx      # 表示層（UIレンダリング）
```

### コンポーネント配置の使い分け

| 配置場所 | 用途 |
|---------|------|
| `app/.../page/_components/` | ページ固有のコンポーネント（container.tsx, presentational.tsx） |
| `app/.../{親ディレクトリ}/_components/` | 同一機能の複数ページで共有するコンポーネント（例: 作成・編集で共通のフォーム） |
| `features/{feature}/components/` | 複数機能で共有するコンポーネント |
| `components/ui/` | 汎用UIコンポーネント |

**例: 作成・編集で共通のフォームコンポーネント**

```
app/(admin)/admin/(authenticated)/products/
├── _components/
│   └── product-form.tsx     ← 作成・編集で共有
├── new/
│   └── _components/
│       ├── container.tsx    ← 作成ページ専用ロジック
│       └── presentational.tsx
└── [id]/
    └── edit/
        └── _components/
            ├── container.tsx    ← 編集ページ専用ロジック
            └── presentational.tsx
```

### container.tsx（ロジック層）

```tsx
// app/(user)/[locale]/(authenticated)/settings/_components/container.tsx
"use client"

import { useMutation } from "@tanstack/react-query"
import { useRouter } from "next/navigation"
import { useLocale } from "next-intl"
import { useState } from "react"
import { toast } from "sonner"
import { getQueryClient } from "@/lib/react-query/query-client"
import { SettingsPresentational } from "./presentational"

export function SettingsContainer() {
  const [isDialogOpen, setIsDialogOpen] = useState(false)
  const locale = useLocale()
  const router = useRouter()
  const queryClient = getQueryClient()

  const deleteMutation = useMutation({
    mutationFn: async () => {
      // API呼び出し
    },
    onSuccess: () => {
      queryClient.clear()
    }
  })

  // ✅ mutateAsync + try-catch パターン（推奨）
  const handleDelete = async () => {
    try {
      await deleteMutation.mutateAsync()
      toast.success("アカウントを削除しました")
      router.push(`/${locale}/sign-in`)
    } catch {
      toast.error("削除に失敗しました")
    }
  }

  return (
    <SettingsPresentational
      isDialogOpen={isDialogOpen}
      isDeleting={deleteMutation.isPending}
      onOpenDialog={() => setIsDialogOpen(true)}
      onCloseDialog={() => setIsDialogOpen(false)}
      onDelete={handleDelete}
    />
  )
}
```

### presentational.tsx（表示層）

```tsx
// app/(user)/[locale]/(authenticated)/settings/_components/presentational.tsx
"use client"

import { useTranslations } from "next-intl"
import { Button } from "@/components/ui/button"
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from "@/components/ui/card"

type SettingsPresentationalProps = {
  isDialogOpen: boolean
  isDeleting: boolean
  onOpenDialog: () => void
  onCloseDialog: () => void
  onDelete: () => void
}

export function SettingsPresentational({
  isDialogOpen,
  isDeleting,
  onOpenDialog,
  onCloseDialog,
  onDelete
}: SettingsPresentationalProps) {
  const t = useTranslations("settings")

  return (
    <div className="container max-w-2xl py-8">
      <h1 className="mb-8 text-2xl font-bold">{t("heading")}</h1>
      <Card>
        <CardHeader>
          <CardTitle>{t("deleteAccount.title")}</CardTitle>
          <CardDescription>{t("deleteAccount.description")}</CardDescription>
        </CardHeader>
        <CardContent>
          <Button variant="destructive" onClick={onOpenDialog}>
            {t("deleteAccount.button")}
          </Button>
        </CardContent>
      </Card>
      {/* AlertDialog は省略 */}
    </div>
  )
}
```

### page.tsx からの呼び出し

```tsx
// app/(user)/[locale]/(authenticated)/settings/page.tsx
import { setRequestLocale } from "next-intl/server"
import { SettingsContainer } from "./_components/container"

type Props = {
  params: Promise<{ locale: string }>
}

export default async function SettingsPage({ params }: Props) {
  const { locale } = await params
  setRequestLocale(locale)

  return <SettingsContainer />
}
```

## ページ実装パターン

### サーバーコンポーネント（認証なし）

```tsx
// app/(user)/[locale]/(public)/example/page.tsx
import { setRequestLocale } from "next-intl/server"

type Props = {
  params: Promise<{ locale: string }>
}

export default async function ExamplePage({ params }: Props) {
  const { locale } = await params
  setRequestLocale(locale)

  return <div>コンテンツ</div>
}
```

### サーバーコンポーネント（認証あり・プリフェッチ）

```tsx
// app/(user)/[locale]/(authenticated)/dashboard/page.tsx
import { setRequestLocale, getTranslations } from "next-intl/server"
import { HydrationBoundary, dehydrate } from "@tanstack/react-query"
import { getQueryClient } from "@/lib/react-query/query-client"
import { featureKey, getFeatureQuery } from "@/features/example/queries/get-feature"
import { DashboardContainer } from "@/features/example/components/layout/DashboardContainer"

type Props = {
  params: Promise<{ locale: string }>
}

export default async function DashboardPage({ params }: Props) {
  const { locale } = await params
  setRequestLocale(locale)
  const t = await getTranslations("dashboard")
  const queryClient = getQueryClient()

  await queryClient.prefetchQuery({
    queryKey: featureKey,
    queryFn: getFeatureQuery
  })

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      <h1>{t("title")}</h1>
      <DashboardContainer />
    </HydrationBoundary>
  )
}
```

## React Query パターン

### QueryClient 設定

```tsx
// lib/react-query/query-client.ts
import { QueryClient } from "@tanstack/react-query"

const createQueryClient = () => {
  return new QueryClient({
    defaultOptions: {
      queries: {
        staleTime: 1000 * 60 * 5,  // 5分
        gcTime: 1000 * 60 * 10,    // 10分
        retry: 1,
        refetchOnMount: true,
        refetchOnWindowFocus: false,
        refetchOnReconnect: true
      }
    }
  })
}

let browserQueryClient: QueryClient | undefined

export const getQueryClient = () => {
  if (typeof window === "undefined") {
    // サーバー: 毎回新しいインスタンス（リクエスト間の混在防止）
    return createQueryClient()
  }
  // ブラウザ: シングルトン
  if (!browserQueryClient) {
    browserQueryClient = createQueryClient()
  }
  return browserQueryClient
}
```

### QueryProvider

```tsx
// providers/QueryProvider.tsx
"use client"

import { QueryClientProvider } from "@tanstack/react-query"
import { useState } from "react"
import { getQueryClient } from "@/lib/react-query/query-client"

export const QueryProvider = ({ children }: { children: React.ReactNode }) => {
  const [queryClient] = useState(() => getQueryClient())

  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  )
}
```

### 型定義

**重要: フロントエンドの型定義はバックエンドから import しない**

型定義は `features/{feature}/types/` に独立して定義する。バックエンドの型を直接 import すると、フロントエンドとバックエンドの結合度が高くなり、変更時の影響範囲が大きくなるため避ける。

**重要: 型アサーション（as）の使用を避ける**

`as` による型アサーションは型安全性を損なうため使用しない。代わりに明示的なマッピングで型を変換する。

**重要: Zodスキーマから `z.infer` で型を推論する**

フォームなどでZodスキーマを定義している場合、手動で型を定義せず `z.infer` を使用する。スキーマと型が常に同期され、乖離を防げる。

```tsx
// features/{feature}/types/product-form.ts
import { z } from "zod"

export const productFormSchema = z.object({
  name: z.string().min(1, "商品名は必須です").max(255),
  description: z.string().max(5000).optional().or(z.literal("")),
  features: z.array(z.string()).optional(),
  displayOrder: z.number().int().min(0).optional()
})

// ❌ NG: 手動で型を定義（スキーマと乖離する可能性）
// export type ProductFormValues = {
//   name: string
//   description?: string
//   features?: string[]
//   displayOrder?: number
// }

// ✅ OK: z.infer でスキーマから型を推論
export type ProductFormValues = z.infer<typeof productFormSchema>
```

```tsx
// features/{feature}/types/product.ts

// ❌ NG: バックエンドから型をインポート
// export type { Product } from "@/backend/modules/billing/presentation/actions/find-products/find-products.action"

// ✅ OK: フロントエンド側で独立して型定義
export type Product = {
  id: string
  name: string
  description: string | null
  active: boolean
  createdAt: string
  updatedAt: string
}

export type ProductFilterStatus = "all" | "active" | "archived"
```

```tsx
// features/{feature}/queries/get-products.ts

// ❌ NG: 型アサーション（as）を使用
// return { products: res.data.products as Product[] }

// ✅ OK: 明示的なマッピングで型変換
const products: Product[] = res.data.products.map((p) => ({
  id: p.id,
  name: p.name,
  description: p.description,
  active: p.active,
  createdAt: p.createdAt,
  updatedAt: p.updatedAt
}))
return { products }
```

**重要: 不要なマッピングをしない**

Action のレスポンス型とフロントエンドの型が構造的に一致している場合、冗長なマッピングは行わず、レスポンスデータをそのまま返す。マッピングは型変換やフィールドの取捨選択が必要な場合にのみ行う。

```tsx
// ❌ NG: 型が一致しているのに冗長なマッピング
const subscription: Subscription = {
  id: res.data.subscription.id,
  name: res.data.subscription.name,
  status: res.data.subscription.status,
  // ... 全フィールドを手動でコピー
}
return { subscription }

// ✅ OK: 型が一致している場合はそのまま返す
return { subscription: res.data.subscription }
```

### Query Key 定義

```tsx
// features/{feature}/queries/keys.ts
export const featureKey = ["feature"] as const
export const featureDetailKey = (id: string) => ["feature", id] as const
```

### Query定義

```tsx
// features/{feature}/queries/get-feature.ts
import { getFeatureAction } from "@/backend/features/{feature}/actions/get-feature"
import { ServerError } from "@/utils/error/server-error"

export const getFeatureQuery = async () => {
  const res = await getFeatureAction()

  if (!res.ok) {
    throw new ServerError(
      res.error.code,
      res.error.status,
      res.error.message,
      res.error.details
    )
  }

  return res.data
}
```

### Query Hook

```tsx
// features/{feature}/hooks/queries/useGetFeatureQuery.ts
import { useQuery } from "@tanstack/react-query"
import { featureKey } from "../../queries/keys"
import { getFeatureQuery } from "../../queries/get-feature"

export const useGetFeatureQuery = () => {
  return useQuery({
    queryKey: featureKey,
    queryFn: getFeatureQuery
  })
}
```

### Mutation定義

**重要: Mutation関数の入力型もフロントエンド側で定義する**

バックエンドのAction型を直接importせず、フロントエンド側で定義した型を使用する。

```tsx
// features/{feature}/mutations/create-product.ts
import { createProductAction } from "@/backend/modules/billing/presentation/actions/create-product/create-product.action"
import { ServerError } from "@/utils/error/server-error"
// ✅ OK: フロントエンド側で定義した型を使用
import type { ProductFormValues } from "../types/product-form"

// ❌ NG: バックエンドの型を直接import
// import type { CreateProductActionRequest } from "@/backend/modules/billing/presentation/actions/create-product/create-product.action"

export const createProductMutation = async (input: ProductFormValues) => {
  const res = await createProductAction(input)

  if (!res.ok) {
    throw new ServerError(
      res.error.code,
      res.error.status,
      res.error.message,
      res.error.details
    )
  }

  return res.data
}
```

```tsx
// features/{feature}/mutations/delete-feature.ts
import { deleteFeatureAction } from "@/backend/features/{feature}/actions/delete-feature"
import { ServerError } from "@/utils/error/server-error"

export const deleteFeatureMutation = async () => {
  const res = await deleteFeatureAction()

  if (!res.ok) {
    throw new ServerError(
      res.error.code,
      res.error.status,
      res.error.message,
      res.error.details
    )
  }
}
```

### Mutation Hook

```tsx
// features/{feature}/hooks/mutations/useDeleteFeatureMutation.ts
import { useMutation } from "@tanstack/react-query"
import { deleteFeatureMutation } from "../../mutations/delete-feature"

export const useDeleteFeatureMutation = () => {
  return useMutation({
    mutationFn: deleteFeatureMutation
  })
}
```

### HydrationBoundary（SSR統合）

サーバーでプリフェッチしたデータをクライアントに引き継ぐ:

```tsx
// app/(user)/[locale]/(authenticated)/layout.tsx
import { dehydrate, HydrationBoundary } from "@tanstack/react-query"
import { getQueryClient } from "@/lib/react-query/query-client"

export const dynamic = "force-dynamic"

export default async function AuthenticatedLayout({ children, params }) {
  const { locale } = await params
  const queryClient = getQueryClient()

  // サーバーサイドでデータプリフェッチ
  await queryClient.prefetchQuery({
    queryKey: featureKey,
    queryFn: getFeatureQuery
  })

  return (
    <HydrationBoundary state={dehydrate(queryClient)}>
      {children}
    </HydrationBoundary>
  )
}
```

## i18n 実装

### メッセージ定義

```json
// messages/ja.json
{
  "feature": {
    "title": "機能タイトル",
    "description": "説明文",
    "button": {
      "submit": "送信",
      "cancel": "キャンセル"
    }
  }
}
```

### 使用方法

```tsx
// サーバーコンポーネント
import { getTranslations } from "next-intl/server"
const t = await getTranslations("feature")
t("title") // "機能タイトル"

// クライアントコンポーネント
import { useTranslations, useLocale } from "next-intl"
const t = useTranslations("feature")
const locale = useLocale()
```

## スタイリング規約

### Tailwind CSS v4 設定

```css
/* app/globals.css */
@import "tailwindcss";
@import "tw-animate-css";

@custom-variant dark (&:is(.dark *));

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-secondary: var(--secondary);
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);
  --color-accent: var(--accent);
  --color-destructive: var(--destructive);
  --color-border: var(--border);
  --color-ring: var(--ring);
  --radius-sm: calc(var(--radius) - 4px);
  --radius-md: calc(var(--radius) - 2px);
  --radius-lg: var(--radius);
  --radius-xl: calc(var(--radius) + 4px);
}

:root {
  --radius: 0.625rem;
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
  --primary: oklch(0.205 0 0);
  --primary-foreground: oklch(0.985 0 0);
  /* その他のカラートークン */
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  /* ダークモード用カラートークン */
}

@layer base {
  * {
    @apply border-border outline-ring/50;
  }
  body {
    @apply bg-background text-foreground;
  }
}
```

### cn() ユーティリティ

```tsx
// lib/shadcn/utils.ts
import { type ClassValue, clsx } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

```tsx
// 使用例
import { cn } from "@/lib/shadcn/utils"

<div className={cn(
  "base-class",
  condition && "conditional-class",
  className
)} />
```

### CVA (Class Variance Authority)

```tsx
import { cva, type VariantProps } from "class-variance-authority"

const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md text-sm font-medium transition-colors",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive: "bg-destructive text-white hover:bg-destructive/90",
        outline: "border bg-background hover:bg-accent",
        ghost: "hover:bg-accent hover:text-accent-foreground"
      },
      size: {
        default: "h-9 px-4 py-2",
        sm: "h-8 rounded-md px-3",
        lg: "h-10 rounded-md px-6",
        icon: "size-9"
      }
    },
    defaultVariants: {
      variant: "default",
      size: "default"
    }
  }
)

type ButtonProps = React.ComponentProps<"button"> &
  VariantProps<typeof buttonVariants>

function Button({ className, variant, size, ...props }: ButtonProps) {
  return (
    <button
      className={cn(buttonVariants({ variant, size, className }))}
      {...props}
    />
  )
}
```

### data-slot 属性

テストやセレクタ用にコンポーネントを識別:

```tsx
<div data-slot="feature-card">
  <h2 data-slot="feature-card-title">{title}</h2>
</div>
```

### Tailwind クラス順序

1. レイアウト（flex, grid, block）
2. サイズ（w-, h-, min-, max-）
3. スペーシング（p-, m-, gap-）
4. ボーダー・背景
5. テキスト
6. 状態（hover:, focus:, disabled:）

## エラーハンドリング

### ErrorAlert コンポーネント

```tsx
import { ErrorAlert } from "@/components/ui/error-alert"

{errors.length > 0 && <ErrorAlert messages={errors} />}
```

### ServerError クラス

```tsx
import { ServerError, ValidationServerError } from "@/utils/error/server-error"

// 一般エラー
throw new ServerError("ERROR_CODE", 500, "エラーメッセージ")

// バリデーションエラー
throw new ValidationServerError("VALIDATION_ERROR", 400, "入力エラー", {
  field: ["エラー詳細"]
})
```

## 認証パターン（Better-Auth）

### 認証クライアント設定

```tsx
// lib/better-auth/auth-client.ts
import "client-only"

import { oneTapClient } from "better-auth/client/plugins"
import { createAuthClient } from "better-auth/react"
import { env } from "@/env"

export const authClient = createAuthClient({
  baseURL: env.NEXT_PUBLIC_ORIGIN,
  plugins: [
    oneTapClient({
      clientId: env.NEXT_PUBLIC_GOOGLE_CLIENT_ID,
      cancelOnTapOutside: false,
      context: "signin",
      promptOptions: {
        // FedCM はHTTPS環境のみ有効
        fedCM: env.NEXT_PUBLIC_ORIGIN.startsWith("https://")
      }
    })
  ]
})
```

### サインイン実装（One-Tap対応）

```tsx
// app/(user)/[locale]/(public)/sign-in/_components/container.tsx
"use client"

import { useLocale } from "next-intl"
import { useState } from "react"
import { useEffectOnce } from "react-use"
import { authClient } from "@/lib/better-auth/auth-client"
import { SignInPresentational } from "./presentational"

export function SignInContainer() {
  const [isLoading, setIsLoading] = useState(false)
  const locale = useLocale()

  const handleGoogleSignIn = async () => {
    setIsLoading(true)
    try {
      await authClient.signIn.social({
        provider: "google",
        callbackURL: `/${locale}/home`
      })
    } finally {
      setIsLoading(false)
    }
  }

  // One-Tap サインインの初期化
  useEffectOnce(() => {
    authClient.oneTap({
      callbackURL: `/${locale}/home`
    })
  })

  return (
    <SignInPresentational
      onGoogleSignIn={handleGoogleSignIn}
      isLoading={isLoading}
    />
  )
}
```

### サインアウト実装

```tsx
"use client"

import { useMutation, useQueryClient } from "@tanstack/react-query"
import { useRouter } from "next/navigation"
import { useLocale } from "next-intl"
import { authClient } from "@/lib/better-auth/auth-client"

export function useSignOut() {
  const locale = useLocale()
  const router = useRouter()
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: async () => {
      await authClient.signOut()
    },
    onSuccess: () => {
      queryClient.clear()  // 全キャッシュをクリア
      router.push(`/${locale}/sign-in`)
    }
  })
}
```

### 認証ガード（レイアウト）

```tsx
// app/(user)/[locale]/(authenticated)/layout.tsx
import { redirect } from "next/navigation"
import { dehydrate, HydrationBoundary } from "@tanstack/react-query"
import { getQueryClient } from "@/lib/react-query/query-client"
import { authUserKey, getAuthUserQuery } from "@/features/auth/queries/get-auth-user"
import { Toaster } from "@/components/ui/sonner"
import { AuthUserMenu } from "@/features/auth/components/layout/AuthUserMenu"

export const dynamic = "force-dynamic"

export default async function AuthenticatedLayout({
  children,
  params
}: {
  children: React.ReactNode
  params: Promise<{ locale: string }>
}) {
  const { locale } = await params
  const queryClient = getQueryClient()

  const { authUser } = await queryClient.fetchQuery({
    queryKey: authUserKey,
    queryFn: getAuthUserQuery
  })

  if (!authUser) {
    redirect(`/${locale}/sign-in`)
  }

  return (
    <div className="min-h-screen">
      <header className="sticky top-0 z-50 border-b bg-background">
        <div className="container flex h-14 items-center justify-end">
          <AuthUserMenu />
        </div>
      </header>
      <main>
        <HydrationBoundary state={dehydrate(queryClient)}>
          {children}
        </HydrationBoundary>
      </main>
      <Toaster />
    </div>
  )
}
```

## グローバルプロバイダー構成

### RootLayoutWrapper

```tsx
// components/layout/wrapper/RootLayoutWrapper/index.tsx
"use client"

import type { PropsWithChildren } from "react"
import { QueryProvider } from "@/providers/QueryProvider"

export const RootLayoutWrapper = ({ children }: PropsWithChildren) => {
  return <QueryProvider>{children}</QueryProvider>
}
```

### ルートレイアウト

```tsx
// app/layout.tsx
import type { Metadata } from "next"
import { RootLayoutWrapper } from "@/components/layout/wrapper/RootLayoutWrapper"
import "./globals.css"

export const metadata: Metadata = {
  title: "App Title",
  description: "App Description"
}

export default function RootLayout({
  children
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body>
        <RootLayoutWrapper>{children}</RootLayoutWrapper>
      </body>
    </html>
  )
}
```

### ロケールレイアウト

```tsx
// app/(user)/[locale]/layout.tsx
import { hasLocale, setRequestLocale } from "next-intl/server"
import { notFound } from "next/navigation"
import { NextIntlClientProvider } from "next-intl"
import { getMessages } from "next-intl/server"
import { routing } from "@/i18n/routing"

export function generateStaticParams() {
  return routing.locales.map((locale) => ({ locale }))
}

export default async function LocaleLayout({
  children,
  params
}: {
  children: React.ReactNode
  params: Promise<{ locale: string }>
}) {
  const { locale } = await params

  if (!hasLocale(routing.locales, locale)) {
    notFound()
  }

  setRequestLocale(locale)
  const messages = await getMessages()

  return (
    <NextIntlClientProvider messages={messages}>
      {children}
    </NextIntlClientProvider>
  )
}
```

## エラーページ実装

### error.tsx

```tsx
// app/(user)/[locale]/error.tsx
"use client"

import { useEffect } from "react"
import { useTranslations } from "next-intl"
import { Button } from "@/components/ui/button"

type Props = {
  error: Error & { digest?: string }
  reset: () => void
}

export default function ErrorPage({ error, reset }: Props) {
  const t = useTranslations("errors.general")

  useEffect(() => {
    console.error(error)
  }, [error])

  return (
    <div className="flex min-h-screen flex-col items-center justify-center gap-4">
      <h2 className="text-2xl font-bold">{t("title")}</h2>
      <p className="text-muted-foreground">{t("description")}</p>
      <Button onClick={() => reset()}>{t("retry")}</Button>
    </div>
  )
}
```

### not-found.tsx

```tsx
// app/(user)/[locale]/not-found.tsx
import { getTranslations } from "next-intl/server"
import { Link } from "@/i18n/navigation"
import { Button } from "@/components/ui/button"

export default async function NotFoundPage() {
  const t = await getTranslations("errors.notFound")

  return (
    <div className="flex min-h-screen flex-col items-center justify-center gap-4">
      <h2 className="text-2xl font-bold">{t("title")}</h2>
      <p className="text-muted-foreground">{t("description")}</p>
      <Button asChild>
        <Link href="/">{t("backToHome")}</Link>
      </Button>
    </div>
  )
}
```

## 実装完了後の必須ステップ

**実装が完了したら必ず以下を実行:**

```bash
pnpm type:check
```

エラーが出た場合は、すべてのエラーを解消するまで修正を続ける。型エラーが残った状態で実装完了としない。

## チェックリスト

新規実装時の確認事項:

### コンポーネント配置
- [ ] `"use client"` の有無を確認
- [ ] ページ固有コンポーネントは `_components/` に配置（container.tsx + presentational.tsx）
- [ ] 同一機能の複数ページで共有するコンポーネントは親ディレクトリの `_components/` に配置
- [ ] 複数機能で共有するコンポーネントは `features/{feature}/components/` に配置
- [ ] 型定義は `features/{feature}/types/` に独立して定義（バックエンドから import しない）
- [ ] Zodスキーマがある場合は `z.infer` で型を推論（手動定義しない）

### React Query
- [ ] Query定義は `features/{feature}/queries/` に配置
- [ ] Mutation定義は `features/{feature}/mutations/` に配置
- [ ] カスタムフックは `features/{feature}/hooks/queries/` または `hooks/mutations/` に配置
- [ ] サーバーコンポーネントで `HydrationBoundary` + `dehydrate` を使用（必要時）

### i18n
- [ ] ja.json, en.json に翻訳追加
- [ ] サーバーコンポーネント: `getTranslations`, `setRequestLocale`
- [ ] クライアントコンポーネント: `useTranslations`, `useLocale`
- [ ] ナビゲーション: `@/i18n/navigation` の `useRouter`, `Link` を使用

### スタイリング
- [ ] `cn()` でクラス合成
- [ ] `data-slot` 属性でコンポーネント識別
- [ ] CVA でバリアント管理（必要時）

### 品質
- [ ] エラーハンドリング実装（ServerError クラス使用）
- [ ] アクセシビリティ対応（aria-* 属性）
- [ ] `pnpm type:check` が通ること（必須）
