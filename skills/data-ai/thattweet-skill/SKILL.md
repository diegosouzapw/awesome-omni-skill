---
name: thattweet-skill
description: 搜索、分析 ThatTweet 备份中的推文内容。支持关键词检索、热度排序、增长追踪、作者筛选，并与 AI 进行深度讨论。
---

# ThatTweet 推文分析

把你的推文库变成可查询的知识资产。

## 快速开始

```bash
# Windows 下使用 UTF-8 模式（避免 emoji 乱码）
python -X utf8 scripts/analyzer.py -f "path/to/backup.json" summary

# 列出可用的 JSON 文件
python -X utf8 scripts/analyzer.py --list-files
```

## 核心命令

| 命令 | 说明 | 示例 |
|------|------|------|
| `search` | 关键词搜索 | `search "Claude"` |
| `top` | 热度排行 | `top --by views --limit 10` |
| `timeline` | 时间筛选 | `timeline --after 2026-01-20` |
| `author` | 按作者筛选 | `author Jackywine` |
| `mentions` | 查找提及 | `mentions vista8` |
| `media` | 筛选媒体 | `media --type video` |
| `quotes` | 引用推文 | `quotes` |
| `growth` | 增长分析 | `growth --limit 5` |
| `export` | 导出结果 | `export --format csv` |

**提示：** 在 search, top, author 等命令后加上 `--ai` 标志（如 `top --by views --ai`），可以输出极简格式，大幅减少 Token 消耗并提升 AI 分析速度。

所有命令都支持 `-f` 参数指定 JSON 文件。

## 数据字段

| 字段 | 说明 |
|------|------|
| `text` | 推文正文 |
| `authorId` | 作者ID（映射为用户名） |
| `favoriteCount / retweetCount / viewCount` | 互动数据 |
| `media` | 图片/视频附件 |
| `metricsHistory` | 互动历史变化 |
| `quoteFromId` | 引用来源 |

## 作者映射

JSON 中作者名为空，脚本内置映射表，可手动扩展：

```python
# scripts/analyzer.py
AUTHOR_MAP = {
    "1404738934043602947": "Jackywine",
    "6844292": "elonmusk",
    # 添加更多...
}
```

## AI 讨论

检索后可以问 AI：
- 这些推文在讨论什么？
- 热门推文有什么共同特点？
- 是否存在认知偏差？（配合 `cognitive-bias` skill）
