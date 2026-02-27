---
name: smart-short-video
description: 智能短影片生成器 - 混合 AI 圖片與原始影片片段
metadata:
  tags: [video, short-form, remix, ai-images, video-segments]
questions:
  - id: video_source
    text: 輸入影片路徑
    type: file
    required: true
  - id: target_duration
    text: 目標影片時長
    type: choice
    options:
      - { label: "30 秒", value: 30 }
      - { label: "60 秒", value: 60 }
      - { label: "90 秒", value: 90 }
      - { label: "120 秒", value: 120 }
    required: true
  - id: library_ratio
    text: 圖庫圖片比例
    type: choice
    options:
      - { label: "0%", value: 0 }
      - { label: "25%", value: 25 }
      - { label: "50%", value: 50 }
      - { label: "75%", value: 75 }
    required: true
  - id: ai_ratio
    text: AI 生圖比例
    type: choice
    options:
      - { label: "0%", value: 0 }
      - { label: "25%", value: 25 }
      - { label: "50%", value: 50 }
      - { label: "75%", value: 75 }
    required: true
  - id: ai_image_service
    text: AI 生圖服務
    type: choice
    options:
      - { label: "Pollinations.ai (免費)", value: pollinations }
      - { label: "DALL-E (需 OpenAI Key)", value: dalle }
      - { label: "GLM 智譜 AI", value: glm }
    required: true
  - id: api_key
    text: API Key (若選 Pollinations 可跳過)
    type: text
    required: false
    placeholder: 請輸入 OpenAI API Key 或 GLM API Key
  - id: transcription_method
    text: 轉錄方式
    type: choice
    options:
      - { label: "本地 Whisper", value: local }
      - { label: "OpenAI API", value: openai }
      - { label: "使用現有結果", value: existing }
    required: false
---

# Smart Short Video - 智能短影片生成器

**混合 AI 圖片與原始影片片段，生成 TikTok/Reels/Shorts 短影片**

---

## 使用者輸入

| 參數 | 說明 |
|------|------|
| `video_source` | 輸入影片路徑 |
| `target_duration` | 輸出時長 (30/60/90/120 秒) |
| `library_ratio` | 圖庫圖片比例 (0/25/50/75%) |
| `ai_ratio` | AI 生圖比例 (0/25/50/75%) |
| `video_ratio` | 原始影片比例 (自動計算 = 100% - 圖庫 - AI) |
| `transcription_method` | 轉錄方式 (local/openai/existing) |
| `openai_api_key` | OpenAI API 金鑰 (用於 DALL-E 或 Whisper API) |
| `glm_api_key` | GLM (智譜 AI) API 金鑰 (用於 glm-image 模型) |
| `ai_image_service` | AI 生圖服務 (pollinations/glm/dalle) |
| `whisper_model` | Whisper 模型選擇 (large-v3-turbo/medium/small/base) |

---

## 🎙️ Whisper 本地模型選擇

### 模型比較

| 模型 | 大小 | 速度 | 準確度 | 推薦用途 |
|------|------|------|--------|----------|
| **large-v3-turbo** | ~3GB | ⚡ 快 | ⭐⭐⭐⭐⭐ | **推薦** 最佳平衡 |
| large-v3 | ~3GB | 🐢 慢 | ⭐⭐⭐⭐⭐ | 最高準確度 |
| medium | ~1.5GB | ⚡ 快 | ⭐⭐⭐ | 一般用途 |
| small | ~500MB | ⚡⚡ 很快 | ⭐⭐ | 快速處理 |
| base | ~150MB | ⚡⚡⚡ 極快 | ⭐ | 草稿級 |

### 安裝 Whisper

```bash
# 安裝 OpenAI Whisper
pip3 install openai-whisper

# 驗證安裝
python3 -m whisper --help
```

### 使用方式

```bash
# 推薦：large-v3-turbo (速度與準確度最佳平衡)
python3 -m whisper video.mp4 --model large-v3-turbo --output_format json --language Zh

# 最高準確度：large-v3
python3 -m whisper video.mp4 --model large-v3 --output_format json --language Zh

# 快速處理：medium
python3 -m whisper video.mp4 --model medium --output_format json --language Zh
```

---

## ⚠️ 執行前必讀

**在開始執行任何步驟之前，必須先使用 `AskUserQuestion` 工具分三批詢問使用者以下參數：**

### 📦 第一批詢問 (4個問題)
1. **target_duration** - 目標時長 (30/60/90/120 秒)
2. **library_ratio** - 圖庫圖片比例 (0/25/50/75%)
3. **ai_ratio** - AI 生圖比例 (0/25/50/75%)
4. **ai_image_service** - AI 生圖服務 (Pollinations/GLM/DALL-E)

### 📦 第二批詢問 (1個問題)
5. **api_key** - API Key (根據 ai_image_service 選擇要求輸入)
   - Pollinations.ai → 可跳過（免費）
   - GLM → 必須輸入 GLM_API_KEY
   - DALL-E → 必須輸入 OPENAI_API_KEY

### 📦 第三批詢問 (1個問題)
6. **transcription_method** - 轉錄方式 (local/openai/existing)

**重要**: 不要跳過詢問直接執行步驟！依序執行三批詢問。

---

## 已修正問題

### ✅ 問題 1：AI 生圖沒有真正執行
**已修正**: 使用 `assemble-scenes-with-ai.js`，調用 Pollinations.ai 免費 API 真正生成 AI 圖片

### ✅ 問題 2：場景順序混亂
**已修正**: 使用 Fisher-Yates 隨機演算法，確保場景類型分布符合設定比例

---

## 執行步驟 (按順序執行)

```bash
# ═══════════════════════════════════════════════════════════
# 步驟 0: 創建專屬工作資料夾
# ═══════════════════════════════════════════════════════════
WORK_DIR="output/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$WORK_DIR"/{segments,transcripts,video-segments,ai-generated}
echo "✅ 步驟 0 完成：創建專屬資料夾 $WORK_DIR"
echo "🔄 執行步驟 1：影片切片..."

# ═══════════════════════════════════════════════════════════
# 步驟 1: 影片切片 (每 3 秒一段)
# ═══════════════════════════════════════════════════════════
cd "$WORK_DIR/segments"
ffmpeg -i "{{video_source}}" -c copy -map 0 -f segment -segment_time 3 -reset_timestamps 1 "segment_%03d.mp4"
echo "✅ 步驟 1 完成：影片切片"
echo "🔄 執行步驟 2：Whisper 轉錄..."

# ═══════════════════════════════════════════════════════════
# 步驟 2: Whisper 轉錄 (使用 Python 直接調用)
# ═══════════════════════════════════════════════════════════
python3 -m whisper "{{video_source}}" --model large-v3-turbo --output_format json --output_dir "$WORK_DIR/transcripts" --task transcribe --language Zh
echo "✅ 步驟 2 完成：Whisper 轉錄"
echo "🔄 執行步驟 3：生成文字分組與關鍵字..."

# ═══════════════════════════════════════════════════════════
# 步驟 3: 生成 grouped.json 和 keywords.json
# ═══════════════════════════════════════════════════════════
# 從轉錄結果按 3 秒分組，生成對應的關鍵字
node -e "
const fs = require('fs');
const path = require('path');

const transcriptPath = '$WORK_DIR/transcripts/*.json';
const transcriptFiles = require('fs').readdirSync('$WORK_DIR/transcripts').filter(f => f.endsWith('.json'));
const transcript = JSON.parse(fs.readFileSync(path.join('$WORK_DIR/transcripts', transcriptFiles[0]), 'utf8'));

const SEGMENT_DURATION = 3;
const totalDuration = transcript.segments[transcript.segments.length - 1].end;
const numSegments = Math.ceil(totalDuration / SEGMENT_DURATION);

const groupedTexts = [];
const keywordsList = [];

for (let i = 0; i < numSegments; i++) {
  const startTime = i * SEGMENT_DURATION;
  const endTime = (i + 1) * SEGMENT_DURATION;
  const relevantSegments = transcript.segments.filter(s => s.start < endTime && s.end > startTime);
  const text = relevantSegments.map(s => s.text.trim()).filter(t => t).join(' ');
  groupedTexts.push(text);
  const keywords = text.replace(/[？！。，、；：「」『』\\s]/g, ' ').split(' ').filter(w => w.length >= 2).slice(0, 5);
  keywordsList.push(keywords);
}

fs.writeFileSync('$WORK_DIR/transcripts/grouped.json', JSON.stringify(groupedTexts, null, 2));
fs.writeFileSync('$WORK_DIR/transcripts/keywords.json', JSON.stringify(keywordsList, null, 2));
console.log('✓ 生成 grouped.json 和 keywords.json');
"
echo "✅ 步驟 3 完成：文字分組與關鍵字"
echo "🔄 執行步驟 4：片段重要性分析..."

# ═══════════════════════════════════════════════════════════
# 步驟 4: 片段重要性分析與選擇
# ═══════════════════════════════════════════════════════════
WORK_DIR="$WORK_DIR" node scripts/analyze-segments.js
echo "✅ 步驟 4 完成：片段重要性分析"
echo "🔄 執行步驟 5：AI 文案重寫..."

# ═══════════════════════════════════════════════════════════
# 步驟 5: AI 文案重寫 (使用 Claude)
# ═══════════════════════════════════════════════════════════
# ⚠️ 重要：此步驟必須由 Claude (AI) 完成，不使用腳本
#
# 執行方式：
# 1. 讀取 $WORK_DIR/selected-segments.json
# 2. 根據目標時長計算場景數量：target_duration / 3
# 3. 使用 Claude 重寫每個片段的文案：
#    - 每個場景 3 秒，文案必須在 12-15 字以內（正常語速 4-5字/秒）
#    - 簡潔有力，去除冗餘和重複內容
#    - 保持核心信息
#    - 適合短影片快節奏
# 4. 輸出格式到 $WORK_DIR/rewritten-text.json：
#    [{ index: 0, originalText: "...", rewrittenText: "...", keywords: [...] }]
#
echo "✅ 步驟 5 完成：AI 文案重寫"
echo "🔄 執行步驟 6：影片片段重新編碼..."

# ═══════════════════════════════════════════════════════════
# 步驟 6: 影片片段重新編碼 (瀏覽器相容格式)
# ═══════════════════════════════════════════════════════════
node -e "
const fs = require('fs');
const path = require('path');
const { execSync } = require('child_process');

const workDir = '$WORK_DIR';
const selected = JSON.parse(fs.readFileSync(path.join(workDir, 'selected-segments.json'), 'utf8'));

console.log('🎬 重新編碼選取的影片片段...');

selected.selectedSegments.forEach((seg, i) => {
  const src = path.join(workDir, \`segments/segment_\${String(seg.index).padStart(3, '0')}.mp4\`);
  const dst = path.join('public/video-segments', \`segment_\${String(i).padStart(3, '0')}.mp4\`);
  if (fs.existsSync(src)) {
    execSync(\`ffmpeg -y -i \"\${src}\" -c:v libx264 -profile:v baseline -level 3.0 -pix_fmt yuv420p -c:a aac -b:a 128k -movflags +faststart \"\${dst}\"\`, { stdio: 'inherit' });
    console.log(\`✓ [\${i}] segment_\${String(seg.index).padStart(3, '0')} -> segment_\${String(i).padStart(3, '0')}\`);
  }
});
"
echo "✅ 步驟 6 完成：影片片段重新編碼"
echo "🔄 執行步驟 7：場景組裝與 AI 生圖..."

# ═══════════════════════════════════════════════════════════
# 步驟 7: 場景組裝
# ═══════════════════════════════════════════════════════════
# ⚠️ 場景數量 = target_duration / 3（例如 60秒 = 20個場景）
# ⚠️ 不要固定 30 個場景！
#
# 根據用戶選擇的 AI 服務執行生圖：
# - pollinations → 使用 Pollinations.ai 免費 API
# - glm → 使用 GLM API (需要 GLM_API_KEY)
# - dalle → 使用 DALL-E API (需要 OPENAI_API_KEY)

echo "✅ 步驟 7 完成：場景組裝"
echo "🔄 執行步驟 8：渲染最終影片..."

# ═══════════════════════════════════════════════════════════
# 步驟 8: 複製場景配置並渲染
# ═══════════════════════════════════════════════════════════
cp "$WORK_DIR/scenes.json" output/scenes.json
npx remotion render AIShortVideo "$WORK_DIR/final-output.mp4" --overwrite --timeout=120000 --concurrency=1
echo "✅ 步驟 8 完成：渲染最終影片"
echo "🔄 執行步驟 9：UI/UX 視覺檢查..."

# ═══════════════════════════════════════════════════════════
# 步驟 9: UI/UX 視覺檢查
# ═══════════════════════════════════════════════════════════
echo ""
echo "🎨 執行 UI/UX 視覺檢查..."
# 檢查場景配置的視覺多樣性和吸引力
node -e "
const fs = require('fs');
const data = JSON.parse(fs.readFileSync('output/scenes.json', 'utf8'));
const scenes = data.scenes;
const videoCount = scenes.filter(s => s.useVideo).length;
const imageCount = scenes.filter(s => !s.useVideo).length;
const aiImageCount = scenes.filter(s => s.imageConfig?.primary?.url?.includes('/ai-generated/')).length;
const libraryImageCount = imageCount - aiImageCount;

console.log('\\n📊 場景分析:');
console.log(\`   🎬 影片場景: \${videoCount} (\${Math.round(videoCount/scenes.length*100)}%)\`);
console.log(\`   ✨ AI 圖片: \${aiImageCount} (\${Math.round(aiImageCount/scenes.length*100)}%)\`);
console.log(\`   📚 圖庫圖片: \${libraryImageCount} (\${Math.round(libraryImageCount/scenes.length*100)}%)\`);

// 分析視覺多樣性
const sceneTypes = scenes.map(s => s.useVideo ? 'video' : s.imageConfig?.primary?.url?.includes('/ai-generated/') ? 'ai' : 'library');
let transitions = 0;
for (let i = 1; i < sceneTypes.length; i++) {
  if (sceneTypes[i] !== sceneTypes[i-1]) transitions++;
}
const diversityScore = Math.round((transitions / (scenes.length - 1)) * 100);

console.log('\\n🎨 視覺多樣性:');
console.log(\`   場景切換次數: \${transitions}/\${scenes.length - 1}\`);
console.log(\`   多樣性評分: \${diversityScore}%\`);

if (diversityScore >= 60) {
  console.log('   ✅ 視覺節奏良好，場景變化豐富');
} else if (diversityScore >= 40) {
  console.log('   ⚠️  視覺節奏一般，可增加場景變化');
} else {
  console.log('   ❌ 視覺節奏單調，建議增加場景類型變化');
}

// 場景順序視覺化
console.log('\\n📲 場景順序:');
console.log('   ' + sceneTypes.map(t => t === 'video' ? '🎬' : t === 'ai' ? '✨' : '📚').join(' '));
"
echo ""
echo "✅ 步驟 9 完成：UI/UX 視覺檢查"
echo ""
echo "🎉 所有步驟完成！輸出影片: $WORK_DIR/final-output.mp4"
```

---

## 關鍵修正：場景順序使用隨機模式

### ✅ 隨機模式做法
```javascript
// 建立類型池
const sceneTypes = [];
for (let i = 0; i < videoScenes; i++) sceneTypes.push('video');
for (let i = 0; i < libraryScenes; i++) sceneTypes.push('library');
for (let i = 0; i < aiScenes; i++) sceneTypes.push('ai');

// Fisher-Yates 隨機打亂
for (let i = sceneTypes.length - 1; i > 0; i--) {
  const j = Math.floor(Math.random() * (i + 1));
  [sceneTypes[i], sceneTypes[j]] = [sceneTypes[j], sceneTypes[i]];
}
```

### UI/UX 視覺多樣性檢查
渲染後會自動分析場景配置，評估視覺多樣性：
- 📊 場景類型統計
- 🎨 場景切換頻率
- ✅/⚠️/❌ 視覺節奏評分

---

## AI 生圖必須真正執行

### ❌ 錯誤做法
```javascript
function simulateAIImage(text, sceneIndex) {
  // 只是隨機選圖庫圖片！
  const allImages = Object.keys(irasutoyaDB.images);
  const randomImage = allImages[Math.floor(Math.random() * allImages.length)];
  return `/ai-images/${randomImage}`;
}
```

### ✅ 正確做法
```javascript
async function generateAIImage(text, sceneIndex) {
  // 檢查是否有 API Key
  if (process.env.OPENAI_API_KEY) {
    // 真正調用 DALL-E API
    const prompt = createImagePrompt(text);
    const imageUrl = await generateWithDalle(prompt, sceneIndex);
    if (imageUrl) return imageUrl;
  }
  // 才使用 fallback
  return getRandomLibraryImage(text);
}
```

---

## 場景組裝腳本：assemble-scenes-with-ai.js

這個腳本：
1. **AI 生圖真正執行** - 使用 Pollinations.ai 免費 API 生成圖片
2. **隨機場景順序** - Fisher-Yates 洗牌演算法
3. **內容匹配圖片** - 根據文案生成對應的 AI 圖片

---

## GLM (智譜 AI) 配置

### ✅ 正確模型名稱
使用 `glm-image` 模型進行 AI 圖像生成。

### API 請求格式
```json
{
  "model": "glm-image",
  "prompt": "futuristic, cyberpunk, digital art, neon, high quality",
  "size": "1024x1024"
}
```

### API 端點
```
POST https://open.bigmodel.cn/api/paas/v4/images/generations
Authorization: Bearer YOUR_GLM_API_KEY
```

---

## 場景數據結構

### 圖片場景 (AI 生圖)
```json
{
  "id": "scene-0",
  "index": 0,
  "startTime": 0,
  "endTime": 3,
  "duration": 3,
  "transcription": "2026年最強個人AI員工來了！",
  "keywords": ["AI", "員工", "2026"],
  "style": "tech",
  "useVideo": false,
  "videoSegmentPath": null,
  "imageConfig": {
    "primary": {
      "url": "/ai-generated/ai_0.png"
    }
  }
}
```

### 圖片場景 (圖庫圖片)
```json
{
  "id": "scene-1",
  "index": 1,
  "startTime": 3,
  "endTime": 6,
  "duration": 3,
  "transcription": "Cloudbot徹底改變我的工作方式",
  "keywords": ["Cloudbot", "工作"],
  "style": "tech",
  "useVideo": false,
  "videoSegmentPath": null,
  "imageConfig": {
    "primary": {
      "url": "/ai-images/VR_1.png"
    }
  }
}
```

### 影片場景
```json
{
  "id": "scene-1",
  "index": 1,
  "startTime": 3,
  "endTime": 6,
  "duration": 3,
  "transcription": "現代最強咒術師，迎來最強大的外星敵人",
  "keywords": ["咒術", "最強"],
  "style": "tech",
  "useVideo": true,
  "videoSegmentPath": "video-segments/segment_000.mp4",
  "imageConfig": {
    "primary": {
      "url": null
    }
  }
}
```

---

## 關鍵檔案位置

| 用途 | 路徑 |
|------|------|
| 輸入影片 | 使用者指定路徑 |
| 專屬資料夾 | `$WORK_DIR` (如 `output/20250128-153045/`) |
| 影片片段 | `$WORK_DIR/segments/*.mp4` |
| 轉錄結果 | `$WORK_DIR/transcripts/*.json` |
| 選取片段 | `$WORK_DIR/selected-segments.json` |
| 重寫文案 | `$WORK_DIR/rewritten-text.json` |
| 場景數據 | `$WORK_DIR/scenes.json` |
| 瀏覽器影片 | `public/video-segments/*.mp4` (共用) |
| 圖片資源 | `public/ai-images/*.png` (共用) |
| AI 生圖 | `$WORK_DIR/ai-generated/*.png` |
| 最終輸出 | `$WORK_DIR/final-output.mp4` |
