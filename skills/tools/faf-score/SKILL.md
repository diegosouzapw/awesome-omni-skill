---
name: faf-score
description: Calculate AI-readiness score (0-100%) for any project with project.faf file. Shows Podium tier (Trophy/Gold/Silver/Bronze), identifies strengths and gaps, provides improvement roadmap. Use when user asks "what's my score", "how AI-ready is this", "check my FAF quality", or wants to measure project context completeness.
allowed-tools: Read, Bash
---

# FAF Score - AI-Readiness Measurement

## Purpose

Calculate and explain the AI-readiness score for any project with a `project.faf` file. The score measures how well AI can understand your project from 0-100%, using the **Podium scoring system**.

**The Goal:** Give developers measurable, actionable feedback on project context quality with clear improvement paths.

## When to Use

This skill activates automatically when the user:
- Asks "What's my AI-readiness score?"
- Says "Check my FAF quality"
- Says "How AI-ready is this project?"
- Says "Score my project.faf"
- Asks "What tier am I at?"
- Wants to measure context completeness
- Just ran `faf init` and wants to see baseline score
- Asks "How can I improve my score?"

**Trigger Words:** score, AI-readiness, quality, tier, measure, check, rating, grade, Podium

## How It Works

### Step 1: Verify project.faf Exists

Check for `project.faf` in current directory:

```bash
ls -la project.faf
```

If not found, suggest running `faf init` first (use **faf-init** skill).

### Step 2: Execute faf score

Run the existing `faf score` command:

```bash
faf score
```

This command (from faf-cli v3.1.1):
- Analyzes project.faf completeness
- Calculates 0-100% score
- Assigns Podium tier
- Identifies strengths and gaps
- Suggests specific improvements

### Step 3: Interpret Results

Parse the output to identify:
- **Score percentage** (0-100%)
- **Podium tier** (Trophy/Gold/Silver/Bronze/etc.)
- **Strengths** (what's working well)
- **Gaps** (what's missing)
- **Next tier target** (what % needed)
- **Improvement suggestions**

### Step 4: Explain to User

Present results in clear, actionable format:

1. **Current Status:** "🥈 Silver (58%) - Good foundation"
2. **What This Means:** Explain their tier
3. **Strengths:** What they did well
4. **Gaps:** What's missing or incomplete
5. **Next Steps:** How to reach next tier
6. **Improvement Path:** Specific actions to take

### Step 5: Guide Next Actions

Based on score, suggest:
- **85%+ (Trophy):** Maintain quality, consider `faf bi-sync`
- **70-84% (Gold):** Minor enhancements, use `faf enhance`
- **55-69% (Silver):** Add missing sections, use `faf enhance`
- **40-54% (Bronze):** Significant gaps, use `faf enhance`
- **<40%:** Major work needed, consider starting over with `faf init`

## The Podium Scoring System

### Tier Breakdown

**🏆 Trophy (85-100%)**
- Elite AI-readiness
- Complete project DNA
- All critical sections present and detailed
- Architecture well documented
- Testing approach clear
- Dependencies fully specified
- **Meaning:** AI has near-perfect understanding

**🥇 Gold (70-84%)**
- Excellent AI-readiness
- Strong foundation with minor gaps
- Core architecture documented
- Most sections complete
- Some details could be enhanced
- **Meaning:** AI understands project very well

**🥈 Silver (55-69%)**
- Good AI-readiness
- Solid baseline established
- Core sections present
- Some important details missing
- Clear improvement path exists
- **Meaning:** AI has good understanding, some questions remain

**🥉 Bronze (40-54%)**
- Basic AI-readiness
- Essential information present
- Multiple gaps in documentation
- Significant room for improvement
- **Meaning:** AI has basic understanding, many questions

**🟢 Green (35-39%)**
- Minimal AI-readiness
- Project.faf exists but sparse
- Major sections missing or incomplete
- **Meaning:** AI understands project type, little else

**🟡 Yellow (20-34%)**
- Very limited AI-readiness
- Bare minimum information
- Almost everything needs work
- **Meaning:** AI knows very little about project

**🔴 Red (10-19%)**
- Critical AI-readiness gaps
- Project.faf nearly empty
- Urgent improvements needed
- **Meaning:** AI barely understands project

**🤍 White (<10%)**
- No effective AI-readiness
- Project.faf exists but provides no value
- Start over with `faf init`
- **Meaning:** AI has no meaningful context

### What Gets Scored

**Critical Sections (highest weight):**
- Project name and purpose
- Architecture type (web app, library, API, etc.)
- Primary language and framework
- Runtime requirements

**Important Sections (medium weight):**
- Dependencies and stack
- Testing framework and approach
- Build requirements
- File structure overview

**Enhancement Sections (lower weight):**
- Architecture decisions
- Design patterns
- API documentation
- Deployment information
- Team workflow notes

## Examples

### Example 1: New React Project (First Score)

**User:** "What's my AI-readiness score?"

**Skill Activates:**
1. Check `project.faf` exists ✓
2. Run `faf score`
3. Output:
   ```
   🥈 Silver (58%) - Good foundation

   Strengths:
   - Project type clearly identified (React web app)
   - TypeScript configuration documented
   - Framework version specified (React 18.2.0)
   - Build tools identified (Vite)

   Gaps:
   - Testing framework not documented
   - Architecture decisions missing
   - Deployment information absent
   - API endpoints not described

   Next tier: 🥇 Gold at 70%

   To reach Gold:
   - Add testing information (Jest/Vitest setup)
   - Document key architecture decisions
   - Include deployment target (Vercel/Netlify/etc.)
   ```

4. Explain to user:
   - "You're at Silver tier (58%) - this is a good starting point!"
   - "Your project.faf clearly identifies the React/TypeScript stack"
   - "To reach Gold (70%), focus on testing and architecture documentation"
   - "Run `faf enhance` for guided improvements"

### Example 2: Mature Next.js Project (High Score)

**User:** "Check my FAF quality"

**Skill Activates:**
1. Run `faf score`
2. Output:
   ```
   🥇 Gold (76%) - Excellent AI-readiness

   Strengths:
   - Comprehensive architecture documentation
   - Testing approach well defined (Jest + Playwright)
   - All dependencies documented
   - Deployment pipeline described
   - API routes documented
   - TypeScript strict mode noted

   Gaps:
   - Design patterns could be more detailed
   - Database schema documentation light
   - Performance considerations not mentioned

   Next tier: 🏆 Trophy at 85%

   To reach Trophy:
   - Add database schema documentation
   - Document key design patterns
   - Include performance optimization notes
   ```

3. Explain:
   - "Excellent! You're at Gold tier (76%)"
   - "Your project.faf provides comprehensive context"
   - "You're only 9 percentage points from Trophy tier"
   - "Focus on database and performance documentation to reach elite status"

### Example 3: Bare Minimum Project (Low Score)

**User:** "How AI-ready is this project?"

**Skill Activates:**
1. Run `faf score`
2. Output:
   ```
   🟡 Yellow (28%) - Very limited AI-readiness

   Strengths:
   - Project name present
   - Language identified (Python)

   Gaps:
   - Purpose/description missing
   - No architecture information
   - Framework not specified
   - Dependencies not documented
   - Testing approach absent
   - No file structure information

   Next tier: 🥉 Bronze at 40%

   To reach Bronze:
   - Add clear project purpose/description
   - Specify framework (Django/Flask/FastAPI)
   - Document key dependencies
   - Add testing framework information

   Consider: Run `faf enhance` for comprehensive improvements
   ```

3. Explain:
   - "Your score is 28% (Yellow tier) - there's significant room for improvement"
   - "The project.faf has minimal information right now"
   - "AI can barely understand what this project does"
   - "Run `faf enhance` to add critical missing sections"
   - "Target Bronze tier (40%) as first milestone"

### Example 4: Perfect Score (Trophy Tier)

**User:** "Score my project.faf"

**Skill Activates:**
1. Run `faf score`
2. Output:
   ```
   🏆 Trophy (89%) - Elite AI-readiness

   Strengths:
   - Complete project DNA present
   - Architecture thoroughly documented
   - Testing strategy comprehensive
   - All dependencies with versions
   - Deployment pipeline detailed
   - API documentation complete
   - Design patterns explained
   - Performance considerations noted
   - Database schema documented
   - File structure clear

   Minor enhancements:
   - Could add more API examples
   - Monitoring setup could be mentioned

   Status: Elite tier - maintain this quality!
   ```

3. Explain:
   - "Outstanding! You're at Trophy tier (89%)"
   - "Your project.faf provides elite-level AI context"
   - "AI can understand your project immediately and completely"
   - "This is championship-grade documentation"
   - "Maintain this quality with `faf bi-sync` to keep CLAUDE.md in sync"

## Understanding Your Score

### What Scores Mean in Practice

**85%+ (Trophy):**
- **AI Understanding:** Complete, immediate comprehension
- **Time Saved:** 95%+ (30 min → <30 seconds)
- **Accuracy:** AI suggests correct patterns first try
- **Questions:** Almost none, AI has full context
- **Maintenance:** Minimal, occasional updates only

**70-84% (Gold):**
- **AI Understanding:** Very good, minor clarifications needed
- **Time Saved:** 80-90% (30 min → 3-5 min)
- **Accuracy:** AI mostly correct, 1-2 questions
- **Questions:** Few, usually about edge cases
- **Maintenance:** Low, update on architecture changes

**55-69% (Silver):**
- **AI Understanding:** Good foundation, some gaps
- **Time Saved:** 60-75% (30 min → 7-12 min)
- **Accuracy:** AI needs some guidance
- **Questions:** Several about architecture/testing
- **Maintenance:** Medium, regular improvements helpful

**40-54% (Bronze):**
- **AI Understanding:** Basic only
- **Time Saved:** 30-50% (30 min → 15-20 min)
- **Accuracy:** AI makes assumptions, often wrong
- **Questions:** Many fundamental questions
- **Maintenance:** High, significant work needed

**<40%:**
- **AI Understanding:** Very limited
- **Time Saved:** Minimal (<25%)
- **Accuracy:** AI guesses frequently
- **Questions:** Constant clarification needed
- **Maintenance:** Critical, major overhaul required

### ROI by Tier

**Trophy (85%+):**
- 5 AI sessions/day × 28 min saved = 140 min/day
- 700 min/week = **11.7 hours/week saved**
- 50 weeks/year = **585 hours/year saved**

**Gold (70-84%):**
- 5 AI sessions/day × 23 min saved = 115 min/day
- 575 min/week = **9.6 hours/week saved**
- 50 weeks/year = **480 hours/year saved**

**Silver (55-69%):**
- 5 AI sessions/day × 18 min saved = 90 min/day
- 450 min/week = **7.5 hours/week saved**
- 50 weeks/year = **375 hours/year saved**

**Bronze (40-54%):**
- 5 AI sessions/day × 12 min saved = 60 min/day
- 300 min/week = **5 hours/week saved**
- 50 weeks/year = **250 hours/year saved**

Even Bronze tier saves significant time. Trophy tier is transformational.

## Improvement Paths

### From Yellow/Red to Bronze (20% → 40%)

**Priority Actions:**
1. Add clear project purpose (1-2 sentences)
2. Specify architecture type (web app/library/API/CLI)
3. Document primary language and framework
4. List key dependencies (top 5-10)
5. Add runtime requirements (Node version, Python version, etc.)

**Time Required:** 15-30 minutes
**Impact:** Basic AI understanding established

### From Bronze to Silver (40% → 55%)

**Priority Actions:**
1. Enhance architecture description (2-3 paragraphs)
2. Document testing framework and approach
3. Add file structure overview
4. Include build/deployment basics
5. Specify development setup steps

**Time Required:** 30-60 minutes
**Impact:** Good AI context foundation

### From Silver to Gold (55% → 70%)

**Priority Actions:**
1. Document architecture decisions (why choices were made)
2. Add comprehensive testing strategy
3. Include API documentation (if applicable)
4. Document database schema (if applicable)
5. Add deployment pipeline details
6. Include performance considerations

**Time Required:** 1-2 hours
**Impact:** Excellent AI understanding

### From Gold to Trophy (70% → 85%)

**Priority Actions:**
1. Document design patterns used
2. Add detailed API examples
3. Include monitoring/observability setup
4. Document security considerations
5. Add troubleshooting guides
6. Include team workflow notes
7. Document edge cases and gotchas

**Time Required:** 2-4 hours
**Impact:** Elite AI comprehension

## Verification & Troubleshooting

### Success Indicators

✅ Score calculated successfully (0-100%)
✅ Podium tier assigned correctly
✅ Strengths identified
✅ Gaps clearly listed
✅ Improvement suggestions provided
✅ Next tier target shown

### Common Issues

**Issue: `faf: command not found`**
```bash
# Solution: Install faf-cli
npm install -g faf-cli

# Verify
faf --version  # Should show v3.1.1 or later
```

**Issue: "No project.faf found"**
```bash
# Solution: Initialize first
faf init

# Then score
faf score
```

**Issue: Score seems wrong (too low)**
```bash
# Solution: Validate file format
faf validate

# Check for YAML syntax errors
# Fix any issues
# Re-run score
faf score
```

**Issue: Score stuck despite improvements**
```bash
# Solution: Check what's missing
faf score --verbose  # (if supported)

# Or use enhance for guided improvements
faf enhance
```

## Supporting Files

This skill works with:
- **faf-cli** (v3.1.1+) - Scoring engine
- **project.faf** - File being scored
- **Podium scorer** - 0-100% algorithm

## Related Skills

After scoring, users typically want:
- **faf-enhance** - Improve score through guided process
- **faf-validate** - Check format compliance
- **faf-sync** - Keep project.faf synced with CLAUDE.md
- **faf-init** - Regenerate if score is very low (<20%)

## Key Principles

**Measurable Progress:**
- Scores are quantifiable (0-100%)
- Tiers provide clear milestones
- Progress is visible (45% → 58% → 72%)
- Improvements are trackable

**Actionable Feedback:**
- Not just "score is low"
- Specific gaps identified
- Clear improvement path shown
- Next tier target provided

**NO BS ZONE:**
- Scores are honest (no inflation)
- Based on actual content analysis
- Transparent criteria
- Real improvement guidance

**Podium Philosophy:**
- Championship-grade standards (F1-inspired)
- Trophy tier = elite, not easy
- Every tier has value
- Progress celebrated at all levels

## Success Metrics

When this skill succeeds, users should:
1. Know their exact AI-readiness score (0-100%)
2. Understand their current Podium tier
3. See specific strengths and gaps
4. Know next tier target percentage
5. Have clear improvement roadmap
6. Feel motivated to enhance their score

## References

- **Podium Scoring System:** faf-cli scoring engine
- **faf score command:** https://faf.one/docs/scoring
- **AI-readiness tiers:** https://faf.one/docs/podium
- **Improvement guides:** https://faf.one/docs/enhance

---

**Generated by FAF Skill: faf-score v1.0.0**
**Podium Edition: Championship-Grade Measurement**
**"Measurable progress. Actionable feedback. Trophy-grade standards."**
