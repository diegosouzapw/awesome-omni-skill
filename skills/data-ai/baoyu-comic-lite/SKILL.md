---
name: baoyu-comic-lite
description: 知识漫画创作工具，支持多种风格（如 Logicomix/Ligne Claire、欧姆社漫画指南）。创作带有详细分镜布局和序列图像生成的原创教育漫画。（精简版，不包含 PDF 合并功能）
---

# 知识漫画创作者 (Knowledge Comic Creator)

创作具有多种视觉风格的原创知识漫画。

## 使用方法

```bash
/baoyu-comic posts/turing-story/source.md
/baoyu-comic  # 然后粘贴内容
```

## 选项

| 选项 | 取值 |
|--------|--------|
| `--style` | classic (默认), dramatic, warm, sepia, vibrant, ohmsha, realistic, wuxia, shoujo, 或自定义描述 |
| `--layout` | standard (默认), cinematic, dense, splash, mixed, webtoon |
| `--aspect` | 3:4 (默认, 纵向), 4:3 (横向), 16:9 (宽屏) |
| `--lang` | auto (默认), zh, en, ja 等 |

风格 (Style) × 布局 (Layout) × 纵横比 (Aspect) 可以自由组合。自定义风格可以用自然语言描述。

**纵横比在漫画的所有页面中保持一致。**

## 自动选择逻辑

| 内容信号 | 风格 (Style) | 布局 (Layout) |
|-----------------|-------|--------|
| 教程, 操作指南, 初学者 | ohmsha | webtoon |
| 计算, AI, 编程 | ohmsha | dense |
| 1950年前, 古典, 古代 | sepia | cinematic |
| 个人故事, 导师 | warm | standard |
| 冲突, 突破 | dramatic | splash |
| 红酒, 美食, 商业, 生活方式, 专业 | realistic | cinematic |
| 武侠, 仙侠, 中国历史 | wuxia | splash |
| 浪漫, 爱情, 校园生活, 友谊, 情感 | shoujo | standard |
| 传记, 均衡型 | classic | mixed |

**代理执行指令**：
无。 (该技能不依赖外部脚本)

## 文件结构

每个会话都会创建一个以内容缩略名（slug）命名的独立目录：

```
comic/{topic-slug}/
├── source-{slug}.{ext}            # 源文件（文本、图像等）
├── analysis.md                    # 深度分析结果 (YAML+MD)
├── storyboard-chronological.md    # 方案 A：时间轴顺序（保留）
├── storyboard-thematic.md         # 方案 B：主题分类（保留）
├── storyboard-character.md        # 方案 C：角色驱动（保留）
├── characters-chronological/      # 方案 A 的角色设定（保留）
│   ├── characters.md
│   └── characters.png
├── characters-thematic/           # 方案 B 的角色设定（保留）
│   ├── characters.md
│   └── characters.png
├── characters-character/          # 方案 C 的角色设定（保留）
│   ├── characters.md
│   └── characters.png
├── storyboard.md                  # 最终选定的分镜
├── characters/                    # 最终选定的角色
│   ├── characters.md
│   └── characters.png
└── prompts/
    ├── 00-cover-[slug].md
    └── NN-page-[slug].md
├── 00-cover-[slug].png
└── NN-page-[slug].png
```

**缩略名 (Slug) 生成规则**：
1. 从内容中提取主题（2-4 个词，使用 kebab-case，如：alan-turing-bio）
2. 示例：“阿兰·图灵传记” → `alan-turing-bio`

**冲突处理**：
如果 `comic/{topic-slug}/` 已存在：
- 追加时间戳：`{topic-slug}-YYYYMMDD-HHMMSS`
- 示例：`turing-story` 已存在 → `turing-story-20260118-143052`

**源文件**：
以 `source-{slug}.{ext}` 命名复制所有源文件：
- `source-biography.md`, `source-portrait.jpg`, `source-timeline.png` 等
- 支持多个来源：对话中的文本、图像、文件

## 工作流

### 第一步：内容分析 → `analysis.md`

读取源内容，必要时保存，并进行深度分析。

**动作**：
1. **保存源内容**（如果尚未作为文件存在）：
   - 如果用户提供了文件路径：直接使用
   - 如果用户粘贴了内容：保存在目标目录的 `source.md` 中
2. 读取源内容
3. 遵循 `references/analysis-framework.md` 进行**深度分析**：
   - 目标受众识别
   - 读者价值主张
   - 核心主题和叙事潜力
   - 关键人物及其故事曲线
4. 检测源语言
5. 确定建议页数：
   - 短篇故事：5-8 页
   - 中等复杂度：9-15 页
   - 完整传记：16-25 页
6. 分析内容信号以推荐风格/布局
7. **保存至 `analysis.md`**

**analysis.md 格式**：

```yaml
---
title: "阿兰·图灵：计算机科学之父"
topic: 传记
time_span: 1912-1954
source_language: en
user_language: zh
aspect_ratio: "3:4"
recommended_page_count: 12
---

## 目标受众

- **核心受众**：对计算历史感兴趣的科技爱好者
- **次要受众**：学习科学突破的学生
- **潜在受众**：对人物传记故事感兴趣的普通读者

## 价值主张

读者将获得：
1. 了解现代计算是如何诞生的
2. 与一位天才但悲剧的人物建立情感联系
3. 理解创新背后的人道代价

## 核心主题

| 主题 | 叙事潜力 | 视觉呈现机会 |
|-------|--------------------|--------------------|
| 天才与社会的对抗 | 高冲突，戏剧性曲线 | 对比鲜明的场景 |
| 密码破译 | 神秘，紧张 | 把技术图表作为艺术呈现 |
| 个人悲剧 | 情感深度 | 亲密、忧郁的面板 |

## 关键人物与故事曲线

### 阿兰·图灵 (主角)
- **曲线**：被误解的天才 → 战争英雄 → 悲剧结局
- **视觉特征**：凌乱的学究形象，眼神深邃
- **关键时刻**：Enigma 破译工作、被捕、最后的日子

### 克里斯托弗·莫科姆 (催化剂)
- **角色**：早年好友，其去世塑造了图灵
- **视觉特征**：年轻、阳光
- **关键时刻**：校园友谊、突然离世

## 内容信号

- "biography" → classic + mixed
- "computing history" → ohmsha + dense
- "personal tragedy" → dramatic + splash

## 建议方案

1. **时间轴顺序** - 遵循生平时间线（推荐用于传记）
2. **主题分类** - 按贡献组织（适合教学重点）
3. **角色驱动** - 关系驱动叙事（适合情感影响）
```

### 第二步：生成 3 个分镜变体

创建三个不同的方案，每个方案结合一种叙事方式和推荐风格。

| 方案 | 叙事方式 | 推荐风格 | 布局 |
|---------|-------------------|-------------------|--------|
| A | 时间轴顺序 | sepia | cinematic |
| B | 主题分类 | ohmsha | dense |
| C | 角色驱动 | warm | standard |

**针对每个变体**：

1. **生成分镜脚本** (`storyboard-{approach}.md`)：
   - YAML 前置数据包含 narrative_approach, recommended_style, recommended_layout, aspect_ratio
   - 封面设计
   - 每一页：布局、面板分解、视觉提示词
   - **使用用户的首选语言编写**
   - 参考：`references/storyboard-template.md`

2. **生成配套角色** (`characters-{approach}/`)：
   - `characters.md` - 符合推荐风格的视觉规范（使用用户的首选语言）
   - `characters.png` - 角色参考表
   - 参考：`references/character-template.md`

**所有变体在选定后都会保留供参考。**

### 第三步：用户确认所有选项

**重要**：在一个 AskUserQuestion 步骤中呈现所有选项。不要通过多个单独的确认来中断工作流。

**确定要问的问题**：

| 问题 | 何时询问 |
|----------|-------------|
| 分镜方案 | 始终（必需） |
| 视觉风格 | 始终（必需） |
| 语言 | 仅当 `source_language ≠ user_language` 时 |
| 纵横比 | 仅当用户可能偏好非默认值（如横向内容）时 |

**语言处理**：
- 如果源语言 = 用户语言：仅告知用户（如，“漫画将采用中文”）
- 如果不同：询问要使用哪种语言

**所有的分镜和提示词都以用户选择/首选的语言生成。**

**纵横比处理**：
- 默认：3:4 (纵向) - 标准漫画格式
- 如果内容适合（如全景场景、技术图表），提供 4:3 (横向)
- 对于电影感内容，提供 16:9 (宽屏)

**AskUserQuestion 示例格式**：

```
问题 1 (分镜方案): 您选择哪种分镜方案？
- A: 时间轴顺序 + sepia (推荐)
- B: 主题分类 + ohmsha
- C: 角色驱动 + warm
- 自定义

问题 2 (风格): 您选择哪种视觉风格？
- sepia (方案推荐)
- classic / dramatic / warm / sepia / vibrant / ohmsha / realistic / wuxia
- 自定义描述

问题 3 (语言) - 仅当不匹配时:
- 中文 (源材料语言)
- 英文 (您的偏好)

问题 4 (纵横比) - 仅当相关时:
- 3:4 纵向 (推荐)
- 4:3 横向
- 16:9 宽屏
```

**确认后**：
1. 将选定的分镜复制到 → `storyboard.md`
2. 将选定的角色复制到 → `characters/`
3. 更新 YAML 前置数据，包含确认的风格、语言和纵横比
4. 如果风格与方案推荐的不同：重新生成 `characters/characters.png`
5. 用户可以直接编辑文件进行微调

### 第四步：生成图像

使用确认的分镜 + 风格 + 纵横比：

**针对每一页（封面 + 正文）**：
1. 将提示词保存至 `prompts/NN-{cover|page}-[slug].md`（使用用户的首选语言）
2. 使用确认的风格和纵横比生成图像
3. 在每次生成后报告进度

**图像生成技能选择**：
- 检查可用的图像生成技能
- 如果有多个技能可用，询问用户偏好

**角色参考处理**：
- 如果技能支持参考图：提供 `characters/characters.png`
- 如果技能不支持参考图：在提示词中包含 `characters/characters.md` 的内容

**会话管理**：
如果图像生成技能支持 `--sessionId`：
1. 生成唯一的会话 ID：`comic-{topic-slug}-{timestamp}`
2. 为所有页面使用相同的会话 ID
3. 确保生成的图像之间视觉风格的一致性


### 第六步：完成报告

```
漫画制作完成！
标题: [title] | 风格: [style] | 页数: [count] | 纵横比: [ratio] | 语言: [lang]
位置: [path]
✓ analysis.md
✓ characters.png
✓ 00-cover-[slug].png ... NN-page-[slug].png
```

## 页面修改

支持在初始生成后修改单个页面。

### 编辑单页

使用修改后的提示词重新生成特定页面：

1. 确定要编辑的页面（例如：`03-page-enigma-machine.png`）
2. 如果需要，更新 `prompts/03-page-enigma-machine.md` 中的提示词
3. 如果内容有显著变化，更新文件名中的 slug
4. 使用相同的会话 ID 和纵横比重新生成图像
### 添加新页面

在指定位置插入新页面：

1. 指定插入位置（例如：在第 3 页之后）
2. 创建带有合适 slug 的新提示词（例如：`04-page-bletchley-park.md`）
3. 生成新的页面图像（相同的纵横比）
4. **重新编号文件**：所有后续页面的 NN 前缀递增 1
   - `04-page-tragedy.png` → `05-page-tragedy.png`
   - Slug 保持不变
5. 在 `storyboard.md` 中更新新页面条目
### 删除页面

删除页面并重新编号：

1. 确定要删除的页面（例如：`03-page-enigma-machine.png`）
2. 删除图像文件和提示词文件
3. **重新编号文件**：所有后续页面的 NN 前缀递减 1
   - `04-page-tragedy.png` → `03-page-tragedy.png`
   - Slug 保持不变
4. 更新 `storyboard.md` 以移除页面条目
### 文件命名规范

文件使用有意义的 slug 以提高可读性：
```
NN-cover-[slug].png / NN-page-[slug].png
NN-cover-[slug].md / NN-page-[slug].md (位于 prompts/ 目录)
```

示例：
- `00-cover-turing-story.png`
- `01-page-early-life.png`
- `02-page-cambridge-years.png`
- `03-page-enigma-machine.png`

**Slug 规则**：
- 源自页面标题/内容 (kebab-case)
- 在漫画内必须唯一
- 当页面内容发生显著变化时，相应地更新 slug

**重新编号规则**：
- 在添加/删除后，更新受影响页面的 NN 前缀
- 除非内容变化，否则 slug 保持不变
- 保持编号顺序，不留空隙

## 特定风格指南

### 欧姆社风格 (`--style ohmsha`)

教育漫画的额外要求：
- **默认：直接使用哆啦A梦角色** - 无需创建新角色
  - 大雄 (Nobita)：学生角色，好奇的传信者
  - 哆啦A梦 (Doraemon)：导师角色，使用道具解释概念
  - 胖虎 (Gian)：对立面/挑战角色，代表障碍或错误观念
  - 静香 (Shizuka)：辅助角色，提出澄清性问题
- 仅在明确要求时才使用自定义角色：`--characters "学生:小明,导师:教授"`
- 必须使用视觉隐喻（道具、动作场景） - 禁止纯粹的“头像对话”
- 页面标题：采用叙事风格，而不是“第 X 页：主题”

**参考**：`references/ohmsha-guide.md` 详细指南。

## 参考资料

`references/` 目录中的详细模板和指南：
- `analysis-framework.md` - 漫画改编的深度内容分析
- `character-template.md` - 角色定义格式和示例
- `storyboard-template.md` - 分镜构造和面板分解
- `ohmsha-guide.md` - 欧姆社漫画风格细节
- `styles/` - 详细风格定义
- `layouts/` - 详细布局定义

## 扩展支持

通过 EXTEND.md 支持自定义风格和配置。

**检查路径**（优先级顺序）：
1. `.baoyu-skills/baoyu-comic-lite/EXTEND.md` (项目级)
2. `~/.baoyu-skills/baoyu-comic-lite/EXTEND.md` (用户级)

如果找到，则在第一步之前加载。扩展内容将覆盖默认设置。
