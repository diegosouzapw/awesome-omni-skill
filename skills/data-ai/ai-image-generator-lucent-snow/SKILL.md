---
name: ai-image-generator
description: 使用 ModelScope 等平台生成 AI 图像。当用户需要生成图像、设计图标、创建角色立绘，或需要帮助编写 AI 绘画提示词时使用此技能。支持直接生成图像和仅优化提示词两种模式。
---

# AI Image Generator

## 概述

AI Image Generator 是一个可扩展的图像生成技能，支持通过多个 AI 平台生成高质量图像。当前支持 ModelScope 平台，未来可轻松扩展到 OpenAI DALL-E、Stability AI 等平台。

**核心功能**:
- 直接生成图像（调用 API）
- 仅优化提示词（不调用 API）
- 多模型支持（FLUX.1、Qwen-Image、ChenkinNoob-XL）
- 分层参考文档（模型介绍、提示词指南、扩展指南）

## 何时使用此技能

当用户请求以下任务时，立即使用此技能：

### 1. 图像生成需求
- "帮我生成一个..."
- "创建一张图像..."
- "我需要一个...的图片"
- "画一个..."

### 2. 提示词优化需求
- "帮我写一个提示词..."
- "如何描述...才能生成好的图像"
- "优化这个提示词..."
- "我想要...的提示词"

### 3. 具体场景
- **产品设计**: 图标、Logo、产品图
- **游戏开发**: 角色立绘、道具图标、场景概念图
- **概念艺术**: 角色设计、场景设计、氛围图
- **营销素材**: 海报、宣传图、社交媒体图片

### 4. 模型相关
- 用户提到 "ModelScope"、"FLUX"、"Qwen"、"SDXL"
- 用户询问不同模型的特点和选择

## 配置

### 首次使用设置

1. **复制配置文件**:
   ```bash
   cp config.json.example config.json
   ```

2. **获取 ModelScope API Key**:
   - 访问: https://modelscope.cn/my/myaccesstoken
   - 登录或注册账户
   - 生成 API Token

3. **编辑 config.json**:
   ```json
   {
     "platforms": {
       "modelscope": {
         "api_key": "YOUR_ACTUAL_API_KEY_HERE",
         "base_url": "https://api-inference.modelscope.cn/v1"
       }
     },
     "default_platform": "modelscope",
     "default_model": "flux-1"
   }
   ```

4. **安装依赖**:
   ```bash
   pip install requests Pillow
   ```

## 工作流程

### 工作流程 1: 直接生成图像

```
用户请求 → 选择模型 → 优化提示词 → 调用脚本 → 返回结果
```

**步骤**:

1. **理解用户需求**
   - 确定用途（图标、角色、概念艺术等）
   - 确定语言（中文/英文）
   - 确定质量要求（快速/高质量）

2. **选择合适模型**
   - 参考 `references/models/overview.md` 进行选择
   - 风景/肖像 → FLUX.1
   - 二次元/动漫 → ChenkinNoob-XL
   - 文字设计 → Qwen-Image
   - 中文提示词 → Qwen-Image

3. **优化提示词**
   - 读取对应模型的提示词指南（`references/prompts/[model].md`）
   - 根据指南优化用户的原始描述
   - 添加风格、细节、质量关键词

4. **调用生成脚本**
   ```bash
   python scripts/generate_image_modelscope.py \
     --prompt "优化后的提示词" \
     --model flux-1 \
     --output output/image.jpg \
     --config config.json
   ```

5. **解析 JSON 输出**
   - 成功: 提取 `image_url` 和 `local_path`
   - 失败: 提取 `error_code` 和 `message`，向用户解释

6. **向用户展示结果**
   - 显示图片 URL
   - 显示本地保存路径
   - 说明使用的模型和生成时间
   - 如果失败，提供清晰的错误说明和解决建议

### 工作流程 2: 仅优化提示词

```
用户请求 → 选择目标模型 → 读取提示词指南 → 优化提示词 → 返回给用户
```

**步骤**:

1. **确认用户意图**
   - 用户只想要提示词，不需要实际生成
   - 或者用户想先看提示词再决定是否生成

2. **选择目标模型**
   - 询问用户打算用哪个平台/模型
   - 如果未指定，根据用途推荐

3. **读取提示词指南**
   - 读取 `references/prompts/[model].md`
   - 了解该模型的特点和最佳实践

4. **优化提示词**
   - 根据指南中的模板和示例
   - 将用户的简单描述扩展为详细提示词
   - 添加适合该模型的关键词

5. **向用户展示**
   - 展示优化后的提示词
   - 解释为什么这样优化
   - 询问是否需要进一步调整或直接生成

## 模型选择指南

### 快速决策

**使用 FLUX.1 当**:
- 生成风景摄影风格图像
- 生成人物肖像
- 需要写实风格
- 需要快速生成（~6秒）

**使用 Qwen-Image 当**:
- 需要文字渲染（海报、标语、Logo）
- 图像中包含中文字或英文字
- 用户使用中文描述
- 中文文化元素

**使用 ChenkinNoob-XL 当**:
- 生成二次元角色
- 生成动漫风格插画
- 游戏角色立绘（动漫风格）
- Anime/Manga 风格作品

### 详细对比

参考 `references/models/overview.md` 查看完整的模型对比表（基于实际测试数据）和选择建议。

## 使用脚本

### ModelScope 平台脚本

**脚本**: `scripts/generate_image_modelscope.py`

**基本用法**:
```bash
python scripts/generate_image_modelscope.py --prompt "一只金色的猫" --output cat.jpg
```

**完整参数**:
```bash
python scripts/generate_image_modelscope.py \
  --prompt "提示词" \
  --model flux-1 \
  --output output/image.jpg \
  --config config.json \
  --width 1280 \
  --height 1280 \
  --steps 30
```

**参数说明**:
- `--prompt`: 图像描述（必需）
- `--model`: 模型 ID，可选值: `flux-1`, `qwen-image`, `chenkinnoob-xl`（默认: flux-1）
- `--output`: 输出文件路径（默认: generated_image.jpg）
- `--config`: 配置文件路径（默认: ../config.json）
- `--width`: 图像宽度（可选，默认使用模型默认值）
- `--height`: 图像高度（可选，默认使用模型默认值）
- `--steps`: 推理步数（可选，默认使用模型默认值）

**输出格式**:

成功时（stdout）:
```json
{
  "status": "success",
  "task_id": "xxx",
  "model": "MusePublic/489_ckpt_FLUX_1",
  "prompt": "a cute cat",
  "image_url": "https://...",
  "local_path": "cat.jpg",
  "generation_time_seconds": 6.5
}
```

失败时（stderr）:
```json
{
  "status": "error",
  "error_code": "AUTH_FAILED",
  "message": "Invalid API key. Please check config.json"
}
```

## 提示词优化指南

### 通用原则

1. **明确主体**: 清楚说明要生成什么
2. **添加风格**: 指定艺术风格或视觉风格
3. **描述细节**: 添加颜色、材质、光照等细节
4. **指定背景**: 说明背景类型（白色、透明、场景等）
5. **质量关键词**: 适当添加质量提升关键词

### 模型特定指南

#### FLUX.1
- **参考**: `references/prompts/flux-1.md`
- **语言**: 英文
- **风格**: 自然英文描述，30-80 词
- **示例**: "a cute orange cat sitting on a sunny windowsill, soft natural lighting, cozy atmosphere"

#### Qwen-Image
- **参考**: `references/prompts/qwen-image.md`
- **语言**: 中文优先
- **风格**: 简洁明了，30-60 字
- **示例**: "一只毛茸茸的橘猫，绿色眼睛，坐在温馨的窗台上，柔和的自然光"

#### ChenkinNoob-XL
- **参考**: `references/prompts/chenkinnoob-xl.md`
- **语言**: 英文
- **风格**: 详细描述，50-150 词
- **示例**: "fantasy warrior character, detailed plate armor with gold trim, heroic standing pose, dramatic lighting, castle ruins background, highly detailed, artstation quality, 8k"

### 场景化模板

参考各模型的提示词指南文件，其中包含：
- 产品图标模板
- 游戏角色模板
- 概念艺术模板
- 产品摄影模板

## 错误处理

### 常见错误及解决方案

#### AUTH_FAILED
**原因**: API Key 无效或未配置
**解决**:
1. 检查 config.json 中的 api_key
2. 确保不是 "YOUR_MODELSCOPE_API_KEY"
3. 访问 https://modelscope.cn/my/myaccesstoken 重新生成

#### RATE_LIMIT
**原因**: 超过免费账户限额（~50-100次/小时）
**解决**:
1. 等待一段时间后重试
2. 减少请求频率
3. 考虑升级付费账户

#### TIMEOUT
**原因**: 任务超时（超过 120 秒）
**解决**:
1. 简化提示词
2. 使用更快的模型（FLUX.1，仅需~6秒）
3. 稍后重试

#### CONFIG_ERROR
**原因**: 配置文件格式错误或缺失
**解决**:
1. 检查 config.json 格式是否正确
2. 确保文件存在于正确位置
3. 参考 config.json.example

#### MODEL_NOT_FOUND
**原因**: 指定的模型在配置中不存在
**解决**:
1. 检查 --model 参数拼写
2. 查看 config.json 中的 models 部分
3. 使用 `flux-1`, `qwen-image`, 或 `chenkinnoob-xl`

## 扩展性

### 添加新平台

本技能设计为可扩展架构，可以轻松添加新的图像生成平台（如 OpenAI DALL-E、Stability AI 等）。

**详细指南**: 参见 `references/integration/new_platform_guide.md`

**步骤概览**:
1. 在 config.json 添加平台配置
2. 创建平台专用脚本（如 `generate_image_openai.py`）
3. 添加平台文档（`references/models/[platform].md`）
4. 为每个模型添加提示词指南（`references/prompts/[model].md`）
5. 更新此 SKILL.md

### 添加新模型

在现有平台添加新模型：
1. 在 config.json 的 models 部分添加模型配置
2. 创建提示词指南（`references/prompts/[model].md`）
3. 更新 `references/models/overview.md` 的对比表

## 参考文档

### 模型介绍层
- `references/models/overview.md` - 模型选择指南和对比
- `references/models/modelscope.md` - ModelScope 平台详细介绍

### 提示词指南层
- `references/prompts/z-image-turbo.md` - Z-Image-Turbo 提示词最佳实践
- `references/prompts/sdxl-base.md` - SDXL-Base 提示词最佳实践

### 扩展指南层
- `references/integration/new_platform_guide.md` - 添加新平台的完整指南

## 使用示例

### 示例 1: 生成产品图标（中文）

**用户**: "帮我生成一个手机App图标，蓝色系，简约风格"

**Claude 操作**:
1. 选择模型: Qwen-Image（中文提示词）
2. 读取: `references/prompts/qwen-image.md`
3. 优化提示词: "手机App图标，简约扁平风格，蓝色渐变背景，白色图形，圆角方形"
4. 调用脚本:
   ```bash
   python scripts/generate_image_modelscope.py \
     --prompt "手机App图标，简约扁平风格，蓝色渐变背景，白色图形，圆角方形" \
     --model qwen-image \
     --output icon.jpg
   ```
5. 解析 JSON，向用户展示结果

### 示例 2: 生成游戏角色（英文，高质量）

**用户**: "I need a fantasy warrior character for my game"

**Claude 操作**:
1. 选择模型: ChenkinNoob-XL（高质量需求）
2. 读取: `references/prompts/chenkinnoob-xl.md`
3. 优化提示词: "fantasy warrior character, detailed plate armor with gold trim, heroic standing pose with sword raised, dramatic lighting from above, castle ruins background with stormy sky, highly detailed, artstation quality, 8k"
4. 调用脚本:
   ```bash
   python scripts/generate_image_modelscope.py \
     --prompt "fantasy warrior character, detailed plate armor with gold trim, heroic standing pose with sword raised, dramatic lighting from above, castle ruins background with stormy sky, highly detailed, artstation quality, 8k" \
     --model chenkinnoob-xl \
     --output warrior.jpg
   ```
5. 解析 JSON，向用户展示结果

### 示例 3: 快速生成（英文）

**用户**: "Generate a cute cat quickly"

**Claude 操作**:
1. 选择模型: FLUX.1（速度最快）
2. 读取: `references/prompts/flux-1.md`
3. 优化提示词: "a cute fluffy orange cat with green eyes, sitting on a cozy windowsill, soft natural lighting, peaceful atmosphere"
4. 调用脚本:
   ```bash
   python scripts/generate_image_modelscope.py \
     --prompt "a cute fluffy orange cat with green eyes, sitting on a cozy windowsill, soft natural lighting, peaceful atmosphere" \
     --model flux-1 \
     --output cat.jpg
   ```
5. 解析 JSON，向用户展示结果（约6秒完成）

### 示例 4: 仅优化提示词

**用户**: "帮我写一个生成游戏道具图标的提示词，我要用 Qwen-Image"

**Claude 操作**:
1. 确认: 用户只需要提示词，不需要实际生成
2. 读取: `references/prompts/qwen-image.md`
3. 根据"产品图标"模板优化
4. 向用户展示:
   ```
   优化后的提示词：
   "游戏道具图标，金色宝剑，奇幻风格，金色发光效果，透明背景，精致"

   这个提示词：
   - 明确了主体（金色宝剑）
   - 指定了风格（奇幻）
   - 添加了视觉效果（金色发光）
   - 指定了背景（透明）
   - 添加了质量词（精致）

   您可以直接使用这个提示词，或者告诉我需要调整的地方。
   需要我帮您生成图像吗？
   ```

## 最佳实践

### 1. 模型选择
- 优先考虑内容类型（风景肖像→FLUX.1，二次元→ChenkinNoob-XL，文字→Qwen-Image）
- 考虑语言（中文→Qwen-Image）
- 考虑风格（写实→FLUX.1，动漫→ChenkinNoob-XL）
- 考虑时间要求（快速→FLUX.1，不急→ChenkinNoob-XL）

### 2. 提示词优化
- 始终读取对应模型的提示词指南
- 根据场景使用合适的模板
- 保持提示词清晰、结构化
- 避免冲突的描述

### 3. 错误处理
- 解析 JSON 输出的 error_code
- 向用户提供清晰的错误说明
- 提供具体的解决步骤
- 必要时建议替代方案

### 4. 用户体验
- 生成前向用户说明选择的模型和原因
- 生成后展示图片 URL 和本地路径
- 如果失败，不要只是转发错误，要解释和建议
- 询问用户是否需要调整或重试

## 故障排查

### 脚本无法运行
1. 检查 Python 版本（建议 3.7+）
2. 安装依赖: `pip install requests Pillow`
3. 检查脚本权限

### 配置问题
1. 确保 config.json 存在
2. 检查 JSON 格式是否正确
3. 确认 API Key 已正确填写

### API 问题
1. 测试 API Key 是否有效
2. 检查网络连接
3. 查看 ModelScope 服务状态

### 生成质量问题
1. 尝试优化提示词
2. 增加推理步数
3. 尝试不同模型
4. 参考提示词指南中的示例

## 限制说明

### 当前限制
- 仅支持 ModelScope 平台
- 免费账户有速率限制（~50-100次/小时）
- 图像尺寸最大 2048x2048
- 不支持图生图（image-to-image）

### 未来计划
- 添加 OpenAI DALL-E 支持
- 添加 Stability AI 支持
- 支持图生图功能
- 支持批量生成
- 本地模型支持（ComfyUI、Automatic1111）

## 许可和使用条款

- 本技能代码遵循 Apache License 2.0
- 生成的图像版权取决于使用的平台和模型
- ModelScope 平台的使用需遵守其服务条款
- 请负责任地使用 AI 图像生成技术

## 反馈和贡献

如果您在使用过程中遇到问题或有改进建议，欢迎：
- 报告问题
- 贡献新的平台支持
- 改进提示词指南
- 分享使用经验

---

**版本**: 1.0.0
**最后更新**: 2025-12-12
**维护者**: Claude Code Community
