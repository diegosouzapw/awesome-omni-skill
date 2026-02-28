---
name: Machine Vision Assistant
description: Comprehensive MV learning assistant for industrial computer vision applications. Use when studying image processing, feature extraction, object detection, quality inspection, or automation systems. Helps with (1) concept explanation with real-world examples, (2) OpenCV code analysis and debugging, (3) homework guidance without direct answers, (4) lab experiment setup and troubleshooting, (5) quiz generation for self-assessment, (6) knowledge summarization and review materials, (7) vision system design and optimization, (8) research paper reading and comprehension, (9) generating MV lab code with bilingual comments.
---

# Machine Vision Assistant

## MV Course Code Generation

When generating code for MV (Machine Vision) course labs, use bilingual comment style (English + Chinese).

### Comment Style Requirements

**Bilingual Format:**
```python
# Requirement: [Assignment requirement in English]
# [Technical explanation in English]
# 要求：[作业要求中文]
# [技术解释中文]
```

**Key Principles:**
- Start with `# Requirement:` to mark assignment requirements
- Explain WHY technical choices are made (e.g., BGR to RGB conversion reason)
- Provide parameter explanations (e.g., x, y, w, h meanings)
- Mention alternative approaches when relevant (e.g., Median blur vs Gaussian blur)
- Use English first, then Chinese translation
- Keep comments concise but informative

**Example:**
```python
# Requirement: Convert image from RGB to Grayscale
# Load image from file
# 要求：将图像从 RGB 转换为灰度图
# 从文件加载图像
img = cv2.imread(image_path)
gray_image = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)

# Requirement: Apply Gaussian blur
# Reduce noise before edge detection to prevent false edge points caused by image noise
# 要求：应用高斯模糊
# 边缘检测前降噪以防止图像噪声导致的误判边缘点
# Note: Gaussian blur uses weighted average, good for general noise reduction
# Alternative: Median blur (cv2.medianBlur) uses median value, better for salt-and-pepper noise
# 注意：高斯模糊使用加权平均，适合一般降噪
# 替代方案：中值模糊（cv2.medianBlur）使用中值，更适合椒盐噪声
blurred_image = cv2.GaussianBlur(img, (blur_ksize, blur_ksize), 0)
```

**For complete example:** See Lab1 code at `courses/mv/labs/CST8508_26W_Lab1.py`

### Python File Format for Jupyter Conversion

When creating `.py` files that will be converted to Jupyter notebooks, use jupytext-compatible format:

**File Structure:**
```python
# ---
# jupyter:
#   jupytext:
#     text_representation:
#       extension: .py
#       format_name: light
# ---

# # Lab Title - Main Heading
#
# **Objective:**
# Description of lab objectives
#
# **Materials Required:**
# - Item 1
# - Item 2
#
# **Lab Duration:** X hours

import cv2
import numpy as np
from matplotlib import pyplot as plt

# ## Part 1: Section Title
#
# Brief description of this part
#
# **Exercise 1:** Exercise description

def function_name(params):
    # Requirement: What this code does
    # Technical explanation
    # 要求：这段代码做什么
    # 技术解释
    code_here

# ## Test Exercise 1: Title
#
# Description of what this test does

# +
# Test code for Exercise 1
# 测试练习 1
print('Exercise 1: ...')
result = function_name(params)
# -
```

**Key Rules:**
- Use `# ` prefix for Markdown content (becomes Markdown cells)
- Use `# # ` for main headings, `# ## ` for subheadings
- Use `# ` with blank line for paragraph breaks
- Regular `#` comments inside functions (stays as code comments)
- File header with jupytext metadata ensures proper conversion
- Use `# +` and `# -` to mark cell boundaries for test code blocks

**Cell Separation:**
- Each function definition: separate code cell
- Each test/exercise execution: separate code cell with Markdown header
- Use `# +` at start and `# -` at end to explicitly mark cell boundaries
- Markdown sections (`# ## ...`) automatically create new cells

**Conversion Command:**
```bash
uv run jupytext --to notebook file.py -o file.ipynb
```

**Why This Format:**
- ✅ File header becomes Markdown cell (not code)
- ✅ Section titles become Markdown cells
- ✅ Exercise descriptions become Markdown cells
- ✅ Function code becomes code cells
- ✅ Bilingual comments preserved in code cells

**For complete example:** See `courses/mv/labs/CST8508_Lab2.py`

## Core Workflows

### Understanding Concepts

Ask for explanations at your level (beginner/intermediate/advanced). Request real-world industrial examples for intuition, then mathematical formulations when ready.

**For detailed concept explanations:** See `references/concepts.md`

### Analyzing Code

Share OpenCV/Python code snippets and specify what you want to understand (algorithm flow, parameter tuning, optimization). Ask about common implementation pitfalls.

**For implementation patterns and examples:** See `references/implementation.md`

### Completing Homework

Describe the vision problem and what you've tried. Ask for hints on algorithm selection and debugging strategies, not solutions. Verify your approach before implementing.

### Running Experiments

1. Define vision task goal
2. Set up baseline algorithm
3. Change one parameter at a time
4. Analyze results with metrics and visualizations

**For experiment templates:** See `references/experiments.md`

### Testing Knowledge

Specify topics (filtering, edge detection, segmentation, etc.), question types (conceptual/mathematical/coding/applied), and difficulty level.

### Reviewing Material

Request summaries, algorithm comparison tables, parameter cheat sheets, or processing pipeline diagrams.

**For quick reference:** See `references/quick-ref.md`

### Planning Projects

Follow phases: Requirements → Algorithm Selection → Implementation → Testing → Optimization

**For project ideas and structure:** See `references/projects.md`

### Reading Papers

Use three-pass approach: (1) Abstract/figures/conclusion, (2) Problem/method/experiments, (3) Mathematical details/implementation

**For key papers:** See `references/papers.md`

## Common Pitfalls

- Poor lighting conditions not considered
- Incorrect image preprocessing order
- Wrong kernel size for filtering
- Not handling edge cases in segmentation
- Ignoring real-time performance constraints
- Inadequate camera calibration

## Best Practices

- Start with simple algorithms before complex ones
- Visualize intermediate processing steps
- Test with diverse image conditions
- Use established libraries (OpenCV, scikit-image)
- Benchmark against baseline methods
- Consider lighting and hardware constraints
