---
name: repository-analyzer
description: "Comprehensive repository analysis using Explore agents, web search, and Context7 to investigate codebase structure, technology stack, configuration, documentation quality, and provide actionable insights. Use this skill when asked to analyze, audit, investigate, or report on a repository or codebase. | Exploreエージェント、Web検索、Context7を用いた包括的なリポジトリ分析。コードベース構造、技術スタック、設定、ドキュメント品質を調査し、実用的な洞察を提供。リポジトリやコードベースの分析、監査、調査、レポート作成を依頼された場合に使用。"
---

# Repository Analyzer Skill

This skill provides a systematic approach to analyzing any code repository through multi-source investigation and critical evaluation.

## Purpose

Enable Claude to perform thorough repository analysis by:
1. Exploring codebase structure and organization using Explore agents
2. Identifying technology stack and dependencies
3. Researching best practices via web search
4. Fetching authoritative documentation via Context7
5. Evaluating configuration quality and completeness
6. Assessing documentation and maintainability
7. Providing actionable recommendations

## When to Use This Skill

Activate this skill when users request:
- "Analyze this repository"
- "Audit the codebase"
- "Report on this repository's structure"
- "Investigate the current repository"
- "What technologies does this repo use?"
- "Review the repository configuration"
- "Assess the documentation quality"

## Analysis Framework

### Phase 1: Repository Discovery & Scoping

**Identify Repository Context:**

1. **Location & Scope**
   ```bash
   # Determine working directory
   pwd

   # Check if it's a git repository
   git rev-parse --show-toplevel

   # Get repository metadata
   git remote -v
   git log --oneline -n 10
   ```

2. **Initial Structure Assessment**
   - Use Explore agent with "medium" thoroughness
   - Identify root-level files and directories
   - Locate key configuration files
   - Determine primary language(s)

**Prompt for Explore Agent (Phase 1):**
```
Explore the repository at [path] with medium thoroughness.

Find and report:
1. Directory structure (major folders and their purposes)
2. Configuration files (package.json, Cargo.toml, pyproject.toml, etc.)
3. Primary programming languages (by file extensions)
4. Entry points (main files, executables)
5. Documentation files (README, docs/, etc.)
6. Build/deployment files (Dockerfile, Makefile, CI configs)
```

### Phase 2: Technology Stack Analysis

**Identify All Technologies:**

1. **Programming Languages**
   - Analyze file extensions
   - Check language-specific config files
   - Determine primary vs. secondary languages

2. **Package Managers & Dependencies**
   - Node.js: `package.json`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`
   - Python: `requirements.txt`, `pyproject.toml`, `Pipfile`, `poetry.lock`
   - Rust: `Cargo.toml`, `Cargo.lock`
   - Ruby: `Gemfile`, `Gemfile.lock`
   - Go: `go.mod`, `go.sum`
   - PHP: `composer.json`, `composer.lock`
   - Other: `Makefile`, `CMakeLists.txt`, etc.

3. **Frameworks & Libraries**
   - Read package manager files
   - Identify major dependencies
   - Categorize: frontend, backend, testing, tooling

4. **Development Tools**
   - Version managers: `.nvmrc`, `.ruby-version`, `.tool-versions`
   - Formatters: `.prettierrc`, `.eslintrc`, `rustfmt.toml`
   - Linters: `.eslintrc`, `pylint.rc`, `.rubocop.yml`
   - Type checkers: `tsconfig.json`, `mypy.ini`

5. **Infrastructure & Deployment**
   - Docker: `Dockerfile`, `docker-compose.yml`
   - CI/CD: `.github/workflows/`, `.gitlab-ci.yml`, `.circleci/`
   - Cloud: `terraform/`, `cloudformation/`, `k8s/`

**Prompt for Explore Agent (Phase 2):**
```
Explore the repository for technology stack identification with very thorough mode.

Search for and analyze:
1. All package manager files (package.json, Cargo.toml, requirements.txt, etc.)
2. Configuration files for development tools
3. Docker and containerization files
4. CI/CD pipeline configurations
5. Infrastructure as code files
6. Build system files

For each found file, provide:
- File path
- Type/purpose
- Key dependencies or configurations
```

### Phase 3: Context7 Documentation Research

**Research Identified Technologies:**

For each major technology/library identified in Phase 2:

1. **Resolve Library IDs**
   ```
   Use Context7's resolve-library-id for:
   - Major frameworks (e.g., "next.js", "react", "vue")
   - Key libraries (e.g., "axios", "lodash", "sqlalchemy")
   - Development tools (e.g., "vitest", "eslint", "prettier")
   ```

2. **Fetch Documentation**
   ```
   Use Context7's get-library-docs with topics:
   - "configuration best practices"
   - "project structure"
   - "common patterns"
   - "performance optimization"
   ```

3. **Compare Against Best Practices**
   - How does the repository's usage align with official recommendations?
   - Are there deprecated patterns being used?
   - Are there missed optimization opportunities?

### Phase 4: Web Search for Best Practices

**Research Industry Standards:**

1. **General Repository Practices**
   - Web search: "[primary language] project structure best practices 2025"
   - Web search: "[framework] application architecture patterns"
   - Web search: "dotfiles management best practices" (if applicable)

2. **Technology-Specific Patterns**
   - Web search: "[technology] configuration optimization"
   - Web search: "[framework] performance best practices"
   - Web search: "[tool] common mistakes to avoid"

3. **Security & Maintenance**
   - Web search: "[language] security best practices"
   - Web search: "dependency management [package manager]"
   - Web search: "CI/CD pipeline optimization"

### Phase 5: Configuration Deep Dive

**Analyze Key Configuration Files:**

1. **Read Critical Configs**
   - Use Read tool for important configuration files
   - Check for:
     - Version pinning vs. version ranges
     - Security configurations
     - Performance settings
     - Environment-specific configurations
     - Secret management

2. **Evaluate Configuration Quality**
   - Completeness: Are all necessary configs present?
   - Consistency: Do configs align across the project?
   - Documentation: Are configs well-commented?
   - Security: Are secrets properly handled?
   - Maintainability: Are configs modular and DRY?

**Prompt for Explore Agent (Phase 5):**
```
Explore configuration files in depth with very thorough mode.

Focus on:
1. Environment configuration (.env templates, config files)
2. Secret management patterns
3. Build configuration optimization
4. Development vs production configs
5. Version locking strategies
```

### Phase 6: Documentation Assessment

**Evaluate Documentation Quality:**

1. **Existence Check**
   - [ ] README.md at root
   - [ ] CONTRIBUTING.md
   - [ ] LICENSE
   - [ ] CHANGELOG.md
   - [ ] docs/ directory
   - [ ] API documentation
   - [ ] Code comments
   - [ ] Type annotations/JSDoc

2. **Quality Assessment**
   - **README.md:**
     - Clear project description?
     - Installation instructions?
     - Usage examples?
     - Configuration guide?
     - Contribution guidelines?
     - License information?

   - **Code Documentation:**
     - Function/method documentation?
     - Complex logic explained?
     - Type signatures/annotations?
     - Examples provided?

3. **Documentation Gaps**
   - Identify missing documentation
   - Note outdated information
   - Flag confusing sections

### Phase 7: Code Quality & Patterns

**Analyze Code Organization:**

1. **Directory Structure**
   - Is structure logical and consistent?
   - Are concerns properly separated?
   - Is there clear separation of layers?
   - Are naming conventions followed?

2. **Code Patterns**
   - Identify architectural patterns
   - Check for anti-patterns
   - Assess modularity
   - Evaluate reusability

3. **Testing Strategy**
   - Test files location and organization
   - Test coverage indicators
   - Testing frameworks used
   - CI test automation

**Prompt for Explore Agent (Phase 7):**
```
Explore code organization and patterns with very thorough mode.

Analyze:
1. Directory structure and naming conventions
2. Test files and testing strategy
3. Shared/common code organization
4. Configuration management patterns
5. Error handling approaches
6. Logging and monitoring setup
```

### Phase 8: Dependencies & Security

**Audit Dependencies:**

1. **Dependency Health**
   - Count total dependencies
   - Identify outdated packages
   - Check for security vulnerabilities
   - Assess dependency tree depth

2. **Version Management**
   - Are versions locked properly?
   - Overly restrictive version constraints?
   - Too loose version ranges?

3. **License Compliance**
   - What licenses are dependencies using?
   - Any licensing conflicts?
   - Attribution requirements met?

### Phase 9: CI/CD & DevOps

**Evaluate Automation:**

1. **CI/CD Pipeline**
   - What tests run automatically?
   - Deployment automation level?
   - Code quality checks?
   - Security scanning?

2. **Development Workflow**
   - Pre-commit hooks?
   - Code review process?
   - Branch protection?
   - Release process?

3. **Infrastructure as Code**
   - Environment reproducibility?
   - Configuration management?
   - Deployment consistency?

### Phase 10: Synthesis & Reporting

**Generate Comprehensive Report:**

## Report Structure

```markdown
# Repository Analysis Report: [Repository Name]

## Executive Summary

**Repository:** [Name/Path]
**Primary Language:** [Language]
**Main Framework/Purpose:** [Description]
**Overall Health Score:** [X/10]

**Key Findings:**
- [Finding 1]
- [Finding 2]
- [Finding 3]

**Critical Issues:** [Count]
**Recommendations:** [Count]

---

## 1. Repository Overview

### Basic Information
- **Location:** [Path]
- **Git Remote:** [URL if available]
- **Last Updated:** [Date]
- **Total Files:** [Count]
- **Total Lines:** [Estimate]

### Purpose & Scope
[Description of what this repository does]

---

## 2. Technology Stack

### Programming Languages
| Language | Percentage | Files | Purpose |
|----------|-----------|-------|---------|
| [Lang]   | [X%]      | [N]   | [Desc]  |

### Major Dependencies

**Production:**
- [Dependency 1] ([version]) - [Purpose]
- [Dependency 2] ([version]) - [Purpose]

**Development:**
- [Dependency 1] ([version]) - [Purpose]
- [Dependency 2] ([version]) - [Purpose]

### Frameworks & Tools
- **Frontend:** [Frameworks]
- **Backend:** [Frameworks]
- **Testing:** [Frameworks]
- **Build Tools:** [Tools]
- **CI/CD:** [Tools]

---

## 3. Repository Structure

### Directory Organization

```
repository/
├── [dir1]/          # [Purpose]
├── [dir2]/          # [Purpose]
└── [dir3]/          # [Purpose]
```

### Key Files
- `[file1]` - [Purpose]
- `[file2]` - [Purpose]

**Structure Assessment:** [Evaluation]

---

## 4. Configuration Analysis

### Configuration Files Found
- [Config file 1] - [Quality: Good/Fair/Poor]
- [Config file 2] - [Quality: Good/Fair/Poor]

### Configuration Quality

**Strengths:**
- [Strength 1]
- [Strength 2]

**Weaknesses:**
- [Weakness 1]
- [Weakness 2]

### Secret Management
[Assessment of how secrets are handled]

---

## 5. Documentation Quality

### Existing Documentation
- [x] README.md - [Quality score]
- [ ] CONTRIBUTING.md - [Missing/Present]
- [x] Code comments - [Quality score]

### Documentation Assessment

**Coverage:** [X/10]
**Quality:** [X/10]
**Maintainability:** [X/10]

**Gaps Identified:**
1. [Gap 1]
2. [Gap 2]

---

## 6. Code Quality & Patterns

### Architectural Patterns
[Identified patterns and their appropriateness]

### Code Organization
**Rating:** [X/10]

**Strengths:**
- [Strength 1]

**Concerns:**
- [Concern 1]

### Testing Strategy
- **Test Coverage:** [Estimated %]
- **Test Frameworks:** [Frameworks]
- **CI Integration:** [Yes/No]

---

## 7. Dependencies & Security

### Dependency Health
- **Total Dependencies:** [Count]
- **Outdated:** [Count]
- **Security Vulnerabilities:** [Count]

### Dependency Management
[Assessment of version locking, update strategy]

---

## 8. DevOps & Automation

### CI/CD Pipeline
[Description of automation setup]

**Automated Checks:**
- [ ] Tests
- [ ] Linting
- [ ] Security scanning
- [ ] Build verification

### Development Workflow
[Description of git workflow, hooks, etc.]

---

## 9. Best Practices Comparison

### Alignment with Industry Standards

| Practice | Current | Recommended | Gap |
|----------|---------|-------------|-----|
| [Practice 1] | [Status] | [Standard] | [Description] |
| [Practice 2] | [Status] | [Standard] | [Description] |

### Context7 Insights
[Findings from official documentation comparison]

### Web Research Findings
[Insights from industry best practices research]

---

## 10. Recommendations

### Critical (Fix Immediately)
1. **[Issue]**
   - **Impact:** [High/Medium/Low]
   - **Effort:** [High/Medium/Low]
   - **Action:** [Specific steps]

### Important (Fix Soon)
1. **[Issue]**
   - **Impact:** [High/Medium/Low]
   - **Effort:** [High/Medium/Low]
   - **Action:** [Specific steps]

### Nice to Have (Consider)
1. **[Issue]**
   - **Impact:** [High/Medium/Low]
   - **Effort:** [High/Medium/Low]
   - **Action:** [Specific steps]

---

## 11. Quick Wins

[List of easy improvements with high impact]

1. [Quick win 1] - [Estimated time: Xm]
2. [Quick win 2] - [Estimated time: Xm]

---

## 12. Technical Debt Assessment

**Overall Technical Debt:** [High/Medium/Low]

**Areas of Concern:**
1. [Area 1]
2. [Area 2]

**Suggested Refactoring:**
1. [Refactoring 1]
2. [Refactoring 2]

---

## 13. Maintainability Score

| Aspect | Score | Notes |
|--------|-------|-------|
| Code Organization | [X/10] | [Notes] |
| Documentation | [X/10] | [Notes] |
| Testing | [X/10] | [Notes] |
| Configuration | [X/10] | [Notes] |
| Dependencies | [X/10] | [Notes] |
| **Overall** | **[X/10]** | |

---

## 14. Research Sources

### Context7 Documentation Consulted
- [Library 1] - [Topic]
- [Library 2] - [Topic]

### Web Research Conducted
- [Search query 1] - [Key finding]
- [Search query 2] - [Key finding]

### Explore Agent Investigations
- [Investigation 1] - [Finding]
- [Investigation 2] - [Finding]

---

## 15. Next Steps

### Immediate Actions
1. [Action 1]
2. [Action 2]

### Short-term (1-2 weeks)
1. [Action 1]
2. [Action 2]

### Long-term (1-3 months)
1. [Action 1]
2. [Action 2]

---

## Appendix

### Full Technology List
[Complete list of all technologies found]

### All Configuration Files
[Complete list with brief descriptions]

### Dependency Tree
[If relevant and not too large]

---

**Report Generated:** [Date]
**Analysis Duration:** [Estimated time]
**Claude Skills Used:** repository-analyzer, deep-research (if applicable)
**Tools Used:** Explore agent, Context7, WebSearch, Read, Grep, Glob
```

## Execution Strategy

### Parallel vs Sequential

**Run in Parallel (when possible):**
- Multiple Explore agent investigations (different aspects)
- Web searches for different technologies
- Context7 lookups for multiple libraries

**Run Sequentially (when needed):**
1. Discovery first (need to know structure)
2. Technology identification second (need to know what to research)
3. Deep dives third (need context from earlier phases)
4. Synthesis last (need all information)

### Time Management

**Estimated Duration:**
- Small repository (< 100 files): 10-15 minutes
- Medium repository (100-1000 files): 15-30 minutes
- Large repository (> 1000 files): 30-60 minutes

**Optimization:**
- Use Explore agent efficiently (right thoroughness level)
- Don't read every file - sample strategically
- Focus on high-impact findings
- Prioritize actionable insights

### Thoroughness Levels

**Quick Audit (10 min):**
- Phase 1-3 only
- Basic structure, tech stack, major issues

**Standard Analysis (30 min):**
- Phase 1-6
- Complete overview with recommendations

**Deep Audit (60 min):**
- All phases
- Comprehensive analysis with detailed recommendations

**Custom Focus:**
Ask user what aspects to prioritize if time is limited.

## Special Repository Types

### Dotfiles Repository (like chezmoi)

**Focus on:**
- File organization and chezmoi-specific patterns
- Template usage and variable management
- Secret management (age encryption, 1Password integration)
- Cross-platform compatibility
- Backup and restore strategies
- Documentation for setup

**Key Files:**
- `.chezmoi.toml.tmpl`, `chezmoi.toml`
- Template files (`.tmpl` extension)
- Encrypted files (`.age` extension)
- Scripts and hooks
- README and setup guides

### Monorepo

**Focus on:**
- Package/workspace organization
- Shared dependencies management
- Build orchestration
- Independent deployments
- Code sharing patterns

### Library/Package

**Focus on:**
- API design and documentation
- Versioning strategy
- Breaking changes handling
- Examples and usage guides
- Publishing workflow

### Web Application

**Focus on:**
- Frontend/backend separation
- State management
- Routing structure
- API design
- Performance optimization
- Security headers and practices

## Quality Checklist

Before finalizing the report, verify:

- [ ] All major directories explored
- [ ] All configuration files identified
- [ ] Technology stack fully documented
- [ ] Context7 consulted for major technologies
- [ ] Web research conducted for best practices
- [ ] Specific, actionable recommendations provided
- [ ] Evidence cited for all claims
- [ ] Quick wins identified
- [ ] Critical issues highlighted
- [ ] Report is well-structured and readable

## Usage Examples

### Example 1: Analyze Current Repository

User: "Analyze this repository"

Claude:
1. Runs `pwd` to get location
2. Launches Explore agent (Phase 1)
3. Identifies it's a chezmoi dotfiles repo
4. Launches Phase 2 exploration for tech stack
5. Uses Context7 for chezmoi documentation
6. Web searches for dotfiles best practices
7. Analyzes configuration quality
8. Generates comprehensive report

### Example 2: Focus on Security

User: "Audit the security configuration of this repository"

Claude:
1. Focuses on Phase 5 (Configuration) and Phase 8 (Security)
2. Searches for secret management patterns
3. Checks for exposed credentials
4. Evaluates dependency vulnerabilities
5. Reviews CI/CD security
6. Provides security-focused report

### Example 3: Technology Stack Report

User: "What technologies does this repository use?"

Claude:
1. Runs Phase 2 (Technology Stack Analysis)
2. Uses Explore agent to find all config files
3. Parses package managers files
4. Uses Context7 to get documentation for major libraries
5. Provides structured technology report

## Tips for Effective Analysis

1. **Start Broad, Then Deep**
   - Get the big picture first
   - Then drill into specifics

2. **Let Explore Agent Do the Work**
   - Don't manually grep every file
   - Use appropriate thoroughness level
   - Multiple focused explorations > one massive exploration

3. **Context7 for Authority**
   - Official documentation is most reliable
   - Use for major frameworks/libraries
   - Compare repo's usage against official recommendations

4. **Web Search for Trends**
   - Industry best practices
   - Common pitfalls
   - Recent developments (2025 standards)

5. **Be Actionable**
   - Every finding should have a recommendation
   - Prioritize by impact and effort
   - Provide specific steps, not vague advice

6. **Be Honest**
   - Acknowledge what you don't know
   - State confidence levels
   - Suggest areas for human expert review

## Notes

- **Scope Management:** For very large repositories, ask user to specify focus areas
- **Permissions:** Some operations may require user approval (file reads, web searches)
- **Time Awareness:** Inform user of estimated duration upfront
- **Iteration:** Offer to deep-dive into specific areas after initial report
- **Export:** Offer to save report as markdown file in repository
