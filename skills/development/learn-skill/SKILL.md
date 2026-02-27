---
name: learn-skill
description: Interactive learning system that researches any coding or development topic and presents it as a structured learning path with progressive difficulty levels (Foundations, Intermediate, Advanced) and use-case guides. Use when the user wants to learn about programming languages, frameworks, development tools, software architecture patterns, coding practices, or any technical development topic. Triggers on requests like "teach me about X", "help me learn Y", "I want to understand Z", or direct invocation via /learn-skill.
---

# Learn Skill

Create interactive, structured learning experiences for any coding or development subject by researching it online and presenting it as a progressive learning path.

## Overview

This skill transforms any technical learning request into a structured, categorized learning experience similar to an AI productivity coach. When invoked, the skill:

1. Checks for existing learning progress in `~/.claude-learn-skill/[subject-name]/`
2. Uses subagents to research the subject comprehensively online (if not already researched)
3. Breaks the subject into logical topics across difficulty levels
4. Organizes content into learning paths and use-case guides
5. Presents a table of all available topics with recommendations
6. **For each topic:** Conducts detailed, personalized research based on user context before teaching
7. Saves all research (subject-level and topic-level), guides, and progress to the learning directory
8. Tracks which topics the user has completed and maintains detailed user context

All learning materials and progress are persisted to `~/.claude-learn-skill/[subject-name]/` so learning can be resumed across sessions.

## Terminology

- **Subject**: The main thing being learned (e.g., "React Hooks", "TypeScript", "Stock Trading")
- **Topic**: Individual learning unit within the subject (e.g., "01: Understanding useState", "02: Working with useEffect")

## Workflow

### Step 1: Understand the Learning Request and Check for Existing Progress

When the user invokes the skill, identify what subject they want to learn. This may be provided directly (e.g., `/learn-skill React hooks`) or require clarification.

**If unclear, ask:**
- "What specific subject would you like to learn about?"
- "Is there a particular aspect or use case you're most interested in?"
- "What's your current experience level with this subject?"

#### Check for Existing Learning Materials

Once the subject is identified, normalize the subject name to create a directory-safe slug (lowercase, hyphens instead of spaces, e.g., "React Hooks" → "react-hooks").

Check if `~/.claude-learn-skill/[subject-slug]/` exists:

**If directory exists:**
- Read `learning_path.json` to get the topic structure
- Read `progress.json` to see what topics the user has completed
- Skip directly to Step 4 (Present the Learning Interface) with progress indication
- Note: If user asks for a specific topic number/name that isn't in the saved research, proceed to Step 2 for additional research on that topic

**If directory does not exist:**
- Proceed to Step 2 (Research the Subject)
- Initialize user context tracking (see User Context Tracking section below)
- After research, create the directory structure:
  ```
  ~/.claude-learn-skill/[subject-slug]/
  ├── learning_path.json          # Subject structure and metadata (lists all topics)
  ├── progress.json                # User's progress tracking
  ├── user_context.md              # Detailed user-specific learning context
  ├── research/                    # Research materials (VERY DETAILED)
  │   └── initial_research.md     # 500-1000+ lines of comprehensive research
  └── topics/                      # Individual topic guides
      ├── 01_topic-name.md
      ├── 02_topic-name.md
      └── ...
  ```

### Step 1.5: User Context Tracking (Throughout Learning Journey)

**CRITICAL: Maintain detailed user-specific context to enable personalized, tailored teaching that persists across sessions.**

As learning progresses, continuously capture and update user-specific information in `~/.claude-learn-skill/[subject-slug]/user_context.md`. This file should be VERY DETAILED and grow over time as you learn more about the user.

**Initialize user_context.md when starting a new subject:**

```markdown
# User Learning Context: [Subject Name]

**Last Updated:** [Timestamp]

## User Background & Experience

### Prior Knowledge
- What the user already knows related to this subject
- Their experience level with prerequisites
- Related technologies/concepts they're familiar with

### Learning Goals
- What they want to accomplish
- Why they're learning this subject
- Specific problems they want to solve
- Success criteria they've mentioned

### Learning Style Observations
- How they respond to explanations (need more examples? prefer theory first?)
- Pace preferences (slow and thorough? quick overviews?)
- What types of explanations resonate best

## User's Situation & Context

### Technical Environment
- Tools they're using
- Platform/OS
- Project context or constraints
- Team or organizational context (if mentioned)

### Real-World Application
- Specific use cases they're working on
- Problems they're trying to solve
- Projects or scenarios they've mentioned
- Business or personal context

### Constraints & Preferences
- Time constraints
- Resource limitations
- Preferences (coding style, frameworks, approaches)
- Things they want to avoid
- Past experiences (good or bad) they've shared

## Learning Progress & Insights

### Topics Covered
[For each topic discussed, maintain detailed notes:]

#### Topic [Number/Name]
**Date:** [When discussed]
**Depth:** [Brief/Detailed/In-depth]

**Key Points Discussed:**
- Specific concepts explained
- Examples provided
- Questions the user asked
- Areas of confusion
- "Aha!" moments

**User's Understanding:**
- What they grasped quickly
- What needed extra explanation
- Misconceptions corrected
- Knowledge gaps identified

**User's Interests:**
- What aspects they found most interesting
- Questions they asked for deeper understanding
- Topics they want to explore further
- Related areas they mentioned

**Practical Application:**
- How they plan to use this
- Examples from their context
- Challenges they anticipate
- Follow-up they requested

### Recurring Themes
- Patterns in what the user asks about
- Common challenges they face
- Consistent interests or focus areas
- Learning obstacles observed

### Personalization Insights
- What teaching approaches work best for this user
- How to tailor future content
- Topics to emphasize or de-emphasize
- Connections to make based on their background

## Action Items & Follow-Ups
- Topics the user wants to revisit
- Questions that need more research
- Examples to provide in future sessions
- Concepts to reinforce

## Session Notes

### [Date] - Session Summary
**Topics:** [What was covered]
**Duration/Depth:** [How much ground was covered]
**User Engagement:** [How active/interested they were]
**Key Takeaways:** [Main points from this session]
**Next Steps:** [What to cover next]

[Add new session notes as learning progresses]
```

**When to Update user_context.md:**

1. **At the start of learning a new subject:** Initialize the file
2. **When starting each topic:** Add a new section for the topic being discussed
2. **During conversations:** Capture important details the user shares:
   - Their specific situation or use case
   - Questions that reveal their thinking
   - Challenges they mention
   - Goals or constraints they describe
   - Preferences they express
3. **After each chunk of learning:** Note how they responded
4. **When user shares context:** Any time they mention their environment, projects, goals, or constraints
5. **End of session:** Add a session summary
6. **When patterns emerge:** Update recurring themes and personalization insights

**What to Capture (Throughout Conversation):**

**During Every Interaction:**
- **User's exact questions:** Word-for-word when revealing (shows thought process and gaps)
- **User's responses:** How they answer your questions, what they say
- **Explicit information:** Things user directly states (their job, project, constraints, goals)
- **Implicit information:** Things you can infer (struggle with abstract concepts, prefer step-by-step)
- **What clicked:** When understanding happens ("Oh! So it's like...")
- **What didn't click:** Concepts needing re-explanation, confusion points
- **Examples that resonated:** Which examples they connected with and why
- **Hands-on results:** What happened when they tried exercises (success, errors, insights)

**Context as it Emerges:**
- **Real-world applications:** Their specific projects, use cases, or scenarios they mention
- **Technical environment:** Tools, platforms, constraints they reveal during conversation
- **Team/work context:** Organizational practices, team preferences mentioned
- **Prior knowledge connections:** Links they make to things they already know

**Learning Patterns:**
- **Teaching style response:** What explanations work best (analogies? step-by-step? code-first?)
- **Pace preferences:** Do they want to slow down, speed up, or dive deeper?
- **Misconceptions corrected:** What they misunderstood and the correction that worked
- **Interests revealed:** Topics they express curiosity about or want to explore
- **Feedback signals:** Engagement level, frustration, excitement, confusion

**Quality Guidelines:**

- **Be specific:** Don't just write "user understood useState" - write "user grasped useState basics quickly, asked insightful question about why state updates are async, wants to use this for form handling"
- **Capture examples:** When user mentions specific scenarios, write them down in detail
- **Note patterns:** After 2-3 topics, identify learning patterns
- **Include quotes:** When user says something revealing about their situation or goals, capture it
- **Update regularly:** Don't wait - update as conversations happen
- **Link topics:** Note connections between what user has learned and what they're learning now

**Example of Good User Context Entry:**

```markdown
#### Topic 03: Managing State with useState

**Date:** 2026-02-04
**Depth:** In-depth with hands-on practice

**Key Points Discussed:**
- Basic useState syntax and when to use it
- How state updates trigger re-renders
- Common mistake: treating state updates as synchronous
- Functional updates when new state depends on old state
- User asked excellent question about why they can't just use regular variables (explained closure and re-render mechanism)

**User's Understanding:**
- Quickly grasped the basic concept and syntax
- Needed extra explanation on why state updates are async (provided promise analogy which clicked)
- Initially confused about functional updates but understood after seeing setState(prev => prev + 1) example
- Made connection to their previous experience with Vue's reactive data

**User's Interests:**
- Very interested in how this works "under the hood"
- Wants to know performance implications of many state updates
- Asked about when to use useState vs useReducer (noted for future topic)
- Mentioned they're building a multi-step form and wondering how to structure state for that

**Practical Application:**
- Working on a project: multi-step registration form for their company's app
- Needs to handle 5 steps with validation
- Currently storing form data in component state
- Concerned about state management getting messy as form grows
- Team uses TypeScript - wants to understand typed state

**Next Time:**
- Can use their form example when teaching more complex patterns
- They'll be ready for useReducer topic soon (mentioned they have complex state logic)
- Should emphasize TypeScript examples since that's their environment
```

This detailed context allows teaching to become increasingly personalized and relevant to the user's specific situation over time.

#### How Conversation Context Feeds Into Future Learning

**When starting the next topic:**
1. **Read user_context.md** to review all previous conversation details
2. **Topic-level research** (Step 5) will explicitly incorporate:
   - What teaching approaches worked in previous topics
   - User's specific projects and use cases mentioned
   - Technical environment and tools they're using
   - Concepts they struggled with vs. grasped quickly
   - Their interests and curiosity areas
   - Real-world applications they need

3. **Personalized teaching** based on conversation history:
   - Reference previous topics: "Remember when we talked about X in Topic 2? This builds on that..."
   - Use their examples: "Let's see how this works with your multi-step form project you mentioned..."
   - Adapt explanations: "Since analogies worked well for you with useState, let me use a similar approach..."
   - Address patterns: "I noticed you're always interested in performance - here's how this optimizes..."

**Example of conversation context informing future topic:**
```
Previous conversation captured:
- User struggled with async/await initially, needed promises analogy
- User mentioned building a dashboard at work
- User asked about performance 3 times
- User's environment: TypeScript, React, uses VSCode

Next topic research includes:
- Extra emphasis on async patterns with promise connections
- Examples using dashboard data fetching scenarios
- Performance implications highlighted upfront
- TypeScript examples with proper typing
```

### Step 2: Research the Subject

**CRITICAL: Research must be comprehensive and thorough.** The research document is the foundation for all learning content and must cover the subject completely to support deep, tailored teaching.

Use the Task tool with subagent_type="general-purpose" to spawn research agents that will:

- Search the web for comprehensive, current information about the subject
- Identify ALL key concepts, patterns, and best practices at every level
- Gather extensive real-world examples and use cases
- Find common learning paths and progression patterns
- Discover practical applications and scenarios
- Include detailed explanations, not just summaries
- Collect code examples, patterns, and anti-patterns
- Document edge cases and common mistakes
- Gather authoritative sources and recent discussions

**Research Quality Target:**
- **Depth:** Each concept should have clear, multi-paragraph explanations with examples
- **Breadth:** Cover foundations, intermediate, advanced, and use-case scenarios comprehensively
- **Sources:** Include extensive links to official docs, tutorials, articles, discussions
- **Current:** Emphasize 2025-2026 best practices and recent changes
- **Completeness:** Cover the topic thoroughly - length should match complexity (simple topics may need less, complex topics more)
- **No padding:** Focus on quality and comprehensiveness, not hitting a line count

**Research prompt template:**
```
Research [SUBJECT] comprehensively for creating a structured learning guide.

**IMPORTANT:** Your research output must be thorough and complete. This research will be the foundation for teaching, so cover all essential concepts, patterns, and use cases. Quality and completeness matter more than length - write as much as needed to cover the subject properly, without artificial padding.

Your research should be structured as follows:

## 1. FOUNDATIONAL CONCEPTS (Beginner Level)
For EACH foundational concept, provide:
- Detailed explanation (3-5 paragraphs minimum)
- Why it exists and what problem it solves
- Clear examples with explanations
- Common misconceptions
- Prerequisites
- Key terminology and definitions

## 2. INTERMEDIATE TOPICS
For EACH intermediate topic, provide:
- How it builds on foundations (detailed explanation)
- Practical patterns and when to use them
- Real-world scenarios with code examples
- Common mistakes and how to avoid them
- Integration with other concepts
- Tools and ecosystem

## 3. ADVANCED CONCEPTS AND PATTERNS
For EACH advanced concept, provide:
- Complex patterns and architectures
- Performance considerations
- Optimization techniques
- Edge cases and handling
- Production considerations
- Trade-offs and alternatives

## 4. USE CASES AND PRACTICAL APPLICATIONS
For EACH major use case, provide:
- Detailed scenario description
- Step-by-step implementation approach
- Complete code examples
- Variations and alternatives
- Common pitfalls
- Best practices

## 5. CURRENT STATE (2025-2026)
- Latest version and recent changes
- Modern best practices
- Deprecated patterns to avoid
- Emerging trends
- Popular tools and libraries
- Community recommendations

## 6. LEARNING PATH STRUCTURE
Based on all the above, organize into:
- Foundations (01-05): Core concepts
- Intermediate (06-09): Building on basics
- Advanced (10-11): Complex patterns
- Use Case Guides (12-17): Scenario-based learning

For each numbered topic, provide:
- Topic name
- Brief description (2-3 sentences)
- Key concepts it covers
- Why it matters

Include thorough details, code examples, patterns, and explanations throughout. This research should be comprehensive enough that someone could create detailed teaching content from it without additional research. Write as much as needed to cover the subject completely - no more, no less.
```

### Step 3: Structure the Learning Path (Topics) and Save Research

Based on research results, organize the content into three main categories:

#### 1. Learning Path (Progressive Difficulty)

Organize topics by difficulty level:

- **Foundations (01-05)**: Core concepts everyone must understand first
- **Intermediate (06-09)**: Build on foundations, assume basic knowledge
- **Advanced (10-11)**: Complex patterns, optimization, architecture

**Criteria for categorization:**
- Foundations: No prerequisites, fundamental concepts
- Intermediate: Requires foundation knowledge, practical application
- Advanced: Requires intermediate knowledge, complex scenarios

#### 2. Use Case Guides (Scenario-Based)

Group topics by practical application scenarios:

- **Day-to-Day (12-14)**: Common daily tasks and workflows
- **Modernization (15-16)**: Updating legacy code, adopting new patterns
- **Quality (17)**: Testing, optimization, best practices

**Alternative categories based on topic:**
- Architecture / Implementation / Integration
- Frontend / Backend / Full-stack
- Setup / Development / Deployment
- Or any other logical grouping that fits the subject

#### 3. Topic Table Structure

Each topic should have:
- **#**: Sequential number (01-17 typical range)
- **Topic**: Clear, specific name (e.g., "The AI Adoption Landscape", "Effective Prompting")
- **Category**: One of the categories above (e.g., "Foundations", "Day-to-Day")

#### Save Learning Path Structure

After organizing the topics, save the learning path structure to `~/.claude-learn-skill/[subject-slug]/learning_path.json`:

```json
{
  "subject": "React Hooks",
  "slug": "react-hooks",
  "created_at": "2026-02-03T...",
  "last_accessed": "2026-02-03T...",
  "total_topics": 17,
  "categories": {
    "foundations": {
      "name": "Foundations",
      "range": "01-05",
      "topics": [
        {"number": 1, "name": "What Are Hooks and Why They Exist", "slug": "what-are-hooks"},
        {"number": 2, "name": "Understanding useState for State Management", "slug": "understanding-usestate"},
        ...
      ]
    },
    "intermediate": {...},
    "advanced": {...},
    "day_to_day": {...},
    "modernization": {...},
    "quality": {...}
  }
}
```

Also save the comprehensive research to `~/.claude-learn-skill/[subject-slug]/research/initial_research.md` for future reference.

**CRITICAL: Research document must be thorough and complete.** This is not a summary - it's a comprehensive knowledge base that will inform all teaching. Include:
- Clear, thorough explanations for every concept
- Multiple code examples with detailed walkthroughs
- Common patterns and anti-patterns
- Real-world scenarios and use cases
- All sources and references
- Modern best practices (2025-2026)
- Edge cases and gotchas
- Tool ecosystem and integrations
- **Quality over quantity:** Write as much as needed to fully cover the subject - complex topics will naturally be longer, simpler topics shorter

Initialize the progress file at `~/.claude-learn-skill/[subject-slug]/progress.json`:

```json
{
  "completed_topics": [],
  "current_topic": null,
  "last_activity": null,
  "notes": {}
}
```

### Step 4: Present the Learning Interface

Format the output as an interactive learning interface:

```markdown
# Welcome to [Subject] Learning!

How would you like to learn today?

## 📚 LEARNING PATH
Systematic skill building from foundations to advanced
→ Foundations (01-05)
→ Intermediate (06-09)
→ Advanced (10-11)

## 🎯 USE CASE GUIDES
Jump to specific scenarios you're working on
→ [Category 1] (12-14)
→ [Category 2] (15-16)
→ [Category 3] (17)

Type a number (01-17), topic name, or describe what you need help with.

---

## Available Topics:

| #  | Topic | Category |
|----|-------|----------|
| 01 | [Topic Name] | Foundations |
| 02 | [Topic Name] | Foundations |
| ... | ... | ... |

---

**💡 Recommendation:**
[Based on progress.json, recommend:
- If no topics completed: Start with topic 01
- If some topics completed: Continue with next logical topic or suggest related topics
- If returning user: "Welcome back! You last completed Topic [X]. Ready to continue with Topic [Y]?"]

What would you like to explore? You can type a number, topic name, or describe what you're trying to accomplish.
```

**When showing progress:**
- Mark completed topics with ✓ in the table
- Show completion percentage
- Highlight recommended next topic

### Step 5: Provide Interactive Topic Learning

When the user selects a topic (by number, name, or description):

1. **Read user context:** Load `~/.claude-learn-skill/[subject-slug]/user_context.md` to understand user's background, goals, prior learning, and situation
2. **Check if topic-specific research exists:** Look for `~/.claude-learn-skill/[subject-slug]/research/topic_[NN]_[topic-slug]_research.md`
3. **If topic research doesn't exist:** **SPAWN RESEARCH AGENT** for this specific topic (see Topic-Level Research below)
4. **Check if topic guide exists:** Look for `~/.claude-learn-skill/[subject-slug]/topics/[NN]_[topic-slug].md`
5. **If topic guide doesn't exist:** Create the guide based on topic research and user context, then save it
6. **Tailor presentation:** Use user context to personalize examples, pace, and emphasis throughout teaching

#### Topic-Level Research (Per Topic)

**CRITICAL: Each topic requires its own thorough research that is personalized to the user's context.**

When starting a new topic, spawn a research agent to create topic-specific research that:
- Is saved to `research/topic_[NN]_[topic-name]_research.md`
- Provides detailed, focused content on this ONE topic (length should match topic complexity)
- Takes into account everything known about the user from user_context.md
- Provides deep coverage of this specific topic (not breadth across the subject)
- Focuses on completeness and quality rather than hitting a specific length

**Topic Research Prompt Template:**
```
Research Topic [NN]: [TOPIC NAME] within [SUBJECT] thoroughly for personalized teaching.

**IMPORTANT:** This research is for ONE specific topic within the broader subject. It must be thorough, complete, and PERSONALIZED based on the user's context. Write as much as needed to fully cover this topic - no more, no less.

## USER CONTEXT (use this to personalize the research):
[Include relevant sections from user_context.md here - this contains ALL previous conversation details:]

**Background & Goals:**
- User's experience level and prior knowledge
- Their learning goals and what they want to accomplish
- Why they're learning this subject

**Technical Context:**
- Their technical environment (tools, platform, IDE)
- Real-world projects they're working on
- Team/organizational context
- Constraints or preferences

**Previous Learning Journey:**
- Topics they've completed and their understanding level
- What concepts they grasped quickly vs. struggled with
- Teaching approaches that worked best (analogies, code-first, step-by-step, etc.)
- Questions they asked in previous topics
- Misconceptions that were corrected
- Pace preferences (slow/fast, detail-oriented/big-picture)

**Conversation Context:**
- Specific examples or scenarios they mentioned
- Hands-on exercises they completed successfully
- Challenges they faced and how they were resolved
- Connections they made to prior knowledge
- Interests and curiosity areas that emerged
- Performance/optimization concerns they've raised
- Real-world applications they need for their work

## RESEARCH REQUIREMENTS:

### 1. Core Concept Deep Dive
- Comprehensive explanation of the topic (5-10 paragraphs)
- Why this topic matters and when to use it
- How it fits into the broader subject
- Prerequisites and dependencies
- Key terminology and definitions

### 2. Personalized Examples
Based on user context, provide examples that:
- Relate to their technical environment and tools
- Connect to their specific projects or use cases
- Match their experience level
- Address their stated goals
- Build on concepts they've already learned

### 3. Detailed Patterns and Approaches
- Common patterns for this topic (with detailed explanations)
- Step-by-step implementation approaches
- Multiple variations and when to use each
- Integration with other concepts they know
- Real-world scenarios relevant to their situation

### 4. Common Challenges and Solutions
- Typical mistakes at their experience level
- Edge cases and gotchas
- Debugging approaches
- Performance considerations
- Best practices (2025-2026)

### 5. Hands-On Exercises
Design exercises that:
- Use their actual technical environment
- Relate to their project context
- Match their learning pace
- Build on their existing knowledge
- Address their specific goals

### 6. Deep Technical Details
- How it works "under the hood" (if user has shown interest in this)
- Advanced configurations and options
- Trade-offs and alternatives
- Connection to related topics in the learning path
- Modern approaches and tools

### 7. Resources and References
- Official documentation sections relevant to this topic
- Code examples from authoritative sources
- Recent articles (2025-2026) on best practices
- Tools and libraries specific to this topic

**Output Format:**
Structure the research as a comprehensive markdown document with clear sections, thorough code examples, detailed explanations, and explicit connections to the user's context wherever possible. Write enough to fully cover the topic - quality and completeness matter more than length. This research will be used to create a highly personalized teaching experience.
```

**After topic research is complete:**
- Save to `research/topic_[NN]_[topic-name]_research.md`
- Use this research along with user_context.md to create the topic guide
- Present the topic interactively, tailored to the user's needs

**CRITICAL: Interactive Learning Approach with Active Conversation Tracking**

**DO NOT dump the entire topic content at once.** Instead, break the topic into small, digestible chunks and present them interactively.

**IMPORTANT: Throughout the conversation, actively track and record what's happening in real-time.** Update user_context.md frequently (not just at the end) to capture the learning journey as it unfolds. This recorded context will be crucial for future topics and sessions.

#### Real-Time Conversation Tracking

**CRITICAL: As the conversation unfolds, continuously capture what's happening.**

After EVERY significant interaction (user question, explanation, example, exercise), immediately update user_context.md with:

**What to capture in real-time:**
- **User's questions:** Exact questions they ask (reveals their thinking and gaps)
- **User's responses:** How they answer your check questions (shows understanding level)
- **What clicked:** Moments when they say "oh I see" or demonstrate understanding
- **What didn't click:** Concepts that needed re-explanation or multiple approaches
- **Examples they relate to:** Which examples resonated most with them
- **Their specific context:** Any projects, constraints, tools, or situations they mention
- **Insights they share:** Connections they make to their prior knowledge
- **Challenges they face:** Problems they encounter with exercises or examples
- **Follow-up interests:** Topics they express curiosity about
- **Pace observations:** Whether they're moving too fast/slow, need more/less detail

**How to update:**
- Don't wait until end of topic - update after each significant exchange
- Add to the current topic's section in user_context.md
- Be specific: "User struggled with async/await concept initially, needed analogy to promises which helped"
- Include quotes when revealing: "User said: 'This reminds me of the Python decorators we use at work'"
- Note patterns: "Third time user has asked about performance implications - clear interest in optimization"

**Why this matters:**
- Future topics will be informed by what worked/didn't work in this conversation
- Topic-level research for next topics will reference these specific learnings
- Teaching approach can adapt based on observed patterns
- User's actual learning journey is preserved, not just what topics they completed

#### Interactive Learning Flow

**1. Start with Brief Introduction (2-3 sentences)**
- Explain what this section covers
- Why it matters
- What they'll be able to do after

**2. Present Content in Small Chunks**
- **One concept at a time** (3-5 paragraphs maximum per chunk)
- **Wait for user acknowledgment** before continuing
- Ask: "Ready to continue?" or "Does this make sense so far?"
- If concept is complex, ask a quick check question
- **→ UPDATE user_context.md:** Record their response, whether concept clicked, any questions they asked

**3. Hands-On Practice (When Applicable)**
- For practical topics, provide a command/task to try
- **Wait for the user to actually do it** and report back
- Ask: "Give this a try and let me know what you see"
- Don't continue until they've tried it or asked to skip
- Help troubleshoot if they encounter issues
- **→ UPDATE user_context.md:** Record what they tried, any issues they hit, their environment details, how they approached the problem

**4. Check Understanding**
- After 2-3 chunks, ask a quick question to verify understanding
- **For multiple choice questions, use the AskUserQuestion tool** for better UX
- Examples:
  - "Can you explain in your own words what X does?"
  - "When would you use Y vs Z?" (can be multiple choice with AskUserQuestion)
  - "What do you think would happen if...?" (can be multiple choice with AskUserQuestion)
- Adapt explanation if they're unclear
- **CRITICAL:** Update user_context.md with their response - captures their understanding level, misconceptions, and learning style

**5. Offer Pacing Control**
- After each chunk: "Would you like to continue, try an example, or take a break?"
- **Use AskUserQuestion tool with multiple choice options** for pacing decisions (e.g., "Continue", "Try an example", "Take a break")
- Let them control the pace: "continue", "example", "slow down", "skip ahead"
- If they seem overwhelmed, suggest breaking for now
- **→ UPDATE user_context.md:** Note their preferred pace, energy level, any requests to slow down/speed up, patterns in their engagement

#### Using ASCII Diagrams to Enhance Learning

**IMPORTANT: Use ASCII diagrams when appropriate to illustrate concepts, making abstract ideas concrete and relationships clear. ASCII diagrams render perfectly in terminal environments.**

**When to Use Diagrams:**

Diagrams significantly enhance learning for:
- **System architecture**: Components and their relationships, data flow between services
- **Workflows and processes**: Step-by-step flows, decision trees, conditional logic
- **Concept relationships**: How different concepts connect, hierarchies, dependencies
- **State changes**: How state evolves over time, lifecycle diagrams
- **Data structures**: Tree structures, graphs, linked lists, object relationships
- **Sequences and interactions**: API calls, request/response flows, event sequences
- **Mental models**: Abstract concepts made visual, comparisons between approaches

**ASCII Diagram Types and Examples:**

**1. Flowcharts** - Decision logic, conditional workflows, process flows

```
User Authentication Flow
========================

    ┌─────────────┐
    │ User Login  │
    │   Request   │
    └──────┬──────┘
           │
           ▼
    ┌──────────────┐      NO
    │  Credentials ├─────────────┐
    │   Valid?     │             │
    └──────┬───────┘             │
           │ YES                 │
           ▼                     ▼
    ┌──────────────┐      ┌─────────────┐
    │ Generate JWT │      │Return Error │
    │    Token     │      │   401       │
    └──────┬───────┘      └─────────────┘
           │
           ▼
    ┌──────────────┐
    │Return Token  │
    │   200 OK     │
    └──────────────┘
```

**2. Hierarchies** - Class structures, organizational charts, inheritance

```
React Component Hierarchy
=========================

              ┌─────────────┐
              │     App     │
              └──────┬──────┘
                     │
        ┌────────────┼────────────┐
        │            │            │
        ▼            ▼            ▼
   ┌────────┐  ┌─────────┐  ┌────────┐
   │ Header │  │  Main   │  │ Footer │
   └────────┘  └────┬────┘  └────────┘
                    │
           ┌────────┼────────┐
           │        │        │
           ▼        ▼        ▼
      ┌────────┐ ┌────┐ ┌────────┐
      │Sidebar │ │Feed│ │ Widget │
      └────────┘ └────┘ └────────┘
```

**3. State Transitions** - Lifecycle, state machines, mode changes

```
Component Lifecycle States
===========================

  ┌──────────────────────────────────────┐
  │                                      │
  │  MOUNTING                            │
  │  ───────────                         │
  │  constructor() ──→ render() ──→ componentDidMount()
  │                                      │
  └──────────────┬───────────────────────┘
                 │
                 ▼
  ┌──────────────────────────────────────┐
  │                                      │
  │  UPDATING                            │
  │  ────────                            │
  │  props/state change ──→ render() ──→ componentDidUpdate()
  │         │                            │
  │         └────────────┐               │
  └──────────────────────┼───────────────┘
                         │
                         ▼
  ┌──────────────────────────────────────┐
  │  UNMOUNTING                          │
  │  ──────────                          │
  │  componentWillUnmount()              │
  └──────────────────────────────────────┘
```

**4. Sequences** - API calls, request/response flows, interactions

```
API Request Flow
================

Client          Frontend         Backend        Database
  │                │                │               │
  │──Login Click──→│                │               │
  │                │──POST /auth───→│               │
  │                │                │──Query User──→│
  │                │                │←──User Data───│
  │                │                │               │
  │                │                │─Validate─     │
  │                │                │  Password     │
  │                │                │               │
  │                │←────JWT────────│               │
  │←─Set Cookie────│                │               │
  │                │                │               │
  │──Page Request─→│                │               │
  │←─Auth Page─────│                │               │
```

**5. Relationships** - Database schemas, data models, connections

```
Database Schema
===============

    ┌─────────────┐          ┌──────────────┐
    │    Users    │          │    Posts     │
    ├─────────────┤          ├──────────────┤
    │ id (PK)     │◄────┐    │ id (PK)      │
    │ email       │     │    │ user_id (FK) │──┐
    │ username    │     └────│ title        │  │
    │ created_at  │          │ content      │  │
    └─────────────┘          │ created_at   │  │
                             └──────────────┘  │
                                    │          │
                                    │          │
                                    ▼          │
                             ┌──────────────┐  │
                             │   Comments   │  │
                             ├──────────────┤  │
                             │ id (PK)      │  │
                             │ post_id (FK) │──┘
                             │ user_id (FK) │──────┐
                             │ content      │      │
                             │ created_at   │      │
                             └──────────────┘      │
                                                   │
                                   ┌───────────────┘
                                   ▼
                           (FK = Foreign Key)
                           (PK = Primary Key)
```

**6. Architecture** - System components, service layouts

```
Microservices Architecture
==========================

                    ┌──────────────┐
                    │  API Gateway │
                    └───────┬──────┘
                            │
            ┌───────────────┼───────────────┐
            │               │               │
            ▼               ▼               ▼
    ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
    │    Auth      │ │   Product    │ │    Order     │
    │   Service    │ │   Service    │ │   Service    │
    └───────┬──────┘ └───────┬──────┘ └───────┬──────┘
            │                │                │
            ▼                ▼                ▼
    ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
    │   Auth DB    │ │  Product DB  │ │   Order DB   │
    │  (Postgres)  │ │   (MongoDB)  │ │  (Postgres)  │
    └──────────────┘ └──────────────┘ └──────────────┘
```

**7. Comparisons** - Side-by-side approaches, before/after

```
useState vs useReducer
======================

useState (Simple State)          useReducer (Complex State)
───────────────────────          ──────────────────────────

┌─────────────────┐              ┌──────────────────────┐
│ const [count,   │              │ const [state,        │
│  setCount] =    │              │  dispatch] =         │
│  useState(0)    │              │  useReducer(reducer) │
└─────────────────┘              └──────────────────────┘
        │                                  │
        ▼                                  ▼
┌─────────────────┐              ┌──────────────────────┐
│ setCount(1)     │              │ dispatch({           │
│ setCount(n+1)   │              │   type: 'INCREMENT', │
│                 │              │   payload: 1         │
│                 │              │ })                   │
└─────────────────┘              └──────────────────────┘

Best for:                        Best for:
• Simple values                  • Complex state logic
• Independent updates            • Related state values
• Few state changes              • State history needed
```

**ASCII Diagram Drawing Guidelines:**

**Box Drawing Characters:**
- `─ │ ┌ ┐ └ ┘ ├ ┤ ┬ ┴ ┼` (single line)
- `═ ║ ╔ ╗ ╚ ╝ ╠ ╣ ╦ ╩ ╬` (double line)
- `▲ ▼ ◄ ►` (arrows for direction)
- `┌─────┐` (boxes)
- `│     │` (vertical borders)
- `└─────┘` (corners)

**Common Patterns:**
```
Flow:        A ──→ B ──→ C
Branch:      A ─┬─→ B
               └─→ C
Decision:    ┌────┐  YES
             │ ?  ├────→
             └────┘  NO
Bi-directional: A ←──→ B
```

**Examples of Effective Diagram Usage:**

- **React hooks**: State transition diagram showing component lifecycle and when hooks execute
- **Async/await**: Sequence diagram showing event loop, promises, and execution order
- **Database relationships**: ER diagram showing table connections and foreign keys
- **API design**: Flowchart showing request validation, processing, and response paths
- **Git workflows**: Diagram showing branch structure and merge strategies
- **Authentication flow**: Sequence showing user, frontend, backend, and auth service interactions

**Integration with Learning Flow:**

- **Before complex concepts**: Show diagram first to provide mental framework
- **During explanation**: Use diagram to illustrate points as you explain
- **After explanation**: Provide diagram summary to reinforce understanding
- **For comparisons**: Side-by-side ASCII diagrams showing different approaches
- **Keep diagrams simple**: Terminal width is limited (~80-100 chars), break complex diagrams into multiple simpler ones
- **Add titles and labels**: Always title diagrams and label components clearly
- **Update user_context.md**: Note if visual explanations resonated with user

**Example Chunk Presentation:**

```markdown
## Understanding What Mise Is

Mise (pronounced "MEEZ ahn plahs") is a modern development environment manager. Think of it as a replacement for tools like nvm, pyenv, and rbenv - but for *all* languages at once, not just one.

The name comes from the French culinary term "mise-en-place" meaning "everything in its place." Just like a chef prepares ingredients before cooking, mise prepares your development environment before you code.

**The Problem It Solves:**
Instead of installing nvm for Node.js, pyenv for Python, rbenv for Ruby, etc., mise handles all of them with a single tool.

*Does this make sense so far?*
```

**Then use AskUserQuestion tool with options like:**
- "Yes, show me a concrete example" (Recommended)
- "Can you explain that differently?"
- "I need more context on why this matters"

**Wait for their response before continuing with next chunk.**

#### Chunk Size Guidelines

- **Foundations topics:** Very small chunks (2-4 paragraphs), more hands-on
- **Intermediate topics:** Medium chunks (4-6 paragraphs), balance theory/practice
- **Advanced topics:** Slightly larger chunks (5-8 paragraphs), assume faster pace
- **Always** break before hands-on exercises
- **Always** pause after introducing new terminology

#### Hands-On Exercise Pattern

```markdown
## Let's Try It Yourself

Let's see mise in action. Try running this command:

```bash
mise --version
```

*Run this command and tell me what version you see. If you get "command not found," that's okay - we'll install it in the next section.*

[WAIT FOR USER RESPONSE - DO NOT CONTINUE]

[After they respond, acknowledge and continue with next chunk]
```

#### Progress Tracking Within Topic

Track which section/chunk of the topic the user is on in progress.json:

```json
{
  "completed_topics": [1],
  "current_topic": 2,
  "current_section": "hands-on-example-1",
  "last_activity": "2026-02-03T14:30:00Z",
  "notes": {
    "topic_2": "User needed extra explanation on shell activation, provided additional examples"
  }
}
```

#### Save Full Topic Guide

Save the complete topic content to `~/.claude-learn-skill/[subject-slug]/topics/[NN]_[topic-name-slug].md` for reference, but **present it interactively in chunks** during the learning session.

### Step 6: Track Progress and Update Files

Update both progress tracking and user context at multiple levels:

**A. User Context Updates (Throughout Learning):**

Update `user_context.md` continuously as you learn about the user:
- When user shares background, goals, or constraints
- After each topic section, capture what they understood/struggled with
- When they ask questions or express interests
- When they mention specific projects, use cases, or situations
- When patterns emerge in their learning style
- At the end of each session, add a session summary

**B. Progress Tracking (Structured Tracking):**

**1. Within-Topic Progress (During Learning):**
Track which section the user is on while learning a topic:

```json
{
  "completed_topics": [1, 2],
  "current_topic": 3,
  "current_section": "hands-on-configuration",  // Where they are within topic 3
  "last_activity": "2026-02-03T14:30:00Z",
  "notes": {
    "topic_1": "User found this helpful, wanted more examples",
    "topic_3": "Currently on hands-on section, going at good pace"
  }
}
```

**2. Topic Completion (When Finished):**
When the user completes a topic (finishes all sections or moves to another topic):

```json
{
  "completed_topics": [1, 2, 3],  // Add the completed topic number
  "current_topic": 4,
  "current_section": null,  // Reset section when starting new topic
  "last_activity": "2026-02-03T14:35:00Z",
  "notes": {
    "topic_1": "User found this helpful, wanted more examples on...",
    "topic_3": "Completed all sections, user asked good questions about shell activation"
  }
}
```

**Update triggers:**

*Progress (progress.json):*
- **After each chunk:** Update current_section if moving to new section
- **When user pauses:** Update last_activity
- **When topic completed:** Add to completed_topics, reset current_section
- **User feedback:** Add to notes for that topic

*User Context (user_context.md):*
- **User shares context:** Any time they mention their situation, goals, projects, constraints, tools
- **After explanations:** How they responded, what clicked, what needed clarification
- **User asks questions:** Capture the question and what it reveals about their thinking
- **During hands-on:** What they tried, challenges faced, insights gained
- **User expresses interest:** Topics they want to explore more, connections they make
- **Session end:** Summary of what was covered and user's engagement
- **Patterns emerge:** Update recurring themes and personalization insights

Help the user navigate their learning journey based on progress:

- **If they completed a foundation topic**: Suggest the next foundation topic or let them know when they're ready for intermediate
- **If they completed an intermediate topic**: Suggest advanced topics or related use cases
- **If they're jumping around**: Offer to return to the main menu or suggest a related topic
- **If they seem stuck**: Ask if they want more examples, clarification, or a different approach
- **If they've completed all topics**: Congratulate them and suggest related learning paths

## Key Guidelines

### Research Quality

**CRITICAL: Research happens at TWO levels, both must be thorough and complete**

**Subject-Level Research (Step 2):**
- Initial comprehensive research when starting a new subject
- Covers the entire subject with appropriate depth (length matches subject complexity)
- Broad overview of all concepts, patterns, and use cases
- Saved to `research/initial_research.md`
- Forms the foundation for structuring the learning path

**Topic-Level Research (Step 5):**
- Detailed research for EACH individual topic before teaching it
- Focuses deeply on ONE topic (length should match topic complexity)
- **PERSONALIZED based on user_context.md**
- Includes examples relevant to user's projects, tools, and goals
- Saved to `research/topic_[NN]_[name]_research.md`
- Used to create highly tailored topic guides and teaching

**Quality Standards (Both Levels):**
- **Completeness over length:** Cover everything needed, avoid padding
- Use current information (2025-2026 best practices)
- Prioritize official documentation and authoritative sources
- Include practical, real-world examples with clear explanations
- Focus on actionable information with comprehensive coverage
- Provide clear, multi-paragraph explanations for each concept
- Include code examples, patterns, anti-patterns, and edge cases
- Document tools, ecosystem, and integrations as relevant
- Capture best practices, common mistakes, and optimization techniques
- **Simple topics = shorter research, complex topics = longer research**

### User Context Tracking & Conversation Recording

**CRITICAL: Maintain detailed user-specific context throughout learning journey**

**Real-Time Updates (During Conversation):**
- Update user_context.md after EVERY significant interaction, not just at end
- Capture user's questions, responses, and reactions as they happen
- Record what examples resonated and what needed re-explanation
- Note hands-on results, challenges faced, and insights gained
- Document pace preferences and engagement patterns
- Track what clicked and what didn't in real-time

**Comprehensive Context:**
- Initialize user_context.md when starting a new subject
- Continuously add user's situation, goals, projects, constraints as they emerge
- Capture teaching approach effectiveness (analogies vs. code-first vs. step-by-step)
- Note user's questions and what they reveal about thinking/interests
- Document specific use cases and real-world applications mentioned
- Track learning patterns for personalization

**Context Utilization:**
- Use conversation history to inform topic-level research
- Reference previous topics and conversations during teaching
- Adapt teaching style based on observed patterns
- Personalize examples to user's actual projects and environment
- Build on successful teaching approaches from earlier topics
- Address recurring interests and concerns proactively

### Content Organization

- Keep foundations truly foundational (no prerequisites)
- Ensure logical progression between difficulty levels
- Make use-case categories mutually exclusive where possible
- Number topics sequentially for easy reference

### User Experience

- Always provide a clear way to navigate (topic menu)
- Give specific recommendations based on context
- Allow flexible navigation (by number, name, or description)
- Track conceptual progress and suggest logical next steps
- **Use AskUserQuestion tool for multiple choice interactions:**
  - Check understanding questions with answer options
  - Pacing control ("Continue", "Try example", "Take a break")
  - Topic navigation choices
  - Learning preference questions
  - Any situation where offering 2-4 clear options improves UX

### Adaptation

- Adjust difficulty levels based on user feedback
- If topics are too easy/hard, acknowledge and adjust
- Allow users to skip around while noting prerequisites
- Offer different explanations if the first doesn't resonate
- **When user seems stuck, use AskUserQuestion to offer options:**
  - Different explanation approaches (analogy, code-first, step-by-step)
  - Whether to continue, revisit basics, or try hands-on
  - Skip to a different topic or dive deeper
  - Provides clear choices instead of open-ended "what do you want to do?"

## Example Invocation Patterns

**Direct subject specification:**
```
User: /learn-skill React hooks
Assistant: [Spawns research agent for "React Hooks" subject, then presents learning interface with topics]
```

**General request:**
```
User: I want to learn TypeScript
Assistant: [Spawns research agent for "TypeScript" subject, then presents learning interface]
```

**With context:**
```
User: Teach me about GraphQL, I already know REST APIs
Assistant: [Spawns research agent, notes user's background, presents interface with intermediate topic recommendations]
```

**Topic selection (within a subject):**
```
User: [After seeing React Hooks menu] 3
Assistant: [Presents detailed guide for Topic 03: Working with useEffect]
```

**Description-based:**
```
User: How do I handle state in React?
Assistant: [Matches to React subject, identifies relevant topics like useState, presents guide]
```

**Resuming previous learning:**
```
User: /learn-skill React hooks
Assistant: [Checks ~/.claude-learn-skill/react-hooks/, finds existing progress and user context]
Assistant: Welcome back to React Hooks! You've completed 3 of 17 topics (18%).
          Last time you finished Topic 03: Working with useEffect for Side Effects.
          Ready to continue with Topic 04: The Rules of Hooks?
```

**Additional topic request:**
```
User: [While learning React Hooks subject] Tell me about useImperativeHandle
Assistant: [Checks if topic exists in saved path for React Hooks subject]
Assistant: That's not in the original 17 topics, but I can research it for you.
          [Spawns research agent, creates new topic guide, saves to topics/]
          [Presents the new guide for this additional topic]
```

## Handling Edge Cases

**If research fails:**
- Acknowledge the limitation
- Offer to research a related, more specific topic
- Suggest alternative learning resources

**If subject is too broad:**
- Break it into multiple subjects
- Ask user to specify which aspect they want to focus on first
- Create a meta-guide that links to more specific subjects

**If subject is too narrow:**
- Embed it as a topic within a broader subject
- Suggest the broader subject as a starting point
- Combine with related topics into a cohesive subject

**If user is experienced:**
- Note prerequisites they already know
- Suggest starting at intermediate or advanced levels
- Focus on new patterns and modern approaches

**If user asks for a topic not in the saved learning path:**
- Check existing research first to see if it can be covered
- Read user_context.md to understand why they're interested in this additional topic (update with this context)
- If truly new/different, spawn a research agent to gather thorough information on that specific topic
- Create and save a new topic guide to `topics/` directory (e.g., `18_new-topic.md`)
- Save additional research to `research/` directory (comprehensive coverage matching topic complexity)
- Optionally add it to the learning_path.json if it fits the subject
- Update user_context.md with this interest and what drove it
- Let user know this was additional topic research beyond the original path

**If returning to existing learning path:**
- Always check `~/.claude-learn-skill/[subject-slug]/` first
- Load saved learning path and progress
- **CRITICAL: Read user_context.md** to understand their background, goals, and previous learning
- Welcome user back with their progress status and personalized context
- Use user context to tailor the session (reference their projects, preferences, past challenges)
- If they were in the middle of a topic, ask if they want to:
  - Resume where they left off (from current_section)
  - Restart the current topic
  - Choose a different topic
- Only do new research if they request something not covered
- **Continue updating user_context.md** as the session progresses

**If resuming within a topic:**
- Check current_section in progress.json
- Read user_context.md to recall their specific situation and learning style
- Check if topic research exists (research/topic_[NN]_research.md)
- If topic research doesn't exist, create it before resuming (using current user context)
- Briefly remind them what they covered so far, referencing their specific context
- Resume from that section with personalized teaching
- Example: "Welcome back! Last time you were learning about shell activation in Topic 02. You completed the installation section. Ready to continue with configuring your shell?"

## Persistence and File Management

### Directory Structure

```
~/.claude-learn-skill/
├── react-hooks/
│   ├── learning_path.json              # Topic structure
│   ├── progress.json                    # User's progress tracking
│   ├── user_context.md                  # DETAILED user-specific learning context
│   ├── research/
│   │   ├── initial_research.md         # Subject-level: comprehensive research
│   │   ├── topic_01_what-are-hooks_research.md      # Topic-level: detailed research
│   │   ├── topic_02_understanding-usestate_research.md
│   │   └── topic_03_working-with-useeffect_research.md
│   └── topics/
│       ├── 01_what-are-hooks.md         # Topic guide (based on topic research + user context)
│       ├── 02_understanding-usestate.md
│       └── ...
├── typescript/
│   ├── learning_path.json
│   ├── progress.json
│   ├── user_context.md                  # User-specific context for TypeScript learning
│   ├── research/
│   │   ├── initial_research.md         # Subject-level research
│   │   ├── topic_01_research.md        # Topic-level research
│   │   └── topic_02_research.md
│   └── topics/
└── ...
```

### File Naming Conventions

- **Subject slugs:** Lowercase, hyphens, no special characters (e.g., "react-hooks", "typescript-fundamentals")
- **Topic files:** `[NN]_[topic-name-slug].md` (e.g., "01_what-are-hooks.md", "12_fetching-data.md")
- **Research files:** Descriptive names (e.g., "initial_research.md", "additional_context_api.md")

### When to Save Files

- **learning_path.json:** After structuring topics (Step 3)
- **progress.json:** After each topic is completed or user navigates (Step 6)
- **user_context.md:**
  - Initialize when starting a new subject (Step 1)
  - Update continuously throughout learning as user shares context (Step 6)
  - Update after each topic (individual learning unit) with observations (Step 6)
  - Update at end of each session with summary (Step 6)
- **research/initial_research.md:** After subject-level research is completed (Step 2) - comprehensive coverage
- **research/topic_[NN]_[name]_research.md:** When starting each topic (Step 5) - thorough coverage, personalized to user
- **topics/[NN]_*.md:** When first presenting a topic to the user, after topic research exists (Step 5)

### Updating vs. Creating

- **Never modify learning_path.json structure** once created (users rely on topic numbers)
- **Always update progress.json** when user completes topics
- **Always update user_context.md** throughout learning - this is a living document that grows over time
- **Create new topic files** only when first needed
- **Append to research/** when doing additional research (maintaining detailed, comprehensive format)

## References

Extended guides and examples are available in the references directory:

- **references/topic_template.md**: Detailed template for creating comprehensive topic guides
- **references/research_checklist.md**: Checklist for ensuring complete research coverage
- **references/example_learning_paths.md**: Examples of well-structured learning paths for various topics
