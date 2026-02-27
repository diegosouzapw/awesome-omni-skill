---
name: multimodal-doc-converter
description: Parse and convert multimodal documents (PDF, DOCX, etc.) into structured Markdown with minimal information loss. Use this skill when users need to: (1) convert documents containing text, images, and audio into Markdown format, (2) extract and OCR text from embedded images, (3) recognize and render mathematical formulas, (4) transcribe embedded audio files, (5) preserve document structure and reading order during conversion. Trigger on requests like "convert this PDF to markdown", "extract content from this document", "turn this docx into markdown with OCR".
---

# Multimodal Document Converter

将 PDF、DOCX 等多模态文档转换为结构化 Markdown,近乎无损地保留文本、图像、音频等内容。

## 核心理念

**不要直接转换 Markdown**,必须先构建中间表示(IR),再重建输出。这是保证结构与顺序不丢失的关键。

## 转换流程

### 1. 文档解析与资源提取

根据文档格式选择解析器:

**PDF 文档:**
```python
import fitz  # PyMuPDF
import pdfplumber

# 提取文本与布局
with pdfplumber.open(pdf_path) as pdf:
    for page in pdf.pages:
        text = page.extract_text()
        tables = page.extract_tables()

# 提取图片、音频等嵌入对象
doc = fitz.open(pdf_path)
for page_num in range(len(doc)):
    page = doc[page_num]
    image_list = page.get_images()
    # 提取音频对象(如有)
```

**DOCX 文档:**
```python
from docx import Document
import docx2python

doc = Document(docx_path)
for element in doc.element.body:
    # 解析段落、图片、音频引用
    pass
```

### 2. 构建文档中间结构

这是**最关键**的步骤,定义统一的数据结构:

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class DocumentBlock:
    block_id: str
    block_type: str  # paragraph/heading/image/audio/formula/table
    text: Optional[str] = None
    media_ref: Optional[str] = None  # 媒体文件路径
    bbox: Optional[tuple] = None  # (x, y, width, height)
    page_index: int = 0
    order_index: int = 0  # 同页内顺序
    style: Optional[dict] = None  # {level: 1, bold: True}
    metadata: Optional[dict] = None
```

**排序规则:**
- 按 `page_index` 排序
- 同页内按 `bbox.y` 坐标(从上到下)
- 识别标题层级(字号、加粗、编号模式)

### 3. 图像处理与 OCR

对每个提取的图片进行分类处理:

**初始化 PaddleOCR:**
```python
from paddleocr import PaddleOCR
ocr = PaddleOCR(use_angle_cls=True, lang='ch')
```

**图片分类策略:**
1. 普通插图 → 保留原图引用
2. 包含文字的图片 → OCR识别
3. 数学公式/几何图形 → LaTeX识别 + 可视化重绘

**OCR文字识别:**
```python
result = ocr.ocr(img_path, cls=True)
for line in result:
    for word_info in line:
        text, confidence = word_info[1]
        bbox = word_info[0]
```

**数学公式处理:**
```python
# 使用 LaTeX OCR 或 PaddleOCR 数学模型
from pix2tex.cli import LatexOCR
model = LatexOCR()
latex_str = model(img_path)

# 可选:用 SymPy 验证公式
from sympy.parsing.latex import parse_latex
expr = parse_latex(latex_str)
```

**数学可视化(可选增强):**
```python
import matplotlib.pyplot as plt
from sympy import plot, symbols

# 函数图像
x = symbols('x')
plot(expr, (x, -10, 10))

# 或使用 Manim 制作动画(高级场景)
```

### 4. 音频提取与转录

**音频提取:**
```python
# PDF 中提取音频
import fitz
doc = fitz.open(pdf_path)
for page in doc:
    for annot in page.annots():
        if annot.type[0] == 17:  # Sound annotation
            sound = annot.get_sound()
```

**音频转码:**
```bash
ffmpeg -i input.mp3 -ar 16000 -ac 1 output.wav
```

**语音识别:**
```python
from paddlespeech.cli.asr import ASRExecutor
asr = ASRExecutor()
result = asr(audio_file='output.wav')
```

**带时间戳转录(可选):**
```python
# 使用 WhisperX 或 PaddleSpeech
asr_result = asr(audio_file, force_yes=True)
# 输出: [(start_time, end_time, text), ...]
```

### 5. Markdown 重建规则

根据 `DocumentBlock` 序列生成 Markdown:

**基础规则:**
```python
def block_to_markdown(block: DocumentBlock) -> str:
    if block.block_type == 'heading':
        level = block.style.get('level', 1)
        return f"{'#' * level} {block.text}\n\n"
    
    elif block.block_type == 'paragraph':
        return f"{block.text}\n\n"
    
    elif block.block_type == 'image':
        md = f"![{block.block_id}]({block.media_ref})\n\n"
        
        # 如果有OCR内容,追加引用块
        if block.metadata and 'ocr_text' in block.metadata:
            ocr_text = block.metadata['ocr_text']
            md += f"> **图片文字识别:**\n> {ocr_text}\n\n"
        
        # 如果有LaTeX公式
        if block.metadata and 'latex' in block.metadata:
            latex = block.metadata['latex']
            md += f"$$\n{latex}\n$$\n\n"
        
        return md
    
    elif block.block_type == 'audio':
        md = f"[🔊 {block.block_id}]({block.media_ref})\n\n"
        
        # 追加转录文本
        if block.metadata and 'transcript' in block.metadata:
            transcript = block.metadata['transcript']
            md += f"> **语音转文字:**\n> {transcript}\n\n"
        
        return md
    
    elif block.block_type == 'formula':
        # 独立公式
        return f"$$\n{block.text}\n$$\n\n"
```

**克制原则:**
- OCR/ASR 内容用引用块标注,避免与原文混淆
- 保持简洁,不过度格式化
- 低置信度内容标注"自动识别"

### 6. 资源管理

**统一目录结构:**
```
output/
├── document.md
└── assets/
    ├── images/
    │   ├── img_001.png
    │   └── formula_002.png
    ├── audio/
    │   └── audio_001.wav
    └── formulas/
        └── rendered_003.png
```

**Markdown 内只用相对路径:**
```markdown
![image](assets/images/img_001.png)
[audio](assets/audio/audio_001.wav)
```

## 完整工作流示例

```python
def convert_document(input_path: str, output_dir: str):
    # 1. 解析文档
    blocks = parse_document(input_path)
    
    # 2. 提取多媒体
    media_files = extract_media(blocks, output_dir)
    
    # 3. OCR 图片
    for block in blocks:
        if block.block_type == 'image':
            ocr_result = ocr.ocr(block.media_ref)
            block.metadata['ocr_text'] = extract_text(ocr_result)
            
            # 检测数学公式
            if is_formula(block.media_ref):
                latex = latex_ocr(block.media_ref)
                block.metadata['latex'] = latex
    
    # 4. 转录音频
    for block in blocks:
        if block.block_type == 'audio':
            transcript = asr(block.media_ref)
            block.metadata['transcript'] = transcript
    
    # 5. 排序 blocks
    blocks.sort(key=lambda b: (b.page_index, b.order_index))
    
    # 6. 生成 Markdown
    markdown = []
    for block in blocks:
        markdown.append(block_to_markdown(block))
    
    # 7. 写入文件
    output_path = os.path.join(output_dir, 'document.md')
    with open(output_path, 'w', encoding='utf-8') as f:
        f.write(''.join(markdown))
```

## 关键难点与对策

1. **"近乎无损"的定义:** 不是文本100%准确,而是**结构与阅读顺序不丢失**
2. **必须有中间结构:** 直接转 Markdown 必然丢信息,IR 是必需品
3. **OCR/ASR 是补充:** 永远不替代原文,只作为引用块追加
4. **数学内容分层:** 区分公式本身(LaTeX) vs 可视化呈现(图片)
5. **模块解耦:** 各处理模块独立,未来可替换模型而无需重写系统

## 依赖安装

```bash
pip install PyMuPDF pdfplumber python-docx --break-system-packages
pip install paddlepaddle paddleocr paddlespeech --break-system-packages
pip install sympy matplotlib --break-system-packages
pip install pix2tex --break-system-packages  # LaTeX OCR
```

## 输出质量标准

转换后的 Markdown 应满足:

1. 标题层级正确反映原文档结构
2. 段落顺序与原文档阅读顺序一致
3. 图片位置保留,OCR内容以引用块形式追加
4. 数学公式用 LaTeX 渲染,复杂图形保留原图
5. 音频文件可点击,转录文本紧随其后
6. 所有资源路径相对化,保证可迁移性

## 注意事项

- 处理大文档时分页处理,避免内存溢出
- OCR 置信度低于0.8的结果标注"需人工核验"
- 音频转录支持多语言,根据文档语言自动切换
- 数学公式识别失败时,保留原图并标注"公式图片"
