---
name: gradle-dependency-checker
description: Executes Gradle dependency check commands, retrieves and analyzes dependency trees, and extracts version information for key dependencies such as kotlin/kotlinx/skiko/androidx. Use when users need to check Gradle project dependency versions or analyze dependency relationships.
---

# Gradle Dependency Checker

**Path Note**: This document uses `{SKILL_ROOT}` as a placeholder for the skill root directory. The AI assistant should dynamically determine the absolute path of the skill root directory based on the currently open file path when executing commands. For example, if the current workspace path is `/Users/username/.cursor/skills/gradle-dependency-checker`, then `{SKILL_ROOT}` should be replaced with that path.

## Overview

This skill is used to execute Gradle dependency check commands, retrieve project dependency trees, and extract version information for key dependencies, specifically:

- `kotlin` - Kotlin standard library version
- `kotlinx.*` - Kotlin extension libraries (e.g., kotlinx-coroutines, kotlinx-serialization, etc.)
- `skiko` - Skia binding library version
- `androidx.*` - AndroidX library versions

## Usage

### View Dependency Reports

First check if the report file `{SKILL_ROOT}/references/compose-multiplatform-dependencies-{VERSION}.md` exists. If the report file does not exist, you need to execute the script to generate the dependency tree first.

### One-Click Script (Recommended)

**It is recommended to use the one-click script**, which automatically completes both steps of dependency tree generation and report generation:

```bash
# Process a single version
{SKILL_ROOT}/scripts/compose-multiplatform-check-and-parse.sh 1.10.0

# Process multiple versions (executed sequentially)
{SKILL_ROOT}/scripts/compose-multiplatform-check-and-parse.sh 1.9.3 1.10.0 1.8.2
```

**Script Description**:

- Script path: `{SKILL_ROOT}/scripts/compose-multiplatform-check-and-parse.sh`
- Supports single or multiple version number parameters
- Automatically completes two steps:
  1. Generate dependency tree (if it doesn't exist or is empty)
  2. Generate Markdown report
- Output files:
  - Dependency tree: `{SKILL_ROOT}/assets/dependency-checker/compose-multiplatform-checker/results/compose-multiplatform-dependencies-{VERSION}.txt`
  - Report: `{SKILL_ROOT}/references/compose-multiplatform-dependencies-{VERSION}.md`
- If the dependency tree file already exists and is not empty, the generation step will be skipped
- Multiple versions are processed sequentially, waiting for each version to complete before executing the next

## Workflow

**Important Note**: The AI assistant **must NOT** directly execute scripts or generate dependency tree/report files. All scripts must be manually executed by the user.

### Step 1: Check Report File

1. **Check report file**: First check if `{SKILL_ROOT}/references/compose-multiplatform-dependencies-{VERSION}.md` exists (the AI needs to dynamically determine the absolute path of `{SKILL_ROOT}` based on the current workspace path)
2. **If the report file exists**: Directly read and display the report content
3. **If the report file does not exist**: **Prompt the user** to execute the following command to generate the dependency tree and report (the AI must not directly execute, but needs to replace `{SKILL_ROOT}` with the actual absolute path)

### Step 2: Generate Dependency Tree and Report (if needed)

**Recommended method (one-click)**: The AI should prompt the user to execute the one-click script (need to replace `{SKILL_ROOT}` with the actual absolute path):

```bash
{SKILL_ROOT}/scripts/compose-multiplatform-check-and-parse.sh {VERSION}
```

One-click script description:

- Automatically completes both steps of dependency tree generation and report generation
- If the dependency tree file already exists and is not empty, the generation step will be skipped
- Supports passing multiple versions, the script will process them sequentially, waiting for completion before executing the next

## Output Format

Analysis results must strictly follow the following format (generated using the `{SKILL_ROOT}/scripts/compose-multiplatform-parse-dependencies.py` script):

```markdown
# Compose Multiplatform 1.10.0 Dependencies

## Kotlin

- `org.jetbrains.kotlin:kotlin-stdlib`: **2.2.20**

## Kotlinx

- `org.jetbrains.kotlinx:atomicfu`: **0.27.0**
- `org.jetbrains.kotlinx:kotlinx-coroutines-core`: **1.9.0**
- `org.jetbrains.kotlinx:kotlinx-serialization-core`: **1.7.3**

## Skiko

- `org.jetbrains.skiko:skiko`: **0.9.37.3**

## Compose Multiplatform

- `org.jetbrains.compose.animation:animation`: **1.10.0**
- `org.jetbrains.compose.foundation:foundation`: **1.10.0**
- `org.jetbrains.compose.runtime:runtime`: **1.10.0**
- `org.jetbrains.compose.ui:ui`: **1.10.0**

## JetBrains AndroidX Lifecycle

- `org.jetbrains.androidx.lifecycle:lifecycle-common`: **2.9.6**
- `org.jetbrains.androidx.lifecycle:lifecycle-runtime-compose`: **2.9.6**

## JetBrains AndroidX SavedState

- `org.jetbrains.androidx.savedstate:savedstate`: **1.3.6**

## AndroidX

- `androidx.compose.runtime:runtime`: **1.10.0**
- `androidx.collection:collection`: **1.5.0**
- `androidx.annotation:annotation`: **1.9.1**
- `androidx.lifecycle:lifecycle-common`: **2.9.4**
```

### Output Format Description

- Title format: `# Compose Multiplatform {VERSION} Dependencies`
- Dependency format: Use backticks to wrap the complete dependency coordinate, version number in bold
- Version information: Display the final resolved version (in bold), automatically handle version upgrade markers `->`
- Categorization: Organize by Kotlin, Kotlinx, Skiko, Compose Multiplatform, JetBrains AndroidX Lifecycle, JetBrains AndroidX SavedState, AndroidX
- Empty lines before and after titles and lists (compliant with MD022 and MD032 standards)
- Empty value handling: If a category has no dependencies, that category can be omitted

## Important Notes

- **AI assistant must NOT directly execute scripts or generate files**: All dependency tree and report generation must be completed by the user manually executing scripts
- **Path handling**: When the AI assistant executes commands or checks files, it needs to dynamically determine the absolute path of `{SKILL_ROOT}` based on the current workspace path and replace it in all paths and commands
- **Recommended to use one-click script**: `{SKILL_ROOT}/scripts/compose-multiplatform-check-and-parse.sh` will automatically complete dependency tree generation and report generation
- Only get dependencies from the `commonMainResolvableDependenciesMetadata` configuration
- Output files are saved at `{SKILL_ROOT}/assets/dependency-checker/compose-multiplatform-checker/results/compose-multiplatform-dependencies-{VERSION}.txt`
- Dependency reports are saved at `{SKILL_ROOT}/references/compose-multiplatform-dependencies-{VERSION}.md`
- **If the output file is empty, the script will automatically re-execute the Gradle command**
- Multiple versions are processed sequentially, cannot be executed in parallel
- Dependency tree output has been filtered to remove build information, only contains dependency tree content
- Dependencies managed by BOM need to identify the actually resolved version
- Output format must strictly follow the specifications in the "Output Format" section
- Use the `{SKILL_ROOT}/scripts/compose-multiplatform-parse-dependencies.py` script to parse the dependency tree and generate Markdown reports
