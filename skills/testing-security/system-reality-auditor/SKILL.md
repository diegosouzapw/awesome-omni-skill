---
name: system-reality-auditor
description: Performs a triple-point audit to synchronize codebase, database schema, and project documentation states. Use this skill to align the PRD, Gap Analysis, and Execution Plan with the actual system reality.
---

# System Reality Auditor

This skill provides instructions on how to perform a comprehensive triple-point audit to ensure all master project documents accurately reflect reality.

## 1. Code Audit
Use tools like `find_by_name`, `list_dir`, or recursive file reading to map all implemented features in the codebase. Verify the actual presence of UI components, server actions, helpers, and API routes.

## 2. DB Audit
Use the Supabase MCP to fetch the latest database schema. Inspect:
- Tables
- Columns
- RPCs (Remote Procedure Calls)

## 3. Alignment Logic
Compare the Code and DB findings against the master project documents (e.g., `PRD.md`, `EXECUTION_PLAN.md`, `GAP_ANALYSIS.md`, `DATABASE_SCHEMA.md`).
- If a task is marked `[ ]` (pending) in the plan but actually exists in the code or database, mark it as `[x]`.
- If a task is marked `[x]` (completed) or done in the plan but the feature does not exist in the code or database, uncheck it to `[ ]` and note the gap.

## 4. Update Authority
You have full update authority over the project documents. Rewrite the 'Status' and 'Gaps' sections of all documents to match the current ground truth. Ensure all documents are strictly synchronized with the codebase and database realities after the audit.
