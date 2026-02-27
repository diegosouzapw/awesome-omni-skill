---
name: doc_processor
description: A comprehensive tool for parsing, cleaning, generating content for, and reconstructing MS Word (.docx) documents.
---

# Doc Processor Skill

This skill allows you to "re-architect" a Word document. It can extract the deep structure, wipe content to create a template, generate new content based on rules or AI, and refill the document.

## Capabilities

1.  **Parse Structure**: Extract a hierarchical JSON representation including Sections, Paragraphs, Tables, and "Slots".
2.  **Clean Template**: Create a "Clean" blank version of the document.
3.  **Generate Content**: Produce a content map based on the parsed structure and a user topic.
    *   **Local Repository Integration**: Automatically queries local question bank for authentic exam materials.
    *   **Source Citation**: All borrowed content is properly annotated with exam source information.
4.  **Build Document**: Inject content back into the Clean Template.

## Usage Workflow

**Task**: "Rewrite this lesson plan for the topic 'Past Tense'."

### ⚠️ 重要：模板文件路径检查

**问题背景**：`/tmp/`目录下的文件在session结束后会被清理。如果用户提供的模板路径是`/tmp/xxx.docx`，在新session中可能已不存在。

**解决方案**：
1. **生成前必须检查**：使用`os.path.exists()`检查模板文件是否存在
2. **文件不存在时**：**必须询问用户**提供正确的模板路径，不要假设文件存在
3. **建议用户**：将模板文件保存在非/tmp/目录（如`~/Documents/`）

### 标准工作流程

1.  **Parse Original**:
    ```bash
    python skills/doc_processor/scripts/parser.py input.docx > structure.json
    ```
    *(Optionally redirect output to file)*

2.  **Create Template (Clean)**:
    ```bash
    python skills/doc_processor/scripts/cleaner.py input.docx template_clean.docx
    ```
    
    **⚠️ 路径保存建议**：
    - 清理后的模板保存在非/tmp/目录，如：`~/Documents/templates/lesson_template_clean.docx`
    - 或保存在工作目录：`/Users/xielk/webdata/english/lesson/templates/`

3.  **Generate Content (The "Brain")**:
    *   **Goal**: Create a `content.json` file that maps `structure.json` IDs to new content.
    *   **Process**:
        1.  Read the `structure.json` to find the Slot IDs (`p_X`, `t_X`) and their types.
        2.  **MANDATORY: Query Local Question Bank via Index System (CRITICAL CONSTRAINT)**
            *   **MUST** use the **Index + On-Demand Loading** system to access exam questions. **NEVER** directly load all docx files (65MB+).
            *   **Workflow**:
                1.  Load index file (`/Users/xielk/webdata/english/lesson/resource/index.json`)
                2.  Search index for matching files (search filename and preview text)
                3.  Load only the most relevant 3-5 docx files on-demand
                4.  Extract questions with proper citations
            *   **Implementation**:
                ```python
                from skills.doc_processor.scripts.searcher import search_question_bank
                
                # Search for questions matching topic and student profile
                results, questions = search_question_bank(
                    topic="非谓语动词",           # Grammar topic
                    district="嘉定",              # Student's district (priority)
                    year="2025"                   # Most recent year (priority)
                )
                
                # questions contains content with source annotations
                for q in questions:
                    print(q['content'])  # Question text
                    print(q['source'])   # Source: (2025 嘉定一模)
                ```
            *   **NEVER** fabricate or hallucinate exam questions. All content MUST be sourced from the local repository.
            *   **Citation Requirement**: EVERY piece of content MUST be annotated: `(YYYY 区域 考试类型)`
                *   Examples: `(2025 徐汇一模)`, `(2024 浦东二模)`, `(2023 嘉定一模)`
            *   **Priority Rules**:
                1. Most recent year (2025 > 2024 > 2023)
                2. Student's district (if specified)
                3. Load max 3-5 files, max 5 questions per file (control token usage)
        3.  **STRICTLY ADHERE to Rules from `.agent/rules/lesson.md`**:
            *   **Length Constraint**: Resulting doc MUST be **> 14 pages**. You must generate EXTENSIVE examples, detailed logic explanations, and sufficient practice questions to meet this. Do not compress content.
            *   **Time Duration**: Content must cover a full **2-hour lesson**.
            *   **Topic Focus**: Single core topic (e.g., "Prepositions") only. All examples must align.
            *   **Structure Mapping**:
                *   Row 1-3: Teaching Objectives & Difficulties.
                *   Row 6: Icebreaker/Review.
                *   Row 7-10: **Knowledge Points (Deep Dive)**. This is the bulk. Use "Methodology + Logic" style (When/Why/Trap/How).
                *   Row 15: Variant Practice (Part A: Drill, Part B: Application).
                *   Row 17: Class Quiz (Part A: Real Exams, Part B: Extension).
                *   Row 18: Reflection.
            *   **Exam Alignment**: Use tags like `(2023 Shanghai Zhongkao)` or `(2024 Pudong Model)`.
            *   **Formatting**: No Markdown symbols (`**`, `|`), use `____` for blanks.
        4.  **Synthesize Content**:
            *   Write a JSON file where Keys = IDs, Values = Strings (or Arrays for Tables).
            *   Ensure all exam questions, reading passages, and reference materials include proper source citations as specified above.
    *   *Action*: Save the result to `content.json`.

4.  **Build Final Doc**:
    Run the builder script to inject your generated content into the clean template.
    ```bash
    python skills/doc_processor/scripts/builder.py <path_to_clean_template_docx> <path_to_content_json> <path_to_final_docx>
    ```
    
    **⚠️ 异常处理流程**：
    
    如果模板文件不存在（FileNotFoundError），**必须**执行以下流程：
    
    ```python
    import os
    
    template_path = "/tmp/xxx.docx"  # 用户提供的路径
    
    if not os.path.exists(template_path):
        # 1. 报告错误
        print(f"❌ 模板文件不存在: {template_path}")
        
        # 2. 解释原因
        print("可能原因：")
        print("  • /tmp/目录文件在session结束后被清理")
        print("  • 文件路径错误")
        print("  • 文件被移动或删除")
        
        # 3. 询问用户
        print("\n💡 请提供正确的模板文件路径:")
        print("   建议将模板复制到非/tmp/目录，如 ~/Documents/templates/")
        
        # 4. 等待用户提供新路径（在对话中）
        # 不要继续生成，避免生成格式错误的文档！
    ```
    
    **在新session中的处理流程**：
    
    ```
    用户：帮我生成教案，模板是 /tmp/template.docx
    
    助手：检查文件是否存在...
    
    如果发现文件不存在：
    "⚠️ 模板文件 /tmp/template.docx 不存在！
    
    /tmp/目录下的文件会在session结束后被清理。
    
    请提供正确的模板路径，或者重新上传模板文件。
    建议将模板保存在 ~/Documents/ 目录下。"
    
    用户：（提供新路径或重新上传）
    
    助手：（使用正确的路径继续生成）
    ```

## Scripts Reference

-   `scripts/parser.py`: Analyzes structure. Returns valid JSON.
-   `scripts/cleaner.py`: Wipes content cells/paragraphs.
-   `scripts/generator.py`: *Optional* mock script. In real usage, the Agent generates the `content.json`.
-   `scripts/builder.py`: Fills blocks by ID. Matches iteration order of `parser.py`.

## Local Question Bank Integration (强制约束)

### Repository Path Configuration

**Default Path**: `/Users/xielk/webdata/english/lesson/resource`

This directory contains authentic exam materials organized by:
-   District (区): `徐汇/`, `浦东/`, `嘉定/`, etc.
-   Year: `2025/`, `2024/`, `2023/`, etc.
-   Type: `一模/`, `二模/`, `中考/`, etc.
-   Category: `语法/`, `阅读/`, `作文/`, etc.

### Index System (索引+按需加载)

**解决大文件问题**: 题库总计约65MB，直接加载所有docx会产生巨大token费用。使用**索引+按需加载**机制：

#### 1. 生成索引（首次使用或更新题库时执行）

```bash
# 创建索引（只需执行一次，约10秒）
python skills/doc_processor/scripts/indexer.py
```

索引文件位置: `/Users/xielk/webdata/english/lesson/resource/index.json`

索引包含：
-   文件路径、文件名
-   年份、区域、考试类型、题型（自动解析）
-   预览内容（前500字符）
-   文件大小、修改时间

#### 2. 搜索使用方式

**方式A：使用Searcher类（推荐）**

```python
from skills.doc_processor.scripts.searcher import QuestionBankSearcher

# 初始化（加载索引，token极少）
searcher = QuestionBankSearcher()

# 搜索索引（仅查索引，不加载docx）
results = searcher.search(
    keyword="非谓语",      # 关键词
    district="徐汇",       # 可选：区域筛选
    year="2025",          # 可选：年份筛选
    limit=10              # 返回结果数
)

# 智能搜索（索引+按需加载docx）
idx_results, questions = searcher.smart_search(
    topic="非谓语",
    district="嘉定",       # 优先学生所在区
    year="2025",
    max_docs=3,           # 最多加载3个文件
    max_questions_per_doc=5  # 每个文件最多5题
)

# questions中包含题目内容和来源标注
for q in questions:
    print(q['content'])     # 题目内容
    print(q['source'])      # 来源：(2025 嘉定一模)
```

**方式B：便捷函数**

```python
from skills.doc_processor.scripts.searcher import search_question_bank

# 一键搜索
results, questions = search_question_bank(
    topic="定语从句",
    district="浦东",
    year="2024"
)
```

#### 3. Token费用对比

| 方式 | Token消耗 | 说明 |
|------|-----------|------|
| 直接加载所有docx（65MB） | **巨大** | ❌ 不推荐 |
| 预转txt后全文搜索 | **大** | ⚠️ 稍好但仍贵 |
| **索引+按需加载** | **极小** | ✅ 只加载需要的3-5个文件 |

### Search Strategy (MUST FOLLOW)

使用索引系统进行搜索：

1.  **加载索引**（token极少，一次性）
2.  **搜索索引**（匹配文件名和预览内容）
3.  **按需加载**（只加载最相关的3-5个docx文件）
4.  **提取题目**（带来源标注）

具体步骤：

```bash
# Step 1: 确保索引已创建
python skills/doc_processor/scripts/indexer.py

# Step 2: 在Python中使用Searcher搜索
python << 'PYEOF'
from skills.doc_processor.scripts.searcher import search_question_bank

# 搜索语法题目（优先嘉定区2025年）
results, questions = search_question_bank("非谓语", "嘉定", "2025")

# 搜索阅读材料
results, passages = search_question_bank("阅读B篇", "徐汇", "2024")

# 搜索作文范文
results, compositions = search_question_bank("中考作文", None, "2023")
PYEOF
```

### Source Citation Format (强制标注)

Every piece of content extracted from the repository MUST include source annotation:

**Format**: `(YYYY 区域 考试类型 [题型])`

**Examples**:
-   `(2025 徐汇一模 语法单选)` - 2025 Xuhui District First Mock Exam, Grammar MCQ
-   `(2024 浦东二模 阅读B篇)` - 2024 Pudong District Second Mock Exam, Reading Passage B
-   `(2023 Shanghai Zhongkao 作文)` - 2023 Shanghai High School Entrance Exam, Composition
-   `(2024 Jiading Model 完形填空)` - 2024 Jiading District Mock Exam, Cloze Test

**Placement**:
-   Place citation **immediately after** the question title or passage title
-   Example:
    ```
    【例题1】选择最佳答案（2025 徐汇一模 语法单选）
    The problem ______ at the meeting tomorrow is important.
    A. to be discussed    B. being discussed    C. discussed    D. to discuss
    ```

### Priority Rules

When multiple sources are available, select in this order:

1.  **Recency**: Prioritize 2025 over 2024 over 2023
2.  **Student's District**: If student is from Jiading, use Jiading papers first
3.  **Difficulty Match**: Select materials matching student's current level (98分 → medium difficulty, avoid too basic)
4.  **Topic Relevance**: Exact topic match > Related topic > General review

### Error Handling

If required content is **NOT found** in the repository:

1.  Expand search to adjacent years (e.g., if 2025 not found, try 2024)
2.  Expand search to other districts (e.g., if 徐汇 not found, try 浦东)
3.  If still not found, inform user: "未在题库中找到[具体年份/区域]的相关题目，已使用[替代来源]的相似题目替代"
4.  **NEVER fabricate** exam questions or pretend they exist in the repository

### Content Types to Search

-   **Grammar Questions**: 单选题, 填空题, 改错题, 完成句子
-   **Reading Materials**: A篇应用文, B篇记叙文, C篇首字母填空, D篇回答问题
-   **Compositions**: 中考作文范文, 满分作文, 常见话题模板
-   **Vocabulary**: 考纲词汇, 高频短语, 固定搭配

### Shanghai Zhongkao Question Type Structure (上海中考题型结构)

**必须理解上海中考英语试卷结构**（与其他地区不同）：

| 题型 | 内容 | 分值 | 特点 |
|------|------|------|------|
| **Part 1** | 听力 | 30分 | 短对话、长对话、短文 |
| **Part 2** | 语音/语法/词汇 | 40分 | 语音、词汇变形、语法选择 |
| **Part 3** | 阅读理解 | 50分 | A/B/C/D四篇 |
| **- A篇** | 应用文阅读 | 约12分 | 广告、通知、指南，3-4题选择题 |
| **- B篇** | 记叙文阅读 | 约12分 | 故事类，3-4题选择题 |
| **- C篇** | **首字母填空** | 14分 | ⚠️ **不是选择题！** 首字母提示填空(7空×2分) |
| **- D篇** | 回答问题 | 12分 | 阅读后回答问题(6题) |
| **Part 4** | 写作 | 20分 | 命题作文(80-100词) |

**⚠️ 常见错误警示**:

❌ **错误理解**: C篇是阅读理解选择题（这是全国卷题型）
✅ **正确理解**: 上海中考C篇是**首字母填空**（Cloze with initial letters）

**C篇特点**:
- 给出一篇150-200词的短文
- 7个空格，每空首字母已给出
- 需根据上下文和首字母填入正确单词
- 考点：词汇拼写、语法搭配、上下文逻辑

**搜索关键词对照**:
- C篇 / 首字母填空 / 首字母
- 不是：阅读理解 / 阅读C篇 / 选择题
