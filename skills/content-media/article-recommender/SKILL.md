---
name: article-recommender
description: Generate three-version article recommendations (standard, concise, and personal commentary) in both Chinese and English for BestBlogs.dev weekly newsletter. Use when users request article recommendations，推荐语，推荐理由，or ask to write recommendations for newsletter content. Triggered by phrases like "帮我编写推荐理由", "生成推荐语", "write a recommendation", or when presenting curated content.
---

# Article Recommender

## Overview

This skill generates professional article recommendations in three distinct versions (standard, concise, personal commentary) with both Chinese and English outputs, specifically designed for BestBlogs.dev's weekly newsletter format.

## Core Principles

### Target Audience
- **Reader Level**: Intermediate to professional/expert level
- **Content Domains**: Artificial Intelligence, Product Development, Business, Programming
- **Content Formats**: Articles, videos, podcasts, tweets, and other formats

### Three-Version Framework

**Version 1: 推荐版本 (Standard Recommendation)**
- Comprehensive overview of content value
- Highlights core insights and key points
- Balanced, professional tone
- 150-200 Chinese characters, 80-120 English words
- Suitable for most newsletter readers

**Version 2: 精炼简洁版本 (Concise Version)**
- Direct and to-the-point
- Essential information only
- 80-120 Chinese characters, 50-80 English words
- Perfect for quick scanning

**Version 3: 个人评论版本 (Personal Commentary)**
- Objective and direct critical analysis
- Points out potential limitations and areas requiring further thought
- Raises questions worth discussing
- May include constructive skepticism
- 180-250 Chinese characters, 100-150 English words
- Adds depth and encourages critical thinking

### Content Quality Standards

**What to Include**:
- Article's core value proposition
- Main arguments and insights
- Target audience relevance
- Practical implications
- Unique perspective or approach

**What to Avoid**:
- Unnecessary quotes and parentheses
- Excessive emphasis (过多的强调标记)
- Complex long sentences that impede reading
- Colloquial transitions (不必要的口语和过渡)
- Exaggerated or pretentious language
- Making claims not supported by article content

### Language and Formatting Requirements

**Chinese Text**:
- Use authentic, natural Chinese expressions (地道的中文表达)
- **Must use full-width Chinese punctuation**: 。，、；：？！（not half-width .,;:?!）
- Use proper quotation marks: "" or 「」（not half-width ""）
- Add space between Chinese and English/numbers (中英文、数字之间增加空格)
- **No spaces around full-width punctuation** (全形标点前后不加空格)
- Follow Chinese typography standards
- Avoid Chinglish patterns

**English Text**:
- Use idiomatic English expressions
- Apply proper English punctuation
- Natural, professional tone
- Avoid overly complex academic language

**Technical Terminology**:
Keep English terms when they are:
- Universal and commonly used (API, SDK, GitHub, LLM, RAG, CI/CD)
- Easy to understand without translation
- Difficult to replace with appropriate Chinese equivalents
- Industry standard terminology

Use Chinese when:
- Clear Chinese translation exists and is widely accepted
- Chinese expression is more natural and fluent
- The term is not technical jargon (e.g., 机器学习 for machine learning)

Mixed usage examples:
- ✓ API 接口设计
- ✓ LLM 应用开发
- ✓ 持续集成 CI/CD 流程
- ✓ 机器学习模型优化

**Markdown Formatting**:
- Use proper Markdown syntax
- Ensure no spaces before/after that break rendering (避免因为前后的空格导致无法渲染)
- Clear section headers for each version
- Separate Chinese and English within each version

## Workflow

### Step 1: Analyze Input

**Required Information**:
- Article summary/摘要
- Main points/主要观点
- Key quotes/核心金句 (if available)
- Full article content (if provided)

**If missing critical information**:
Ask for:
- Article's core argument
- Target audience
- Key insights or takeaways

### Step 2: Understand Context

**Identify Content Domain and Format**:
Load `references/domain_format_guidelines.md` for domain and format-specific guidance.

- **Domain**: AI, Product, Business, or Programming?
- **Format**: Article, video, podcast, tweet/thread, or paper?
- **Cross-domain**: Does it span multiple domains?

**Quick Assessment**:
- What makes this content valuable?
- Who benefits from reading/watching/listening?
- What's unique or noteworthy?
- Does it offer actionable insights?
- Any limitations or caveats worth noting?
- What's the reader's time investment? (especially for videos/podcasts)

### Step 3: Craft Three Versions

**For Version 1 (推荐版本)**:
1. Lead with article's core value
2. Highlight 2-3 key insights
3. Explain practical relevance
4. Maintain balanced, informative tone
5. End with target audience fit

**For Version 2 (精炼简洁版本)**:
1. One-sentence core value
2. List 1-2 main points
3. Brief practical implication
4. Direct, no-frills language

**For Version 3 (个人评论版本)**:
1. Start with personal observation or critical hook
2. Present main content objectively
3. **Add objective, direct critical analysis**:
   - Point out potential limitations or gaps
   - Identify areas requiring further thought
   - Raise questions worth discussing
   - Note what the content doesn't address
   - Highlight assumptions that may not hold universally
4. Provide constructive recommendations or context
5. Maintain professional tone while being direct
6. **Criticism should be**:
   - Based on objective analysis, not personal preference
   - Constructive and thoughtful, not dismissive
   - Specific about what's missing or questionable
   - Balanced with recognition of value where it exists

**Example critical elements for Version 3**:
```
Chinese:
"文章的分析很全面，但有几个现实问题没有深入讨论：首先是学习曲线..."
"这个方案看起来很理想，但实际应用中可能面临..."
"作者的观点有道理，但忽略了一个关键问题..."
"值得注意的是，文章基于的假设在某些场景下可能不成立..."

English:
"The analysis is comprehensive, but several practical issues aren't deeply discussed..."
"This solution looks ideal, but in practice may face..."
"The author's point is valid, but overlooks a key issue..."
"Worth noting that the assumptions may not hold in certain scenarios..."
```

### Step 4: Bilingual Output

**Technical Terminology**:
Load `references/terminology_guidelines.md` for detailed guidance on handling technical terms.

Apply the principle:
- Keep English for: universal, commonly-used, easy-to-understand terms with no good Chinese equivalent
- Use Chinese for: terms with clear, natural Chinese translations
- Maintain consistency across all three versions

For each version:
1. Write Chinese version first (typically more challenging)
2. Create English version (not direct translation, but equivalent)
3. Ensure both convey same core message with language-appropriate expressions
4. Apply proper formatting for each language

### Step 5: Quality Check

**Content**:
- ✓ Information accurate and based on article
- ✓ Clear structure and flow
- ✓ Engaging but not exaggerated
- ✓ All three versions serve distinct purposes

**Language**:
- ✓ Natural, idiomatic expressions
- ✓ Chinese uses full-width punctuation (。，、；：？！) not half-width (.,;:?!)
- ✓ Spaces added correctly in Chinese (中英文、数字之间有空格)
- ✓ No spaces around full-width punctuation (全形标点前后不加空格)
- ✓ No Markdown rendering issues

**Tone**:
- ✓ Professional yet accessible
- ✓ Version 3 shows personality without being overly casual
- ✓ Appropriate depth for newsletter format

## Output Template

Use this template structure:

```markdown
## 版本一：推荐版本

### 中文

[Standard Chinese recommendation]

### English

[Standard English recommendation]

---

## 版本二：精炼简洁版本

### 中文

[Concise Chinese recommendation]

### English

[Concise English recommendation]

---

## 版本三：个人评论版本

### 中文

[Personal commentary in Chinese]

### English

[Personal commentary in English]
```

## Examples of Good Practices

**Strong opening (Version 1)**:
- ✓ "GitHub 官方发布的 Copilot 代码审查指令文件实战指南"
- ✗ "这篇文章介绍了关于 GitHub Copilot 的一些内容"

**Conciseness (Version 2)**:
- ✓ "GitHub 团队总结的 Copilot 代码审查指令编写最佳实践"
- ✗ "这是一篇由 GitHub 团队编写的、关于如何更好地使用 Copilot 进行代码审查的最佳实践指南"

**Personal voice (Version 3)**:
- ✓ "这篇文章来得正是时候。很多团队兴冲冲地写了一堆指令文件，结果发现 AI 根本不按套路出牌"
- ✗ "这篇文章很好地解决了用户在使用过程中可能遇到的问题"

## Common Pitfalls to Avoid

1. **使用英文标点**: Chinese text MUST use full-width punctuation
   - ✗ 这篇文章介绍了API设计的核心原则,包括一致性、简洁性和可扩展性.
   - ✓ 这篇文章介绍了 API 设计的核心原则，包括一致性、简洁性和可扩展性。
   - ✗ GitHub团队发布了新功能!开发者可以更方便地进行代码审查了.
   - ✓ GitHub 团队发布了新功能！开发者可以更方便地进行代码审查了。

2. **标点前后加空格**: No spaces around full-width punctuation
   - ✗ 刚刚买了一部 iPhone ，好开心 ！
   - ✓ 刚刚买了一部 iPhone，好开心！

3. **过度引用**: Don't use excessive quotation marks
   - ✗ "AI"模型需要"结构化"的"短指令"
   - ✓ AI 模型需要结构化的短指令

4. **括号滥用**: Minimize parenthetical asides
   - ✗ GitHub 团队 (基于大量用户反馈) 提供了 (经过验证的) 编写策略
   - ✓ GitHub 团队基于大量用户反馈，提供了经过验证的编写策略

5. **口语过渡**: Avoid unnecessary conversational fillers
   - ✗ 那么，让我们来看看这篇文章主要讲了什么
   - ✓ 文章主要介绍了...

6. **浮夸表述**: Keep praise measured and credible
   - ✗ 这是史上最强大的、绝无仅有的、不可或缺的指南
   - ✓ 这是一份不可多得的实操手册

7. **空格错误**: Ensure proper spacing in Chinese
   - ✗ GitHub官方发布的Copilot代码审查指南
   - ✓ GitHub 官方发布的 Copilot 代码审查指南

## Notes for BestBlogs.dev Context

- Audience: Developers interested in programming, AI, product development
- Tone: Professional but accessible, technical but not academic
- Focus: Practical value and actionable insights
- Format: Weekly newsletter curation
- Goal: Help readers quickly decide if article is worth their time
- Balance: Informative without spoiling the content

## Integration with Other Skills

- **deep-reading-analyst**: Can use analysis results as input
- **content-synthesizer**: Shares voice preservation principles
- May reference user's reading history or preferences from memory
