---
name: stardew-wiki-advisor
description: Query Stardew Valley Wiki using natural language. Ask about crops, NPCs, strategies, and more.
user-invocable: true
metadata:
  {
    "openclaw":
    {
      "emoji": "🌾",
      "requires": {
        "bins": ["python3", "curl"],
        "pythonBin": "{baseDir}/.venv/bin/python3",
        "env": []
      },
      "homepage": "https://zh.stardewvalleywiki.com/"
    }
  }
---

# Stardew Valley Wiki Advisor

問牧場物語的任何問題。由本機 AI 和本地 Wiki 向量資料庫驅動。

## Setup (First Time Only)

在使用前，需要初始化向量資料庫。

```bash
# 1. 安裝依賴
pip install requests beautifulsoup4 lxml numpy faiss-cpu ollama

# 2. 下載 Embedding 模型
ollama pull qwen3-embedding

# 3. 下載 LLM 模型
ollama pull qwen3:8b

# 4. 爬取 Wiki (100 頁)
python3 {baseDir}/scripts/crawl_wiki.py --max-pages 100 --output {baseDir}/data

# 5. 建立向量索引
python3 {baseDir}/scripts/build_vectors.py --input {baseDir}/data/raw_pages.json --output {baseDir}/data --model qwen3-embedding

# 初始化完成！
```

Takes ~20-30 minutes on first run:
- 5 min: crawl 100 Wiki pages
- 12-15 min: vectorization (100 pages × 10 chunks = 1000+ embeddings)
- Ollama model downloads: ~10-20 min (first time only)

## Query

> Note (OpenClaw sandbox): `{baseDir}/venv` in this repo is a **macOS** venv (Mach-O) and will not run inside the Linux Docker sandbox.
> OpenClaw will create a Linux venv at `{baseDir}/.venv` inside the sandbox and install deps from `{baseDir}/requirements.txt`.

```bash
{baseDir}/.venv/bin/python3 {baseDir}/scripts/query.py \
  --data-dir {baseDir}/data \
  --query "你的問題" \
  --embedding-model qwen3-embedding \
  --llm-model qwen3:8b
```

**Output Format for Agents:**
When responding to user queries:
1. State estimated completion time (typically 5-10 seconds)
2. Execute query and return results directly
3. Do NOT describe the process or show progress messages
4. Include Wiki links and similarity scores in results

## Examples

- "怎樣賺最多錢？"
- "Elliott 喜歡什麼禮物？"
- "漁場怎樣升級？"
- "哪些作物收益最高？"
- "怎樣快速提升人氣度？"
- "能源核心在哪裡找？"

## Requirements

- Python 3.8+
- **Ollama with models** (必須已下載):
  - `qwen3-embedding` (4.7 GB, for embeddings)
  - `qwen3:8b` (5.2 GB, for LLM inference)
- Python venv 已配置完成，所有依賴已安裝（numpy, faiss-cpu, ollama, requests, beautifulsoup4）
- 30 GB 磁盤空間（模型 + 爬蟲結果 + 索引）
- 8-10 GB RAM (qwen3:8b is lighter than GLM-4.7-Flash)

## How It Works

1. **Crawl** — BFS 爬取 zh.stardewvalleywiki.com（100+ 頁）
   - 動態發現：從首頁自動提取所有內容連結
   - 去重機制：URL 規範化 + MD5 hash 防重複

2. **Chunk** — 將每頁分成 500 字的段落

3. **Embed** — 用 qwen3-embedding 生成 4096 維向量
   - 100 頁 → 1063+ embeddings chunks
   - 總大小：~17 MB

4. **Index** — 用 FAISS 建立本機索引
   - 秒級相似度搜尋
   - 無需網路或 API

5. **Query** — 用戶提問 → 相似度搜尋 → qwen3:8b 生成答案
   - 返回可溯源的 Wiki 連結和相似度分數

## Performance

- Query latency: 5-10 seconds（使用 qwen3:8b，比 glm-4.7-flash 快 5-10 倍）
  - Embedding generation: <1 sec
  - FAISS search: ~0.1 sec
  - LLM generation: ~5-8 sec
- Throughput: ~6-10 queries/minute
- Memory: ~8-10 GB
- Storage: ~17 MB (indexed data)

## Troubleshooting

- **"Ollama connection failed"** → Make sure `ollama serve` is running
- **"Model not found"** → Download with `ollama pull qwen3:8b`
- **"No embeddings found"** → Run setup first
- **"Slow response"** → Check if Ollama is overloaded; try reducing --top-k

## Update Data

To refresh Wiki data:

```bash
python3 {baseDir}/scripts/crawl_wiki.py --max-pages 100 --output {baseDir}/data
python3 {baseDir}/scripts/build_vectors.py --input {baseDir}/data/raw_pages.json --output {baseDir}/data
```

