---
name: Plexus Classifier Guidelines Management
description: The format for guidelines documents for Plexus scorecard scores and the validation tool.
---

## Instructions

This skill helps create and update classifier guidelines documents for Plexus scorecard score configurations.  The user will provide you with information from subject matter experts that will commonly come from emails, chat messages, or other documents. **This source information may be formatted for different audiences (e.g., agent instructions, training materials, operational procedures) rather than for classifier design.**

**Your job is to transform this information into the Plexus guidelines standard format**, which is specifically designed to help distinguish between classification classes. The guidelines must be organized around **how to tell the difference between classes** (e.g., what makes something "Yes" vs "No"), NOT around operational procedures or agent instructions.

### Key Transformation Principles:

1. **Source Format → Classifier Format**: Convert operational rules ("agents must do X") into classification criteria ("classify as No if X is missing")
2. **Focus on Distinguishing Classes**: Organize information around "Definition of [Class]" and "Conditions for [Class]" sections
3. **Make Conditional Logic Explicit**: When requirements depend on context (e.g., "if metadata contains X, then Y is required"), express both sides of the condition clearly in the classification criteria
4. **Extract Classification Criteria**: Identify what observable features distinguish one class from another, even if the source material doesn't explicitly frame it that way

After you make any change to the guidelines you need to use the tool to validate the guidelines file.

You can use the Plexus MCP tools to pull score versions, including either the champion or specific versions.  And you can use the MCP tool for pushing new score versions with updated guidelines, after you make changes to the guidelines and validate the changes using the tool in this skill.  You may NOT push updates without first validating them, and you may not push guidelines documents that are invalid.  Making changes to the score configuration is out of scope for this skill, this is all about the guidelines.

## Context

Plexus (AnthusAI/Plexus on GitHub) uses human-readable Guidelines documents (Markdown format) alongside YAML-based classifier configurations. The Guidelines express how to make classification decisions and serve as the source of truth for alignment between human subject-matter experts (SMEs), AI/ML engineers, and the LLM-based classifiers.

## Guidelines Format Standards

There are three types of classifier guidelines, each with required elements:

### Binary Classifier
Required sections (marked with *):
- Classifier Name (title)
- Objective
- Classes (with metadata: Valid labels, Target class, Default class)
- Definition of No
- Conditions for No
- Definition of Yes

Optional sections:
- Examples (Clear No Cases, Clear Yes Cases, Boundary Cases)

### Binary Classifier with Abstentions
Required sections (marked with *):
- Classifier Name (title)
- Objective
- Classes (with metadata: Valid labels, Target class, Default class, Abstain class)
- Definition of No
- Conditions for No
- Definition of NA
- Conditions for NA
- Definition of Yes

Optional sections:
- Examples (Clear No Cases, Clear Yes Cases, Clear NA Cases, Boundary Cases)

### Multi-Class Classifier
Required sections (marked with *):
- Classifier Name (title)
- Objective
- Classes (with metadata: Valid labels as list)
- Definition of [Each Class]
- Conditions for [Each Class]

Optional sections:
- Boundary Conditions ([Class A] vs [Class B] for each pair)
- Examples (Clear [Class] Cases for each class, Boundary Cases)

## Workflow

When creating or updating guidelines:

1. **Identify the classifier type** (binary, binary with abstentions, or multi-class)
2. **Use the validation tool** to check which required sections exist
3. **Work with the user** to fill in missing required sections
4. **Create or update** the guidelines document
5. **Run validation again** to confirm all required sections are present
6. **Iterate** until validation passes

## Key Principles

- **Transform, don't copy** - Source material is often written for agents/operations; transform it into classifier decision criteria
- **Classification-focused** - Guidelines must help distinguish between classes, not just describe procedures
- **Required sections are non-negotiable** - guidelines documents must include all required sections
- **Make conditionals explicit** - When classification depends on context (metadata, modality, etc.), state both branches clearly
- **Work collaboratively** - if information is missing, ask the user rather than making assumptions
- **Validate after every update** - always run the validation tool after modifying guidelines
- **Preserve existing content** - when updating, maintain all existing sections unless explicitly asked to remove them
- **Follow the format exactly** - section headers must match the standard format

## Validation Tool

Use the `validate_guidelines.py` tool to check guidelines documents for compliance. The tool:
- Detects classifier type automatically
- Checks for all required sections
- Reports missing or malformed sections
- Provides actionable feedback

Always run this tool after creating or updating guidelines documents.

**Usage:**
```bash
python validate_guidelines.py guidelines.md
```

**Exit codes:**
- 0: Validation passed
- 1: Validation failed

**Example output:**
```
Validation Results for: content_moderation.md

Classifier Type: Binary

Detected binary classifier
✓ All required sections present

Found Sections:
  • Objective ✓
  • Classes ✓
  • Definition of No ✓
  • Conditions for No ✓
  • Definition of Yes ✓
  • Examples (optional)

✓ Validation PASSED
```

## Example Usage

**Creating new guidelines:**
1. User provides classifier objective and class information
2. Determine classifier type from class count and structure
3. Run validation to see what's missing
4. Work with user to define each required section
5. Create complete guidelines document
6. Run validation to confirm success

**Updating existing guidelines:**
1. User provides guidelines file and requested changes
2. Run validation on current document
3. Make requested changes while preserving structure
4. Run validation to ensure no required sections were lost
5. If validation fails, work with user to fix issues

## Common Issues

- **Missing Classes metadata**: Ensure Target class and Default class are specified for binary classifiers
- **Missing class definitions**: Multi-class classifiers need Definition and Conditions sections for every class
- **Inconsistent class names**: Class names in Conditions sections must exactly match those in Classes metadata
- **Malformed headers**: Section headers must match the standard format (e.g., "## Definition of No" not "## No Definition")

## Instructions for Claude

When using this skill:

1. **Always validate first**: Before making changes, run the validation tool on any existing guidelines
2. **Show validation results**: Display the output to the user so they can see what's missing
3. **Work iteratively**: For missing required sections, ask the user for information one section at a time
4. **Validate after changes**: After creating or updating guidelines, run validation again
5. **Don't assume**: If information is missing, ask rather than inventing content
6. **Preserve structure**: Keep all existing sections unless explicitly asked to remove them
7. **Use exact format**: Follow the standard section headers precisely

**Example interaction:**
```
User: Create guidelines for a spam classifier
Claude: I'll help you create guidelines for a spam classifier. First, let me understand the structure.

Is this a binary classifier (Spam/Not Spam) or does it need an NA class for uncertain cases?

User: Binary - just Spam or Not Spam

Claude: Got it. For a binary classifier, I need:
- Objective (what is the purpose of this classifier?)
- Target class (which class are we trying to detect - I assume "Spam"?)
- Default class (what do we classify when conditions aren't met - I assume "Not Spam"?)
- Definition of Spam
- Conditions for identifying Spam
- Definition of Not Spam

Let's start with the Objective. What should this classifier do?

[User provides information]

[Claude creates draft guidelines]

Let me validate this:
[Runs validation tool]
[Shows results]

The guidelines are now complete and valid. Would you like to add optional Examples sections?
```

**Example transformation scenario:**
```
User: Here are the agent instructions for branding requirements: "Agents must state school name, program name, degree type, and modality."

Claude: I need to transform these agent instructions into classification criteria. Let me think about how to distinguish between Yes and No:

For a binary classifier about branding compliance:
- "Conditions for No" would include: "Rep fails to state school name" (converting "must state" into a failure condition)
- "Conditions for Yes" would include: "Rep states school name" (the positive case)

I also notice this rule might have conditional logic. Does the degree type requirement depend on anything (like whether it's in metadata)?

[User clarifies conditional requirements]

[Claude creates guidelines with explicit conditional logic in both Conditions for No and Conditions for Yes sections]
```
