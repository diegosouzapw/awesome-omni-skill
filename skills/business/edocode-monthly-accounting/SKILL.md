---
name: edocode-monthly-accounting
description: EDOCODE 株式会社の毎月の経理処理（証憑ファイル回収）を行うスキル。ユーザーから「EDOCODE の経理処理を始めよう」「EDOCODE の月次精算をお願い」などの依頼があった場合に使用する。Claude Code（`claude --chrome`）または Cowork 環境で、Claude in Chrome を使ったブラウザ操作により、各サービスの管理画面から請求書・領収書 PDF を自動回収する。ログイン等ユーザーにしかできない操作は都度依頼する。
---

# EDOCODE Monthly Accounting v1

EDOCODE 株式会社で共通利用しているサービスのうち、管理画面から請求書・領収書 PDF を直接ダウンロードする必要があるものについて、Claude が主導して証憑ファイルを回収する。

## Prerequisites（前提条件）

- Claude Code（`claude --chrome` で起動）または Cowork 環境で実行すること
- Claude in Chrome 拡張機能が有効であること
- Keeper Fill が Chrome にインストール済みであること
- Google Workspace（`yamada.k@edocode.co.jp`）にログイン済みであること
- Cowork 環境の場合：ネットワーク allowlist に各サービスのドメインが追加されていること

## Preflight Checks（事前チェック）

Phase 1 に入る前に、以下のチェックを順番に実行する。すべてクリアしてから処理を開始する。

### Check 1: Chrome Connection（Chrome 接続確認）

Claude in Chrome との接続状態を確認する。

- 接続済みの場合 → Check 2 に進む
- 未接続の場合 → ユーザーに「Claude in Chrome が接続されていません。Chrome で Claude 拡張機能が有効になっていることを確認し、接続してください。」と依頼する。接続が確認できるまで待機する。

### Check 2: Downloads Folder Access（ダウンロードフォルダのアクセス確認）

このワークフローでは PDF をダウンロードするため、ダウンロードフォルダへのアクセスが必要。

- Claude Code の場合：ダウンロードフォルダ（通常 `~/Downloads`）にアクセスできることを確認する
- Cowork の場合：ダウンロードフォルダを作業フォルダとして追加するようユーザーに依頼する
- アクセスが確認できたら → Preflight Complete に進む

### Preflight Complete

すべてのチェックが完了したら：

1. 対象月の確認を行う（Common: Month Calculation 参照）
2. 作業フォルダを作成する（Common: Workspace Setup 参照）
3. 監査ログを初期化する（Common: Audit Log 参照）
4. 「事前チェック完了。証憑ファイルの回収を開始します。」と通知し、Phase 1 に進む

---

## Target Services（対象サービス）

| # | Service | Type | Login Method | Login ID | Billing URL |
|---|---------|------|-------------|----------|-------------|
| 1 | fondesk | 電話代行 | メール + パスワード（Keeper Fill） | `service-account@edocode.co.jp` | `https://www.fondesk.jp/app/account/invoices` |
| 2 | Figma | デザインツール | Google SSO（ポップアップ型） | `yamada.k@edocode.co.jp` | `https://www.figma.com/files/team/690131495837577799/team-admin-console/billing/invoices` |
| 3 | TimeRex | 日程調整 | Google SSO（ページ遷移型） | `yamada.k@edocode.co.jp` | `https://timerex.net/user/team/1/billing` |
| 4 | Bitly | URL短縮 | メール + パスワード（Keeper Fill） | `service-account@edocode.co.jp` | `https://app.bitly.com/settings/organization/Okbk5aX3uEc/billing` |
| 5 | Krisp | ノイズキャンセル | Google SSO（ページ遷移型） | `yamada.k@edocode.co.jp` | `https://account.krisp.ai/billing-team/details` |
| 6 | n8n | ワークフロー自動化 | メール + パスワード（2段階ログイン） | `yamada.k@edocode.co.jp` | `https://app.n8n.cloud/manage/billing` |
| 7 | Microsoft | クラウドサービス | Microsoft SSO（ユーザー操作） | `yamada.k@edocode.co.jp` | `https://admin.cloud.microsoft/#/billoverview/invoice-list` |
| 8 | OpenAI | AI サービス | Google SSO（ページ遷移型）+ MFA | `yamada.k@edocode.co.jp` | `https://chatgpt.com/admin/billing?tab=invoices` |

## Execution Order（実行順序）

上記の番号順に実行する。各サービスの開始時にユーザーに「Service {#}: {サービス名} を開始します」と通知する。

---

## Common: Login Handling（共通：ログイン処理）

各サービスのページを開いた際、以下の判定を行う：

1. ページ内容からログイン状態を判定する
2. **ログイン済みの場合** → そのままデータ取得を続行
3. **未ログイン（ログインページにリダイレクトされた等）の場合：**

### Pattern A: メール + パスワード（Keeper Fill）

fondesk, Bitly など、メールアドレスとパスワードでログインするサービス。

1. ログインページが表示されたら、メールアドレス欄にスキルに登録された ID が入力されているか確認する
2. Keeper Fill がパスワードを自動入力しているか確認する
3. **正しい ID と Keeper Fill のパスワードが入力されている場合：**
   - `find` ツールでログインボタンを検索する（例：`find "続ける submit button"`、`find "Log in button"`）
   - `ref` でボタンをクリックしてログインを完了する
4. **Keeper Fill が自動入力していない、または複数アカウントが候補に出ている場合：**
   - ユーザーに「{サービス名} に {メールアドレス} でログインが必要です。Keeper Fill でログイン情報を入力してください。完了したら OK と教えてください。」と伝える
   - ユーザーの「OK」を待つ

### Pattern B: Google SSO（ページ遷移型）

TimeRex など、Google ログインが同じタブ内でページ遷移するサービス。

1. ログインページで「Google でログイン」ボタンを `find` ツールで検索する（例：`find "Googleでログイン link"`）
2. `ref` でクリックする
3. Google アカウント選択画面が表示されたら、スキルに登録されたアカウントを `find` ツールで検索する（例：`find "yamada.k@edocode.co.jp account option"`）
4. `ref` でクリックしてログインを完了する
5. サービスのページに自動リダイレクトされることを確認する

### Pattern C: Google SSO（ポップアップ型）

Figma など、Google ログインが別ウィンドウのポップアップで開くサービス。

1. Claude からはポップアップウィンドウを操作できないため、ユーザーに依頼する
2. ユーザーに「{サービス名} に Google アカウント（{メールアドレス}）でログインしてください。完了したら OK と教えてください。」と伝える
3. ユーザーの「OK」を待つ

### Pattern D: Microsoft SSO（ユーザー操作）

Microsoft 365 Admin Center など、Microsoft アカウントでログインするサービス。

1. Microsoft SSO（`login.microsoftonline.com`）はポップアップやリダイレクトの挙動が環境依存であり、Claude からの操作が不安定なため、ユーザーに依頼する
2. ユーザーに「{サービス名} に Microsoft アカウント（{メールアドレス}）でログインしてください。完了したら OK と教えてください。」と伝える
3. ユーザーの「OK」を待つ

### Pattern E: Google SSO（ページ遷移型）+ MFA

OpenAI など、Google SSO 後にサービス独自の多要素認証（MFA）が求められるサービス。

1. Pattern B と同様に Google SSO でアカウント選択まで完了する
2. Google SSO 完了後、サービス独自の MFA ページ（`auth.openai.com/mfa-challenge` 等）にリダイレクトされる
3. MFA（ワンタイムパスワード入力）は Claude では操作できないため、ユーザーに依頼する：
   - ユーザーに「{サービス名} の多要素認証（MFA）が求められています。認証アプリのワンタイムコードを入力して『続行』をクリックしてください。完了したら OK と教えてください。」と伝える
4. ユーザーの「OK」を待つ

**重要：ログインフォームにパスワードを直接入力してはならない。Keeper Fill がフォームを自動入力する。**

## Common: Month Calculation（共通：対象月の計算）

経理処理は月初に前月分を処理する。現在日付から前月を自動計算する。

- 現在が 2026年2月 → 対象月は 2026年1月
- 処理開始時にユーザーに「対象月: {YYYY年M月} で処理します。変更する場合は教えてください。」と確認する

## Common: Workspace Setup（共通：作業フォルダ）

対象月の確定後、プロジェクトディレクトリ内に作業フォルダを作成する。

```
{プロジェクトディレクトリ}/
└── {YYYY-MM}/              ← 作業フォルダ（対象月）
    ├── downloads/          ← ダウンロードファイル保管
    └── audit.log           ← 監査ログ
```

- フォルダ名は対象月の `YYYY-MM` 形式（例：`2026-01`）
- `downloads/` サブフォルダを作成する
- 作業中にダウンロードした PDF はすべて `~/Downloads` から `downloads/` フォルダに移動して保管する
- ファイル名は `{YYYY-MM}_{サービス名}.pdf` の形式にリネームする（例：`2026-01_fondesk.pdf`）

## Common: Audit Log（共通：監査ログ）

作業の監査・追跡を目的として、作業フォルダ内に `audit.log` を作成する。後から「何をどう判断し、何をダウンロードしたか」を辿れることが目的。

### ログフォーマット

```
[YYYY-MM-DD HH:MM:SS] CATEGORY: message
```

CATEGORY は以下のいずれか：
- `INIT` — 作業開始・パラメータ確定
- `SELECT` — データ選択の判断根拠
- `DOWNLOAD` — ファイルダウンロード実行
- `COMPLETE` — Phase/作業完了

### 記録タイミングと内容

**INIT（Preflight Complete 時）:**
```
[timestamp] INIT: target_month=2026-01, workspace={作業フォルダパス}
```

**SELECT（各サービスで対象月の請求書を特定した時）:**
```
[timestamp] SELECT: {サービス名} invoice found, date={日付}, amount={金額}
```

**DOWNLOAD（PDF ダウンロード完了時）:**
```
[timestamp] DOWNLOAD: {サービス名} file={ファイル名}, saved_as={保存先パス}
```

**COMPLETE（全サービス完了時）:**
```
[timestamp] COMPLETE: all services finished, files_downloaded={件数}
```

---

## Service 1: fondesk

### Login

- **Login ID**: `service-account@edocode.co.jp`
- **Login Method**: Pattern A（メール + パスワード / Keeper Fill）
- **Login URL**: `auth.fondesk.jp/u/login`（`fondesk.jp/app/*` にアクセスすると自動リダイレクト）
- **Login Page判定**: ページタイトルに「ログイン」を含む、または URL が `auth.fondesk.jp` を含む場合はログインが必要
- **Login手順**:
  1. メールアドレス欄に `service-account@edocode.co.jp` が入力されているか確認
  2. Keeper Fill がパスワードを自動入力しているか確認
  3. `find "続ける submit button"` でログインボタンを取得
  4. `ref` でクリック

### Download手順

1. Claude in Chrome で以下の URL を開く：
   ```
   https://www.fondesk.jp/app/account/invoices
   ```
2. ログイン判定を行い、必要であればログイン処理を実施する
3. 請求書一覧が表示される。各行は「{YYYY}年{M}月10日 発行分 #{番号}」の形式。対象月分を特定する
   - 例：対象月が 2026年1月 → 「2026年1月10日 発行分」を探す
4. 対象月の行の右端にある「請求書」リンクをクリックする
   - `find "請求書 link near {対象年}年{対象月}月10日"` で要素を取得
   - `ref` でクリック
5. **新しいタブ**で Stripe の請求書ページ（`invoice.stripe.com`）が開く
6. 新しいタブに切り替える（`tabs_context_mcp` でタブ一覧を取得し、Stripe のタブ ID を確認）
7. 「請求書をダウンロード」ボタンをクリック：
   - `find "請求書をダウンロード button"` で要素を取得
   - `ref` でクリック
8. `~/Downloads` に `Invoice-{番号}.pdf` がダウンロードされる
9. ダウンロードしたファイルを作業フォルダの `downloads/` に移動しリネーム

### Download対象

- **請求書**をダウンロードする（領収書ではない）

---

## Service 2: Figma

### Login

- **Login ID**: `yamada.k@edocode.co.jp`
- **Login Method**: Pattern C（Google SSO / ポップアップ型）
- **Login URL**: `figma.com/login`（請求書ページからリダイレクト）
- **Login Page判定**: ページタイトルに「ログイン」を含む、または URL が `figma.com/login` を含む場合はログインが必要
- **Login手順**:
  1. ユーザーに「Figma に Google アカウント（`yamada.k@edocode.co.jp`）でログインしてください。完了したら OK と教えてください。」と伝える
  2. ユーザーの「OK」を待つ

### Download手順

1. Claude in Chrome で以下の URL を開く：
   ```
   https://www.figma.com/files/team/690131495837577799/team-admin-console/billing/invoices
   ```
2. ログイン判定を行い、必要であればログイン処理を実施する
3. Billing > Invoices ページが表示される。各行は「{Month} {DD}, {YYYY}」形式の Due date で表示される。毎月12日に Monthly invoice が発行される
4. 対象月の行をクリックする（行のどこでもクリック可能）
   - 例：対象月が 2026年1月 → 「January 12, 2026」の行をクリック
5. 右側に請求書詳細パネルが開く
6. パネルの下部までスクロールする（右側パネル内をスクロール）
7. 「Download PDF」リンクをクリック：
   - `find "Download PDF link"` で要素を取得
   - `ref` でクリック
8. **新しいタブ**が一瞬開いてすぐ閉じ、PDF がダウンロードされる
9. `~/Downloads` に `Invoice-{番号}.pdf` がダウンロードされる
10. ダウンロードしたファイルを作業フォルダの `downloads/` に移動しリネーム

---

## Service 3: TimeRex

### Login

- **Login ID**: `yamada.k@edocode.co.jp`
- **Login Method**: Pattern B（Google SSO / ページ遷移型）
- **Login URL**: `timerex.net/auth/login`（`timerex.net/user/*` にアクセスすると自動リダイレクト）
- **Login Page判定**: ページタイトルに「ログイン」を含む、または URL が `timerex.net/auth/login` を含む場合はログインが必要
- **Login手順**:
  1. `find "Googleでログイン link"` でボタンを取得
  2. `ref` でクリック → Google アカウント選択画面にページ遷移
  3. `find "yamada.k@edocode.co.jp account option"` でアカウントを取得
  4. `ref` でクリック → TimeRex にリダイレクトされログイン完了

### Download手順

1. Claude in Chrome で以下の URL を開く：
   ```
   https://timerex.net/user/team/1/billing
   ```
2. ログイン判定を行い、必要であればログイン処理を実施する
3. 請求履歴が表示される。各行は「決済日時 / オーダー番号 / 金額（税込）/ 領収書・明細」のカラム。**一覧は古い順**なので、ページ最下部までスクロールが必要
4. 対象月分の行を特定する（決済日時の月で判断。毎月14日頃に決済）
   - 例：対象月が 2026年1月 → 「2026/1/14」の行を探す
5. 「表示する」リンクをクリック：
   - `find "表示する link near {YYYY/M/14}"` で要素を取得
   - `ref` でクリック
6. **新しいタブ**で「利用明細書・領収書」HTML ページが開く
7. このページは PDF ダウンロードボタンがないため、ユーザーに手動保存を依頼する：
   - ユーザーに「TimeRex の領収書ページが開きました。ブラウザで Cmd+P → 『PDF として保存』でダウンロードしてください。ファイル名は `TimeRex_{オーダー番号}.pdf` にしてください。完了したら OK と教えてください。」と伝える
   - ユーザーの「OK」を待つ
8. ダウンロードしたファイルを作業フォルダの `downloads/` に移動しリネーム

---

## Service 4: Bitly

### Login

- **Login ID**: `service-account@edocode.co.jp`
- **Login Method**: Pattern A（メール + パスワード / Keeper Fill）
- **Login URL**: `app.bitly.com`（ログインページにリダイレクト）
- **Login Page判定**: URL が `bitly.com/a/sign_in` または `bitly.com/a/oauth` を含む場合はログインが必要
- **Login手順**:
  1. メールアドレス欄に `service-account@edocode.co.jp` が入力されているか確認
  2. Keeper Fill がパスワードを自動入力しているか確認
  3. `find "Log in submit button"` でログインボタンを取得
  4. `ref` でクリック

### Download手順

1. Claude in Chrome で以下の URL を開く：
   ```
   https://app.bitly.com/settings/organization/Okbk5aX3uEc/billing
   ```
2. ログイン判定を行い、必要であればログイン処理を実施する
3. Billing and usage ページが表示される。下にスクロールすると請求一覧がある
4. 請求一覧は新しい順で表示される。各行は「{MM/DD/YYYY} / Basic Subscripti... / ${金額} USD / ...」の形式
5. 対象月の行の「...」メニューボタンをクリックする
   - 対象月の日付は毎月20日（例：01/20/2026）
   - `...` ボタンは行の右端にある
6. ドロップダウンメニューが表示されたら「Download PDF」をクリック：
   - `find "Download PDF menu option"` で要素を取得
   - `ref` でクリック
7. `~/Downloads` に `bitly-invoice-{YYYY-MM-DD}.pdf` がダウンロードされる
8. ダウンロードしたファイルを作業フォルダの `downloads/` に移動しリネーム

---

## Service 5: Krisp

### Login

- **Login ID**: `yamada.k@edocode.co.jp`
- **Login Method**: Pattern B（Google SSO / ページ遷移型）
- **Login URL**: `https://account.krisp.ai/login`
- **Login Page判定**: URL が `account.krisp.ai/login` を含む場合はログインが必要
- **Login手順**:
  1. `find "Continue with Google button"` でボタンを取得
  2. `ref` でクリック → Google アカウント選択画面にページ遷移
  3. `find "yamada.k@edocode.co.jp account option"` でアカウントを取得
  4. `ref` でクリック → Krisp にリダイレクトされログイン完了

### Logout手順（参考）

1. 左サイドバー最下部の「Kyo Yamada」ユーザーメニューをクリック
   - `find "Kyo Yamada user menu or profile button"` で取得 → `ref` でクリック
2. メニューから「サインアウト」をクリック
   - `find "サインアウト link"` で取得 → `ref` でクリック
3. ログインページにリダイレクトされる

### Download手順

1. Claude in Chrome で以下の URL を開く：
   ```
   https://account.krisp.ai/billing-team/details
   ```
2. ログイン判定を行い、必要であればログイン処理を実施する
3. Billing ページが表示される。「Invoice history」セクションに請求履歴がある
4. 対象月の「請求書を見る」リンクをクリック：
   - `find "請求書を見る link for {対象月の英語名} {対象日} {対象年}"` で要素を取得（例：`find "請求書を見る link for Jan 25 2026"`）
   - `ref` でクリック
5. **新しいタブ**で Stripe の領収書ページ（`pay.stripe.com`）が開く
6. 新しいタブに切り替える（`tabs_context_mcp` でタブ一覧を取得し、Stripe のタブ ID を確認）
7. 「Download invoice」リンクをクリック：
   - `find "Download invoice link"` で要素を取得
   - `ref` でクリック
8. `~/Downloads` に `Invoice-{番号}.pdf` がダウンロードされる
9. ダウンロードしたファイルを作業フォルダの `downloads/` に移動しリネーム

### Download対象

- **請求書（Invoice）** をダウンロードする

---

## Service 6: n8n

### Login

- **Login ID**: `yamada.k@edocode.co.jp`
- **Login Method**: 2段階ログイン（Cloud Admin Panel → Instance）
- **Login URL（Stage 1）**: `https://app.n8n.cloud/login`（Cloud Admin Panel）
- **Login URL（Stage 2）**: `https://edocode.app.n8n.cloud/signin`（Instance）
- **Login Page判定**: URL が `app.n8n.cloud/login` を含む場合はログインが必要

**重要**: n8n は Cloud Admin Panel（`app.n8n.cloud`）と Instance（`edocode.app.n8n.cloud`）の2つのポータルがあり、認証セッションが別々。Billing は Cloud Admin Panel 側にある。

- **Login手順（Stage 1 - Cloud Admin Panel）**:
  1. `https://app.n8n.cloud/login` が表示される
  2. アカウント名入力欄に `edocode` を入力する。**重要：フィールドに値がプリフィルされている場合でも、JSフレームワーク（Nuxt.js）が値を認識しないことがある。以下の手順で確実に入力する：**
     - `find "account name input field"` で入力欄を取得
     - `ref` でトリプルクリック（`triple_click`）して既存のテキストを全選択
     - `type` アクションで `edocode` を入力
     - `key` アクションで `Return` を押す（Enter で Submit）
  3. Instance のログインページ（`edocode.app.n8n.cloud/signin`）にリダイレクトされる

- **Login手順（Stage 2 - Instance）**:
  1. メールアドレス欄とパスワード欄が表示される
  2. Keeper Fill がパスワードを自動入力しているか確認する
  3. **正しい ID と Keeper Fill のパスワードが入力されている場合：**
     - `find "Sign in button"` でログインボタンを取得
     - `ref` でクリック
  4. **Keeper Fill が自動入力していない場合：**
     - ユーザーに「n8n に `yamada.k@edocode.co.jp` でログインが必要です。Keeper Fill でログイン情報を入力してください。完了したら OK と教えてください。」と伝える
  5. ログイン完了後、Instance のダッシュボードにリダイレクトされる
  6. **Billing ページに移動するため** `https://app.n8n.cloud/manage/billing` にナビゲートする。Instance にログイン済みであれば Cloud Admin Panel も自動認証される

### Logout手順（参考）

n8n は Cloud Admin Panel と Instance で別々にログアウトが必要：

**Cloud Admin Panel のログアウト:**
1. 右上の「Sign out」ボタンをクリック

**Instance のログアウト:**
1. 左下の「Kyo Yamada」ユーザーメニューボタンをクリック
   - `find "Kyo Yamada user menu button"` で取得 → `ref` でクリック
2. メニューから「Sign out」をクリック
   - `find "Sign out menu item"` で取得 → `ref` でクリック

### Download手順

1. Claude in Chrome で以下の URL を開く：
   ```
   https://app.n8n.cloud/manage/billing
   ```
2. ログイン判定を行い、必要であればログイン処理を実施する
3. Cloud Admin Panel の Billing ページが表示される。「Transaction history」セクションに取引履歴がある
4. 対象月の行をクリックする（行のどこでもクリック可能）
   - 行が展開され「Open」ボタンが表示される
5. 「Open」ボタンをクリック：
   - `find "Open button"` で要素を取得
   - `ref` でクリック
6. **新しいタブ**で Paddle.com の領収書ページ（`my.paddle.com/invoice/...`）が開く
7. **Paddle の領収書ページは HTML 表示であり、PDF ダウンロードボタンがない**ため、ユーザーに手動保存を依頼する：
   - ユーザーに「n8n の Paddle.com 領収書ページが開きました。ブラウザで Cmd+P → 『PDF として保存』でダウンロードしてください。ファイル名は `n8n_receipt_{YYYY-MM}.pdf` にしてください。完了したら OK と教えてください。」と伝える
   - ユーザーの「OK」を待つ
8. ダウンロードしたファイルを作業フォルダの `downloads/` に移動しリネーム

### Download対象

- **領収書（Receipt）** を PDF 保存する（Paddle.com は HTML のみ提供）

---

## Service 7: Microsoft

### Login

- **Login ID**: `yamada.k@edocode.co.jp`
- **Login Method**: Pattern D（Microsoft SSO / ユーザー操作）
- **Login URL**: Microsoft 365 Admin Center（`admin.cloud.microsoft`）は通常 Microsoft SSO で自動ログインされる
- **Login Page判定**: URL が `login.microsoftonline.com` を含む場合はログインが必要
- **Login手順**:
  1. ユーザーに「Microsoft 365 Admin Center に Microsoft アカウント（`yamada.k@edocode.co.jp`）でログインしてください。完了したら OK と教えてください。」と伝える
  2. ユーザーの「OK」を待つ

### Download手順

1. Claude in Chrome で以下の URL を開く：
   ```
   https://admin.cloud.microsoft/#/billoverview/invoice-list
   ```
2. ログイン判定を行い、必要であればログイン処理を実施する
3. 「請求と支払い」ページが表示される。請求書の一覧が表示される
4. 対象月の請求書 ID リンクをクリックする：
   - `find "{請求書ID} invoice link"` で要素を取得（例：`find "G139153324 invoice link"`）
   - `ref` でクリック
5. 請求書の詳細ページが開く
6. 「ダウンロード」ドロップダウンボタンをクリック：
   - `find "ダウンロード dropdown button"` で要素を取得
   - `ref` でクリック
7. ドロップダウンメニューから「請求書をダウンロード」をクリック：
   - `find "請求書をダウンロード menu option"` で要素を取得
   - `ref` でクリック
8. `~/Downloads` に `{請求書ID}_{ハッシュ}.pdf` がダウンロードされる
9. ダウンロードしたファイルを作業フォルダの `downloads/` に移動しリネーム

### 請求書一覧ページへの到達方法（代替）

Billing URL で直接請求書一覧が表示されない場合：
1. `https://admin.cloud.microsoft/` を開く
2. 左サイドバーの「課金情報」を展開
3. `find "請求と支払い menu item"` で要素を取得 → `ref` でクリック

### Download対象

- **請求書** をダウンロードする

---

## Service 8: OpenAI (ChatGPT)

### Login

- **Login ID**: `yamada.k@edocode.co.jp`
- **Login Method**: Pattern E（Google SSO / ページ遷移型 + MFA）
- **Login URL**: `https://chatgpt.com/` → 右上「ログイン」ボタン
- **Login Page判定**: ページ右上に「ログイン」ボタンが表示されている場合はログインが必要
- **Login手順**:
  1. `chatgpt.com` のトップページで右上の「ログイン」ボタンをクリック：
     - `find "ログイン button"` で取得 → `ref` でクリック
  2. 「ログインまたはサインアップ」モーダルが表示される
  3. 「Google で続行」ボタンをクリック：
     - `find "Google で続行 button"` で取得 → `ref` でクリック
  4. Google アカウント選択画面にページ遷移する
  5. `find "yamada.k@edocode.co.jp account option"` でアカウントを取得
  6. `ref` でクリック
  7. **MFA ページ**（`auth.openai.com/mfa-challenge`）にリダイレクトされる
  8. ユーザーに「OpenAI の多要素認証（MFA）が求められています。認証アプリのワンタイムコードを入力して『続行』をクリックしてください。完了したら OK と教えてください。」と伝える
  9. ユーザーの「OK」を待つ

### Logout手順（参考）

1. ChatGPT メイン画面の左下「Kyo Yamada / EDOCODE Inc.」をクリック
   - `find "Kyo Yamada user menu or profile button"` で取得 → `ref` でクリック
2. メニューから「ログアウト」をクリック
   - `find "ログアウト menu item"` で取得 → `ref` でクリック
3. 確認ダイアログ「ログアウトしますか？」が表示される
4. 「ログアウト」ボタンをクリック：
   - `find "ログアウト confirm button in dialog"` で取得 → `ref` でクリック

### Download手順

1. Claude in Chrome で以下の URL を開く：
   ```
   https://chatgpt.com/admin/billing?tab=invoices
   ```
2. ログイン判定を行い、必要であればログイン処理を実施する
3. EDOCODE Inc. の管理者 > 請求 > 請求書タブが表示される。請求書一覧が表示される
4. 対象月の請求書の行にある外部リンクアイコン（緑色の矢印アイコン）をクリック：
   - `find "external link icon for {対象年}年{対象月}月{日}日 invoice"` で要素を取得（例：`find "external link icon for 2026年1月31日 invoice"`）
   - **注意**: 行自体をクリックすると次ページに遷移してしまうので、必ず外部リンクアイコン（`<a>` タグ）を `find` で正確に検索すること
   - `ref` でクリック
5. **新しいタブ**で Stripe の請求書ページ（`invoice.stripe.com`）が開く
6. 新しいタブに切り替える（`tabs_context_mcp` でタブ一覧を取得し、Stripe のタブ ID を確認）
7. 「請求書をダウンロード」ボタンをクリック：
   - `find "請求書をダウンロード button"` で要素を取得
   - `ref` でクリック
8. `~/Downloads` に `Invoice-{番号}.pdf` がダウンロードされる
9. ダウンロードしたファイルを作業フォルダの `downloads/` に移動しリネーム

### Download対象

- **請求書** をダウンロードする（領収書ではない）

---

## Completion（完了処理）

全サービス完了後、ユーザーに以下のサマリーを表示する：

```
## EDOCODE 月次証憑ファイル回収 完了サマリー

対象月: {YYYY年M月}

### 回収ファイル一覧
| # | サービス | ファイル名 | ステータス |
|---|---------|----------|----------|
| 1 | fondesk | {ファイル名} | 完了 |
| 2 | Figma | {ファイル名} | 完了 |
| 3 | TimeRex | {ファイル名} | 完了 |
| 4 | Bitly | {ファイル名} | 完了 |
| ... | ... | ... | ... |

保存先: {作業フォルダ}/downloads/
```

## Security Notes（セキュリティに関する注意事項）

- **パスワードやログイン認証情報を会話コンテキストに出力してはならない**
- ログインは必ず Keeper Fill 経由で行い、Claude がフォームに直接入力しない
- Google SSO のアカウント選択のみ Claude が操作する（パスワード入力は発生しない）
- 各サービスの URL やデータ内容は業務上必要な範囲でのみ表示する
- PDF に含まれる個人情報・機密情報は必要最小限の抽出にとどめる

## Click Operation Guidelines（クリック操作の指針）

Web ページのクリック操作は失敗しやすいため、以下のルールを厳守する：

1. **座標クリックではなく、必ず `find` → `ref` クリックを使う**
   - まず `find` ツールで対象要素を自然言語で検索する
   - 取得した `ref_xxx` を使って `ref` パラメータでクリックする
2. **`find` の検索キーワードは具体的に書く**
   - 良い例：`find "Download PDF menu option"`, `find "続ける submit button"`, `find "表示する link near 2026/1/14"`
   - 悪い例：`find "button"`, `find "link"`
3. **新しいタブが開くケースに注意する**
   - fondesk の Stripe ページ、TimeRex の領収書ページなど、リンクが新タブで開くケースがある
   - `tabs_context_mcp` でタブ一覧を更新し、正しいタブ ID で操作する
4. **スクロールが必要なケースに注意する**
   - Figma の詳細パネル内スクロール、TimeRex の古い順リスト最下部など
