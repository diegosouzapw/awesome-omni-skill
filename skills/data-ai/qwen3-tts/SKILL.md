---
name: qwen3-tts
description: Qwen3-TTS 文本转语音技能，支持多语言语音合成、语音克隆和语音设计。当用户需要生成语音、转换文本为语音、或使用 TTS 功能时使用此技能。
allowed-tools: Bash(~/.local/bin/mlx_audio.tts.generate, uvx mlx_audio.tts.generate)
---

# Qwen3-TTS 文本转语音

基于 MLX 框架的本地 TTS（文本转语音）解决方案，专为 Apple Silicon M 系列芯片优化。

## 快速开始

```bash
# 安装（仅需一次）
uv tool install --force mlx-audio --prerelease=allow

# 基础使用
~/.local/bin/mlx_audio.tts.generate \
  --model mlx-community/Qwen3-TTS-12Hz-1.7B-CustomVoice-bf16 \
  --text "你好，欢迎体验 Qwen3-TTS！" \
  --voice "serena" \
  --play
```

## 使用已克隆的音色

当用户已经保存了克隆语音配置时（在 `~/.qwen3_tts_voices/` 目录下），可以直接使用保存的语音生成音频：

### 检查是否有保存的克隆语音

```bash
# 列出所有保存的语音
ls -1 ~/.qwen3_tts_voices/
```

### 使用保存的克隆语音（推荐方式）

```bash
# 使用 voice-clone.sh 脚本（最简单）
~/.claude/skills/qwen3-tts/templates/voice-clone.sh use my_voice "你好，这是测试。" --play
```

### 直接使用命令

```bash
# 读取配置文件获取参数
CONFIG_FILE="$HOME/.qwen3_tts_voices/my_voice/config.json"

# 如果配置文件存在，直接使用
if [ -f "$CONFIG_FILE" ]; then
  ~/.local/bin/mlx_audio.tts.generate \
    --model "mlx-community/Qwen3-TTS-12Hz-0.6B-Base-bf16" \
    --text "你好，这是测试。" \
    --ref_audio "$HOME/.qwen3_tts_voices/my_voice/reference.wav" \
    --ref_text "你好，这是我的声音克隆测试。今天天气真不错，我很高兴能体验这个功能。" \
    --lang_code "zh" \
    --play
fi
```

### 技能调用流程

当用户要求"使用我的音色"或类似请求时：

1. **检查保存的语音**：`ls ~/.qwen3_tts_voices/`
2. **读取配置**：`cat ~/.qwen3_tts_voices/my_voice/config.json`
3. **生成语音**：使用 `--ref_audio` 和 `--ref_text` 参数

示例对话：
```
用户：用我的声音说"你好，世界"
AI：让我使用你保存的克隆语音来生成...
    [执行 voice-clone.sh use my_voice "你好，世界" --play]
```

## 可用模型

| 模型 | 大小 | 特点 | 用途 |
|------|------|------|------|
| `Qwen3-TTS-12Hz-0.6B-Base-bf16` | 0.6B | 基础模型 | **语音克隆**（推荐） |
| `Qwen3-TTS-12Hz-0.6B-CustomVoice-bf16` | 0.6B | 9 个预设语音 | 使用预设语音生成 |
| `Qwen3-TTS-12Hz-1.7B-CustomVoice-bf16` | 1.7B | 9 个预设语音 | 更高质量的预设语音 |
| `Qwen3-TTS-12Hz-1.7B-VoiceDesign-bf16` | 1.7B | 语音设计 | 创建自定义语音描述 |

## CustomVoice 模型可用语音

### 中文语音
- `serena` - 温柔女声（推荐中文）
- `vivian` - 知性女声
- `ryan` - 成熟男声

### 英文语音
- `aiden` - 青年男声
- `ono_anna` - 日式女声
- `sohee` - 韩式女声
- `eric` - 欧美男声
- `dylan` - 青少年男声
- `uncle_fu` - 特色男声

## 基础用法

### 生成并播放语音
```bash
~/.local/bin/mlx_audio.tts.generate \
  --model mlx-community/Qwen3-TTS-12Hz-1.7B-CustomVoice-bf16 \
  --text "你好，这是一段测试语音。" \
  --voice "serena" \
  --play
```

### 保存语音文件
```bash
~/.local/bin/mlx_audio.tts.generate \
  --model mlx-community/Qwen3-TTS-12Hz-1.7B-CustomVoice-bf16 \
  --text "你好，这是保存的音频。" \
  --voice "vivian" \
  --file_prefix my_audio \
  --audio_format wav
```

## 高级用法

### 调整语速
```bash
~/.local/bin/mlx_audio.tts.generate \
  --model mlx-community/Qwen3-TTS-12Hz-1.7B-CustomVoice-bf16 \
  --text "这是加快语速的示例。" \
  --voice "serena" \
  --speed 1.5
```

### 使用温度控制语音多样性
```bash
~/.local/bin/mlx_audio.tts.generate \
  --model mlx-community/Qwen3-TTS-12Hz-1.7B-CustomVoice-bf16 \
  --text "同样的文本，不同的语气。" \
  --voice "serena" \
  --temperature 0.8
```

### 批量生成长文本
```bash
~/.local/bin/mlx_audio.tts.generate \
  --model mlx-community/Qwen3-TTS-12Hz-1.7B-CustomVoice-bf16 \
  --text "第一段。第二段。第三段。" \
  --voice "serena" \
  --split_pattern "。" \
  --join_audio
```

### 详细输出信息
```bash
~/.local/bin/mlx_audio.tts.generate \
  --model mlx-community/Qwen3-TTS-12Hz-1.7B-CustomVoice-bf16 \
  --text "显示详细生成信息。" \
  --voice "serena" \
  --verbose
```

## 语音克隆

### 重要：语音克隆必须使用 Base 模型

**语音克隆应该使用 Base 模型，而不是 CustomVoice 模型！**

Base 模型支持从参考音频克隆任意声音，而 CustomVoice 模型只支持预设语音。

### 基础克隆（正确方式）

```bash
# 使用 Base 模型进行语音克隆
~/.local/bin/mlx_audio.tts.generate \
  --model mlx-community/Qwen3-TTS-12Hz-0.6B-Base-bf16 \
  --text "这是克隆的语音。" \
  --ref_audio /path/to/reference.wav \
  --ref_text "参考音频的文本内容"
```

**注意**：
- 使用 `Base` 模型，不是 `CustomVoice`
- **不需要** `--voice` 参数（让模型完全从参考音频克隆）
- 只需要 `--ref_audio` 和 `--ref_text`

### 参考音频要求

- **格式**: WAV, MP3, FLAC, M4A 等常见格式
- **时长**: 建议 3-10 秒
- **内容**: 清晰的语音，无背景噪音
- **文本**: `--ref_text` 必须与参考音频的内容完全匹配

### 引导用户录音流程

由于命令行录音需要用户确认，推荐使用以下步骤引导用户录音：

```bash
# 步骤 1: 引导用户使用 macOS 录音应用
echo "请按以下步骤录音："
echo "1. 打开「语音备忘录」应用"
echo "2. 点击红色录音按钮"
echo "3. 清晰朗读以下文本："
echo '   "你好，这是我的声音克隆测试。今天天气真不错，我很高兴能体验这个功能。"'
echo "4. 点击停止按钮"
echo "5. 将录音共享到桌面，命名为 'voice_reference.m4a'"
echo ""
echo "完成后告诉我，继续下一步..."

# 步骤 2: 查找桌面上的录音文件
# 用户录音完成后，查找最新的 m4a 文件
LATEST_AUDIO=$(ls -t ~/Desktop/*.m4a 2>/dev/null | head -1)

# 步骤 3: 转换音频格式（如果需要）
if [ -f "$LATEST_AUDIO" ]; then
  ffmpeg -i "$LATEST_AUDIO" -ar 24000 -ac 1 -acodec pcm_s16le ~/reference.wav -y
  echo "音频已转换: ~/reference.wav"
fi

# 步骤 4: 使用 Base 模型进行语音克隆
~/.local/bin/mlx_audio.tts.generate \
  --model mlx-community/Qwen3-TTS-12Hz-0.6B-Base-bf16 \
  --text "你好，这是测试语音克隆效果。" \
  --ref_audio ~/reference.wav \
  --ref_text "你好，这是我的声音克隆测试。今天天气真不错，我很高兴能体验这个功能。" \
  --play
```

### 保存和复用克隆语音

使用 `templates/voice-clone.sh` 脚本可以保存克隆语音配置：

```bash
# 首次克隆并保存配置
./templates/voice-clone.sh save my_voice \
  --ref_audio ~/reference.wav \
  --ref_text "你好，这是我的声音克隆测试。今天天气真不错，我很高兴能体验这个功能。"

# 之后直接使用保存的语音
./templates/voice-clone.sh use my_voice "这是要生成的新文本" --play
```

语音配置保存在 `~/.qwen3_tts_voices/` 目录，包含：
- `config.json` - 语音配置（参考音频路径、文本等）
- 参考音频文件的副本

**完整工作流程示例**：
```bash
# 1. 引导用户录音（使用语音备忘录）
# 见上面的"引导用户录音流程"

# 2. 保存为克隆语音
./templates/voice-clone.sh save my_custom_voice \
  --ref_audio ~/reference.wav \
  --ref_text "你好，这是我的声音"

# 3. 使用克隆语音生成任意文本
./templates/voice-clone.sh use my_custom_voice "今天天气真好" --play

# 4. 查看所有保存的语音
./templates/voice-clone.sh list
```

### 批量克隆管理

```bash
# 列出所有保存的克隆语音
./templates/voice-clone.sh list

# 删除某个克隆语音
./templates/voice-clone.sh delete my_voice

# 更新克隆语音的参考音频
./templates/voice-clone.sh update my_voice --ref_audio ~/new_reference.wav
```

## 参数说明

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--model` | 模型路径或 HuggingFace ID | `mlx-community/Kokoro-82M-bf16` |
| `--text` | 要转换的文本 | 从 stdin 读取 |
| `--voice` | 语音名称（仅 CustomVoice 模型需要） | 模型默认 |
| `--speed` | 播放速度 | 1.0 |
| `--temperature` | 采样温度 (0.1-1.5) | 0.7 |
| `--top_p` | Top-p 采样 | 0.9 |
| `--top_k` | Top-k 采样 | 50 |
| `--max_tokens` | 最大生成 token 数 | 1200 |
| `--file_prefix` | 输出文件前缀 | `audio` |
| `--audio_format` | 音频格式 (wav/flac/mp3) | `wav` |
| `--play` | 播放生成的音频 | false |
| `--join_audio` | 合并多个音频段 | false |
| `--split_pattern` | 分段模式 | `\n` |
| `--verbose` | 显示详细信息 | false |
| `--ref_audio` | 参考音频路径（语音克隆必需） | null |
| `--ref_text` | 参考音频文本（语音克隆必需） | null |
| `--lang_code` | 语言代码（如 zh, en） | 自动检测 |

## 输出格式

### 控制台输出
```
Text: 你好，欢迎体验 Qwen3-TTS！
Voice: None  # Base 模型显示 None
Speed: 1.0x
Language: zh
==========
Duration:              00:00:08.123
Samples/sec:           24000.0
Prompt:                245 tokens, 30.12 tokens-per-sec
Audio:                 194952 samples, 24000.0 samples-per-sec
Real-time factor:      0.42x
Processing time:       19.32s
Peak memory usage:     2.15GB
✅ Audio successfully generated and saving as: audio_000.wav
```

### 文件输出
- 默认输出为 WAV 格式
- 采样率：24000 Hz
- 文件命名：`{file_prefix}_{索引:03d}.{audio_format}`

## 使用 uvx（无需安装）

```bash
uvx mlx_audio.tts.generate \
  --model mlx-community/Qwen3-TTS-12Hz-0.6B-Base-bf16 \
  --text "使用 uvx 运行" \
  --ref_audio reference.wav \
  --ref_text "参考文本"
```

## 常见问题

### 1. 语音克隆不像我的声音
- 确保使用 **Base 模型**，不是 CustomVoice 模型
- 确保 `--ref_text` 与参考音频内容完全匹配
- 参考音频应清晰、无背景噪音
- 尝试使用更长的参考音频（5-10 秒）

### 2. 音频无法播放
确保已安装音频驱动：
```bash
# macOS 自带 afplay
afplay audio_000.wav

# 或使用 ffplay（需要安装 ffmpeg）
brew install ffmpeg
ffplay audio_000.wav
```

### 3. 内存不足
- 使用较小的模型（0.6B）
- 减少 `--max_tokens` 值
- 关闭其他应用释放内存

### 4. 中文发音不准
- 使用 `--lang_code zh` 参数
- 调整 `--temperature` 参数
- 使用 `--speed` 调整语速

### 5. 生成速度慢
- 正常现象，实时因子约为 0.3-0.5x
- 即生成 10 秒音频需要 20-30 秒

## 实用脚本

### Ready-to-use 模板脚本

`templates/` 目录提供了多个开箱即用的脚本：

| 脚本 | 描述 |
|------|------|
| `simple-tts.sh` | 简单语音生成，适合快速测试 |
| `batch-tts.sh` | 批量生成多种语音的相同文本 |
| `file-to-tts.sh` | 从文本文件读取并生成语音 |
| `voice-clone.sh` | **语音克隆管理**（保存、使用、列表、删除） |
| `record-reference.sh` | **录制参考音频**用于语音克隆 |

使用示例：
```bash
# 快速生成语音
./templates/simple-tts.sh "你好，世界" "serena"

# 录制参考音频
./templates/record-reference.sh my_voice.wav 5

# 保存并使用克隆语音
./templates/voice-clone.sh save my_voice \
  --ref_audio my_voice.wav \
  --ref_text "你好，世界"

./templates/voice-clone.sh use my_voice "新的文本内容" --play
```

### 批量生成多个语音
```bash
#!/bin/bash
TEXT="你好，这是测试文本。"

for voice in serena vivian ryan; do
  ~/.local/bin/mlx_audio.tts.generate \
    --model mlx-community/Qwen3-TTS-12Hz-1.7B-CustomVoice-bf16 \
    --text "$TEXT" \
    --voice "$voice" \
    --file_prefix "output_${voice}" \
    --audio_format wav
done
```

### 从文件读取文本
```bash
~/.local/bin/mlx_audio.tts.generate \
  --model mlx-community/Qwen3-TTS-12Hz-1.7B-CustomVoice-bf16 \
  --voice "serena" \
  --play \
  < input.txt
```

### 流式生成（边生成边播放）
```bash
~/.local/bin/mlx_audio.tts.generate \
  --model mlx-community/Qwen3-TTS-12Hz-1.7B-CustomVoice-bf16 \
  --text "这是一段很长的文本..." \
  --voice "serena" \
  --stream \
  --streaming_interval 2.0
```

## Python API 用法

```python
from mlx_audio.tts.utils import load_model
import sounddevice as sd
import numpy as np

# 加载模型
model = load_model("mlx-community/Qwen3-TTS-12Hz-0.6B-Base-bf16")

# 语音克隆
text = "你好，这是测试语音克隆效果。"
results = list(model.generate(
    text=text,
    ref_audio="reference.wav",
    ref_text="你好，这是我的声音克隆测试。今天天气真不错，我很高兴能体验这个功能。",
    verbose=True
))

# 播放音频
audio = results[0].audio
sample_rate = results[0].sample_rate
sd.play(np.array(audio), sample_rate)
sd.wait()
```

## 技术规格

- **框架**: MLX (Apple Silicon 优化)
- **采样率**: 24000 Hz
- **音频编码**: SNAC (SoundStream)
- **基础模型**: Qwen3 (12Hz)
- **支持语言**: 中文、英文、日文、韩文等多语言
- **硬件要求**: Apple Silicon (M1/M2/M3/M4)
- **Python 要求**: 3.10+

## 参考资源

- [MLX-Audio GitHub](https://github.com/Blaizzy/mlx-audio)
- [Qwen3-TTS 模型页面](https://huggingface.co/Qwen/Qwen3-TTS-12Hz-1.7B-CustomVoice)
- [MLX 官方文档](https://ml-explore.github.io/mlx/)
- [mlx-community Qwen3-TTS Collection](https://huggingface.co/collections/mlx-community/qwen3-tts)
