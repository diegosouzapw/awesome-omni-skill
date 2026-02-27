---
name: agent-ide
description: TS/JS 程式碼智能重構工具。重命名、移動檔案、清理 dead code、修改函式簽章時自動更新所有引用。
---

# Agent IDE

TS/JS 程式碼智能重構工具。

## 功能列表

| 功能 | 用途 |
|------|------|
| [snapshot](references/snapshot.md) | 專案/模組 API 快照 |
| [cycles](references/cycles.md) | 循環依賴檢測 |
| [impact](references/impact.md) | 變更影響分析 |
| [find-references](references/find-references.md) | 符號引用查找 |
| [call-hierarchy](references/call-hierarchy.md) | 呼叫層次追蹤 |
| [rename](references/rename.md) | 重命名符號 + 自動更新引用 |
| [move](references/move.md) | 移動檔案/成員 + 自動更新 import |
| [change-signature](references/change-signature.md) | 修改函式參數 + 更新呼叫點 |
| [deadcode](references/deadcode.md) | 檢測/清理未使用程式碼 |

## 最佳實踐

### 重構標準流程

```text
1. deadcode     → 清理未使用程式碼（清場）
2. snapshot     → 了解專案架構（偵察）
3. cycles       → 檢測循環依賴（診斷）
4. move         → 重組檔案/成員（重構）
5. rename       → 修正命名風格（收尾）
```

### 場景選用指南

| 場景 | 功能 | 說明 |
|------|-------|------|
| 接手新專案 | `deadcode` → `snapshot` | 先清垃圾，再看全貌 |
| 重構前診斷 | `cycles` → `impact` | 找問題點，評估影響範圍 |
| 檔案重組 | `move src/a.ts src/b/` | 自動更新所有 import |
| 抽取函式 | `move src/a.ts:25 src/b.ts` | 移動成員到新位置 |
| 統一命名 | `rename --from userId --to uid` | 全專案一致性 |
| 修改 API | `change-signature` | 參數順序/名稱變更 |
| 追蹤呼叫 | `call-hierarchy` → `find-references` | 理解函式使用情況 |

### 安全操作原則

1. **先 `--dry-run` 再執行**：變更類功能務必預覽
2. **清理優先於重構**：`deadcode` 減少不必要的移動/重命名
3. **小步快跑**：一次只做一種類型的變更，便於回滾
4. **重構後驗證**：`cycles` 確保沒引入循環依賴
