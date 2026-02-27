---
name: zed-editor
description: Zed Editorの特徴、ACP（Agent Client Protocol）の仕様、AI統合、パフォーマンス特性、設定方法について包括的な知識を提供します。Zed Editorに関する質問、ACP対応エージェントの統合、設定のカスタマイズが必要な場合に使用してください。
allowed-tools: Read, Write, Edit, Bash, WebFetch, WebSearch
---

# Zed Editor Skill

このスキルは、Zed Editorとその革新的なAgent Client Protocol (ACP)について包括的な知識を提供します。

## Zed Editorとは

Zed は、**Rustで完全に書き直された次世代のコードエディタ**です。以下の特徴を持ちます：

### コア特性
- **高速パフォーマンス**: 複数のCPUコアとGPUを効率的に活用
  - 起動時間が極めて高速
  - UIインタラクションの遅延がほぼゼロ
  - タイピング遅延が最小限
- **Rust製**: メモリ安全性とパフォーマンスを両立
- **オープンソース**: Apache License 2.0の下で公開

### 主要機能

#### 1. AI統合
- **Agentic Editing**: エージェントに作業を委譲し、進捗をリアルタイムで追跡
- **Edit Prediction**: 次の入力を予測するオープンソース言語モデル
- **Inline Assistant**: 選択したコードをLLMに送信して変換
- **Text Threads**: プレーンテキストインターフェースでLLMと対話

#### 2. コラボレーション機能
- チームメンバーとのチャット
- 共同でのノート作成
- スクリーンとプロジェクトの共有
- すべてデフォルトで利用可能

#### 3. 開発者機能
- Language Server Protocol対応
- アウトラインビュー
- リモート開発
- マルチバッファ編集
- Vimバインディング
- デバッガ
- Git統合

## Agent Client Protocol (ACP)

### ACPの概念

ACPは、**コードエディタとAIコーディングエージェント間の通信を標準化するプロトコル**です。Language Server Protocol (LSP)がIDEから言語インテリジェンスを切り離したように、ACPはエディタからAIエージェントを切り離します。

```
┌─────────────┐         ACP         ┌──────────────┐
│   Editor    │◄──────────────────►│    Agent     │
│  (Client)   │   JSON-RPC/stdio   │   (Server)   │
└─────────────┘                     └──────────────┘
```

### ACPの設計原理

1. **エディタ非依存**: どのエディタでも同じエージェントが動作
2. **標準化された通信**: JSON-RPCを使用したstdio通信
3. **MCPとの統合**: Model Context Protocol (MCP)の仕様を可能な限り再利用
4. **セキュリティ**: エディタがファイル、ターミナル、ツールへのアクセスを仲介

### 技術仕様

#### 通信方式
- **プロトコル**: JSON-RPC over stdio
- **起動**: エージェントプロセスはコードエディタによって起動
- **スキーマ**: `schema/schema.json`で標準化

#### 公式SDK

| 言語 | パッケージ名 | 配布場所 |
|------|-------------|---------|
| TypeScript | `@agentclientprotocol/sdk` | NPM |
| Rust | `agent-client-protocol` | crates.io |
| Kotlin | `acp-kotlin` | JVM（他ターゲット開発中） |

### エコシステム

#### サポートされているエディタ
- **Zed**: ネイティブサポート
- **Neovim**: CodeCompanion、avante.nvim
- **Emacs**: agent-shell plugin
- **JetBrains IDEs**: 開発中（公式連携）
- **Eclipse**: プロトタイプ実装
- **marimo**: Python notebook環境

#### サポートされているエージェント
1. **Claude Code**: Anthropic製（パブリックベータ）
2. **Gemini CLI**: Google製（デフォルト）
3. **Codex CLI**: OpenAI製
4. カスタムACP対応エージェント

## Zed EditorでのACP使用方法

### デフォルトエージェントの使用

#### Gemini CLI
```json
{
  "bindings": {
    "cmd-alt-g": ["agent::NewExternalAgentThread", { "agent": "gemini" }]
  }
}
```

認証方法：
- Googleログイン
- Gemini APIキー
- Vertex AI

#### Claude Code

1. 初回起動時に自動インストール
2. `/login`コマンドで認証（APIキーまたはClaude Pro）
3. カスタム実行ファイルの設定（オプション）：

```json
{
  "agent_servers": {
    "claude": {
      "env": {
        "CLAUDE_CODE_EXECUTABLE": "/path/to/executable"
      }
    }
  }
}
```

#### Codex CLI

認証方法（3種類）：
- ChatGPTアカウント
- `CODEX_API_KEY`環境変数
- `OPENAI_API_KEY`環境変数

### カスタムエージェントの追加

任意のACP対応エージェントを追加できます：

```json
{
  "agent_servers": {
    "Custom Agent": {
      "command": "node",
      "args": ["~/projects/agent/index.js", "--acp"],
      "env": {
        "CUSTOM_ENV_VAR": "value"
      }
    }
  }
}
```

### デバッグ

コマンドパレット → "dev: open acp logs" でエージェント間の通信メッセージを確認できます。

## Claude Code in Zed

### 統合の特徴

Zedは、Claude Codeを**ネイティブに統合**しています：

1. **リアルタイムトラッキング**: 複数ファイルにわたる変更を構文ハイライトと共に追跡
2. **細かいコードレビュー**: マルチバッファ形式で個別の変更を承認/却下
3. **タスクリストの可視化**: サイドバーに永続的に表示
4. **カスタムワークフロー**: スラッシュコマンド対応

### 技術アーキテクチャ

Zedは**アダプター方式**を採用：
- Claude Code SDKの操作をACPのJSON RPC形式に変換
- Claude Codeは独立して動作
- ZedがUIレイヤーを提供

アダプターはApacheライセンスで[オープンソース化](https://github.com/zed-industries/claude-code-adapter)されており、他のACP対応エディタでも利用可能です。

### 現在の制限事項

ベータ版では以下の機能が未サポート：
- Plan mode
- 組み込みスラッシュコマンド

これらはAnthropicによるSDK拡張待ちです。

## MCP (Model Context Protocol) との関係

### MCPサポート状況

| エージェント | MCPサポート |
|-------------|-----------|
| Claude Code | ✅ 対応 |
| Codex CLI | ✅ 対応 |
| Gemini CLI | ❌ 未対応 |

### MCPとACPの違い

- **MCP**: モデル（AI）がコンテキスト（データ）にアクセスするためのプロトコル
- **ACP**: エディタとエージェント間の通信プロトコル
- **関係**: ACPはMCPの仕様を可能な限り再利用しつつ、独自の型も追加

## パートナーシップとエコシステム

### JetBrains連携

JetBrainsとZedは、ACP駆動の体験をJetBrains IDEでネイティブに実装する取り組みを進めています：
- JetBrains IDEでのネイティブな統合
- エコシステム全体での互換性維持
- オープンで移植可能な実装

### コミュニティの広がり

ACPは、複数のエディタコミュニティで採用されています：
- Neovim: 2つのプラグインで実装
- Emacs: agent-shellプラグイン
- marimo: Pythonノートブック環境
- Eclipse: プロトタイプ実装

## 設定ファイルの場所

### Zed設定ファイル

| 設定タイプ | ファイルパス |
|----------|------------|
| ユーザー設定 | `~/.config/zed/settings.json` |
| キーバインド | `~/.config/zed/keymap.json` |
| エージェント設定 | settings.jsonの`agent_servers`セクション |

### 設定例

```json
{
  // 基本設定
  "theme": "One Dark",
  "vim_mode": true,
  
  // エージェント設定
  "agent_servers": {
    "claude": {
      "env": {}
    },
    "gemini": {
      "env": {}
    }
  },
  
  // キーバインド
  "bindings": {
    "cmd-alt-c": ["agent::NewExternalAgentThread", { "agent": "claude" }],
    "cmd-alt-g": ["agent::NewExternalAgentThread", { "agent": "gemini" }]
  }
}
```

## ベストプラクティス

### エージェント選択の指針

1. **Claude Code**: 複雑なコード編集、リファクタリング、アーキテクチャ設計
2. **Gemini CLI**: Google Cloud連携、Vertex AI利用
3. **Codex CLI**: OpenAI APIとの統合

### パフォーマンス最適化

- 不要なエージェントは無効化
- デバッグログは必要時のみ有効化
- 大規模プロジェクトでは言語サーバーの設定を調整

### セキュリティ考慮事項

1. **信頼できるソースのみ**: エージェントは信頼できるソースからのみインストール
2. **環境変数の管理**: APIキーは環境変数で管理
3. **アクセス制御**: エージェントのファイルアクセスを適切に制限

## トラブルシューティング

### エージェントが起動しない

1. ACPログを確認: `dev: open acp logs`
2. 実行ファイルのパスを確認
3. 環境変数が正しく設定されているか確認

### 認証エラー

1. APIキーの有効期限を確認
2. 環境変数の設定を確認
3. エージェントの再認証を試行

### パフォーマンス問題

1. 不要なエージェントを無効化
2. Zedのバージョンを最新に更新
3. システムリソースの使用状況を確認

## 参考リンク

### 公式ドキュメント
- [Zed Editor公式サイト](https://zed.dev/)
- [ACP GitHub Repository](https://github.com/zed-industries/agent-client-protocol)
- [Zed AI Documentation](https://zed.dev/docs/ai/external-agents)

### ブログ記事
- [Claude Code via ACP](https://zed.dev/blog/claude-code-via-acp)
- [ACP Progress Report](https://zed.dev/blog/acp-progress-report)
- [JetBrains × Zed Partnership](https://blog.jetbrains.com/ai/2025/10/jetbrains-zed-open-interoperability-for-ai-coding-agents-in-your-ide/)

### コミュニティリソース
- Zed Discord
- GitHub Discussions
- ACP Ecosystem Projects

## まとめ

Zed Editorは、高速なRust製エディタとして、Agent Client Protocolを通じて真にオープンなAIエージェント統合を実現しています。ACPは、LSPが言語ツールに対して行ったことをAIエージェントに対して実現し、開発者に選択肢と柔軟性を提供します。

このスキルを使用することで、Zed Editorの設定、ACPエージェントの統合、トラブルシューティング、最適化を効率的に行うことができます。
