---
name: context-session-end
description: AI behavioral guideline for autonomously detecting work session boundaries and proposing updates to current_focus.md. The AI monitors conversation flow for natural breakpoints and acts without explicit invocation.
---

# Context Session End

作業の自然な区切りをAIが自律的に検知し、current_focus.md への追記を提案する行動規範。

## 基本方針

ユーザーが「セッション終了スキルを実行して」と言う必要はない。AIは会話の流れから作業の区切りを自然に検知し、記録すべき内容があれば自分から提案する。

## 検知パターン

以下の自然な区切りを検知したら自動的に追記を提案する:

- 「ありがとう」「助かった」
- 「一旦ここまで」「今日はこれで」
- まとまった作業（複数ステップ）が一段落したとき

提案しない場合: 短い質問応答（「このSQL正しい？」→「OK」）程度の場合。

## 手順

### 1. AI作業分の要約

セッション中にAIが関与した作業を振り返り、以下を抽出:

- やったこと（コード作成、設計レビュー等）
- 決まったこと（あれば）
- 残っていること

### 2. 追記案の提示

```
📝 current_focus.md に追記しますか？

【最近あったこと】に追加:
  + [AI] CRUD API 5本作成完了
  + [AI] SQLインデックス追加

【次やること】に追加:
  + E2Eテスト作成

（はい / 修正あり / 不要）
```

### 3. Decision Log 提案（該当する場合のみ）

重要な決定があった場合のみ、1行で:

```
💡 「ページネーションをcursor方式に決定」→ Decision Logに記録しますか？
```

### 4. Tensions 更新提案（該当する場合のみ）

以下のいずれかに該当する場合のみ提案する:

新たな未解決課題が発生した場合:
```
⚠️ tensions.md に追記しますか？ → {未解決課題の要約}
```

既存の tensions.md 項目が解決した場合:
```
tensions.md の「{項目名}」はこのセッションで解決しました。削除しますか？
```

### 5. 更新実行

承認後:
- current_focus.md の該当セクションに追記（既存内容は触らない）
- `[AI]` プレフィックスをつける
- 末尾の「更新」日付を更新

## ルール

- ✅ AI作業分のみ追記提案する
- ✅ 既存の人間が書いた行はそのまま残す
- ✅ `[AI]` プレフィックスで区別する
- ✅ 3-5行以内で簡潔に
- ❌ 人間が書いた行を編集・削除しない
- ❌ current_focus.md を全面書き換えしない
- ❌ 承認なしにファイルを更新しない
