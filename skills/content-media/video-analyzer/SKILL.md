---
name: video-analyzer
version: 1.0.10
description: 鏅鸿兘鍒嗘瀽 Bilibili/YouTube/鏈湴瑙嗛锛岀敓鎴愯浆鍐欍€佽瘎浼板拰鎬荤粨銆傛敮鎸佸叧閿抚鎴浘鑷姩宓屽叆銆?
author: user
tags: [video, transcription, analysis, bilibili, youtube, ai]
---

# Video Analyzer Skill

鏅鸿兘鍒嗘瀽 Bilibili銆乊ouTube 鎴栨湰鍦拌棰戯紝鐢熸垚杞啓銆佽瘎浼板拰鎬荤粨銆傛敮鎸佸叧閿抚鎴浘鑷姩宓屽叆銆?

## When to Use This Skill

褰撶敤鎴锋彁鍒颁互涓嬪唴瀹规椂婵€娲绘鎶€鑳斤細
- "鍒嗘瀽瑙嗛"
- "杞啓瑙嗛"
- "鎬荤粨瑙嗛鍐呭"
- "璇勪及杩欎釜瑙嗛"
- "瑙嗛鍐呭鍒嗘瀽"
- "鎻愬彇瑙嗛鏂囧瓧"

## How It Works

姝ゆ妧鑳戒細锛?
1. **涓嬭浇瑙嗛**锛氭敮鎸?Bilibili銆乊ouTube 鎴栨湰鍦版枃浠?
2. **璇煶杞啓**锛氫娇鐢?Whisper AI 妯″瀷杩涜楂樼簿搴﹁浆鍐?
3. **鍏抽敭甯ф彁鍙?*锛氭櫤鑳介€夋嫨鍏抽敭鑺傜偣骞舵彁鍙栨埅鍥撅紙榛樿鍚敤锛?
4. **鍐呭鍒嗘瀽**锛氫娇鐢?LLM 杩涜鍐呭璇勪及銆佹€荤粨銆佹牸寮忓寲
5. **缁撴灉淇濆瓨**锛氱敓鎴?Markdown 鏍煎紡鐨勫垎鏋愭姤鍛?

## Usage

### 鍩虹鐢ㄦ硶

```bash
python .claude/skills/video-analyzer/run.py --url "<VIDEO_URL>"
```

### 甯哥敤鍙傛暟

- `--url`: 瑙嗛閾炬帴鎴栨湰鍦版枃浠惰矾寰勶紙蹇呭～锛?
- `--whisper-model`: Whisper 妯″瀷锛堥粯璁? large-v2锛?
  - 鍙€? tiny, base, small, medium, large-v2, large-v3, turbo
- `--analysis-types`: 鍒嗘瀽绫诲瀷锛堥粯璁? evaluation,summary锛?
  - 鍙€? evaluation, summary, format
- `--output-dir`: 杈撳嚭鐩綍锛堥粯璁? ./video-analysis锛?
- `--summary-style`: 鎬荤粨椋庢牸锛堝彲閫? concise, deep, social, study锛?
- `--no-screenshots`: 绂佺敤鍏抽敭甯ф埅鍥?

### 绀轰緥

```bash
# 鍩虹鍒嗘瀽
python run.py --url "https://www.bilibili.com/video/BV1xx411c7mD"

# 蹇€熻浆鍐欙紙灏忔ā鍨嬶級
python run.py --url "https://youtu.be/xxx" --whisper-model small

# 鍙仛鎬荤粨
python run.py --url "./video.mp4" --analysis-types summary
```

## Configuration

棣栨浣跨敤闇€瑕侀厤缃?API key锛?

1. 澶嶅埗 `config.example.json` 涓?`config.json`
2. 濉叆浣犵殑 API key锛?

```json
{
  "llm": {
    "provider": "openai",
    "api_key": "your-api-key",
    "base_url": "https://api.openai.com/v1",
    "model": "gpt-4o-mini"
  }
}
```

鏀寔鐨勬彁渚涘晢锛?
- **OpenAI**: gpt-4o-mini, gpt-4o
- **Anthropic**: claude-3-5-sonnet-20241022
- **鍏煎 OpenAI API 鐨勬湇鍔?*: 濡?Gemini銆丏eepSeek 绛?

## Dependencies

### 绯荤粺渚濊禆
- **FFmpeg**锛堝繀闇€锛? 瑙嗛澶勭悊
  - Windows: `winget install ffmpeg`
  - macOS: `brew install ffmpeg`
  - Linux: `sudo apt install ffmpeg`

### Python 渚濊禆
棣栨杩愯浼氳嚜鍔ㄦ鏌ュ苟瀹夎锛?
- yt-dlp - 瑙嗛涓嬭浇
- faster-whisper - 璇煶杞啓
- openai / anthropic - LLM API
- 鍏朵粬渚濊禆瑙?`requirements.txt`

## Features

- 馃幀 澶氬钩鍙版敮鎸侊紙B绔欍€乊ouTube銆佹湰鍦版枃浠讹級
- 馃帳 楂樼簿搴﹁浆鍐欙紙Whisper AI锛?
- 馃 鏅鸿兘鍒嗘瀽锛堝唴瀹硅瘎浼般€佹€荤粨锛?
- 馃摳 鍏抽敭甯ф埅鍥撅紙鑷姩鎻愬彇骞跺祵鍏ワ級
- 馃寪 澶氳瑷€鏀寔锛堜腑鑻辨枃绛夛級

## Troubleshooting

**Q: 鎻愮ず缂哄皯 FFmpeg**
A: 瀹夎 FFmpeg锛堣涓婃柟绯荤粺渚濊禆锛?

**Q: API 璋冪敤澶辫触**
A: 妫€鏌?`config.json` 涓殑 API key 鏄惁姝ｇ‘

**Q: 瑙嗛涓嬭浇澶辫触**
A: 妫€鏌ョ綉缁滆繛鎺ワ紝鎴栦娇鐢ㄦ湰鍦拌棰戞枃浠?

**Q: 鎴浘鍔熻兘涓嶅伐浣?*
A: 纭繚 FFmpeg 宸插畨瑁咃紝涓斾娇鐢ㄧ殑鏄棰?URL锛堜笉鏄函闊抽锛?

## Notes

- 棣栨杩愯浼氫笅杞?Whisper 妯″瀷锛堢害 3GB锛屼娇鐢ㄥ浗鍐呴暅鍍忥級
- 瑙嗛涓存椂涓嬭浇鍒扮郴缁熶复鏃剁洰褰曪紝鍒嗘瀽瀹屾垚鍚庤嚜鍔ㄦ竻鐞?
- 澶фā鍨嬶紙large-v2锛夌簿搴﹂珮浣嗛€熷害鎱紝灏忔ā鍨嬶紙small锛夐€熷害蹇絾绮惧害杈冧綆
- 鍚敤鎴浘闇€瑕佷笅杞藉畬鏁磋棰戯紙鑰岄潪浠呴煶棰戯級

## Code-level Feishu Publishing (Required)

Do not store Feishu app credentials in this skill config.
Publishing is handled by built-in Python flow (`feishu_publisher.py`) after analysis.

After analysis succeeds:
1. Use `video_title` as Feishu doc title exactly.
2. Merge all generated content into one markdown body (summary + evaluation + transcript).
3. Create a wiki docx node under configured `space_id` + `parent_node_token`.
4. Write the full markdown body into the created doc token.
5. Return publish result (`doc_token`/`doc_url`) in output field `feishu_publish`.

Credential/target resolution priority:
- `feishu_space_id` / `feishu_parent_node_token` parameters
- `FEISHU_SPACE_ID` / `FEISHU_PARENT_NODE_TOKEN` env
- `config.json` -> `feishu.space_id` / `feishu.parent_node_token`

App credentials are loaded from:
- `FEISHU_APP_ID` / `FEISHU_APP_SECRET` env, or
- OpenClaw `openclaw.json` -> `channels.feishu`

If publish fails, keep analysis result and return `feishu_publish.success=false` with error details.
