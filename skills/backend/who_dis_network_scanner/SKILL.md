---
name: who_dis_network_scanner
description: AI 驅動的本地網絡掃描器與安全分析工具 (Local Network Scanner and AI Security Analyzer)
version: 1.1.0
os: windows
dependencies:
  - python>=3.10
  - npcap
  - ollama
---

# WhoDis Network Scanner Skill

是一個類似 GlassWire 的 Windows 應用程式，結合了網絡掃描與本地 AI 分析功能。

## 📂 專案結構 (Project Structure)

```
src/
├── app.py        # FastAPI 網頁伺服器主程式
├── scanner.py    # 網路掃描（ARP + Port 掃描 + 主機名稱解析）
├── analyzer.py   # AI 分析模組（Ollama 串流）
├── database.py   # SQLite 掃描歷史存儲
└── static/       # 前端頁面（HTML/CSS/JS）
```

### 核心模組

1. **`src/scanner.py`**:
   - ARP 協定掃描區域網絡
   - TCP Port 掃描（深度掃描模式）
   - DNS 反查 + NetBIOS 主機名稱解析
   - MAC Address 廠商識別

2. **`src/analyzer.py`**:
   - 介接本地 Ollama AI 模型
   - 串流模式 (SSE) 回傳分析結果
   - 繁體中文風險評估報告

3. **`src/database.py`**:
   - SQLite 資料庫管理
   - 掃描歷史 CRUD 操作

4. **`src/app.py`**:
   - FastAPI 網頁伺服器
   - RESTful API 端點
   - 自動開啟瀏覽器

## 🚀 使用指令 (Usage)

```powershell
# 請確保終端機具有管理員權限
conda activate whodis
python src/app.py
```

瀏覽器自動開啟 `http://localhost:8000`

## 🔌 API 端點

| 端點 | 方法 | 說明 |
|------|------|------|
| `/` | GET | 首頁 |
| `/api/scan` | POST | 執行掃描 (`{deep_scan: bool}`) |
| `/api/analyze` | POST | AI 分析 (SSE 串流) |
| `/api/history` | GET | 掃描歷史 |

## ⚠️ 疑難排解 (Troubleshooting)

- **Scapy/Permission Error**: 確認是否已安裝 Npcap 並以管理員身分執行
- **AI 無回應**: 檢查 Ollama 是否在 `http://localhost:11434` 正常運作
- **Port 8000 被佔用**: 修改 `app.py` 中的 port 參數
