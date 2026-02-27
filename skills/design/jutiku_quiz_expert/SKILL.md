---
name: Jutiku_Quiz_Expert
description: 专门用于从文档生成结构化试题的智能体。分析内容属性（大纲型 vs 知识型），提取关键点，并生成 JSON 或 Markdown 格式的高质量试题。当用户要求"根据文件出题"、"创建测验"、"制作试卷"或"提取考题"时使用此技能。
version: 1.2
author: budaobu
---

# 出题专家 (Quiz Expert)

本技能将引导你完成从源文档生成专业题库的全过程。

## 路径约定

在开始执行任务前，先确定关键路径：

- **$WORK_DIR**：当前工作目录。执行 `pwd` (Linux/macOS) 或 `Get-Location` (Windows) 获取，作为所有相对路径的基准。
- **$SKILL_DIR**：本 skill 所在目录。根据 skill 名称 `Jutiku_Quiz_Expert` 定位：
  - **Windows**: `Get-ChildItem -Path . -Filter "SKILL.md" -Recurse | Where-Object { $_.FullName -like "*skills\Jutiku_Quiz_Expert*" }`
  - **macOS/Linux**: `find . -ipath "*/skills/Jutiku_Quiz_Expert/SKILL.md"`
  - 获取到 SKILL.md 文件路径后，取其所在目录作为 $SKILL_DIR
- **源文件目录**：`$WORK_DIR/`（用户上传的文件直接放在工作目录根部）
- **转换文件目录**：`$WORK_DIR/source/`（存放 markitdown 转换后的 .md 文件）
- **输出目录**：`$WORK_DIR/quiz/`（生成的题库文件，直接交付给用户）
- **引用文件**：使用相对于 $SKILL_DIR 的路径，如 `$SKILL_DIR/references/QUIZ_JSON.md`

路径结构示意：
```
$WORK_DIR/                       # 当前工作目录
├── 2023年度报告.pdf             # 用户源文件
├── source/                      # 转换后的 markdown 文件
│   └── 2023年度报告.md
├── quiz/                        # 输出目录（直接交付）
│   └── 2024-01-29_100000.json  # 或 .md（根据用户指定）
└── temp/                        # 存放其他临时处理文件

$SKILL_DIR/                      # skill 所在目录
├── SKILL.md             # skill 文件，即本文件
└── references/
    ├── QUIZ_JSON.md
    └── QUIZ_MARKDOWN.md
```

---

## 工作流程 (Workflow)

### 0. 初始化路径
1. **确定工作目录**：
   - Linux/macOS: 执行 `pwd` 获取当前工作目录，设为 `$WORK_DIR`
   - Windows: 执行 `Get-Location` 或 `cd` 获取当前工作目录
   
2. **定位 skill 目录**：
   - **Linux/macOS**: 
     ```bash
     SKILL_PATH=$(find . -ipath "*/skills/Jutiku_Quiz_Expert/SKILL.md" 2>/dev/null | head -1)
     SKILL_DIR=$(dirname "$SKILL_PATH")
     ```
   - **Windows PowerShell**:
     ```powershell
     $skillFile = Get-ChildItem -Path . -Filter "SKILL.md" -Recurse | Where-Object { $_.FullName -like "*skills\Jutiku_Quiz_Expert*" } | Select-Object -First 1
     $SKILL_DIR = $skillFile.DirectoryName
     ```

3. **创建必要的子目录**：
   - Linux/macOS: `mkdir -p "$WORK_DIR/source" "$WORK_DIR/quiz"`
   - Windows: `New-Item -ItemType Directory -Force -Path "$WORK_DIR\source", "$WORK_DIR\quiz"`

### 1. 文件定位与准备
1.  **定位源文件**：在 `$WORK_DIR/` 根目录下查找用户指定的文件。
    - 使用 `ls $WORK_DIR/` 列出可用文件。
    - 如果用户提供的是模糊名称，查找最匹配的文件。
    
2.  **格式转换**：
    - 如果文件**不是** `.md` 格式（例如 .doc, .docx, .pdf）：
        - **环境检查**：检查是否已安装 `python` (或 `python3`) 和 `pip`。
            - **如果未安装 Python**：
                - **主动询问安装**：向用户说明缺少 Python 环境无法转换文件，询问：“是否允许我为您尝试自动安装 Python？”
                - **若用户同意**：
                    - **Windows**: 尝试运行 `winget install -e --id Python.Python.3` (需管理员权限)。
                    - **macOS**: 尝试运行 `brew install python` (需已安装 Homebrew)。
                    - **Linux**: 尝试运行 `sudo apt update && sudo apt install -y python3 python3-pip`。
                - **若用户拒绝或自动安装失败**：
                    - 暂停任务。
                    - 输出手动安装指南，建议用户访问 [python.org](https://www.python.org/downloads/) 或使用系统包管理器安装，完成后再次触发。
        - **工具检查与安装**：
            - 运行 `markitdown --version` 检查是否已安装。
            - 如果未安装：
              - Linux: `pip install markitdown --break-system-packages`
              - macOS/Windows: `pip install markitdown`
        - **转换**：
            - 使用 `markitdown` 命令行工具将文件转换为 Markdown 格式。
            - **命令格式**：
              - Linux/macOS: `markitdown "$WORK_DIR/{源文件名}" > "$WORK_DIR/source/{同文件名}.md"`
              - Windows: `markitdown "$WORK_DIR\{源文件名}" > "$WORK_DIR\source\{同文件名}.md"`
            - 注意：使用双引号包裹文件名以处理空格。
    - 如果已经是 `.md` 文件，复制到 `$WORK_DIR/source/{文件名}.md` 进行处理。

### 2. 内容预处理与分析
1.  **读取规则文档**：
    - 根据用户指定的输出格式，读取对应的规范文档：
      - **JSON 格式**：读取 `$SKILL_DIR/references/QUIZ_JSON.md` 了解 JSON Schema 规范
      - **Markdown 格式**：读取 `$SKILL_DIR/references/QUIZ_MARKDOWN.md` 了解 Markdown 模板规范

2.  **读取与分块检查**：
    - 读取 `$WORK_DIR/source/` 中转换后的 `.md` 文件内容。
    - **智能分块 (Smart Chunking)**：检查文本长度。
        - **如果 字符数 > 15,000**：
            - **切分策略**：按 Markdown 二级标题 (`##`) 或三级标题 (`###`) 将文档切分为多个逻辑块 (Chunks)。确保切分点不破坏段落完整性。
            - **配额分配**：根据各块的篇幅比例或重要性，分配题目生成数量（如总共 20 题，A 章节占 20% 篇幅，则分配 4 题）。
            - **执行模式**：标记为 `Batch_Mode`，后续将对每个块独立生成，最终统一进入质量优化阶段进行合并去重。
        - **否则**：保持单一大文本，标记为 `Single_Mode`。

3.  **属性打分**：分析文本结构和内容（若是分块模式，取前 30% 内容或摘要进行分析），判断其类型（分值 0.0 - 1.0）：
    - **大纲型 (Outline-oriented)**：具有结构化的标题、要点、简短摘要。
    - **知识/资料型 (Knowledge/Data-oriented)**：密集的段落、详细的解释、事实、数据。

4.  **策略选择**：
    - **如果 大纲型得分 > 0.7**：使用 **大纲扩展模式**。
        - 提取大纲结构。
        - 针对每个关键点/标题，生成涵盖核心概念的题目。
    - **如果 知识型得分 > 0.7**：使用 **全覆盖模式**。
        - 分析全文内容（或逐个处理块）。
        - 生成覆盖整个内容范围的题目，确保没有遗漏重要信息。
    - **否则**：使用平衡方法。

### 3. 题目生成规则
根据选定的策略和输出格式生成题目，严格遵守对应规范文档中定义的**生成规则与约束**。

主要约束包括：
- **题型**：单选、多选、判断、填空、简答。
- **判断题**：选项固定为"A. 正确"和"B. 错误"。
- **难度**：按 3:5:2 比例分布简单、中等、困难题目。
- **结构**：确保所有必填字段完整（qid, type, explanation 等）。

### 4. 质量验证与优化 (Quality Assurance & Optimization)
在生成题目之后（特别是 Batch_Mode 下合并所有分块题目后），执行以下优化流程：

1.  **全局去重 (Global Deduplication)**：
    - **语义比对**：检查所有生成的题目。如果两道题考察相同的知识点且问法高度相似，保留描述更清晰或难度更符合当前分布要求的一道。
    - **覆盖率检查**：确认合并后的题库是否覆盖了文档的所有关键章节。如果某章节题目被大量去重导致缺失，需针对该章节补充生成。

2.  **分布再平衡 (Distribution Rebalancing)**：
    - 统计当前题库的**难度比例**。如果偏离 3:5:2 严重，优先删除过多的难度层级题目，或尝试修改题目难度。
    - 统计**题型分布**。确保符合用户期望（如有）。

3.  **准确性核查 (Fact Check)**：
    - 针对每一道题，**回溯原文**。
    - 验证：正确答案是否能从原文中找到明确依据？
    - 修正：如果发现幻觉或推断过度，根据原文修正答案或删除该题。

4.  **逻辑性核查 (Logic Check)**：
    - 检查选项：是否存在歧义？干扰项是否过于明显错误？
    - 检查解析：`explanation` 是否清晰解释了为什么选 A 而不选 B？

5.  **格式核查 (Format Check)**：
    - 验证输出结构是否符合对应的 Schema 或模板规范。
    - 确保所有必需字段 (`qid`, `lev`, `point` 等) 均已填充且类型正确。

### 5. 输出与交付
根据用户指定的格式（默认 JSON）在 `$WORK_DIR/quiz/` 目录下生成文件。

#### JSON 格式（默认）
- **路径**：`$WORK_DIR/quiz/{YYYY-MM-DD_HHMMSS}.json`
- **格式**：遵循严格的 JSON 语法。
- **Schema**：参见 `$SKILL_DIR/references/QUIZ_JSON.md`

#### Markdown 格式（可选）
- **路径**：`$WORK_DIR/quiz/{YYYY-MM-DD_HHMMSS}.md`
- **格式**：用户友好的可读格式。
- **模板**：参见 `$SKILL_DIR/references/QUIZ_MARKDOWN.md`

#### 交付方式
文件保存到 `$WORK_DIR/quiz/` 后，用户可直接前往查看。

### 6. 响应模板 (Response Template)

任务成功完成后,请参照以下模板格式向用户汇报结果:

---

已根据文件 `{源文件名}` 的内容生成了 `{题目数量}` 道题目。

{处理说明}(例如:由于源文件为二进制 .doc 格式,已通过智能提取算法获取关键文本信息,并据此生成了包含{题型列表}的完整试卷。)

**生成文件信息:**
- **文件路径**:`{输出文件绝对路径}`
- **题目数量**:`{题目数量}` 道
- **包含内容**:
  - {内容概览点1}(例如:常识判断、核心概念等)
  - {内容概览点2}
  - {内容概览点3}
  - 最多列出10个

您可以直接使用该 `{文件格式}` 文件,或告知如果需要 Markdown 可读格式。

---

## 工具与命令
- 使用 `pwd` / `Get-Location` 获取路径。
- 使用 `view` / `ls` / `dir` 查看目录或文件。
- 使用 `bash_tool` 或 PowerShell 执行命令。
- 使用 `create_file` 保存最终输出文件。


## 用户请求示例

### 示例 1：默认 JSON 格式
> "根据 `2023年度报告.pdf` 生成 20 道题目"

执行流程：
1. **初始化**：获取 $WORK_DIR 和 $SKILL_DIR，创建子目录，识别输出格式为 JSON（默认）
2. **读取规范**：`view $SKILL_DIR/references/QUIZ_JSON.md`
3. **查找**：在 `$WORK_DIR/` 找到 `2023年度报告.pdf`
4. **转换**：`markitdown "$WORK_DIR/2023年度报告.pdf" > "$WORK_DIR/source/2023年度报告.md"`
5. **分析**：数据密集 → 知识型模式
6. **生成**：创建 20 道题目（财务统计、战略目标等）
7. **输出**：保存到 `$WORK_DIR/quiz/2024-01-29_100000.json`
8. **响应**:按照响应模板向用户汇报结果

**响应示例:**

已根据文件 `2023年度报告.pdf` 的内容生成了 20 道题目。

由于源文件为 PDF 格式,已通过 markitdown 工具提取文本内容,并据此生成了包含单选题、多选题、判断题的完整试卷。

**生成文件信息:**
- **文件路径**:`$WORK_DIR/quiz/2024-01-29_100000.json`
- **题目数量**:20 道
- **包含内容**:
  - 财务数据分析(营收、利润、增长率等)
  - 战略目标与业务规划
  - 风险管理与合规要求

您可以直接使用该 JSON 文件,或告知如果需要 Markdown 可读格式。

### 示例 2：指定 Markdown 格式
> "根据 `Python入门.docx` 生成 15 道题目，输出为 markdown 格式"

执行流程：
1. **初始化**：获取路径，创建子目录，识别输出格式为 Markdown
2. **读取规范**：`view $SKILL_DIR/references/QUIZ_MARKDOWN.md`
3. **查找与转换**：处理 `Python入门.docx`
4. **分析**：大纲结构 → 大纲扩展模式
5. **生成**：创建 15 道题目
6. **输出**：保存到 `$WORK_DIR/quiz/2024-01-29_100000.md`
7. **响应**:按照响应模板向用户汇报结果

**响应示例:**

已根据文件 `Python入门.docx` 的内容生成了 15 道题目。

源文件采用大纲式结构,已针对各章节关键知识点生成了包含单选题、填空题、简答题的试卷。

**生成文件信息:**
- **文件路径**:`$WORK_DIR/quiz/2024-01-29_100000.md`
- **题目数量**:15 道
- **包含内容**:
  - 基础语法(变量、数据类型、运算符)
  - 控制流程(条件判断、循环结构)
  - 函数与模块(定义、调用、导入)

您可以直接使用该 Markdown 文件,或告知如果需要 JSON 结构化格式。