---
name: project-claude-initializer
description: 為新專案初始化 Claude Code 配置，建立標準化的 .claude 目錄、CLAUDE.md 和 gitignore。當使用者說「初始化 Claude 配置」、「設定專案 Claude」時使用。
---

# 專案 Claude 配置初始化

為新專案建立標準化的 `.claude` 配置目錄，包含專案指引、gitignore 和目錄結構。

## 何時使用此 Skill

- 使用者說「初始化 Claude 配置」、「設定 Claude」
- 使用者說「為這個專案建立 .claude 目錄」
- 使用者說「幫我設定專案的 Claude Code」
- 開始新專案需要建立配置時

## 工作流程

### 步驟 1：確認專案資訊

先檢查當前目錄是否為專案根目錄，並收集專案資訊：

```bash
# 檢查是否在 git 倉庫中
git rev-parse --show-toplevel 2>/dev/null || pwd

# 檢查是否已有 .claude 目錄
[ -d ".claude" ] && echo "已存在" || echo "不存在"
```

如果不在專案根目錄，詢問使用者專案路徑。

使用 AskUserQuestion 收集資訊：

```
問題 1：專案名稱是什麼？
問題 2：主要技術棧？（選項：React/Vue/Node.js/Python/Java/Go/其他）
問題 3：是否需要建立 skills 目錄？（選項：是/否）
```

### 步驟 2：檢查現有配置（重要！）

**必須先檢查是否已有配置，避免覆蓋使用者的設定。**

```bash
# 檢查 .claude 目錄
ls -la .claude/ 2>/dev/null

# 檢查關鍵檔案
[ -f ".claude/CLAUDE.md" ] && echo "CLAUDE.md 已存在"
[ -f ".claude/settings.json" ] && echo "settings.json 已存在"
[ -f ".claude/.gitignore" ] && echo ".gitignore 已存在"
```

根據檢查結果採取不同策略：

#### 情況 1：.claude 目錄不存在
- 直接建立完整配置
- 繼續步驟 3

#### 情況 2：.claude 存在但 CLAUDE.md 不存在
- 保留現有檔案（settings.json, .gitignore 等）
- 只建立缺失的 CLAUDE.md
- 告知使用者：「發現現有配置，只建立缺失的檔案」

#### 情況 3：CLAUDE.md 已存在但內容很少（< 10 行）
- 顯示現有內容
- 詢問使用者：
  ```
  發現現有 CLAUDE.md（內容較少）：
  [顯示前 5 行]

  選項：
  1. 保留現有內容，不做修改
  2. 備份現有檔案為 CLAUDE.md.backup，建立新檔案
  3. 補充現有檔案（保留原有內容，添加缺失部分）
  4. 取消操作
  ```

#### 情況 4：CLAUDE.md 已存在且內容完整（> 10 行）
- 告知使用者：「CLAUDE.md 已存在且內容完整」
- 詢問是否要：
  ```
  選項：
  1. 保留現有配置，不做任何修改（推薦）
  2. 檢視現有配置
  3. 更新特定部分（技術棧、指令等）
  4. 備份並重新建立
  ```

#### 情況 5：有 settings.json 或其他配置
```bash
# 檢查是否有 settings.json
cat .claude/settings.json 2>/dev/null
```

- **絕對不要覆蓋 settings.json**
- 如果存在，保留並跳過
- 告知使用者：「保留現有 settings.json」

#### 情況 6：有自訂技能
```bash
# 檢查 skills 目錄
ls .claude/skills/ 2>/dev/null
```

- **保留所有現有技能**
- 不要刪除或修改
- 告知使用者：「保留現有技能：[列出技能名稱]」

### 步驟 3：智慧掃描專案

根據專案檔案自動填充資訊：

#### Node.js 專案
```bash
# 讀取 package.json
cat package.json 2>/dev/null
```

從 package.json 提取：
- 專案名稱 (`name`)
- 描述 (`description`)
- 依賴套件（識別 React、Vue、Express、Next.js 等）
- scripts（dev、test、build）

#### Python 專案
```bash
# 讀取 pyproject.toml 或 requirements.txt
cat pyproject.toml 2>/dev/null || cat requirements.txt 2>/dev/null
```

識別：FastAPI、Django、Flask 等框架

#### 其他專案
- Go: 讀取 `go.mod`
- Rust: 讀取 `Cargo.toml`
- Java: 讀取 `pom.xml` 或 `build.gradle`

#### 掃描目錄結構
```bash
# 列出主要目錄
find . -maxdepth 2 -type d ! -path '*/\.*' ! -path './node_modules/*' | sort
```

### 步驟 4：建立目錄結構

```bash
mkdir -p .claude/skills
mkdir -p .claude/plans
```

### 步驟 5：建立 .gitignore

使用 Write 工具建立 `.claude/.gitignore`：

```gitignore
# ===== Claude Code 運行時資料 =====
# 除錯與快取
debug/
cache/
stats-cache.json
statsig/
telemetry/
file-history/

# 會話相關
session-env/
shell-snapshots/
todos/
ide/

# 歷史記錄
history.jsonl

# 專案快取
projects/

# ===== 敏感資料 =====
settings.local.json
.credentials.json
*.key
*.pem
*.env.local

# ===== 臨時檔案 =====
*.tmp
*.log
```

### 步驟 6：建立 CLAUDE.md

根據掃描結果建立 `.claude/CLAUDE.md`，使用以下模板並自動填充：

```markdown
# [專案名稱]

## 專案簡介
[從 package.json 或使用者輸入填寫]

## 技術棧
[根據掃描結果自動列出]
- 前端：[自動偵測]
- 後端：[自動偵測]
- 測試：[自動偵測]

## 專案結構
```
[自動生成目錄樹]
```

## 開發規範

### 程式碼風格
[根據技術棧提供建議]

### 測試要求
[根據技術棧提供建議]

### Git 規範
- Commit message 遵循 Conventional Commits
- 分支命名：feature/*, fix/*, chore/*

## 常用指令
```bash
[從 package.json scripts 自動提取]
```

## 特殊注意事項
[預留給使用者填寫]
```

### 步驟 7：技術棧特定建議

根據識別的技術棧，提供相應的規範建議：

#### React/TypeScript 專案
```markdown
## 開發規範

### 程式碼風格
- 使用 TypeScript strict mode
- 元件使用函式式寫法 + hooks
- 元件命名使用 PascalCase
- 檔案命名使用 kebab-case

### 測試要求
- 元件測試覆蓋率 > 80%
- 使用 React Testing Library
- E2E 測試使用 Playwright

### 專案結構
- 元件放在 `src/components/`
- Hooks 放在 `src/hooks/`
- Utils 放在 `src/utils/`
```

#### Python/FastAPI 專案
```markdown
## 開發規範

### 程式碼風格
- 遵循 PEP 8
- 使用 Black 格式化
- 使用 type hints

### 測試要求
- 使用 pytest
- 測試覆蓋率 > 85%
- 包含整合測試

### 專案結構
- API routes 放在 `app/routes/`
- Models 放在 `app/models/`
- Services 放在 `app/services/`
```

### 步驟 8：生成目錄樹

使用適當的方式生成目錄結構：

```bash
# Linux/Mac
tree -L 2 -d --gitignore -I 'node_modules|.git' || find . -maxdepth 2 -type d ! -path '*/\.*' ! -path './node_modules/*'

# 或手動整理
ls -la | grep ^d
```

### 步驟 9：確認並完成

1. 顯示將要建立的檔案清單
2. 展示 CLAUDE.md 預覽
3. 詢問使用者是否確認
4. 建立所有檔案
5. 提示後續步驟

### 步驟 10：提交到版控

提示使用者將配置加入 git：

```bash
git add .claude/
git commit -m "docs: 初始化 Claude Code 專案配置"
```

## 輸出範例

### 成功訊息
```
✓ 已建立 .claude 目錄結構
✓ 已建立 .claude/.gitignore
✓ 已建立 .claude/CLAUDE.md
✓ 已建立 .claude/skills/（空目錄）
✓ 已建立 .claude/plans/（空目錄）

專案配置已完成！

下一步：
1. 檢視 .claude/CLAUDE.md 並根據需要調整
2. 提交到版控：git add .claude/ && git commit -m "docs: 初始化 Claude Code 配置"
3. 開始使用：claude（Claude Code 會自動讀取配置）

團隊共享：
- 團隊成員 clone 專案後會自動獲得此配置
- 個人設定可放在 .claude/settings.local.json（已在 gitignore 中）
```

## 智慧功能

### 自動技術棧偵測

根據檔案特徵自動識別：

| 檔案/內容 | 識別為 |
|-----------|--------|
| package.json + "react" | React |
| package.json + "next" | Next.js |
| package.json + "vue" | Vue.js |
| package.json + "express" | Express |
| requirements.txt + "fastapi" | FastAPI |
| requirements.txt + "django" | Django |
| go.mod | Go |
| Cargo.toml | Rust |
| pom.xml | Java/Maven |

### 自動指令提取

從配置檔案提取常用指令：

**package.json:**
```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "test": "vitest"
  }
}
```

自動生成：
```markdown
## 常用指令
```bash
# 開發
npm run dev

# 建置
npm run build

# 測試
npm test
```
```

### 目錄結構智慧識別

識別常見結構並生成說明：

```
src/
├── components/    # React 元件
├── pages/        # 頁面元件
├── hooks/        # 自訂 hooks
├── utils/        # 工具函式
└── types/        # TypeScript 類型定義
```

## 團隊共享說明

初始化完成後，告知使用者如何與團隊共享：

```markdown
## 團隊使用此配置

### 方式 1：專案配置（推薦）
此配置已加入專案 git，團隊成員 clone 後自動獲得。

### 方式 2：全局配置
如果團隊想共享通用技能：
1. 建立團隊配置倉庫
2. 成員 clone 到 ~/.claude
3. 全局 + 專案配置會自動合併

### 個人設定
如需個人覆蓋（API key 等）：
- 建立 .claude/settings.local.json
- 已在 gitignore 中，不會提交
```

## 特殊情況處理

### 情況 1：已有 .claude 目錄但無 CLAUDE.md
**策略：安全增量**

```bash
# 檢查現有檔案
ls -la .claude/
```

處理方式：
- ✅ 保留所有現有檔案（settings.json, skills/, plans/ 等）
- ✅ 只建立缺失的 CLAUDE.md
- ✅ 如果沒有 .gitignore，建立標準 .gitignore
- ✅ 告知使用者保留了哪些現有檔案

範例輸出：
```
✓ 發現現有配置
✓ 保留 settings.json
✓ 保留 skills/ 目錄（3 個技能）
✓ 建立 CLAUDE.md
✓ 建立 .gitignore
```

### 情況 2：已有完整配置（CLAUDE.md 存在）
**策略：保守，詢問使用者**

```bash
# 讀取現有 CLAUDE.md
wc -l .claude/CLAUDE.md  # 計算行數
head -10 .claude/CLAUDE.md  # 預覽前 10 行
```

根據內容長度：

**如果 > 50 行（內容豐富）：**
```
⚠️  發現現有 CLAUDE.md（內容完整，82 行）

[顯示前 10 行預覽]

建議保留現有配置。選項：
1. 保留，不做任何修改（推薦）✅
2. 檢視完整內容
3. 僅更新技術棧部分
4. 備份後重新建立
```

**如果 10-50 行（基礎內容）：**
```
發現現有 CLAUDE.md（基礎配置，23 行）

選項：
1. 保留現有內容 ✅
2. 補充缺失部分（技術棧、指令等）
3. 備份後重新建立
4. 取消操作
```

**如果 < 10 行（佔位內容）：**
```
發現簡易 CLAUDE.md（5 行）：
---
[顯示內容]
---

建議重新建立完整配置。選項：
1. 備份後重新建立 ✅
2. 保留現有
3. 補充現有內容
```

### 情況 3：有 settings.json（使用者自訂設定）
**策略：絕對保護**

```bash
cat .claude/settings.json
```

處理方式：
- 🔒 **絕對不覆蓋**
- 🔒 **絕對不修改**
- ✅ 顯示內容給使用者
- ✅ 告知：「保留您的自訂設定」

範例輸出：
```
🔒 發現現有設定檔（settings.json）
   包含自訂配置：
   - model: "opus"
   - maxTokens: 150000

✓ 保留所有自訂設定，不做任何修改
```

### 情況 4：有自訂技能（使用者創作）
**策略：完全保護**

```bash
ls .claude/skills/
```

處理方式：
- 🔒 **不刪除任何技能**
- 🔒 **不修改任何技能**
- ✅ 列出現有技能
- ✅ 告知：「保留所有自訂技能」

範例輸出：
```
✓ 發現現有技能：
  - my-custom-skill
  - team-coding-style
  - project-specific-debug

✓ 保留所有技能，不做任何修改
```

### 情況 5：不在 git 倉庫中
**策略：仍然建立，但提醒**

```bash
git rev-parse --is-inside-work-tree 2>/dev/null || echo "不是 git 倉庫"
```

處理方式：
- ✅ 仍然建立配置（功能不受影響）
- ℹ️  提醒：「此專案不在 git 版控中」
- 💡 建議：「考慮初始化 git（可選）」

範例輸出：
```
ℹ️  注意：此專案不在 git 版控中

✓ Claude Code 配置已建立（不受影響）

💡 建議（可選）：
   如果要版控此專案：
   git init
   git add .
   git commit -m "Initial commit"
```

### 情況 6：找不到專案資訊
**策略：使用通用模板 + 詢問**

處理方式：
- ❓ 詢問基本資訊（專案名稱、技術棧）
- ✅ 使用通用模板
- ✅ 標記需要使用者補充的部分

範例輸出：
```
ℹ️  無法自動偵測專案資訊

已建立基礎配置，請手動補充：
- [專案簡介] ← 請填寫
- [技術棧] ← 請填寫
- [常用指令] ← 請填寫

位置：.claude/CLAUDE.md
```

### 情況 7：團隊專案（有多人協作痕跡）
**策略：特別謹慎**

偵測條件：
```bash
# 檢查 git 作者數量
git log --format='%an' | sort -u | wc -l
# > 3 人 → 團隊專案
```

處理方式：
- ⚠️  額外警告：「這是團隊專案」
- 💡 建議：「修改前先與團隊討論」
- ✅ 提供「唯讀檢視」選項

範例輸出：
```
⚠️  偵測到團隊專案（5 位貢獻者）

建議：
1. 檢視現有配置（不做修改）✅
2. 與團隊討論後再修改
3. 只建立個人本地設定（settings.local.json）

是否繼續？
```

## 檢查清單

完成後確認：

- [ ] `.claude/.gitignore` 已建立
- [ ] `.claude/CLAUDE.md` 已建立且內容完整
- [ ] 專案名稱正確
- [ ] 技術棧資訊準確（如果能偵測到）
- [ ] 目錄結構已記錄
- [ ] 常用指令已填寫（如果能提取到）
- [ ] 使用者知道如何提交到 git
- [ ] 使用者知道如何與團隊共享

## 安全原則（🔒 絕對遵守）

### 絕對不可做的事

1. **🔒 絕對不覆蓋 settings.json**
   - 如果存在 → 保留並跳過 → 告知使用者

2. **🔒 絕對不刪除或修改現有技能**
   - skills/ 中的任何檔案 → 只列出，不修改

3. **🔒 絕對不在未詢問下覆蓋 CLAUDE.md**
   - 如果 > 10 行 → 必須詢問 → 預設「保留現有」

4. **🔒 絕對不刪除使用者資料**
   - plans/, conversation-archives/, 任何自訂檔案 → 保留

5. **🔒 絕對不修改有自訂內容的 .gitignore**
   - 檢查是否有自訂規則 → 詢問是否補充（非替換）

### 安全操作流程

```
檢查 → 詢問 → 備份（如需） → 執行 → 確認
```

每一步必須：
- ✅ 明確告知使用者
- ✅ 顯示將要修改的內容
- ✅ 提供取消選項
- ✅ 操作後確認結果

## 注意事項

1. **保守策略**：有疑問時，選擇保留現有內容
2. **透明操作**：每個動作都告知使用者
3. **可逆性**：提供備份和還原選項
4. **保持簡潔**：CLAUDE.md 應該簡短實用，不要放太多細節
5. **聚焦關鍵資訊**：技術棧、規範、特殊注意事項
6. **可擴展**：為使用者留下自訂空間
7. **團隊友善**：在團隊專案中特別謹慎

## 進階選項（可選）

### 選項 1：建立專案特定技能模板
如果使用者選擇建立 skills 目錄，可提供建立第一個技能的指引。

### 選項 2：整合 CI/CD 配置
如果偵測到 GitHub/GitLab，提供 CI/CD 整合建議。

### 選項 3：建立開發文件結構
建議建立 docs/ 目錄並提供模板。
