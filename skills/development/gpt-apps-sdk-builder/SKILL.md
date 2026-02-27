---
name: gpt-apps-sdk-builder
description: GPT Apps SDKを用いたアプリ開発を設計・実装・検証する
---

# 🧩 GPT Apps SDK Builder

あなたはGPT Apps SDKのアーキテクト兼実装パートナーとして、MCPサーバーとUIを一貫して設計・実装・検証・公開まで導く。

---

## Prerequisites

- Node.js 18+ と npm
- もしくは Python 3.10+ と pip
- 公開URLを用意するトンネルまたはホスティング（例: ngrok）
- ChatGPT の開発者モード有効化
- 環境変数: `PORT`（任意）, `PUBLIC_MCP_URL`（公開時）, `APP_NAME`

---

## Workflow

### Step 1: 要件ブリーフを作成する

アプリの目的、ユーザー体験、必要なツール、データソース、認証要否を短く整理する。

**Input**: ユーザー要望、対象ユーザー、操作シナリオ  
**Output**: templates/app_brief_template.md を埋めた要件ブリーフ  
**If this fails**: 不明点を列挙し、最小構成の仮ブリーフを作成して前進する

### Step 2: ツール仕様とデータモデルを定義する

ツール名、入力スキーマ、出力形式、状態管理を決定する。

**Input**: 要件ブリーフ  
**Output**: templates/tool_spec_template.md を埋めたツール仕様  
**If this fails**: ツール数を削減し、1ツール単位で段階的に定義する

### Step 3: UIコンポーネントを設計する（任意）

UIが必要な場合は、MCP Apps UIブリッジに対応したHTML/JSを設計する。

**Input**: UI要件、ツール仕様  
**Output**: templates/ui_widget_template.html を基にしたUI  
**If this fails**: UIを後回しにしてツールのみのMCPサーバーに切り替える

### Step 4: MCPサーバーを実装する

ツールとUIリソースを登録し、/mcp エンドポイントを提供する。

**Input**: ツール仕様、UIリソース（任意）  
**Output**: templates/node_mcp_server_template.md を基にしたサーバー実装  
**If this fails**: 依存関係、CORS、/mcp ルート、ESM設定を再確認する

### Step 5: ローカル検証を実行する

MCP Inspectorでツール呼び出しとUI連携を確認する。

```bash
npx @modelcontextprotocol/inspector@latest --server-url http://localhost:8787/mcp --transport http
```

**Input**: ローカルMCP URL（例: http://localhost:8787/mcp）  
**Output**: ツールが期待通りの structuredContent を返すこと  
**If this fails**: inspectorのログを確認し、ツール定義と入力スキーマを調整する

### Step 6: 公開URLで接続を確認する

公開URLに /mcp を付与してChatGPTに追加し、実際の会話で動作確認する。

**Input**: PUBLIC_MCP_URL  
**Output**: ChatGPT上でのツール実行とUI表示  
**If this fails**: HTTPS、CORS、/mcp パス、レスポンス形式を再確認する

### Step 7: 反復と提出準備を行う

ツール説明、エラーメッセージ、UI導線を改善し、提出ガイドラインに沿って最終確認する。

**Input**: テスト結果、使用ログ、改善点  
**Output**: resources/launch_checklist.md の完了  
**If this fails**: 直近の変更を差し戻し、最小構成で再確認する

---

## Key Concepts

| Concept | Description |
|---|---|
| MCP | ChatGPTとアプリを接続するプロトコル。/mcp エンドポイントを提供する |
| Tool | モデルが呼び出す機能単位。入力スキーマとレスポンスが重要 |
| Resource | UIやデータを公開する仕組み。UIは resourceUri で関連付ける |
| UI Bridge | postMessageによるJSON-RPC通信でUIとモデルを連携する |
| structuredContent | UIレンダリングや状態共有に使う構造化データ |
| Connector | ChatGPTに追加する接続設定。公開URLと説明が必要 |

---

## Error Handling

| Error | Cause | Fix |
|---|---|---|
| 404 /mcp | ルート設定やパスが不一致 | /mcp ルートとHTTPメソッド対応を確認 |
| CORS error | プリフライト未対応 | OPTIONSを処理しヘッダーを付与 |
| Tool not found | tool名が不一致 | register名と呼び出し名を一致させる |
| UIが表示されない | resourceUri不一致 | UI resourceUri と toolの ui.resourceUri を合わせる |
| structuredContentが空 | レスポンス形式が不一致 | structuredContentに必要なキーを返す |

---

## Examples

### Example 1: UI付きタスク管理アプリ

**User says**: "タスク追加と完了ができるアプリを作りたい。UIも欲しい。"

**Agent does**:
1. app_brief_template.md を埋め、ツールは add_task と complete_task に限定する
2. ui_widget_template.html をベースにUIのイベントとツール呼び出しを接続する
3. node_mcp_server_template.md を基にMCPサーバーを実装し、Inspectorで検証する

### Example 2: UIなしの翻訳ツール

**User says**: "UIなしで翻訳ツールだけを使えるようにしたい。"

**Agent does**:
1. ツール仕様のみ定義し、UI resource を登録しない
2. MCPサーバーに翻訳ツールを登録し、ツール出力を structuredContent で返す
3. 公開URLでChatGPTに接続し、会話内でツールを実行する

---

## References

- https://chatgpt.com/apps
- https://developers.openai.com/apps-sdk/
- https://developers.openai.com/apps-sdk/quickstart
- https://note.com/npaka/n/nce50143f6064
- https://dev.classmethod.jp/articles/apps-in-chatgpt-apps-sdk-chatgpt/
- https://zenn.dev/himara2/articles/ae362b516e9e52
- https://www.youtube.com/watch?v=YdEK6M0dOmM
- https://note.com/npaka/n/nd69e0f856bea
