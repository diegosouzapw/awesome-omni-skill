---
name: maximize-ai-tone
description: Comprehensive tone, audience, and content guide for writing chapters in "Maximize Your Life with AI: Build an AI Operating System for Learning, Productivity, and Life". Use this alongside prose-generation for the soul, structure, and drive of the book.
---

# Maximize Your Life with AI: Tone & Content Guide

This guide ensures that any agent writing or editing chapters for **Maximize Your Life with AI** maintains the exact audience understanding, outcome goals, tone, content structure, and driving philosophy established in the first five chapters.

## 1. Audience

- **Primary:** University students balancing coursework, jobs, and life (across all disciplines).
- **Secondary:** Faculty learning to teach _with_ AI rather than policing it.
- **Tertiary:** Working professionals, creators, and lifelong learners seeking an "operating system" for personal and professional productivity.
- **Empathy & Needs:** The audience is capable but busy, overwhelmed by unstructured information, and looking for immediate, practical leverage. They need workflows that compound, not just a list of tools to try once.

## 2. Outcome (The Promise)

- **Core Goal:** Move the reader from being a passive consumer of AI (which research shows leads to "cognitive offloading" and declining abilities) to an active director who uses AI as a true tool, a personal "Chief of Staff," and an operating system. This is not about technical evangelism; it is about practical, everyday leverage.
- **Deliverables:** By the end of the book, readers build a complete portfolio: prompt libraries, custom assistants, study guides, weekly review templates, career materials, and an ethical framework. They will have tactile examples in place for task management, document manipulation, and autonomous intel.
- **Value Proposition:** Save time, manage cognitive overhead, produce higher-quality work, and automate repetitive tasks across five domains: Academics, Career, Health/Wellness, Finance, and Relationships/Communication.

## 3. Tone & Voice

The tone must be **authoritative yet accessible, highly practical, modern, and empowering.**

- **Perspective:** Second-Person ("You"). Speak directly to the reader as a smart, capable peer/mentor.
- **Evidence-Based Authority:** Back up claims with recent, high-quality research (e.g., Harvard, McKinsey, World Economic Forum, Anthropic). Mention the studies efficiently to build trust.
- **Rhythm & Style:**
  - **Aggressively vary sentence length.** Use short sentences for punchy emphasis.
  - **No rule-of-three patterns or anaphora.** Avoid repetitive structural gimmicks.
  - **Avoid tidy conclusions.** Do not use "Ultimately," "In the end," or "In summary." End sections with forward momentum, not neat bows.
  - **Reduce em dashes.** Keep them out of prose; restrict to code blocks, tables, and specific formatting.
  - **No excessive signposting.** (e.g., "In the next section, we will discuss...") The structure should flow naturally.
- **Mindset:** "Human Judgment Is the Product." The AI does not replace the human; it augments human intellect.

## 4. Content Structure

Every chapter MUST precisely follow the **Standard Chapter Template** and use the exact XML elements found in the DocBook/MyEducator schema (`<itemizedlist>`, `<informalfigure>`, `<section>`, `<important>`, etc.):

1. **Introduction:** A compelling hook, a visual/cinematic image description (`<informalfigure>`), and a bold "Learning Objectives" paragraph containing an `<itemizedlist>`. Ends with "Builds on:" and "Prepares for:".
2. **Core Concepts:** 2-3 subsections max conveying the overarching frameworks and research principles.
3. **AI Platform Features:** Single section. Starts with Google Gemini (free for students), followed by Claude (first half chapters) or ChatGPT (second half chapters). Includes video placeholders and feature comparisons.
4. **Builds (Usually 2-4 per chapter):** Self-contained, step-by-step activities applying the **CRSI Loop** (Capture → Run → Store → Improve). Includes workflow explanation, prompt templates, example outputs, and check-ins. Do NOT put workflows in a separate section.
5. **Prompt Library:** A summary table/guidance for retaining the prompts built, placed AFTER all Builds.
6. **Apply and Reflect:** Wraps up use cases, common mistakes, ethics, integration, and checklists.
7. **Assignments:** Student-facing portfolio builds, concept quizzes (progressive MC/open balance), and rubrics.
8. **Instructor Resources:** Discussion questions, in-class activities, grading, assessment, and slide directions.
9. **References:** Wrapper: `<section id="chN_references">` with `<title>References</title>`. Format: APA 7th edition, alphabetically sorted. Each reference is an individual `<para>` tag (no lists) with `<ulink>` tags for URLs. It must be the last child of the section.

## 5. The Drive (The Philosophy)

- **Systems Over Prompts:** Individual prompts are fragile. Workflows and systems compound over time.
- **Direct, Then Verify:** AI hallucinates. The reader must always direct with clear context and verify the output with integrity.
- **Augmentation, Not Automation:** Do not use AI to do the thinking for you. Use it to extend your capabilities.
- **The CRSI Loop (Capture, Run, Store, Improve):** This captures the mechanical drive of the book. Turning scattered inputs into stored assets that get refined week over week until it is an automatic system.
- **Active vs. Passive Use:** Emphasize that passive AI use erodes skills, while active, structured use multiplies productivity and quality.

**_Usage Note:_** _Combine this document with the `.agents/skills/prose-generation/SKILL.md` skill to ensure the actual XML formatting, Flesch readability targets, visual layout, and chapter lengths meet the required technical capabilities._
