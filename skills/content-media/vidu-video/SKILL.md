---
name: vidu-video
description: 使用 Vidu Q3 Pro 模型生成视频。当用户想要文生视频、生成带音频的视频，或提到 vidu 时使用此 skill。
category: video
tags: [video, text-to-video, vidu, audio]
---

# Vidu Q3 视频生成

Vidu 最新 Q3 Pro 文生视频模型，支持多种分辨率（360p~1080p），可自定义时长和宽高比，支持直接生成带音频的视频。

## 可用模型

| 模型 ID | 功能 | 说明 |
|--------|------|------|
| `fal-ai/vidu/q3/text-to-video` | 文生视频 | 从文字描述生成视频，支持带音频输出 |

## 定价

按 **视频秒数 × 分辨率** 计费：

| 分辨率 | 价格（积分/秒） | 5秒视频 | 10秒视频 |
|--------|---------------|---------|---------|
| 360p / 540p | 28 积分/秒 | 140 积分 | 280 积分 |
| 720p / 1080p | 62 积分/秒 | 310 积分 | 620 积分 |

## 工作流

### 1. 调用 submit_task

使用 MCP 工具 `submit_task` 提交任务：

```json
{
  "model_id": "fal-ai/vidu/q3/text-to-video",
  "parameters": {
    "prompt": "In an ultra-realistic fashion photography style featuring light blue and pale amber tones, an astronaut in a spacesuit walks through the fog.",
    "duration": 5,
    "resolution": "720p",
    "aspect_ratio": "16:9",
    "audio": true
  }
}
```

### 参数说明

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|-----|------|-----|-------|------|
| prompt | string | **是** | - | 视频生成提示词（最大 2000 字符） |
| duration | integer | 否 | 5 | 视频时长（秒），1-16 |
| resolution | string | 否 | "720p" | 分辨率：360p / 540p / 720p / 1080p |
| aspect_ratio | string | 否 | "16:9" | 宽高比：16:9 / 9:16 / 4:3 / 3:4 / 1:1 |
| audio | boolean | 否 | true | 是否生成带音频的视频 |
| seed | integer | 否 | - | 随机种子，用于复现结果 |

## 查询任务状态

提交任务后会返回 `task_id`，使用 `get_task` 查询结果：

```json
{
  "task_id": "返回的任务ID"
}
```

任务状态：
- `pending` - 排队中
- `processing` - 处理中
- `completed` - 完成，结果在 `result` 中
- `failed` - 失败，查看 `error` 字段

## 完整示例

**用户请求**：生成一段宇航员在雾中行走的视频

**执行步骤**：

1. 调用 `submit_task`：
```json
{
  "model_id": "fal-ai/vidu/q3/text-to-video",
  "parameters": {
    "prompt": "In an ultra-realistic fashion photography style featuring light blue and pale amber tones, an astronaut in a spacesuit walks through the fog.",
    "duration": 5,
    "resolution": "720p",
    "aspect_ratio": "16:9"
  }
}
```

2. 获取 `task_id` 后调用 `get_task` 查询结果

## 提示词技巧

1. **描述清晰**：详细描述场景、动作、光线和色调
2. **英文优先**：使用英文提示词通常效果更好
3. **风格关键词**：加入 "cinematic"、"ultra-realistic"、"photography style" 等风格词
4. **分辨率选择**：360p/540p 成本低适合测试，720p/1080p 适合正式输出
5. **音频功能**：默认开启 audio=true，生成的视频会自带声音

## 注意事项

- 720p 和 1080p 分辨率价格是 360p/540p 的 2.2 倍
- 视频时长支持 1-16 秒灵活选择
- 支持多种宽高比，适合不同平台的内容需求
