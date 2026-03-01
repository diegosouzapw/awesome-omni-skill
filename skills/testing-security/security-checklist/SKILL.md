---
name: security-checklist
description: Analyze a security checklist Excel file by interactively identifying the relevant sheet, countermeasure entries, and output column, then checking whether GitHub Enterprise supports each item using the customer-security-trust FAQ and past reference checklists.
compatibility: Designed for GitHub Copilot CLI
metadata:
  author: user
  version: "1.0"
---

# Security Checklist Analysis

You are tasked with analyzing a Japanese security checklist (セキュリティチェックリスト) from an Excel file. Your goal is to extract attack countermeasure entries and determine whether GitHub Enterprise (Cloud or Server) supports each requirement.

## Step 0: Interactive Setup (MANDATORY)

Before starting any analysis, you **MUST** ask the user these questions one at a time using the `ask_user` tool. **Do NOT assume default values.**

### 0.1 — Identify the sheet
1. Open the Excel file using the `xlsx` skill.
2. List all available sheet names to the user.
3. Ask: **"Which sheet contains the security checklist?"**
   - Provide the sheet names as choices.

### 0.2 — Identify the items column
1. Once the sheet is selected, scan column headers (typically row 1–5).
2. Show the user the detected headers/columns.
3. Ask: **"Which column contains the items to analyze?"**
   - Provide the column letters as choices with a preview of their content.

### 0.3 — Identify the output column
1. Ask: **"Which column should I write the analysis results into?"**
   - Suggest columns that appear to be for notes/remarks (補足, 備考, コメント, etc.) or empty columns.
   - Make clear that existing content in that column will be preserved (new content prepended above it).

### 0.4 — Customer name
1. Ask: **"What is the customer name? This will be used to organize the output folder."**

Once all answers are collected, confirm the configuration summary with the user before proceeding.

## Reference Sources

### Primary: GitHub Customer Security Trust FAQ
Use the GitHub customer-security-trust FAQ as your primary reference:
- Repository: `github/customer-security-trust`
- File: `FAQ/FAQ.md`
- Fetch this file using the `github-mcp-server-get_file_contents` tool at the start of each analysis.

### Secondary: Previous Checklists (Reference Material)
Check the `50-security-checklists/references/` folder for previously completed checklists.
- If reference files exist, read them to find how similar security questions were answered before.
- Use past answers to ensure consistency across customer analyses.
- Mention when an answer is based on a precedent from a previous customer checklist.

> ***Note by Theo**: The Google Drive folder originally referenced as a secondary source (https://drive.google.com/drive/folders/0BzGOJmkt0vlTUmY0YkhCT0tRVzA) is **not accessible from Copilot CLI** — it requires Google authentication which the CLI does not have. Reference checklists from that folder should be manually downloaded and placed in `50-security-checklists/references/` to be usable.*

## Process Steps

### Step 1: Parse the Excel File

1. Navigate to the sheet identified in Step 0.
2. Scan for the countermeasure column/section identified by the user.
3. Extract all attack countermeasure entries, preserving:
   - The item number/ID (if present)
   - The countermeasure description
   - Any category or grouping information
   - Any additional context columns (priority, notes, etc.)

### Step 2: Fetch the GitHub Security FAQ

1. Use the GitHub MCP tool to fetch the FAQ:
   ```
   github-mcp-server-get_file_contents:
     owner: github
     repo: customer-security-trust
     path: FAQ/FAQ.md
   ```
2. Parse the FAQ content to build a searchable knowledge base of GitHub Enterprise security capabilities.
3. Pay attention to sections covering:
   - Authentication & access control (認証・アクセス制御)
   - Encryption (暗号化)
   - Audit logging (監査ログ)
   - Network security (ネットワークセキュリティ)
   - Data protection (データ保護)
   - Vulnerability management (脆弱性管理)
   - Incident response (インシデント対応)
   - Compliance & certifications (コンプライアンス・認証)

### Step 3: Check Reference Checklists

1. Look in `50-security-checklists/references/` for any existing Excel files.
2. If found, scan them for similar security questions and how they were previously answered.
3. Build a mapping of common questions → proven answers to ensure consistency.

### Step 4: Match Each Entry Against GitHub Capabilities

For each countermeasure entry:

1. **Translate/understand** the Japanese requirement into its security concept.
2. **Search the FAQ** for relevant answers about GitHub Enterprise support.
3. **Check reference checklists** for precedents (how was this answered before?).
4. **Classify** each entry as one of:
   - ✅ **Supported** — GitHub Enterprise directly supports this requirement. Include the relevant FAQ excerpt.
   - ⚠️ **Partially Supported** — GitHub Enterprise covers some aspects but not all. Explain what is and isn't covered.
   - ❌ **Not Supported** — No evidence in the FAQ that GitHub Enterprise supports this. Note if it might be achievable via third-party integrations.
   - ❓ **Unclear / Needs Confirmation** — The FAQ doesn't clearly address this item. Suggest follow-up questions for the GitHub account team.

### Step 5: Write Results Back to the Excel File

After analyzing each item, write your findings into the **output column identified in Step 0** using `openpyxl`.

**Do NOT modify any other columns** — only write to the designated output column.

Use this structured format:

```
【判定】✅ Supported / ⚠️ Partially Supported / ❌ Not Supported / ❓ Unclear
【詳細】[Concise explanation of how GitHub Enterprise supports (or doesn't support) this item, in Japanese or English]
```

**Example entries:**

- `【判定】✅ Supported\n【詳細】GitHub implements protocol filtering using load balancer infrastructure, DDoS protection, and DNS blackholing.`
- `【判定】⚠️ Partially Supported\n【詳細】GitHub does not hold ISO/IEC 27017 directly, but achieved CSA STAR Level 2 and ISO 27018 which include equivalent cloud security controls.`

**Rules for writing:**
- If the cell already has existing content, **prepend** your analysis above it, separated by a blank line, so existing notes are preserved.
- Use `\n` for line breaks within the cell.
- Save the updated workbook to `50-security-checklists/<customer-name>/<original-filename>-analyzed.xlsx`.

### Step 6: Present Results as Markdown Report

Save a structured markdown report to `50-security-checklists/<customer-name>/analysis-report.md`:

```markdown
## Security Checklist Analysis Report — [Customer Name]

**Date:** [Analysis date]
**Source file:** [Original filename]
**Sheet analyzed:** [Sheet name from Step 0]
**Countermeasure column:** [Column from Step 0]
**Output column:** [Column from Step 0]

### Summary
- Total items analyzed: X
- ✅ Supported: X
- ⚠️ Partially Supported: X
- ❌ Not Supported: X
- ❓ Unclear: X

### Detailed Results

| # | Countermeasure Item | Status | GitHub Enterprise Support Details | FAQ Reference |
|---|---------------------|--------|-----------------------------------|---------------|
| 1 | [Item description] | ✅ | [How GHE supports this] | [FAQ section] |
| 2 | [Item description] | ⚠️ | [What's covered / what's not] | [FAQ section] |
| ... | ... | ... | ... | ... |

### Items Requiring Follow-Up
[List any items marked ❓ with suggested questions for the GitHub team]

### Recommendations
[Any additional notes or recommendations based on the analysis]
```

Also display the summary in the CLI output.

## Important Notes

- **Always fetch the latest FAQ** — Do not rely on cached or prior knowledge of the FAQ content. Fetch it fresh each time.
- **Be conservative in classification** — If there's any doubt, mark as ⚠️ or ❓ rather than ✅.
- **IDS/IPS: Double-check carefully** — The FAQ explicitly states **"No"** to having IDS/IPS. GitHub uses alternative controls (protocol filtering, DDoS protection, DNS blackholing, anomaly detection) but does NOT have traditional IDS or IPS. These items should be classified as ⚠️ Partially Supported at best, NOT ✅ Supported.
- **Preserve original Japanese text** — Keep the original descriptions alongside any translations.
- **Distinguish between GHEC and GHES** — If support differs between GitHub Enterprise Cloud and Server, note the distinction.
- **Ask for clarification** — If anything is ambiguous during analysis, ask the user before making assumptions.
- **Large files** — If the FAQ is too large to process at once, use grep or targeted searches for specific security topics relevant to each checklist item.
