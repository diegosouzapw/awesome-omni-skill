---
name: card-generator
description: 创建可下载的卡片式宣传网页/海报。当用户需要制作产品介绍卡片、教程卡片、知识科普卡片、小红书风格图文、PPT式滑动展示页时使用。支持多种预设模板（科技风、简约风、渐变风、暗黑风等），生成包含React+SVG的单HTML文件，内置ZIP打包下载功能。
---

# Card Generator Skill

生成精美的卡片式HTML网页，支持一键下载为PNG图片压缩包。

## 核心能力

1. 将用户内容转换为3:4比例的卡片组
2. 生成单HTML文件（React + Tailwind + SVG）
3. 内置JSZip下载功能，导出高清PNG

## 模板选择

根据用户需求选择合适的模板，模板文件位于 `assets/templates/`:

| 模板 | 文件 | 适用场景 |
|------|------|----------|
| 科技渐变 | `tech-gradient.html` | 产品发布、技术教程、工具介绍 |
| 简约清新 | `minimal-clean.html` | 知识科普、生活方式、读书笔记 |
| 暗黑终端 | `dark-terminal.html` | 开发者工具、命令行教程、黑客风 |
| 活力手绘 | `vibrant-pop.html` | 备忘录风格、手写笔记、涂鸦 |
| 商务专业 | `business-pro.html` | 企业介绍、数据报告、行业分析 |
| 极光玻璃 | `glass-morphism.html` | 潮流前沿、设计趋势、艺术感 |
| 轻奢黑金 | `luxury-gold.html` | 高端品牌、VIP邀请、会员卡 |
| 几何包豪斯 | `geometric-bauhaus.html` | 艺术展览、设计理论、复古海报 |
| 自然清新 | `nature-fresh.html` | 有机食品、生活方式、环保话题 |
| 复古蒸汽波 | `retro-vaporwave.html` | 音乐活动、游戏宣发、怀旧主题 |
| 立体投影 | `shadow-stacked.html` | 图文排版、长文分享、个性语录 |
## 使用流程

### 1. 分析用户内容

将用户提供的内容拆分为4-6张卡片：
- 封面卡：标题 + 副标题 + 品牌标识
- 内容卡：核心信息点，每卡1个主题
- 结尾卡：总结/行动召唤/联系方式

### 2. 选择并读取模板

```bash
view assets/templates/<template-name>.html
```

### 3. 基于模板生成

复制模板结构，替换以下内容：
- `<title>` 标签内容
- 各 SVG 组件的文字和图形
- 配色方案（如需自定义）
- 卡片数量和布局

### 4. SVG卡片设计规范

```jsx
// 每张卡片的基础结构
const CardComponent = () => (
  <svg viewBox="0 0 300 400" className="w-full h-full">
    {/* 背景 */}
    <rect width="300" height="400" fill="..." />
    
    {/* 装饰元素 */}
    <circle cx="..." cy="..." r="..." fill="..." opacity="0.1" />
    
    {/* 主标题 - y位置约在 80-150 */}
    <text x="150" y="120" textAnchor="middle" 
          fill="..." fontSize="28" fontWeight="900">
      标题文字
    </text>
    
    {/* 内容区域 - y位置约在 180-300 */}
    <g transform="translate(30, 180)">
      {/* 内容元素 */}
    </g>
    
    {/* 底部信息 - y位置约在 350-380 */}
    <text x="150" y="370" textAnchor="middle" 
          fill="..." fontSize="14">
      底部说明
    </text>
  </svg>
);
```

### 5. 必需的下载功能

确保包含以下依赖和下载逻辑：

```html
<!-- 头部引入 -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.10.1/jszip.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/2.0.5/FileSaver.min.js"></script>
```

```jsx
// 下载函数核心逻辑
const handleDownload = async () => {
  const zip = new JSZip();
  const cardIds = ['card-1', 'card-2', ...]; // 所有卡片ID
  
  for (const [index, id] of cardIds.entries()) {
    const svg = document.querySelector(`#${id} svg`);
    const svgBlob = new Blob([new XMLSerializer().serializeToString(svg)], 
                            {type: 'image/svg+xml;charset=utf-8'});
    const url = URL.createObjectURL(svgBlob);
    
    const img = new Image();
    await new Promise(r => { img.onload = r; img.src = url; });
    
    const canvas = document.createElement('canvas');
    canvas.width = 600; canvas.height = 800; // 2x缩放
    const ctx = canvas.getContext('2d');
    ctx.scale(2, 2);
    ctx.drawImage(img, 0, 0);
    
    const blob = await new Promise(r => canvas.toBlob(r, 'image/png'));
    zip.file(`card-${index + 1}.png`, blob);
    URL.revokeObjectURL(url);
  }
  
  const content = await zip.generateAsync({type: 'blob'});
  saveAs(content, 'cards.zip');
};
```

## 设计原则

1. **视觉层次**：标题 > 核心内容 > 装饰元素
2. **留白充足**：内容不超过卡片面积的60%
3. **配色一致**：同一套卡片使用统一色系
4. **字体大小**：标题24-32px，正文12-16px，说明10-12px
5. **移动优先**：确保在手机上可正常查看

## 输出位置

生成的HTML文件保存到 `/mnt/user-data/outputs/` 并使用 `present_files` 工具展示给用户。
