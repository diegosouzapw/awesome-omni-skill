---
name: remediation-options
description: Standardized remediation workflow presenting three options for handling config violations - Conform to template, Update template, or Ignore. Adapts recommendations based on repository type (library vs consumer). Use when audit violations are found and user needs to decide how to address them.
---

# Remediation Options Skill

## Purpose

Provides standardized remediation workflow for config violations, presenting three consistent options with intelligent recommendations based on repository type.

## The 3-Option Pattern

When config violations are detected, ALWAYS present exactly 3 options:

1. **Conform to template** - Make repository match agent standard
2. **Ignore** - Skip this violation (acceptable for library repos)
3. **Update template** - Evolve the standard to match repository

## Option Details

### Option 1: Conform to Template

**What it does:**

- Overwrites existing config with standard template
- Re-audits automatically after changes
- Most common choice for consumer repos

**When recommended:**

- Consumer repositories (strict standards)
- First-time setup
- Standardizing across multiple packages

**Implementation:**

```typescript
async function conformToTemplate(
  configPath: string,
  configType: string,
): Promise<void> {
  // 1. Load template
  const template = await loadTemplate(configType);

  // 2. Backup existing (optional)
  await backupConfig(configPath);

  // 3. Write template
  await writeFile(configPath, template);

  // 4. Create additional required files
  await createRequiredFiles(configType, configPath);

  // 5. Re-audit to verify
  const auditResult = await runAudit(configType, configPath);

  if (auditResult.passing) {
    console.log("✅ Config now conforms to standard");
  } else {
    console.log("⚠️  Some issues remain after conforming");
  }
}
```

### Option 2: Ignore

**What it does:**

- Skips this violation
- No changes made
- Violation remains in future audits

**When recommended:**

- Library repositories (intentional differences)
- Temporary exceptions
- Edge cases being evaluated

**Implementation:**

```typescript
async function ignoreViolation(
  violation: string,
  reason?: string,
): Promise<void> {
  console.log(`
ℹ️  Violation ignored: ${violation}
${reason ? `   Reason: ${reason}` : ""}

   This violation will appear in future audits unless:
   - The config is fixed
   - An exception is declared in package.json
`);

  // Optionally: Store in memory for reporting
  await storeIgnoredViolation(violation, reason);
}
```

### Option 3: Update Template

**What it does:**

- Updates agent's template to match current repo config
- Evolves the standard for all consumer repos
- Re-audit all consumer repos recommended after

**When recommended:**

- Evolving standards across organization
- New best practice discovered
- Template is outdated

**CRITICAL:** Only for consumer repos - library repos are NOT the source of truth for consumer standards

**Implementation:**

```typescript
async function updateTemplate(
  configType: string,
  newConfig: any,
  reason: string,
): Promise<void> {
  // 1. Confirm with user (this affects all repos)
  const confirmed = await confirmTemplateUpdate({
    configType,
    reason,
    impact: "All consumer repos will use this new standard",
  });

  if (!confirmed) {
    console.log("❌ Template update cancelled");
    return;
  }

  // 2. Update template
  const templatePath = `.claude/templates/common/${configType}`;
  await writeFile(templatePath, newConfig);

  // 3. Document change
  await documentTemplateChange(configType, reason);

  // 4. Recommend re-audit
  console.log(`
✅ Template updated: ${templatePath}

⚠️  RECOMMENDED: Re-audit all consumer repos to verify consistency
   Run: /ms "audit all ${configType} configs across consumer repos"
`);
}
```

## Presentation Format

### Standard Output Template

```typescript
function presentRemediationOptions(
  repoType: string,
  configType: string,
  violations: string[],
): string {
  const recommendation = getRecommendation(repoType);

  return `
──────────────────────────────────────────────
Remediation Options:
──────────────────────────────────────────────

  1. Conform to template (fix ${configType} to match standard)
     ${getOption1Details(configType)}

  2. Ignore (skip for now)

  3. Update template (evolve the standard)

💡 Recommendation: ${recommendation.option} (${recommendation.label})
   ${recommendation.reason}

Your choice (1-3):
`;
}
```

### Option 1 Details by Config Type

```typescript
function getOption1Details(configType: string): string {
  const details = {
    eslint: "→ Overwrites .eslintrc.js\n     → Re-audits automatically",
    prettier: "→ Overwrites .prettierrc\n     → Re-audits automatically",
    typescript:
      "→ Overwrites tsconfig.json\n     → Verifies extends paths\n     → Re-audits automatically",
    vitest:
      "→ Overwrites vitest.config.ts\n     → Creates ./vitest.setup.ts if frontend package\n     → Re-audits automatically",
    tailwind:
      "→ Overwrites tailwind.config.js\n     → Creates src/index.css if missing\n     → Re-audits automatically",
  };

  return (
    details[configType] || "→ Overwrites config\n     → Re-audits automatically"
  );
}
```

## Smart Recommendations

### Based on Repository Type

```typescript
interface Recommendation {
  option: number; // 1, 2, or 3
  label: string; // "Conform to template" | "Ignore" | "Update template"
  reason: string;
  emoji: string;
}

function getRecommendation(repoType: string): Recommendation {
  if (repoType === "library") {
    return {
      option: 2,
      label: "Ignore",
      reason:
        "Library repo (@metasaver/multi-mono) is intentionally different.",
      emoji: "⭐",
    };
  }

  // Consumer repo
  return {
    option: 1,
    label: "Conform to template",
    reason: `Consumer repos should follow standard ${configType} configuration.`,
    emoji: "💡",
  };
}
```

### Based on Violation Severity

```typescript
function getRecommendationBySeverity(
  severity: "critical" | "warning" | "info",
  repoType: string,
): Recommendation {
  if (severity === "critical") {
    // Critical violations: Always conform (even library repos should consider)
    return {
      option: 1,
      label: "Conform to template",
      reason: "Critical violation affects functionality or security.",
      emoji: "🚨",
    };
  }

  if (severity === "warning" && repoType === "consumer") {
    return {
      option: 1,
      label: "Conform to template",
      reason: "Consumer repos should address warnings for consistency.",
      emoji: "💡",
    };
  }

  // Info-level or library repo: Ignore is acceptable
  return {
    option: 2,
    label: "Ignore",
    reason:
      repoType === "library"
        ? "Library repo differences are expected."
        : "Low-priority issue, can be addressed later.",
    emoji: "ℹ️",
  };
}
```

## User Interaction Pattern

### Using AskUserQuestion Tool

```typescript
async function promptUserForRemediation(
  repoType: string,
  configType: string,
  violations: string[],
): Promise<number> {
  const recommendation = getRecommendation(repoType);

  const answer = await AskUserQuestion({
    questions: [
      {
        question: `${violations.length} violation(s) found in ${configType}. How would you like to proceed?`,
        header: "Remediation",
        multiSelect: false,
        options: [
          {
            label: "Conform to template",
            description: `Fix ${configType} to match MetaSaver standard`,
          },
          {
            label: "Ignore for now",
            description: "Skip this violation (will appear in future audits)",
          },
          {
            label: "Update template",
            description: "Evolve the standard to match current config",
          },
        ],
      },
    ],
  });

  const choices = {
    "Conform to template": 1,
    "Ignore for now": 2,
    "Update template": 3,
  };
  return choices[answer.Remediation];
}
```

### Handling User Choice

```typescript
async function executeRemediationChoice(
  choice: number,
  context: RemediationContext,
): Promise<void> {
  const { repoType, configType, configPath, violations } = context;

  switch (choice) {
    case 1: // Conform
      console.log(`\n🔧 Conforming ${configType} to template...\n`);
      await conformToTemplate(configPath, configType);
      break;

    case 2: // Ignore
      console.log(`\nℹ️  Ignoring ${violations.length} violation(s)\n`);
      await ignoreViolation(violations.join(", "));
      break;

    case 3: // Update template
      console.log(`\n📝 Updating ${configType} template...\n`);

      if (repoType === "library") {
        console.log(`
⚠️  WARNING: Library repo is NOT the source of truth for consumer standards.
   Are you sure you want to update the consumer template based on library config?
`);

        const confirmed = await confirmDangerousAction();
        if (!confirmed) {
          console.log("❌ Template update cancelled");
          return;
        }
      }

      const reason = await askForUpdateReason();
      await updateTemplate(configType, await readFile(configPath), reason);
      break;
  }
}
```

## Exception Declaration Helper

```typescript
async function offerExceptionDeclaration(
  configType: string,
  reason: string,
): Promise<void> {
  console.log(`
💡 TIP: To skip this violation in future audits, declare an exception:

Add to package.json:
{
  "metasaver": {
    "configExceptions": {
      "${configType}": "custom-config",
      "reason": "${reason}"
    }
  }
}
`);
}
```

## Batch Remediation

For multiple violations across multiple configs:

```typescript
async function batchRemediation(
  violations: ConfigViolation[],
  repoType: string,
): Promise<void> {
  console.log(`
Found violations in ${violations.length} config(s).

Options:
  1. Fix all automatically (conform all to templates)
  2. Review each individually
  3. Ignore all

Your choice (1-3):
`);

  const choice = await getUserChoice();

  if (choice === 1) {
    // Fix all
    for (const violation of violations) {
      await conformToTemplate(violation.path, violation.configType);
    }
  } else if (choice === 2) {
    // Review individually
    for (const violation of violations) {
      await promptUserForRemediation(repoType, violation.configType, [
        violation.message,
      ]);
    }
  } else {
    // Ignore all
    console.log("ℹ️  All violations ignored");
  }
}
```

## Complete Example Usage

```typescript
// After audit workflow completes
const auditResults = await runAudit("eslint", ".");

if (auditResults.failing > 0) {
  // Present remediation options
  const choice = await promptUserForRemediation(
    auditResults.repoType,
    "eslint",
    auditResults.violations,
  );

  // Execute choice
  await executeRemediationChoice(choice, {
    repoType: auditResults.repoType,
    configType: "eslint",
    configPath: ".eslintrc.js",
    violations: auditResults.violations,
  });

  // Offer exception declaration if ignored
  if (choice === 2) {
    await offerExceptionDeclaration(
      "eslint",
      "Custom ESLint rules for this project",
    );
  }
}
```

## Output Examples

### Example 1: Consumer Repo with Violations

```
──────────────────────────────────────────────
Remediation Options:
──────────────────────────────────────────────

  1. Conform to template (fix ESLint config to match standard)
     → Overwrites .eslintrc.js
     → Re-audits automatically

  2. Ignore (skip for now)

  3. Update template (evolve the standard)

💡 Recommendation: Option 1 (Conform to template)
   Consumer repos should follow standard ESLint configuration.

Your choice (1-3):
```

### Example 2: Library Repo with Differences

```
──────────────────────────────────────────────
Remediation Options:
──────────────────────────────────────────────

  1. Conform to template (fix Prettier config to match standard)
     → Overwrites .prettierrc
     → Re-audits automatically

  2. Ignore (skip for now)

  3. Update template (evolve the standard)

⭐ Recommendation: Option 2 (Ignore)
   Library repo (@metasaver/multi-mono) is intentionally different.

Your choice (1-3):
```

## Used By

- All config agents after audit completion
- Audit workflows
- CI/CD remediation pipelines
- Interactive configuration setup
