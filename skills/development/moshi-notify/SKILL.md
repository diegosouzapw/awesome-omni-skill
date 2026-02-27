---
name: moshi-notify
description: Send a push notification to Moshi app with custom title and message
argument-hint: "<title>" "<message>"
disable-model-invocation: true
allowed-tools: Bash(curl *)
---

# Moshi Notification Skill

Send a push notification to the Moshi app.

## Usage

```
/moshi-notify "タイトル" "メッセージ"
```

## Examples

```
/moshi-notify "Task Complete" "Build finished successfully. All 42 tests passed."
/moshi-notify "完了" "ビルドが成功しました。\n\n- コンパイル完了\n- テスト通過\n- デプロイ準備完了"
/moshi-notify "エラー" "テストが失敗しました。\n\n詳細はログを確認してください。"
```

## Instructions

Parse the user's input:
- **Title**: Short summary (first argument or first line, max ~50 chars)
- **Message**: Full detailed message (second argument or full text)

If only one argument is provided, use it as both title (truncated) and message (full).

If no arguments provided, ask the user what to notify.

Execute using Bash:

```bash
curl -s -X POST https://api.getmoshi.app/api/webhook \
  -H "Content-Type: application/json" \
  -d '{"token":"YOUR_MOSHI_TOKEN","title":"SHORT_TITLE","message":"FULL_MESSAGE"}'
```

- Escape JSON special characters in title and message
- Title should be concise (under 100 chars)
- Message can be the full detailed text with newlines

After sending, confirm: "通知を送信しました"
