---
name: redbook-creator-publish
description: "小红书帖子创作与发布技能。用于:(1) 生成小红书风格的帖子内容(标题+正文+标签)(2) 获取/生成帖子配图 (3) 自动上传到小红书创作者平台。触发词:小红书创作、create redbook、小红书、红书、笔记创作、帖子创作"
allowedCommands:
  - "python3"
  - "python3 *"
  - "curl *"
  - "mkdir *"
  - "ls *"
  - "rm *"
  - "file *"
---

# 小红书帖子创作与发布技能

## 概述

快速创作并发布小红书帖子,包括标题、正文、标签、配图,自动上传到小红书平台。

**核心特点(v5.3)**:
- 🚀 **极简流程**:搜索 → 生成内容 → 获取配图 → 验证图片 → 自动上传
- 🖼️ **智能配图**:从网络下载公开图片或 AI 生成
- ✅ **图片验证**:自动检测无效图片并删除,确保5-6张有效图片
- 📁 **精简输出**:只生成 preview.html、config.json、images/
- 🤖 **自动上传**:直接触发,减少手动确认
- 🔐 **登录检测**:智能检测登录状态,20秒等待提示
- 🐍 **零依赖**:Python 原生能力为主

---

## 前置检查(第一步,自动执行)

### 检查 Python

```bash
python3 --version  # 要求 3.7+
```

### 全局检测并安装 Playwright

**重要**: 只在未安装时才安装,避免重复安装

```bash
# 检查 playwright 是否已安装(全局检测)
if python3 -c "import playwright" 2>/dev/null; then
    echo "✅ Playwright 已安装,跳过安装"
else
    echo "📦 正在安装 Playwright 到用户全局环境..."
    # 安装到用户全局环境
    pip3 install --user playwright
    # 安装 Chromium 浏览器
    python3 -m playwright install chromium
    echo "✅ Playwright 安装完成"
fi
```

**检测逻辑**:
- 先检查系统是否已安装 playwright 模块
- 如果已安装,跳过安装步骤
- 如果未安装,安装到用户全局环境(`--user`)
- 只需安装一次,后续使用无需重复安装

输出:`✅ 环境检查完成`

---

## 工作流程

### 步骤1:深度搜索话题

使用 WebSearch 工具搜索 2-3 个相关关键词,获取最新数据和案例。

### 步骤2:生成帖子内容

- **标题**:不超过20个字符(含emoji),包含钩子词,简洁有力
- **正文**:300-500字,开头钩子 → 核心内容 → 总结互动
- **标签**:5个(精选核心标签,不要超过5个)

### 步骤3:获取配图(5-6张)

**优先级**:
1. 从网络下载公开图片(Unsplash/Pexels/Pixabay)
2. AI 生成补充(可选,如果模型无法生成则跳过)

**要求**:
- 数量:**5-6张(1封面 + 4-5内容图)**
- 命名:cover.png, image_1.png, image_2.png, image_3.png, image_4.png...
- 格式:PNG/JPG

#### 🎯 精准配图关键词生成策略(重要!)

**核心原则**: 搜索关键词必须高度具体化,避免泛化词汇

**关键词生成规则**:
1. **主体词必须具体化**: 不用泛化词,用具体实体名
   - ❌ 错误: "游戏" "game" "gaming"
   - ✅ 正确: "Minecraft" "我的世界方块" "Minecraft Steve"

2. **场景/画面词优先**: 描述具体画面内容
   - ❌ 错误: "美食" "food"
   - ✅ 正确: "拉面特写" "ramen noodles closeup"

3. **英文关键词为主**: 图库用英文搜索效果更好
   - 将中文主题翻译成对应的英文专有名词
   - 游戏:用游戏官方英文名
   - 品牌:用品牌英文名

**不同主题的关键词构建**:

| 主题类型 | 泛化词(❌避免) | 精准词(✅使用) |
|---------|--------------|--------------|
| 游戏《我的世界》| game, gaming, 游戏 | Minecraft, Minecraft blocks, Minecraft world, pixel art cube |
| 游戏《原神》| game, RPG | Genshin Impact, Genshin character, anime game |
| 游戏《王者荣耀》| MOBA, 手游 | Honor of Kings, MOBA hero |
| 美食-火锅 | food, 美食 | hotpot, Chinese hotpot, spicy pot |
| 旅行-日本 | travel, 旅行 | Tokyo tower, Japan temple, cherry blossom |
| 科技-AI | technology | AI robot, artificial intelligence, neural network |
| 穿搭-韩系 | fashion, 穿搭 | Korean outfit, OOTD, minimalist style |

**配图搜索执行步骤**:
1. 分析帖子主题,提取**核心实体**(游戏名/品牌名/地点名等)
2. 将核心实体转换为**英文专有名词**
3. 组合搜索词: `[英文专有名词] + [画面描述词]`
4. 每张图使用不同角度的关键词组合

**示例 - 主题"我的世界游戏"**:
```
封面图: "Minecraft world landscape" 或 "Minecraft building"
图1: "Minecraft blocks closeup"
图2: "Minecraft Steve character"
图3: "Minecraft castle build"
图4: "Minecraft pixel art"
图5: "Minecraft survival gameplay"
```

**示例 - 主题"火锅推荐"**:
```
封面图: "Chinese hotpot restaurant"
图1: "hotpot ingredients fresh"
图2: "spicy hotpot soup"
图3: "hotpot meat slices"
图4: "hotpot dipping sauce"
```

```bash
mkdir -p ./redbook-article/[主题]-[日期]/images/
# 使用精准关键词下载图片到 images 目录
```

### 步骤3.5:验证图片有效性(重要!)

**下载完成后必须验证每张图片**,删除无效图片并补充下载,确保最终有5-6张有效图片。

```python
import os
import struct

def is_valid_image(file_path):
    """验证图片文件是否有效"""
    if not os.path.exists(file_path):
        return False

    # 检查文件大小(至少1KB)
    if os.path.getsize(file_path) < 1024:
        return False

    try:
        with open(file_path, 'rb') as f:
            header = f.read(32)

            # PNG: 89 50 4E 47 0D 0A 1A 0A
            if header[:8] == b'\x89PNG\r\n\x1a\n':
                return True

            # JPEG: FF D8 FF
            if header[:3] == b'\xff\xd8\xff':
                return True

            # GIF: GIF87a 或 GIF89a
            if header[:6] in (b'GIF87a', b'GIF89a'):
                return True

            # WebP: RIFF....WEBP
            if header[:4] == b'RIFF' and header[8:12] == b'WEBP':
                return True

    except Exception:
        return False

    return False

def validate_and_clean_images(images_dir):
    """验证并清理无效图片,返回有效图片列表"""
    valid_images = []
    invalid_images = []

    for filename in sorted(os.listdir(images_dir)):
        if filename.lower().endswith(('.png', '.jpg', '.jpeg', '.gif', '.webp')):
            file_path = os.path.join(images_dir, filename)
            if is_valid_image(file_path):
                valid_images.append(filename)
                print(f"✅ {filename} - 有效")
            else:
                invalid_images.append(filename)
                print(f"❌ {filename} - 无效,已删除")
                os.remove(file_path)

    return valid_images, invalid_images

# 使用示例
images_dir = '/path/to/images'
valid, invalid = validate_and_clean_images(images_dir)
print(f"\n有效图片: {len(valid)} 张")
print(f"已删除无效图片: {len(invalid)} 张")

# 如果有效图片不足5张,需要补充下载
if len(valid) < 5:
    print(f"⚠️ 图片不足,需要补充 {5 - len(valid)} 张")
```

**验证流程**:
1. 下载所有图片后,运行验证脚本
2. 删除无效图片(文件头不正确或大小过小)
3. 统计有效图片数量
4. **如果有效图片少于5张,必须补充下载新图片**
5. 重复验证直到有5-6张有效图片

### 步骤4:生成本地文件

**目录结构**:
```
redbook-article/
└── [主题]-[日期]/
    ├── preview.html
    ├── config.json
    └── images/
        ├── cover.png
        ├── image_1.png
        ├── image_2.png
        ├── image_3.png
        └── image_4.png
```

**生成 config.json**(重要:使用 json.dump 自动转义):
```python
import json

config = {
  "title": title,
  "content": content,  # json.dump 会自动处理转义
  "tags": tags,
  "cover": "images/cover.png",
  "images": ["images/image_1.png", "images/image_2.png", "images/image_3.png", "images/image_4.png"]
}

with open('config.json', 'w', encoding='utf-8') as f:
    json.dump(config, f, ensure_ascii=False, indent=2)
```

**生成 preview.html**(使用 Newsprint 风格模板):
1. 读取模板:`/Users/a58/.claude/skills/redbook-creator-publish/assets/preview-template.html`
2. 替换变量:
   - `{{TITLE}}` → 标题
   - `{{CONTENT}}` → 正文
   - `{{DATE}}` → 日期
   - `{{TAGS}}` → 标签HTML(每个标签包裹在 `<span class="tag">` 中)
3. 添加封面图和配图的 `<img>` 标签
4. 保存为 `preview.html`

### 步骤5:自动上传到小红书

**重要**:必须触发自动上传,即使前面有错误也要尝试。

```bash
python3 /Users/a58/.claude/skills/redbook-creator-publish/scripts/auto_upload_playwright.py \
  --config ./redbook-article/[主题]-[日期]/config.json
```

**流程(v5.1 优化)**:
1. 打开小红书创作者平台
2. **智能登录检测**:
   - 自动检测上传控件DOM是否存在
   - 如未登录,提示用户在 20 秒内登录
   - 每秒检测一次,登录成功后自动继续
   - 20 秒后仍未检测到控件,退出并提示手动上传
3. 上传图片(cover.png 第一张)
4. 填写标题
5. 填写正文
6. 逐个输入标签(间隔1秒+回车)
7. 点击发布
8. **浏览器保持打开**(不自动关闭)

---

## 注意事项

### 内容创作
- **标题**:不超过20个字符(含emoji)
- 正文字数:300-500字
- **配图数量:5-6张(1封面+4-5副图)**
- **标签**:5个(不要超过5个)
- 图片来源:公开免费图库或 AI 生成
- **图片验证**:下载后必须验证有效性,删除无效图片并补充

### 自动上传(v5.1)
- 必须触发上传脚本
- **首次使用**:在20秒提示期内登录小红书账号
- **后续使用**:自动保持登录,无需重复登录
- 标签间隔 1 秒并回车
- **浏览器不会自动关闭**,可继续查看或编辑
- 如 20 秒内未登录,请手动上传或重新执行

### 技术配置
- Python 3.7+
- Playwright(**只需安装一次**,全局检测)
- config.json 中文符号需正确转义

---

## 手动上传备选方案

如果自动上传失败:

1. 打开:https://creator.xiaohongshu.com/publish/publish
2. 按顺序上传图片(cover.png 第一张)
3. 从 preview.html 复制标题和正文
4. 添加标签
5. 点击发布

---

## 参考文档

- [references/style-guide.md](references/style-guide.md) - 小红书内容风格规范
- [scripts/auto_upload_playwright.py](scripts/auto_upload_playwright.py) - Playwright 自动上传脚本
- [assets/preview-template.html](assets/preview-template.html) - Newsprint 风格预览模板

---

## 更新日志

### v5.4 (2026-01-27)
- ✅ 新增精准配图关键词生成策略
- ✅ 添加不同主题的关键词构建规则表
- ✅ 强调使用英文专有名词搜索
- ✅ 提供游戏/美食/旅行等主题的搜索关键词示例
- ✅ 更新style-guide.md增加主题精准匹配原则

### v5.3 (2026-01-27)
- ✅ 新增图片有效性验证(检测文件头和大小)
- ✅ 自动删除无效图片并提示补充下载
- ✅ 配图数量调整为5-6张(1封面+4-5副图)
- ✅ 添加 allowedCommands 配置,减少手动确认提示

### v5.2 (2026-01-27)
- ✅ 标题限制改为20个字符(含emoji)
- ✅ 标签数量从8-10个改为5个
- ✅ 配置自动批准权限,减少确认提示

### v5.1 (2026-01-27)
- ✅ 新增智能登录检测(检测上传控件DOM)
- ✅ 20秒登录等待提示,每秒检测
- ✅ 浏览器不自动关闭,保持打开状态
- ✅ Playwright 全局安装检测,避免重复安装

### v5.0 (2026-01-27)
- 极简版发布

---

**版本**:v5.4(精准配图版)
**更新日期**:2026-01-27
