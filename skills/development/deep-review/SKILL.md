---
name: deep-review
description: Run a comprehensive deep review combining architecture analysis, code review, error handling audit, type design analysis, comment verification, test coverage analysis, accessibility audit, localization review, concurrency analysis, performance analysis, code simplification, and platform-specific reviews (iOS, macOS, Android, Angular, TypeScript, Next.js, Vue.js, Python, Django, Ruby, Rust, Go, Rails, Flutter, Java/Spring Boot, C#/.NET, PHP/Laravel, C/C++, React Native, Svelte/SvelteKit, Elixir/Phoenix, Kotlin Server, Scala, Terraform, Shell/Bash, Docker, Kubernetes, GraphQL, GitHub Actions, SQL, Swift Data). Platform reviewers are automatically included when relevant. Distinguishes between NEW issues (introduced by PR) and PRE-EXISTING issues (technical debt). Use when reviewing PR changes, before merging, or for thorough code quality assessment. Supports flags --pr, --branch, --changes for scope detection.
argument-hint: "[aspects] [--pr|--branch|--changes|path]"
---

# Deep Review Skill

Run a comprehensive deep review using a team of specialized agents covering architecture, code quality, error handling, types, comments, tests, accessibility, localization, concurrency, performance, and simplification.

## When to Use

- Before creating or merging a PR
- After completing a feature branch
- For thorough code quality assessment
- To identify technical debt and architectural issues
- When you want a complete picture of code health

## Issue Classification

When reviewing PR changes, all issues are classified as:

- **[NEW]**: Issues in code **added or modified** by this PR. These must be addressed before merge.
- **[PRE-EXISTING]**: Issues in code **not changed** by this PR. These are technical debt observations that should not block the PR but are valuable to track.

This distinction helps reviewers focus on what's actionable for the current PR while still surfacing important context about surrounding code quality.

## Scope Detection

Determine the analysis scope from flags or arguments:

1. **If `--pr` or `--branch` flag provided**:
   - Detect base branch: check for `main`, then `master`, or use `git merge-base`
   - Run `git diff --name-only <base>...HEAD` to get all files changed in this branch
   - Analyze those files and their immediate dependencies

2. **If `--changes` flag provided**:
   - Run `git diff --name-only HEAD` and `git diff --name-only --cached` for uncommitted changes
   - Analyze those files and their immediate dependencies

3. **If path argument provided** (e.g., `/deep-review src/features`):
   - Analyze only that path

4. **If no arguments**:
   - Default to `--pr` behavior (analyze current branch changes)

## Review Aspects

Select which aspects to review. Default is `core` (code + errors + arch).

| Aspect | Description |
|--------|-------------|
| `code` | CLAUDE.md compliance, bugs, code quality |
| `errors` | Silent failures, catch blocks, error handling |
| `arch` | Dependencies, cycles, hotspots, patterns, scale |
| `types` | Type invariants, encapsulation, design quality |
| `comments` | Comment accuracy, rot, maintainability |
| `tests` | Test coverage, quality, critical gaps |
| `simplify` | Code clarity, refactoring opportunities |
| `a11y` | WCAG compliance, ARIA, keyboard nav, screen readers |
| `l10n` | Hardcoded strings, i18n readiness, locale handling, RTL |
| `concurrency` | Race conditions, deadlocks, thread safety, async pitfalls |
| `perf` | Algorithmic complexity, allocations, caching, rendering, N+1 queries |
| `core` | code + errors + arch (default) |
| `full` | All cross-cutting aspects (does not include platform-specific) |

**Platform-specific aspects** (automatically included when relevant, or explicitly requested):

| Aspect | Description |
|--------|-------------|
| `ios` | Swift/SwiftUI/UIKit lifecycle, ARC, Apple APIs, App Store compliance |
| `android` | Activity/Fragment lifecycle, Compose, manifest, Android security |
| `ts-frontend` | React/Vue/Angular state, SSR/hydration, component patterns, browser APIs |
| `ts-backend` | Node.js event loop, middleware, ORM, auth, graceful shutdown, API design |
| `python` | Pythonic idioms, type hints, Django/FastAPI/Flask, packaging |
| `rust` | Ownership idioms, unsafe auditing, error handling, trait design |
| `go` | Go idioms, interface design, context propagation, module hygiene |
| `rails` | Rails conventions, ActiveRecord, migration safety, background jobs |
| `flutter` | Widget design, state management, Dart idioms, platform channels |
| `java` | Spring Boot, JPA/Hibernate, bean lifecycle, enterprise patterns |
| `dotnet` | ASP.NET Core, Entity Framework, LINQ, C# idioms |
| `php` | Laravel, Composer, Eloquent, Blade, PHP 8+ features |
| `cpp` | Modern C++ (11/14/17/20), memory safety, RAII, STL, templates |
| `react-native` | Bridge perf, native modules, platform-specific code paths |
| `svelte` | Svelte reactivity, SvelteKit routing, compile-time patterns |
| `elixir` | OTP/GenServer, Phoenix LiveView, BEAM concurrency |
| `kotlin-server` | Ktor, coroutines, Kotlin idioms for server-side |
| `scala` | Functional patterns, Akka/Spark, implicits, effect systems |
| `macos` | AppKit, SwiftUI for macOS, sandboxing, XPC, notarization, desktop integration |
| `nextjs` | Server/Client Components, App Router, caching, Server Actions, middleware |
| `vue` | Vue 3 Composition API, Nuxt 3, Pinia, reactivity patterns, template syntax |
| `django` | Django ORM, DRF, migrations, template security, middleware, signals |
| `ruby` | Ruby idioms, metaprogramming safety, gem hygiene, RSpec/Minitest patterns |
| `terraform` | HCL, state management, IAM security, module design, blast radius control |
| `shell` | Bash/POSIX sh quoting, error handling, portability, CI/CD script safety |
| `angular` | Angular DI, RxJS, change detection, signals, template safety |
| `docker` | Dockerfile layers, multi-stage builds, security, PID 1, Compose |
| `kubernetes` | K8s manifests, resource limits, security contexts, RBAC, probes, Helm |
| `graphql` | Schema design, resolver N+1, query security, authorization, DataLoader |
| `github-actions` | Workflow security, secret handling, action pinning, runner config |
| `sql` | SQL query optimization, schema design, migration safety, injection, ORM fallback |
| `swift-data` | SwiftData, Core Data, GRDB persistence patterns, migrations, concurrency |
| `mobile` | ios + android |
| `ts` | ts-frontend + ts-backend |
| `jvm` | java + kotlin-server + scala |
| `apple` | ios + macos |
| `infra` | terraform + shell |
| `containers` | docker + kubernetes |

Platform reviewers are **automatically included** when the team lead determines they are relevant based on the changed files and project context. For example, changing `.swift` files in an iOS project will include the iOS reviewer. The team lead uses its judgment to disambiguate — `.swift` in a macOS project triggers macOS (not iOS), `.kt` in a Ktor server won't trigger Android, `.py` in a Django project triggers django (not just python), `.vue` files trigger vue (not ts-frontend), Next.js projects trigger nextjs (not just ts-frontend), `.sql` files trigger sql, Swift projects using SwiftData/CoreData/GRDB trigger swift-data. Users can also explicitly request platform aspects (e.g., `/deep-review ios`). Platform aspects are never included in `core` or `full` unless detected or explicitly requested.

**Usage examples:**
```
/deep-review                    # core review of PR changes (+ auto-detected platforms)
/deep-review --pr               # explicit PR scope (+ auto-detected platforms)
/deep-review --changes          # uncommitted changes only (+ auto-detected platforms)
/deep-review full --pr          # all cross-cutting agents on PR (+ auto-detected platforms)
/deep-review code errors        # specific aspects only (+ auto-detected platforms)
/deep-review types tests --pr   # type and test analysis of PR (+ auto-detected platforms)
/deep-review a11y --pr          # accessibility audit of PR
/deep-review l10n --pr          # localization review of PR
/deep-review concurrency --pr   # concurrency analysis of PR
/deep-review perf --pr          # performance analysis of PR
/deep-review ios --pr           # explicitly include iOS reviewer
/deep-review apple --pr         # iOS + macOS reviewers
/deep-review ts --pr            # both TypeScript frontend + backend reviewers
/deep-review mobile --pr        # iOS + Android reviewers
/deep-review nextjs --pr        # Next.js reviewer (Server Components, App Router)
/deep-review vue --pr           # Vue.js reviewer (Composition API, Nuxt)
/deep-review django --pr        # Django reviewer (ORM, DRF, migrations)
/deep-review angular --pr       # Angular reviewer (RxJS, DI, change detection)
/deep-review containers --pr    # Docker + Kubernetes reviewers
/deep-review graphql --pr       # GraphQL reviewer (schema, resolvers, security)
/deep-review infra --pr         # Terraform + Shell reviewers
/deep-review python rust --pr   # explicitly include Python and Rust reviewers
/deep-review sql --pr           # SQL reviewer (queries, schema, migrations, injection)
/deep-review swift-data --pr    # Swift Data reviewer (SwiftData, Core Data, GRDB)
/deep-review src/features       # analyze specific path (+ auto-detected platforms)
```

## Agent Dispatch Table

| Agent ID | Aspect | Model | Agent File |
|----------|--------|-------|------------|
| code-reviewer | code | opus | agents/code-reviewer.md |
| silent-failure-hunter | errors | inherit | agents/silent-failure-hunter.md |
| dependency-mapper | arch | inherit | agents/dependency-mapper.md |
| cycle-detector | arch | inherit | agents/cycle-detector.md |
| hotspot-analyzer | arch | inherit | agents/hotspot-analyzer.md |
| pattern-scout | arch | inherit | agents/pattern-scout.md |
| scale-assessor | arch | inherit | agents/scale-assessor.md |
| type-design-analyzer | types | inherit | agents/type-design-analyzer.md |
| comment-analyzer | comments | inherit | agents/comment-analyzer.md |
| test-analyzer | tests | inherit | agents/test-analyzer.md |
| code-simplifier | simplify | opus | agents/code-simplifier.md |
| accessibility-scanner | a11y | inherit | agents/accessibility-scanner.md |
| localization-scanner | l10n | inherit | agents/localization-scanner.md |
| concurrency-analyzer | concurrency | inherit | agents/concurrency-analyzer.md |
| performance-analyzer | perf | inherit | agents/performance-analyzer.md |
| ios-platform-reviewer | ios | inherit | agents/ios-platform-reviewer.md |
| android-platform-reviewer | android | inherit | agents/android-platform-reviewer.md |
| ts-frontend-reviewer | ts-frontend | inherit | agents/ts-frontend-reviewer.md |
| ts-backend-reviewer | ts-backend | inherit | agents/ts-backend-reviewer.md |
| python-reviewer | python | inherit | agents/python-reviewer.md |
| rust-reviewer | rust | inherit | agents/rust-reviewer.md |
| go-reviewer | go | inherit | agents/go-reviewer.md |
| rails-reviewer | rails | inherit | agents/rails-reviewer.md |
| flutter-reviewer | flutter | inherit | agents/flutter-reviewer.md |
| java-reviewer | java | inherit | agents/java-reviewer.md |
| dotnet-reviewer | dotnet | inherit | agents/dotnet-reviewer.md |
| php-reviewer | php | inherit | agents/php-reviewer.md |
| cpp-reviewer | cpp | inherit | agents/cpp-reviewer.md |
| react-native-reviewer | react-native | inherit | agents/react-native-reviewer.md |
| svelte-reviewer | svelte | inherit | agents/svelte-reviewer.md |
| elixir-reviewer | elixir | inherit | agents/elixir-reviewer.md |
| kotlin-server-reviewer | kotlin-server | inherit | agents/kotlin-server-reviewer.md |
| scala-reviewer | scala | inherit | agents/scala-reviewer.md |
| macos-platform-reviewer | macos | inherit | agents/macos-platform-reviewer.md |
| nextjs-reviewer | nextjs | inherit | agents/nextjs-reviewer.md |
| vue-reviewer | vue | inherit | agents/vue-reviewer.md |
| django-reviewer | django | inherit | agents/django-reviewer.md |
| ruby-reviewer | ruby | inherit | agents/ruby-reviewer.md |
| terraform-reviewer | terraform | inherit | agents/terraform-reviewer.md |
| shell-reviewer | shell | inherit | agents/shell-reviewer.md |
| angular-reviewer | angular | inherit | agents/angular-reviewer.md |
| docker-reviewer | docker | inherit | agents/docker-reviewer.md |
| kubernetes-reviewer | kubernetes | inherit | agents/kubernetes-reviewer.md |
| graphql-reviewer | graphql | inherit | agents/graphql-reviewer.md |
| github-actions-reviewer | github-actions | inherit | agents/github-actions-reviewer.md |
| sql-reviewer | sql | inherit | agents/sql-reviewer.md |
| swift-data-reviewer | swift-data | inherit | agents/swift-data-reviewer.md |

All agents use `subagent_type: "general-purpose"` (needed for file writing).

## Instructions

### Phase 1: Determine Scope

1. Parse arguments to extract:
   - Scope flag: `--pr`, `--branch`, `--changes`, or path
   - Aspects: list of aspects or `core`/`full`

2. Get changed files based on scope:
   ```bash
   # For --pr/--branch (detect base branch first)
   BASE=$(git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null || echo "HEAD~10")
   git diff --name-only $BASE...HEAD

   # For --changes
   git diff --name-only HEAD
   git diff --name-only --cached
   ```

3. **Get detailed diff with line numbers** (for distinguishing new vs pre-existing issues):
   ```bash
   # Get the unified diff showing which lines were added/modified
   git diff $BASE...HEAD --unified=0 | grep -E '^@@|^diff --git'
   ```
   This output shows the exact line ranges that were changed. Parse it to build a map of `{file: [changed_line_ranges]}`.

4. Build the scope context string (referred to as `SCOPE_CONTEXT` below):
   ```
   SCOPE: Focus analysis on these files and their direct dependencies:
   {list of changed files}

   CHANGED LINE RANGES (for classifying issues):
   {file1}: lines {start1}-{end1}, {start2}-{end2}, ...
   {file2}: lines {start1}-{end1}, ...

   IMPORTANT - Issue Classification:
   When reporting issues, you MUST classify each issue as one of:
   - **[NEW]**: Issue is in code that was ADDED or MODIFIED in this PR (within the changed line ranges above)
   - **[PRE-EXISTING]**: Issue is in code that was NOT changed by this PR (outside the changed line ranges)

   This distinction is critical for PR review. New issues should be fixed before merge.
   Pre-existing issues are technical debt to track but should not block the PR.
   ```

### Phase 1.5: Determine Platform Reviewers

After obtaining the list of changed files, determine which platform-specific reviewers to include. Available platform reviewers and what they cover:

| Aspect | Covers |
|--------|--------|
| `ios` | Swift/SwiftUI/UIKit lifecycle, ARC, Apple APIs, App Store compliance |
| `macos` | AppKit, SwiftUI for macOS, sandboxing, XPC, notarization, desktop integration |
| `android` | Activity/Fragment lifecycle, Compose, manifest, Android security |
| `ts-frontend` | React/Vue/Angular state, SSR/hydration, component patterns, browser APIs |
| `ts-backend` | Node.js event loop, middleware, ORM, auth, graceful shutdown, API design |
| `nextjs` | Server/Client Components, App Router, caching, Server Actions, middleware |
| `vue` | Vue 3 Composition API, Nuxt 3, Pinia, reactivity patterns, template syntax |
| `python` | Pythonic idioms, type hints, Django/FastAPI/Flask, packaging |
| `django` | Django ORM, DRF, migrations, template security, middleware, signals |
| `ruby` | Ruby idioms, metaprogramming safety, gem hygiene, RSpec/Minitest patterns |
| `rust` | Ownership idioms, unsafe auditing, error handling, trait design |
| `go` | Go idioms, interface design, context propagation, module hygiene |
| `rails` | Rails conventions, ActiveRecord, migration safety, background jobs |
| `flutter` | Widget design, state management, Dart idioms, platform channels |
| `java` | Spring Boot, JPA/Hibernate, bean lifecycle, enterprise patterns |
| `dotnet` | ASP.NET Core, Entity Framework, LINQ, C# idioms |
| `php` | Laravel, Composer, Eloquent, Blade, PHP 8+ features |
| `cpp` | Modern C++ (11/14/17/20), memory safety, RAII, STL, templates |
| `react-native` | Bridge perf, native modules, platform-specific code paths |
| `svelte` | Svelte reactivity, SvelteKit routing, compile-time patterns |
| `elixir` | OTP/GenServer, Phoenix LiveView, BEAM concurrency |
| `kotlin-server` | Ktor, coroutines, Kotlin idioms for server-side |
| `scala` | Functional patterns, Akka/Spark, implicits, effect systems |
| `terraform` | HCL, state management, IAM security, module design, blast radius control |
| `shell` | Bash/POSIX sh quoting, error handling, portability, CI/CD script safety |
| `angular` | Angular DI, RxJS, change detection, signals, template safety |
| `docker` | Dockerfile layers, multi-stage builds, security, PID 1, Compose |
| `kubernetes` | K8s manifests, resource limits, security contexts, RBAC, probes, Helm |
| `graphql` | Schema design, resolver N+1, query security, authorization, DataLoader |
| `github-actions` | Workflow security, secret handling, action pinning, runner config |
| `sql` | SQL query optimization, schema design, migration safety, injection, ORM fallback |
| `swift-data` | SwiftData, Core Data, GRDB persistence patterns, migrations, concurrency |

**If the user explicitly requested platform aspects** (e.g., `/deep-review ios`, `/deep-review python rust`), use those directly.

**If the user did not request any platform aspects**, look at the changed files and the project context to decide which platform reviewers are relevant. Use your judgment — examine file extensions, imports, build files, and project structure to determine the right reviewers. Be precise: `.swift` files in a macOS project should trigger macOS (not iOS), `.kt` files in a Ktor server should not trigger Android, `.ts` files in an Express app should trigger `ts-backend` not `ts-frontend`, `.vue` files should trigger `vue` (not `ts-frontend`), projects with `next.config.*` should trigger `nextjs`, projects with Django's `settings.py`/`manage.py` should trigger `django`, `.tf` files should trigger `terraform`, `.sh`/`.bash` files should trigger `shell`, Angular projects (`angular.json`) should trigger `angular`, `Dockerfile`/`docker-compose.yml` should trigger `docker`, K8s manifests (YAML with `apiVersion`/`kind`) should trigger `kubernetes`, `.graphql`/`.gql` files or GraphQL schema definitions should trigger `graphql`, `.github/workflows/*.yml` files should trigger `github-actions`, `.sql` files or SQL migration directories should trigger `sql`, Swift projects with SwiftData imports (`import SwiftData`), `.xcdatamodeld` directories (Core Data), or GRDB imports (`import GRDB`) should trigger `swift-data`. The `sql` reviewer also acts as a fallback for ORM patterns not covered by a dedicated framework reviewer — if the project uses an ORM like Sequelize, Prisma, SQLAlchemy, Knex, or Diesel without a framework-specific reviewer, include `sql`. When genuinely uncertain, skip rather than guess wrong — the user can always request a platform reviewer explicitly.

**Group alias expansion**:
- `mobile` → `ios`, `android`
- `ts` → `ts-frontend`, `ts-backend`
- `jvm` → `java`, `kotlin-server`, `scala`
- `apple` → `ios`, `macos`
- `infra` → `terraform`, `shell`
- `containers` → `docker`, `kubernetes`

**Merge behavior**:
- Platform aspects are **added to** whatever cross-cutting aspects the user requested
- Platform aspects are never included in `core` or `full` expansion — they only come from auto-detection or explicit request
- Deduplicate: if auto-detection finds `ts-frontend` and the user also typed `ts`, only include `ts-frontend` once

### Phase 2: Determine Which Agents to Launch

Based on selected aspects (including any auto-detected platform aspects from Phase 1.5):

| Aspect | Agents to Launch |
|--------|-----------------|
| `core` | Code Reviewer, Silent Failure Hunter, all 5 Architecture agents |
| `full` | All agents below |
| `code` | Code Reviewer |
| `errors` | Silent Failure Hunter |
| `arch` | Dependency Mapper, Cycle Detector, Hotspot Analyzer, Pattern Scout, Scale Assessor |
| `types` | Type Design Analyzer |
| `comments` | Comment Analyzer |
| `tests` | Test Analyzer |
| `simplify` | Code Simplifier |
| `a11y` | Accessibility Scanner |
| `l10n` | Localization Scanner |
| `concurrency` | Concurrency Analyzer |
| `perf` | Performance Analyzer |
| `ios` | iOS Platform Reviewer |
| `macos` | macOS Platform Reviewer |
| `android` | Android Platform Reviewer |
| `ts-frontend` | TypeScript Frontend Reviewer |
| `ts-backend` | TypeScript Backend Reviewer |
| `nextjs` | Next.js Reviewer |
| `vue` | Vue.js Reviewer |
| `python` | Python Reviewer |
| `django` | Django Reviewer |
| `ruby` | Ruby Reviewer |
| `rust` | Rust Reviewer |
| `go` | Go Reviewer |
| `rails` | Rails Reviewer |
| `flutter` | Flutter Reviewer |
| `java` | Java Reviewer |
| `dotnet` | .NET Reviewer |
| `php` | PHP Reviewer |
| `cpp` | C/C++ Reviewer |
| `react-native` | React Native Reviewer |
| `svelte` | Svelte Reviewer |
| `elixir` | Elixir Reviewer |
| `kotlin-server` | Kotlin Server Reviewer |
| `scala` | Scala Reviewer |
| `terraform` | Terraform Reviewer |
| `shell` | Shell/Bash Reviewer |
| `angular` | Angular Reviewer |
| `docker` | Docker Reviewer |
| `kubernetes` | Kubernetes Reviewer |
| `graphql` | GraphQL Reviewer |
| `github-actions` | GitHub Actions Reviewer |
| `sql` | SQL Reviewer |
| `swift-data` | Swift Data Reviewer |

### Phase 3: Create Results Directory and Launch Background Agents

1. **Create results directory**:
   ```bash
   mkdir -p /tmp/deep-review-$(uuidgen | tr '[:upper:]' '[:lower:]')/
   ```
   Store the path as `REVIEW_DIR`.

2. **Spawn all analysis agents in parallel as background tasks**:
   Use a **single message** with multiple Task tool calls (one per agent) so they all launch concurrently:
   - `subagent_type`: `"general-purpose"`
   - `model`: from dispatch table (`opus` or omit for inherit)
   - `run_in_background`: `true`
   - `prompt`: use the Agent Prompt Template below, filled in with the agent's details
   - Do NOT use `team_name` or `name` — these are standalone background agents, NOT team members

   Each Task call returns immediately with a `task_id`. Store the mapping of `{agent-id → task_id}`.

### Phase 4: Wait for Completion and Collect Results

1. **Poll for output files** using a bash loop (keeps context minimal compared to reading TaskOutput for every agent):
   ```bash
   # Wait until all expected output files exist (poll interval: 5s)
   EXPECTED_FILES=("{agent-id-1}.md" "{agent-id-2}.md" ...)
   TIMEOUT=600; ELAPSED=0
   while [ $ELAPSED -lt $TIMEOUT ]; do
     ALL_DONE=true
     for f in "${EXPECTED_FILES[@]}"; do
       [ ! -f "{REVIEW_DIR}/$f" ] && ALL_DONE=false && break
     done
     $ALL_DONE && break
     sleep 5; ELAPSED=$((ELAPSED + 5))
   done
   ```
   Adapt the file list and REVIEW_DIR to actual values.

2. **If the timeout expires with missing files**: For each missing agent, call `TaskOutput` with `block: false` using the stored `task_id` to check whether the agent is still running or has failed. If still running, continue waiting (repeat the poll loop). If failed, record it in the gap report and move on.

3. After all agents have completed (or been recorded as failed), check which output files exist.
4. Build a gap report string listing any agents that failed to produce output.

### Phase 5: Launch Synthesis Agent

1. **Spawn the synthesis agent** (as a background task to minimize context):
   - `subagent_type`: `"general-purpose"`
   - `run_in_background`: `true`
   - `prompt`: Include the following in the prompt:
     - Path to the synthesis instructions file: `agents/synthesizer.md`
     - The `REVIEW_DIR` path
     - The list of expected output files (one per agent that was launched)
     - The gap report (if any agents failed)
     - The scope description (for the report header)
     - Instruction to write the final report to `{REVIEW_DIR}/REPORT.md`

   Store the returned `task_id` for the synthesis agent.

2. **Wait for completion** using `TaskOutput` with `block: true` and `timeout: 600000` (10 minutes) on the synthesis `task_id`. This blocks until the agent actually finishes — no arbitrary timeout guessing. The returned output can be ignored; the report is in the file.

### Phase 6: Present Report

1. **Read the report**: Read `{REVIEW_DIR}/REPORT.md` and present its contents to the user
2. **Inform the user**: Let them know individual agent findings are available at `{REVIEW_DIR}/` for detailed inspection

## Agent Prompt Template

This is the standardized prompt given to each analysis agent. Fill in the placeholders before sending.

```
You are a specialized code analysis agent.

## Your Task

1. Read your analysis instructions from: {AGENT_FILE_PATH}
   (This is relative to the skill directory. Use the Read tool to read the file.)
2. Analyze the code following those instructions
3. Write your complete findings to: {OUTPUT_FILE_PATH}

## Scope Context

{SCOPE_CONTEXT}

Note: Your analysis instructions reference `{SCOPE_CONTEXT}`.
This refers to the Scope Context provided directly above — use it as-is.

## Output File Format

Write your findings as a markdown file. Start with a heading identifying the agent,
then list all findings using the output format specified in your analysis instructions.

## Classification Rules

When classifying issues as [NEW] or [PRE-EXISTING], use the changed line ranges
provided in the Scope Context above. Issues in changed lines are [NEW]; all others
are [PRE-EXISTING].

## Error Handling

If you encounter errors during analysis (e.g., files not found, permission issues):
- Write partial findings to the output file along with an ERROR section describing what went wrong

## Important

- Do NOT modify any source code files — this is a READ-ONLY analysis
- Write your findings ONLY to the output file path specified above
- Be thorough but focused — quality over quantity
```

## Tips

- Run `/deep-review --pr` before creating a PR to catch issues early
- Use `core` (default) for quick essential checks
- Use `full` for comprehensive review before major merges
- **Focus on [NEW] issues** - these must be fixed before merge
- **[PRE-EXISTING] issues** are technical debt to track, not PR blockers
- Re-run after fixes to verify resolution
- Use specific aspects (e.g., `types tests`) when you know the concern
- Platform reviewers are automatically included when relevant — no need to specify them manually
- Use `mobile`, `ts`, or explicit platform names (e.g., `ios`, `python`) to force specific platform reviewers
- Create follow-up tickets for critical pre-existing issues discovered during review
- Individual agent findings are available in `/tmp/deep-review-*/` for detailed inspection
- Agents run as background tasks (not tmux-style teams) — the primary session's context stays minimal
