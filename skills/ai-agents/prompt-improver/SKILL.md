---
name: prompt-improver
description: Improve prompts for AI agents and Telegram bots using OpenAI's prompt engineering best practices. Analyzes clarity, specificity, context, and output format. Returns structured improvements.
---

# Prompt Improver Skill

**Version:** 1.0  
**Domain:** AI Prompting & Bot Communication  
**Focus:** Optimize prompts for Telegram bots, increase clarity and response quality

## Overview

The Prompt Improver skill enhances prompts designed for bots by applying OpenAI's prompt engineering best practices, domain-specific optimizations, and bot-communication patterns. It analyzes prompts against a systematic framework and provides structured improvements.

## Core Capabilities

### 1. **Prompt Analysis Framework**
- **Clarity Assessment** - Measures clarity on 1-10 scale
- **Specificity Check** - Evaluates technical detail level
- **Context Sufficiency** - Validates background information
- **Role Definition** - Confirms actor/persona clarity
- **Output Format** - Checks expected response structure
- **Constraint Identification** - Lists limitations and requirements

### 2. **Improvement Strategies**

#### ✅ Clarity Enhancements
- Remove ambiguous language
- Define technical terms inline
- Use concrete examples
- Replace vague phrases with specific actions

#### ✅ Specificity Optimization
- Add measurable outcomes
- Include input/output examples
- Define edge cases to handle
- Specify tone and style

#### ✅ Bot-Specific Optimizations
- Telegram command syntax (`/start`, `/help`)
- Message formatting (markdown, buttons, keyboards)
- Conversation flow patterns
- Error handling strategies
- Response length constraints (4096 char limit)

#### ✅ Context Enrichment
- Suggest background information
- Include relevant examples
- Define expected user scenarios
- Add domain-specific terminology

### 3. **Analysis Output Structure**

```markdown
## 📊 ORIGINAL PROMPT ANALYSIS
- Clarity Score: [1-10]
- Specificity Level: [Low/Medium/High]
- Context Richness: [Insufficient/Adequate/Rich]
- Bot Compatibility: [⚠️ Issues / ✅ Compatible]
- Key Issues: [List]

## 🎯 IMPROVEMENT RECOMMENDATIONS
1. [Specific improvement with reasoning]
2. [Next improvement]
...

## ✨ IMPROVED PROMPT
[Optimized version]

## 📝 EXPLANATION
- Why changes were made
- Expected quality improvement
- Bot-specific considerations
```

## Usage Patterns

### Pattern 1: Bot Command Enhancement
```
User: "Improve this prompt for my Telegram bot"
[Prompt about listing products]

Skill Response:
- Analysis of current prompt
- Bot-specific suggestions (keyboard layout, response format)
- Improved version with Telegram best practices
```

### Pattern 2: Conversation Flow Design
```
User: "Design a conversational flow for..."
Skill provides:
- Multi-turn conversation structure
- Button/keyboard layouts
- Error handling prompts
- Exit strategies
```

### Pattern 3: AI Integration Optimization
```
User: "Improve this prompt for Gemini/ChatGPT API calls"
Skill provides:
- Model-specific optimizations
- Token efficiency improvements
- System prompt + user prompt separation
- Temperature/parameter suggestions
```

## Bot-Specific Best Practices

### Telegram Bots
- **Message Limits:** 4096 characters per message
- **Markdown:** Use `**bold**`, `_italic_`, `` `code` ``
- **Keyboards:** Suggest inline buttons or reply keyboards
- **Callbacks:** Design stateless callback handlers
- **Rate Limits:** Consider API rate limiting

### WhatsApp Bots (WAHA)
- **Templates:** Use message templates for broadcasts
- **Media:** Support image/document responses
- **Formatting:** Limited markdown support
- **Buttons:** Button payloads max 256 chars

### Multi-Platform Coordination
- **Prompt Versioning:** Different prompts per platform
- **Fallbacks:** Handle unsupported features gracefully
- **Context Preservation:** Maintain conversation state

## Integration Points

### With Telegram UI Design Skill
- Complements keyboard/button design
- Provides instruction text for UI elements
- Optimizes for message formatting constraints

### With Backend Services
- Prepares prompts for API calls
- Structures responses for database storage
- Defines error handling responses

## Evaluation Metrics

| Metric | Goal | Measurement |
|--------|------|------------|
| Clarity | +3 points | User understanding increase |
| Specificity | Clear outputs | Unambiguous bot response |
| Token Efficiency | -20% tokens | Reduced API costs |
| Bot Compatibility | 0 errors | No formatting violations |

## Templates & Examples

### Template: System Prompt for Affiliate Bot
```
You are an Affiliate Product Recommendation Bot for [Your Platform Name].
Your role: Help users find the best deals on products.
Constraints: Keep responses under 300 words, max 2 product recommendations per message.
Format: Use inline buttons [View Deal] [Add to List] [Share]
Tone: Friendly, helpful, professional
Error Handling: If product not found, suggest similar categories
```

### Template: Conversation Flow for Product Search
```
[START] → Ask category preference
  ↓
[CATEGORY_SELECT] → Show category buttons
  ↓
[FILTER_OPTIONS] → Price range, brand, rating
  ↓
[RESULTS] → Display products with inline buttons
  ↓
[DETAIL_VIEW] → Full product info with affiliate link
  ↓
[ACTION] → Add to list, share, or new search
```

## Command Patterns for Automation

```bash
# Improve prompt for telegram
@prompt-improver Optimize this prompt: "List products"

# Design conversation flow
@prompt-improver Design a product search flow for Telegram

# API integration prep
@prompt-improver Prepare prompt for Gemini API integration

# Error handling
@prompt-improver Improve error messages for this bot
```

## Advanced Features

### 1. **Prompt Versioning**
- Track prompt iterations
- Compare performance metrics
- A/B test different versions

### 2. **Domain-Specific Libraries**
- Affiliate marketing prompts
- E-commerce product descriptions
- Customer service patterns
- Content generation templates

### 3. **Performance Tuning**
- Optimize for response latency
- Reduce token consumption
- Improve accuracy metrics
- Measure user satisfaction

## References & Learning Resources

- [OpenAI Prompt Engineering Guide](https://platform.openai.com/docs/guides/prompt-engineering)
- [Telegram Bot API Documentation](https://core.telegram.org/bots/api)
- [Claude Prompt Engineering](https://claude.ai/resources)
- [WAHA Documentation](https://waha.dev/)

## Changelog

- **v1.0** (2025-12-19): Initial release with Telegram/WhatsApp optimization, conversation flow design, API integration prep

---

**Status:** ✅ Production Ready  
**Maintenance:** Active  
**Last Updated:** December 19, 2025
