---
name: ui-ux-pro-max
description: "UI/UX 設計智能助手。50 種風格、21 套配色、50 組字體搭配、20 種圖表、8 種技術棧（React、Next.js、Vue、Svelte、SwiftUI、React Native、Flutter、Tailwind）。動作：規劃、建構、創建、設計、實作、審查、修復、改進、優化、增強、重構、檢查 UI/UX 程式碼。專案：網站、登陸頁、儀表板、管理面板、電商、SaaS、作品集、部落格、行動應用程式、.html、.tsx、.vue、.svelte。元素：按鈕、彈窗、導航列、側邊欄、卡片、表格、表單、圖表。風格：玻璃擬態、黏土擬態、極簡主義、野獸派、新擬態、便當格、深色模式、響應式、擬物設計、扁平設計。主題：配色方案、無障礙設計、動畫、佈局、排版、字體搭配、間距、懸停效果、陰影、漸層。"
---

# UI/UX Pro Max - 設計智能

可搜尋的 UI 風格、配色方案、字體搭配、圖表類型、產品建議、UX 指南和技術棧最佳實踐資料庫。

## 先決條件

檢查是否已安裝 Python：

```bash
python3 --version || python --version
```

如果未安裝 Python，請根據作業系統安裝：

**macOS：**
```bash
brew install python3
```

**Ubuntu/Debian：**
```bash
sudo apt update && sudo apt install python3
```

**Windows：**
```powershell
winget install Python.Python.3.12
```

---

## 如何使用此技能

當使用者請求 UI/UX 工作（設計、建構、創建、實作、審查、修復、改進）時，遵循此工作流程：

### 步驟 1：分析使用者需求

從使用者請求中提取關鍵資訊：
- **產品類型**：SaaS、電商、作品集、儀表板、登陸頁等
- **風格關鍵字**：極簡、活潑、專業、優雅、深色模式等
- **產業**：醫療、金融科技、遊戲、教育等
- **技術棧**：React、Vue、Next.js，或預設為 `html-tailwind`

### 步驟 2：搜尋相關領域

使用 `search.py` 多次搜尋以收集完整資訊。持續搜尋直到有足夠的上下文。

```bash
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "<關鍵字>" --domain <領域> [-n <最大結果數>]
```

**建議搜尋順序：**

1. **Product** - 取得產品類型的風格建議
2. **Style** - 取得詳細風格指南（顏色、效果、框架）
3. **Typography** - 取得字體搭配與 Google Fonts 匯入
4. **Color** - 取得配色方案（主色、次色、CTA、背景、文字、邊框）
5. **Landing** - 取得頁面結構（如果是登陸頁）
6. **Chart** - 取得圖表建議（如果是儀表板/分析）
7. **UX** - 取得最佳實踐和反模式
8. **Stack** - 取得技術棧專屬指南（預設：html-tailwind）

### 步驟 3：技術棧指南（預設：html-tailwind）

如果使用者未指定技術棧，**預設為 `html-tailwind`**。

```bash
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "<關鍵字>" --stack html-tailwind
```

可用技術棧：`html-tailwind`、`react`、`nextjs`、`vue`、`svelte`、`swiftui`、`react-native`、`flutter`

---

## 搜尋參考

### 可用領域

| 領域 | 用途 | 範例關鍵字 |
|--------|---------|------------------|
| `product` | 產品類型建議 | SaaS、電商、作品集、醫療、美容、服務 |
| `style` | UI 風格、顏色、效果 | 玻璃擬態、極簡主義、深色模式、野獸派 |
| `typography` | 字體搭配、Google Fonts | 優雅、活潑、專業、現代 |
| `color` | 按產品類型的配色方案 | saas、ecommerce、healthcare、beauty、fintech、service |
| `landing` | 頁面結構、CTA 策略 | hero、hero-centric、testimonial、pricing、social-proof |
| `chart` | 圖表類型、函式庫建議 | trend、comparison、timeline、funnel、pie |
| `ux` | 最佳實踐、反模式 | animation、accessibility、z-index、loading |
| `prompt` | AI 提示詞、CSS 關鍵字 | （風格名稱） |

### 可用技術棧

| 技術棧 | 重點 |
|-------|-------|
| `html-tailwind` | Tailwind 工具類、響應式、無障礙（預設） |
| `react` | 狀態、Hooks、效能、模式 |
| `nextjs` | SSR、路由、圖片、API 路由 |
| `vue` | Composition API、Pinia、Vue Router |
| `svelte` | Runes、stores、SvelteKit |
| `swiftui` | Views、State、Navigation、Animation |
| `react-native` | Components、Navigation、Lists |
| `flutter` | Widgets、State、Layout、Theming |

---

## 範例工作流程

**使用者請求：** "做一個專業護膚服務的登陸頁"

**AI 應該：**

```bash
# 1. 搜尋產品類型
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "beauty spa wellness service" --domain product

# 2. 搜尋風格（根據產業：美容、優雅）
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "elegant minimal soft" --domain style

# 3. 搜尋排版
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "elegant luxury" --domain typography

# 4. 搜尋配色方案
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "beauty spa wellness" --domain color

# 5. 搜尋登陸頁結構
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "hero-centric social-proof" --domain landing

# 6. 搜尋 UX 指南
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "animation" --domain ux
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "accessibility" --domain ux

# 7. 搜尋技術棧指南（預設：html-tailwind）
python3 .claude/skills/ui-ux-pro-max/scripts/search.py "layout responsive" --stack html-tailwind
```

**然後：** 綜合所有搜尋結果並實作設計。

---

## 更好結果的技巧

1. **關鍵字要具體** - "healthcare SaaS dashboard" > "app"
2. **多次搜尋** - 不同關鍵字會揭示不同見解
3. **組合領域** - Style + Typography + Color = 完整設計系統
4. **總是檢查 UX** - 搜尋 "animation"、"z-index"、"accessibility" 以找出常見問題
5. **使用 stack 參數** - 取得實作專屬的最佳實踐
6. **迭代** - 如果第一次搜尋不符合，嘗試不同關鍵字

---

## 專業 UI 的通用規則

這些是經常被忽略、會讓 UI 看起來不專業的問題：

### 圖示與視覺元素

| 規則 | 應該 | 不應該 |
|------|----|----- |
| **不使用表情符號圖示** | 使用 SVG 圖示（Heroicons、Lucide、Simple Icons） | 使用 🎨 🚀 ⚙️ 等表情符號作為 UI 圖示 |
| **穩定的懸停狀態** | 懸停時使用顏色/透明度過渡 | 使用會移動版面的縮放變形 |
| **正確的品牌標誌** | 從 Simple Icons 研究官方 SVG | 猜測或使用錯誤的標誌路徑 |
| **一致的圖示大小** | 使用固定 viewBox (24x24) 配合 w-6 h-6 | 隨意混用不同圖示大小 |

### 互動與游標

| 規則 | 應該 | 不應該 |
|------|----|----- |
| **游標指標** | 為所有可點擊/可懸停的卡片添加 `cursor-pointer` | 互動元素保持預設游標 |
| **懸停回饋** | 提供視覺回饋（顏色、陰影、邊框） | 沒有指示元素可互動 |
| **平滑過渡** | 使用 `transition-colors duration-200` | 即時狀態變化或太慢 (>500ms) |

### 淺色/深色模式對比度

| 規則 | 應該 | 不應該 |
|------|----|----- |
| **玻璃卡片淺色模式** | 使用 `bg-white/80` 或更高透明度 | 使用 `bg-white/10`（太透明） |
| **淺色文字對比** | 文字使用 `#0F172A` (slate-900) | 內文使用 `#94A3B8` (slate-400) |
| **淺色次要文字** | 最低使用 `#475569` (slate-600) | 使用 gray-400 或更淺 |
| **邊框可見性** | 淺色模式使用 `border-gray-200` | 使用 `border-white/10`（看不見） |

### 版面與間距

| 規則 | 應該 | 不應該 |
|------|----|----- |
| **浮動導航列** | 添加 `top-4 left-4 right-4` 間距 | 導航列貼在 `top-0 left-0 right-0` |
| **內容內距** | 考慮固定導航列高度 | 讓內容隱藏在固定元素後面 |
| **一致的最大寬度** | 使用相同的 `max-w-6xl` 或 `max-w-7xl` | 混用不同容器寬度 |

---

## 交付前檢查清單

交付 UI 程式碼前，驗證這些項目：

### 視覺品質
- [ ] 沒有使用表情符號作為圖示（改用 SVG）
- [ ] 所有圖示來自一致的圖示集（Heroicons/Lucide）
- [ ] 品牌標誌正確（已從 Simple Icons 驗證）
- [ ] 懸停狀態不會造成版面位移
- [ ] 直接使用主題顏色 (bg-primary) 而非 var() 包裝

### 互動
- [ ] 所有可點擊元素都有 `cursor-pointer`
- [ ] 懸停狀態提供清晰的視覺回饋
- [ ] 過渡平滑 (150-300ms)
- [ ] 鍵盤導航可見焦點狀態

### 淺色/深色模式
- [ ] 淺色模式文字有足夠對比度（最低 4.5:1）
- [ ] 玻璃/透明元素在淺色模式可見
- [ ] 兩種模式邊框都可見
- [ ] 交付前測試兩種模式

### 版面
- [ ] 浮動元素與邊緣有適當間距
- [ ] 沒有內容隱藏在固定導航列後面
- [ ] 320px、768px、1024px、1440px 響應式
- [ ] 行動裝置沒有水平捲軸

### 無障礙
- [ ] 所有圖片有 alt 文字
- [ ] 表單輸入有標籤
- [ ] 顏色不是唯一指示器
- [ ] 遵循 `prefers-reduced-motion`
