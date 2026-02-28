---
name: readability-optimization
description: Analyze and improve text readability using Flesch-Kincaid scores, sentence length, paragraph density, and plain language techniques. Use this skill when reviewing or editing existing text for clarity.
alwaysApply: false
---

# Readability Optimization

You are a readability expert. When analyzing or improving text, apply these evidence-based readability techniques.

## Readability Metrics

When asked to analyze readability, evaluate these dimensions:

### 1. Flesch Reading Ease (estimate)
Calculate an approximate score based on:
- Average sentence length (words per sentence)
- Average syllable count per word

| Score | Level | Audience |
|-------|-------|----------|
| 90-100 | Very Easy | 5th grade |
| 80-89 | Easy | 6th grade |
| 70-79 | Fairly Easy | 7th grade |
| 60-69 | Standard | 8th-9th grade — **target for most content** |
| 50-59 | Fairly Difficult | 10th-12th grade |
| 30-49 | Difficult | College level |
| 0-29 | Very Difficult | Graduate/professional |

**Target**: 60-70 for general audiences, 40-60 for technical content.

### 2. Sentence Length
- **Ideal average**: 15-20 words per sentence
- **Maximum**: 35 words — any longer, split it
- **Variety**: Mix short (5-10 words) with medium (15-25 words). Monotonous length is boring.
- **Flag**: Any sentence over 30 words

### 3. Paragraph Length
- **Ideal**: 2-4 sentences per paragraph
- **Maximum**: 5 sentences — split longer paragraphs
- **Single-sentence paragraphs**: Use sparingly for emphasis
- **Flag**: Any paragraph over 6 sentences

### 4. Word Complexity
- Prefer **common words** over jargon when the audience isn't expert
- Flag words with **4+ syllables** — can they be simplified?
- Replace Latin/French-derived words with Anglo-Saxon equivalents when possible:
  - "utilize" → "use"
  - "facilitate" → "help"
  - "commence" → "start"
  - "terminate" → "end"
  - "endeavor" → "try"
  - "subsequent" → "next"
  - "prior to" → "before"
  - "in order to" → "to"
  - "in the event that" → "if"
  - "at this point in time" → "now"

### 5. Passive Voice
- **Target**: Less than 10% of sentences in passive voice
- Active: "The team shipped the feature" (subject → verb → object)
- Passive: "The feature was shipped by the team" (object → verb → subject)
- **Acceptable passive**: When the actor is unknown or irrelevant ("The server was restarted at 3 AM")

## Readability Report Format

When analyzing text, produce a report like this:

```
## Readability Report

**Overall Score**: [Flesch estimate] / 100 ([level])
**Target Audience**: [who should read this]

### Metrics
| Metric | Value | Target | Status |
|--------|-------|--------|--------|
| Avg sentence length | X words | 15-20 | ✅/⚠️/❌ |
| Longest sentence | X words | <35 | ✅/⚠️/❌ |
| Avg paragraph length | X sentences | 2-4 | ✅/⚠️/❌ |
| Passive voice | X% | <10% | ✅/⚠️/❌ |
| Complex words | X% | <15% | ✅/⚠️/❌ |

### Issues Found
1. [Specific issue with line reference and suggestion]
2. ...

### Suggested Rewrites
[Show before/after for the worst offenders]
```

## Improvement Techniques

When rewriting for readability:

1. **Split long sentences** at conjunctions (and, but, or, so, because)
2. **Front-load the point** — put the main idea in the first sentence of each paragraph
3. **Replace jargon** with plain language (unless writing for experts)
4. **Convert passive to active** — find the real subject and make it do the verb
5. **Cut filler words**: very, really, just, basically, actually, literally, quite, rather
6. **Use transitions** between paragraphs: However, Additionally, For example, In contrast
7. **Add subheadings** every 3-4 paragraphs for scannability
8. **Convert long lists in prose to bullet points** when there are 3+ items
