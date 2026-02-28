---
name: PatchEvergreen Breaking Changes Analyzer
description: Expert skill for analyzing breaking changes, compatibility issues, and migration planning for programming libraries across multiple languages using the PatchEvergreen database.
version: 1.0.0
author: PatchEvergreen
category: Development Tools
tags:
  - breaking-changes
  - dependency-management
  - migration-planning
  - library-analysis
  - compatibility
  - python
  - javascript
  - java
  - php
  - ruby
  - go
  - rust
mcp_servers:
  - PatchEvergreen
---

# PatchEvergreen Breaking Changes Analyzer

## Overview

The PatchEvergreen Breaking Changes Analyzer is a specialized skill that helps developers understand, analyze, and plan migrations for library version upgrades. It provides expert guidance on breaking changes, compatibility issues, and migration strategies across multiple programming languages.

This skill works with the PatchEvergreen MCP server to access a comprehensive database of breaking changes for libraries in Python, JavaScript, Java, PHP, Ruby, Go, Rust, and other languages.

## Capabilities

This skill can help you:

- **Analyze Breaking Changes**: Identify and understand breaking changes between library versions
- **Plan Migrations**: Create detailed upgrade plans with step-by-step guidance
- **Assess Compatibility**: Evaluate how breaking changes will impact your codebase
- **Conduct Dependency Audits**: Systematically review all project dependencies for issues
- **Generate Migration Reports**: Create comprehensive reports with actionable recommendations

## Prerequisites

To use this skill, you need:

1. **PatchEvergreen MCP Server**: The skill requires access to the PatchEvergreen MCP server (either stdio or SSE transport)
2. **Library Information**: Know the library name as it appears in package managers (e.g., `requests`, `lodash`, `django`, `phpmailer/phpmailer`)
3. **Language Context**: Identify the programming language of the library

## Library Naming Convention

**Important**: Libraries must be named exactly as they appear in their respective package manager files:

- **Python**: As in `requirements.txt`, `setup.py`, or `pyproject.toml` (e.g., `requests`, `django`, `numpy`)
- **JavaScript/Node.js**: As in `package.json` (e.g., `lodash`, `express`, `react`)
- **PHP**: As in `composer.json` (e.g., `phpmailer/phpmailer`, `symfony/symfony`)
- **Ruby**: As in `Gemfile` (e.g., `rails`, `nokogiri`)
- **Rust**: As in `Cargo.toml` (e.g., `serde`, `tokio`)
- **Java**: As in `pom.xml`, `build.gradle.kts`, or `libs.versions.toml` (e.g., `org.springframework:spring-core`)
- **Go**: As in `go.mod` (e.g., `github.com/gin-gonic/gin`)

## Workflows

### 1. Quick Breaking Changes Check

**When to use**: You need a quick overview of breaking changes for a specific library.

**Process**:
1. Identify the library name and programming language
2. Use the `get_issues_for_library` tool with the library name and language
3. Review the returned breaking changes data
4. Summarize the most critical issues that could affect your codebase

**Example**:
```
Check breaking changes for the Python library "requests"
```

### 2. Comprehensive Breaking Changes Analysis

**When to use**: You need a detailed analysis with migration guidance for a library upgrade.

**Process**:
1. Use the `analyze_breaking_changes` prompt template
2. Provide the library name and language
3. The skill will guide you through:
   - Breaking changes summary
   - Version impact assessment
   - Migration priority ranking
   - Code examples (before/after)
   - Action plan
   - Testing recommendations
   - Timeline estimation

**Example**:
```
Analyze breaking changes for django in Python and provide migration guidance
```

### 3. Dependency Audit

**When to use**: You want to audit all dependencies in a project for breaking changes.

**Process**:
1. Use the `dependency_audit_report` prompt template
2. Specify the project's primary programming language
3. For each dependency you provide:
   - The skill will fetch breaking changes data
   - Classify impact (High/Medium/Low/No Impact)
   - Create an update strategy
   - Provide timeline and resource planning

**Example**:
```
Create a dependency audit report for my Python project. Dependencies: requests, django, numpy
```

### 4. Version Upgrade Planning

**When to use**: You're planning to upgrade a library from one specific version to another.

**Process**:
1. Use the `version_upgrade_planner` prompt template
2. Provide:
   - Library name
   - Programming language
   - Current version
   - Target version
3. The skill will create:
   - Pre-upgrade assessment
   - Incremental upgrade strategy
   - Code impact analysis
   - Testing strategy
   - Implementation plan
   - Change management procedures

**Example**:
```
Plan an upgrade for express (JavaScript) from version 4.17.0 to 5.0.0
```

### 5. Compatibility Impact Summary

**When to use**: You need a focused summary of how breaking changes will affect compatibility.

**Process**:
1. Use the `compatibility_impact_summary` prompt template
2. Provide library name and language
3. The skill will analyze:
   - API breaking changes
   - Configuration changes
   - Behavioral changes
   - Integration impact
   - Migration effort assessment
   - Compatibility recommendations

**Example**:
```
Summarize compatibility impacts for phpmailer/phpmailer in PHP
```

## Best Practices

### Library Name Accuracy
- Always use the exact library name from your package manager file
- For namespaced packages (like PHP's `vendor/package`), include the full namespace
- When in doubt, check your `package.json`, `composer.json`, `requirements.txt`, etc.

### Language Specification
- Use lowercase language names: `python`, `javascript`, `java`, `php`, `ruby`, `go`, `rust`
- For JavaScript, use `javascript` (not `node` or `nodejs`)
- For TypeScript libraries, still use `javascript` as the language

### Incremental Analysis
- Start with a quick check to see if breaking changes exist
- Then use more detailed analysis prompts for libraries with significant changes
- Prioritize high-impact dependencies first

### Version Context
- When planning upgrades, always specify both current and target versions
- Consider intermediate versions if a direct upgrade seems risky
- Review changelogs alongside PatchEvergreen data for complete context

## Integration with MCP Server

This skill is designed to work with the PatchEvergreen MCP server. The server provides:

- **Tool**: `get_issues_for_library(library: str, language: str)` - Fetches breaking changes data
- **Prompts**: Five specialized prompt templates for different analysis scenarios

### MCP Server Connection

**For SSE (Server-Sent Events)**: The server runs on a hosted endpoint and can be accessed via HTTP/SSE transport.

**For stdio**: The server can run locally and communicate via standard input/output.

## Error Handling

If you encounter errors:

1. **Library not found**: Verify the library name matches exactly what's in your package manager file
2. **Language mismatch**: Ensure you're using the correct language identifier (lowercase, standard name)
3. **API errors**: The PatchEvergreen API may be temporarily unavailable; retry after a short delay
4. **No breaking changes**: Some libraries may not have breaking changes recorded in the database

## Examples

### Example 1: Quick Check
```
User: "What breaking changes exist for the Python requests library?"
Skill: Uses get_issues_for_library("requests", "python") and summarizes key breaking changes
```

### Example 2: Migration Planning
```
User: "I need to upgrade Django from 3.2 to 4.2. Help me plan this."
Skill: Uses version_upgrade_planner with library="django", language="python",
       current_version="3.2", target_version="4.2" to create comprehensive plan
```

### Example 3: Project Audit
```
User: "Audit my JavaScript project dependencies: express, lodash, axios"
Skill: Uses dependency_audit_report for each library, creates impact matrix,
       and prioritizes updates
```

## Limitations

- The PatchEvergreen database may not contain all libraries or all breaking changes
- Some breaking changes may be undocumented or not yet recorded
- The skill provides guidance but cannot guarantee complete coverage
- Always test upgrades in a development environment before production deployment

## Support

For issues, questions, or contributions related to this skill or the PatchEvergreen MCP server, please refer to the project repository.

---

*This skill leverages the PatchEvergreen database to help developers make informed decisions about library upgrades and migrations.*
