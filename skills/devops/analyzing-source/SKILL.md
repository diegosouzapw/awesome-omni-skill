---
name: analyzing-source
description: Conducts in-depth analysis of a specific source or topic, producing comprehensive summaries for research synthesis. Use when you need detailed analysis and documentation of individual sources as part of a larger research effort.
---

# Analyzing Source

This skill guides you through analyzing a single source in depth and creating a comprehensive summary suitable for research synthesis.

## Process

Follow these steps to analyze a source and create a comprehensive summary:

### Step 1: Source Discovery and Retrieval

**If given a URL:**

- Fetch it directly using WebFetch
- Verify the content is accessible and relevant

**If given a topic or search query:**

- Use WebSearch to find the best source on the topic
- Prioritize authoritative, detailed sources
- Fetch the most relevant result using WebFetch

**If source is inaccessible or low-quality:**

- Try alternative sources
- Be persistent in finding substantive information
- Note any access issues in your summary

### Step 2: Deep Analysis

Conduct thorough analysis focusing on:

- **Core concepts, definitions, and frameworks** presented in the source
- **Main arguments, claims, and findings** - what is the source asserting?
- **Evidence, data, and examples** - what supports the claims?
- **Methodologies or approaches** - how was this work conducted?
- **Limitations, caveats, and counterarguments** - what are the boundaries?
- **Connections to broader themes** - how does this relate to the research focus?
- **Quality and credibility** - how reliable is this source?
- **Unique insights or perspectives** - what new understanding does this offer?

### Step 3: Create Comprehensive Summary

Use the template from `./templates/article-summary.md` to create your summary.

**VERY IMPORTANT:** Your summary must be concise yet thorough, which means being extreme information-dense and leveraging key data as much as possible.

**Template structure includes:**
- Executive summary
- Key concepts & definitions
- Main arguments/findings with evidence
- Methodology/approach
- Specific examples & case studies
- Notable quotes
- Critical evaluation
- Relevance to research focus
- Practical implications

**Key principles:**

- Include specific quotes and examples, not just paraphrasing
- Provide analytical insights about significance and relevance
- Make clear connections to the research focus provided
- Be detailed enough that someone can understand the source without reading the original

### Step 4: Save Summary File

**Create filename:**

- Use a descriptive slug based on the source
- Example: `kubernetes-scaling-patterns.md`, `netflix-chaos-engineering.md`

**Save location:**

- Save to: `{working_directory}/summaries/{filename}.md`
- Use the complete template structure
- Ensure all sections are filled out

### Step 5: Report Results

Provide a brief report including:
1. Confirmation of what source you analyzed
2. The file path where you saved the summary
3. A 2-3 sentence overview of the most important insights discovered

## Important Guidelines

- **Be thorough, not brief**: This is deep research, not light scanning. Capture nuance and detail.
- **Include specific evidence**: Direct quotes, data points, examples - not just general statements.
- **Think critically**: Note limitations, assess quality, identify assumptions.
- **Stay focused**: While being comprehensive, ensure everything relates to the research focus.
- **Be self-contained**: Your summary should make sense without reading the original source.
- **Save your work**: Always save the summary file - the main coordinator depends on it.

## Example Execution

```
Input received:
- Source topic: "Kubernetes horizontal pod autoscaling best practices"
- Research focus: "Scalability patterns in cloud-native systems"
- Working directory: /Users/research/cloud-native-scaling

Step 1: Using WebSearch to find authoritative source on Kubernetes HPA...
Found: kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/
Fetching with WebFetch...

Step 2: Analyzing content...
- Identified core HPA concepts: target metrics, scale-up/down policies, cooldown periods
- Found detailed configuration examples with CPU and custom metrics
- Noted limitations around cluster resources and metric collection latency

Step 3: Creating comprehensive summary using template...
- Executive summary: 3 paragraphs covering main patterns and tradeoffs
- Key concepts: HPA, target utilization, metric servers, custom metrics API
- Main findings: 5 configuration patterns with evidence from examples
- 8 notable quotes extracted from official docs and linked blog posts
- Evidence quality: High (official documentation + real-world examples)

Step 4: Saving summary...
Created: /Users/research/cloud-native-scaling/summaries/kubernetes-hpa-best-practices.md

Step 5: Report
Source analyzed: Kubernetes official documentation on Horizontal Pod Autoscaling
Saved to: /Users/research/cloud-native-scaling/summaries/kubernetes-hpa-best-practices.md

Key insights: This source provides detailed HPA configuration patterns with real-world examples from production systems at scale. Most valuable finding is the discussion of custom metrics integration and the tradeoffs between reactive vs predictive scaling approaches. Also documents common pitfalls like resource request misconfiguration causing scaling issues.
```
