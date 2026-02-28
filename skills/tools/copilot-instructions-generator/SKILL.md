---
name: copilot-instructions-generator
description: 'Generate and maintain high-quality GitHub Copilot instruction files (.github/copilot-instructions.md). Use this skill when asked to create copilot instructions, generate copilot-instructions.md, set up copilot config, or update copilot instructions for any project or tech stack.'
---

# Copilot Instructions Generator

Generate professional `.github/copilot-instructions.md` files following GitHub's official best practices.

## When To Use This Skill

Trigger phrases:
- "create copilot instructions"
- "generate copilot-instructions.md"
- "set up copilot config"
- "update copilot instructions"
- "make a copilot instructions file"

## Workflow

### Step 1: Determine Project Type

- **Greenfield**: No existing codebase - gather requirements via interview
- **Existing**: Analyze codebase first, then interview

### Step 2: Analyze Codebase (Existing Projects)

```bash
# Tech stack detection
ls -la package.json pyproject.toml Cargo.toml go.mod pom.xml *.csproj 2>/dev/null

# Directory structure
find . -type d -maxdepth 3 -not -path '*/\.*' | head -30

# Check existing configs
cat .editorconfig .prettierrc* .eslintrc* tsconfig.json 2>/dev/null | head -100
```

### Step 3: Research Best Practices

Search for current practices:
- `{language} coding standards best practices 2024`
- `{framework} project structure conventions`
- `github copilot instructions {tech stack}`

### Step 4: Select Tech-Stack Template

Load the appropriate example from `assets/examples/`:

| Technology | Example File |
|------------|--------------|
| .NET/C# | `dotnet-csharp.md` |
| Python | `python.md` |
| FastAPI | `python-fastapi.md` |
| Python AI/ML | `python-ai.md` |
| Java | `java.md` |
| Spring Boot | `spring-boot.md` |
| Go | `go-service.md` |
| Scala | `scala.md` |
| Node.js | `nodejs.md` |
| TypeScript | `typescript.md` |
| React | `react-typescript.md` |
| Angular | `angular.md` |
| AngularJS | `angularjs.md` |
| Next.js | `nextjs.md` |
| Blazor | `blazor.md` |
| Android Kotlin | `android-kotlin.md` |
| Android Java | `android-java.md` |
| Kotlin Multiplatform | `kotlin-multiplatform.md` |
| PySpark | `pyspark.md` |
| Scala Spark | `scala-spark.md` |
| Terraform | `terraform.md` |
| Terraform AWS | `terraform-aws.md` |
| Terraform Azure | `terraform-azure.md` |
| Bicep | `bicep.md` |
| Tableau | `tableau-cloud.md` or `tableau-desktop.md` |

### Step 5: Generate Using Template

Use `references/template.md` structure:

```markdown
# Project Name

Brief description of what the project does.

## Tech Stack

- **Language**: [language]
- **Framework**: [framework]
- **Database**: [database]
- **Testing**: [test framework]

## Project Structure

- `src/` - Source code
- `tests/` - Test files
[etc.]

## Development Commands

- **Build**: `[command]`
- **Test**: `[command]`
- **Lint**: `[command]`

## Before Committing

- [pre-commit step 1]
- [pre-commit step 2]

## Coding Guidelines

### [Category]
- [guideline]
- [guideline]

## Key Guidelines

1. [most important rule]
2. [second most important]
3. [third most important]
```

### Step 6: Interview User

Ask focused questions:
1. "Are there specific coding patterns your team enforces?"
2. "Do you have preferred naming conventions?"
3. "Any libraries Copilot should always/never use?"
4. "What testing requirements should be followed?"
5. "Any security practices that must be applied?"

### Step 7: Validate Output

Check against `references/validation-checklist.md`:
- [ ] File at `.github/copilot-instructions.md`
- [ ] Length under 500 lines
- [ ] Clear headings and bullets
- [ ] Specific, actionable instructions
- [ ] Copy-pasteable commands
- [ ] No anti-patterns (external refs, style instructions)

## Writing Rules

### DO:
- Short, imperative statements
- Specific, actionable guidance
- Concrete examples
- Copy-pasteable commands
- Under 500 lines

### DON'T:
- ❌ "Follow styleguide.md in repo X"
- ❌ "Answer in a friendly tone"
- ❌ "Keep responses brief"
- ❌ Long paragraphs

### Good Examples:
```
✅ "Use Bazel for Java dependencies, not Maven"
✅ "Run `make fmt` before committing"
✅ "JavaScript uses double quotes and tabs"
```

## Output Location

Always create at: `.github/copilot-instructions.md`

## References

- [Template](./references/template.md) - Full template structure
- [Validation Checklist](./references/validation-checklist.md) - Quality checks
- [Official Resources](./references/official-resources.md) - GitHub docs
- [Examples](./assets/examples/) - Tech-stack specific examples
