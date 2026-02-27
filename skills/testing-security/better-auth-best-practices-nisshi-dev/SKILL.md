---
name: better-auth-best-practices
description: Better Auth（TypeScript 認証フレームワーク）の統合ガイド。セットアップ、セッション管理、プラグイン、セキュリティ設定に精通。
---

# Better Auth 統合ガイド

**コード例と最新 API は [better-auth.com/docs](https://better-auth.com/docs) を必ず参照すること。**

Better Auth は TypeScript ファーストでフレームワーク非依存の認証フレームワーク。メール/パスワード、OAuth、マジックリンク、パスキーなどをプラグインでサポート。

---

## クイックリファレンス

### 環境変数
- `BETTER_AUTH_SECRET` - 暗号化シークレット（最低 32 文字）。生成: `openssl rand -base64 32`
- `BETTER_AUTH_URL` - ベース URL（例: `https://example.com`）

環境変数が設定されていない場合のみ、config で `baseURL`/`secret` を定義する。

### ファイルの配置場所
CLI は `auth.ts` を以下の順で検索: `./`, `./lib`, `./utils`, または `./src` 配下。カスタムパスは `--config` で指定。

### CLI コマンド
- `npx @better-auth/cli@latest migrate` - スキーマ適用（組み込みアダプター）
- `npx @better-auth/cli@latest generate` - Prisma/Drizzle 用スキーマ生成
- `npx @better-auth/cli mcp --cursor` - AI ツールに MCP を追加

**プラグインの追加・変更後は必ず再実行すること。**

---

## コア設定オプション

| オプション | 説明 |
|-----------|------|
| `appName` | 表示名（任意） |
| `baseURL` | `BETTER_AUTH_URL` 未設定時のみ |
| `basePath` | デフォルト `/api/auth`。ルートに配置する場合は `/` |
| `secret` | `BETTER_AUTH_SECRET` 未設定時のみ |
| `database` | ほとんどの機能で必須。アダプタードキュメント参照 |
| `secondaryStorage` | セッション・レート制限用の Redis/KV |
| `emailAndPassword` | `{ enabled: true }` で有効化 |
| `socialProviders` | `{ google: { clientId, clientSecret }, ... }` |
| `plugins` | プラグインの配列 |
| `trustedOrigins` | CSRF ホワイトリスト |

---

## データベース

**直接接続:** `pg.Pool`、`mysql2` プール、`better-sqlite3`、または `bun:sqlite` インスタンスを渡す。

**ORM アダプター:** `better-auth/adapters/drizzle`、`better-auth/adapters/prisma`、`better-auth/adapters/mongodb` からインポート。

**重要:** Better Auth はアダプターのモデル名を使用し、DB のテーブル名は使わない。Prisma モデルが `User` でテーブルが `users` にマッピングされている場合、`modelName: "user"`（Prisma の参照名）を使い、`"users"` ではない。

---

## セッション管理

**ストレージの優先順位:**
1. `secondaryStorage` が定義されている場合 → セッションはそこに保存（DB ではない）
2. DB にも永続化する場合は `session.storeSessionInDatabase: true` を設定
3. DB なし + `cookieCache` → 完全ステートレスモード

**Cookie キャッシュ戦略:**
- `compact`（デフォルト）- Base64url + HMAC。最小サイズ
- `jwt` - 標準 JWT。読み取り可能だが署名付き
- `jwe` - 暗号化。最大セキュリティ

**主要オプション:** `session.expiresIn`（デフォルト 7 日）、`session.updateAge`（リフレッシュ間隔）、`session.cookieCache.maxAge`、`session.cookieCache.version`（変更するとすべてのセッションを無効化）。

---

## ユーザーとアカウント設定

**ユーザー:** `user.modelName`、`user.fields`（カラムマッピング）、`user.additionalFields`、`user.changeEmail.enabled`（デフォルト無効）、`user.deleteUser.enabled`（デフォルト無効）。

**アカウント:** `account.modelName`、`account.accountLinking.enabled`、`account.storeAccountCookie`（ステートレス OAuth 用）。

**登録に必須:** `email` と `name` フィールド。

---

## メールフロー

- `emailVerification.sendVerificationEmail` - 認証を機能させるには定義が必須
- `emailVerification.sendOnSignUp` / `sendOnSignIn` - 自動送信トリガー
- `emailAndPassword.sendResetPassword` - パスワードリセットメールハンドラー

---

## セキュリティ

**`advanced` 内:**
- `useSecureCookies` - HTTPS Cookie を強制
- `disableCSRFCheck` - セキュリティリスクあり
- `disableOriginCheck` - セキュリティリスクあり
- `crossSubDomainCookies.enabled` - サブドメイン間で Cookie を共有
- `ipAddress.ipAddressHeaders` - プロキシ用カスタム IP ヘッダー
- `database.generateId` - カスタム ID 生成、または `"serial"`/`"uuid"`/`false`

**レート制限:** `rateLimit.enabled`、`rateLimit.window`、`rateLimit.max`、`rateLimit.storage`（"memory" | "database" | "secondary-storage"）。

---

## フック

**エンドポイントフック:** `hooks.before` / `hooks.after` - `{ matcher, handler }` の配列。`createAuthMiddleware` を使用。`ctx.path`、`ctx.context.returned`（after）、`ctx.context.session` にアクセス可能。

**データベースフック:** `databaseHooks.user.create.before/after`、`session`、`account` も同様。デフォルト値の追加や作成後のアクションに有用。

**フックコンテキスト (`ctx.context`):** `session`、`secret`、`authCookies`、`password.hash()`/`verify()`、`adapter`、`internalAdapter`、`generateId()`、`tables`、`baseURL`。

---

## プラグイン

**ツリーシェイキングのため専用パスからインポート:**
```
import { twoFactor } from "better-auth/plugins/two-factor"
```
`from "better-auth/plugins"` からはインポートしない。

**主要プラグイン:** `twoFactor`、`organization`、`passkey`、`magicLink`、`emailOtp`、`username`、`phoneNumber`、`admin`、`apiKey`、`bearer`、`jwt`、`multiSession`、`sso`、`oauthProvider`、`oidcProvider`、`openAPI`、`genericOAuth`。

クライアントプラグインは `createAuthClient({ plugins: [...] })` に配置。

---

## クライアント

インポート元: `better-auth/client`（バニラ）、`better-auth/react`、`better-auth/vue`、`better-auth/svelte`、`better-auth/solid`。

主要メソッド: `signUp.email()`、`signIn.email()`、`signIn.social()`、`signOut()`、`useSession()`、`getSession()`、`revokeSession()`、`revokeSessions()`。

---

## 型安全性

型の推論: `typeof auth.$Infer.Session`、`typeof auth.$Infer.Session.user`。

クライアント/サーバーが別プロジェクトの場合: `createAuthClient<typeof auth>()`。

---

## よくあるハマりどころ

1. **モデル名 vs テーブル名** - 設定では ORM のモデル名を使用し、DB のテーブル名は使わない
2. **プラグインのスキーマ** - プラグイン追加後に CLI を再実行する
3. **セカンダリストレージ** - セッションはデフォルトでそちらに保存され、DB には保存されない
4. **Cookie キャッシュ** - カスタムセッションフィールドはキャッシュされず、常に再取得される
5. **ステートレスモード** - DB なし = セッションは Cookie のみ、キャッシュ期限でログアウト
6. **メールアドレス変更フロー** - まず現在のメールに送信、その後新しいメールに送信

---

## リソース

- [ドキュメント](https://better-auth.com/docs)
- [オプションリファレンス](https://better-auth.com/docs/reference/options)
- [LLMs.txt](https://better-auth.com/llms.txt)
- [GitHub](https://github.com/better-auth/better-auth)
- [初期化オプションのソース](https://github.com/better-auth/better-auth/blob/main/packages/core/src/types/init-options.ts)
