---
name: story-display-skill
description: 负责小说情节的持久化写入、生成校验以及终端的完整展示。通过原生指令确保万字剧情不被截断。
---

# 小说情节展示技能 (Story Display Skill)

## 概述
本技能是“万言小说”最后落地的关键环节。它通过“写入 -> 校验 -> 原生展示”的闭环，确保每一回剧情都能安全存盘并 1:1 地呈现在玩家终端上。

## 核心职责
1. **稳健写入**：将 `story-engine` 生成的内容写入 `./history/chapters/`。
2. **落盘校验**：写入后通过脚本检测文件是否存在且字节数正常。
3. **原生展示**：调用系统原生指令（如 `type`）完整输出内容，规避 AI 模型输出长度限制。

## 使用流程
1. 当一章剧情生成完毕后，调用：
   `python .agent/skills/story-display-skill/scripts/display_chapter.py --chapter <N> --content "<CONTENT>"`
2. 脚本会自动处理写入、校验并回显。

## 脚本索引
- `scripts/display_chapter.py`: 处理核心流转。
