---
name: workflow-composer
description: Chain multiple skills together into automated workflows with conditional logic and parallel execution
allowed-tools: ["Read", "Write", "Bash", "Task", "TodoWrite"]
version: 1.0.0
author: GLINCKER Team
license: Apache-2.0
keywords: [automation, workflow, orchestration, skill-chaining, multi-agent, pipeline]
---

# Workflow Composer

**⚡ UNIQUE FEATURE**: First skill that lets you chain multiple Claude Code skills together into automated workflows with conditional logic, parallel execution, and error handling. Create complex automation pipelines with simple YAML configs.

## What This Skill Does

Orchestrates multiple skills into powerful automated workflows:

- **Chain skills together**: Output of one skill becomes input to the next
- **Parallel execution**: Run multiple skills simultaneously
- **Conditional logic**: IF/THEN/ELSE based on skill results
- **Error handling**: Retry logic and fallback strategies
- **Progress tracking**: Real-time status of workflow execution
- **Workflow templates**: Save and reuse common workflows
- **Schedule workflows**: Run workflows on triggers or schedules

## Why This Is Revolutionary

This is the **first skill-composition framework** for Claude Code:
- **No coding required**: Define workflows in simple YAML
- **Visual workflow builder**: Generate workflow configs interactively
- **Reusable components**: Build once, use everywhere
- **Enterprise-ready**: Error handling, logging, notifications
- **Community workflows**: Share workflows with others

## Workflow Structure

```yaml
name: my-workflow
description: What this workflow does
version: 1.0.0

# Input parameters
inputs:
  project_path:
    type: string
    required: true
  run_tests:
    type: boolean
    default: true

# Workflow steps
steps:
  # Sequential steps
  - name: generate-readme
    skill: readme-generator
    inputs:
      path: ${inputs.project_path}

  - name: generate-tests
    skill: unit-test-generator
    inputs:
      target: ${inputs.project_path}/src
    when: ${inputs.run_tests}

  # Parallel execution
  - name: quality-checks
    parallel:
      - name: security-scan
        skill: security-auditor
        inputs:
          path: ${inputs.project_path}

      - name: lint-code
        skill: code-linter
        inputs:
          path: ${inputs.project_path}

  # Conditional logic
  - name: fix-issues
    skill: auto-fixer
    when: ${steps.quality-checks.security-scan.issues > 0}
    inputs:
      issues: ${steps.quality-checks.security-scan.results}

  # Error handling
  - name: deploy
    skill: deploy-orchestrator
    inputs:
      environment: production
    retry:
      max_attempts: 3
      backoff: exponential
    on_failure:
      - skill: rollback
      - skill: notify-team
        inputs:
          message: "Deployment failed"

# Output mapping
outputs:
  readme_path: ${steps.generate-readme.output.file_path}
  test_coverage: ${steps.generate-tests.output.coverage}
  security_issues: ${steps.quality-checks.security-scan.issues}
```

## Instructions

### Creating a Workflow

When a user wants to create a workflow:

1. **Interactive Workflow Builder**:
   ```
   Ask user:
   - What's the goal of this workflow?
   - Which skills should be included?
   - Should any skills run in parallel?
   - What are the inputs needed?
   - What conditions/logic are needed?
   ```

2. **Generate Workflow YAML**:
   - Create well-structured YAML file
   - Add comments explaining each step
   - Include error handling
   - Validate syntax

3. **Save Workflow**:
   - Use Write to create `.claude-workflows/workflow-name.yml`
   - Add to workflow registry
   - Create documentation

### Running a Workflow

When a user wants to run a workflow:

1. **Load Workflow**:
   ```bash
   Use Read to load workflow YAML from .claude-workflows/
   ```

2. **Validate Inputs**:
   - Check all required inputs provided
   - Validate input types
   - Set defaults for optional inputs

3. **Create Execution Plan**:
   ```
   - Parse workflow steps
   - Identify parallel vs sequential
   - Build dependency graph
   - Prepare Task agents
   ```

4. **Execute Workflow**:
   ```
   For each step:
     - Check 'when' condition (skip if false)
     - If parallel block:
       - Launch all Task agents simultaneously
       - Wait for all to complete
     - If sequential:
       - Execute skill via Task agent
       - Capture output
       - Pass to next step
     - Handle errors:
       - Retry if configured
       - Run on_failure steps
       - Stop or continue based on config
   ```

5. **Track Progress**:
   - Use TodoWrite to show workflow progress
   - Update status for each step
   - Show current step and completion percentage

6. **Generate Report**:
   ```markdown
   # Workflow Execution Report

   **Workflow**: ${workflow.name}
   **Started**: ${start_time}
   **Duration**: ${duration}
   **Status**: ✅ Success / ⚠️ Partial / ❌ Failed

   ## Steps
   1. ✅ generate-readme (2.3s)
   2. ✅ generate-tests (5.1s)
   3. ⏭️ fix-issues (skipped - no issues found)
   4. ✅ deploy (12.4s)

   ## Outputs
   - readme_path: /project/README.md
   - test_coverage: 87%
   - deployment_url: https://app.example.com
   ```

## Example Workflows

### Example 1: Complete Project Setup

```yaml
name: project-setup
description: Initialize new project with README, tests, CI/CD, and docs

steps:
  - name: scaffold
    skill: project-scaffolder
    inputs:
      type: ${inputs.project_type}
      name: ${inputs.project_name}

  - name: generate-files
    parallel:
      - name: readme
        skill: readme-generator

      - name: gitignore
        skill: gitignore-generator

      - name: license
        skill: license-picker
        inputs:
          license_type: MIT

  - name: setup-tests
    skill: unit-test-generator
    inputs:
      coverage_target: 80

  - name: setup-cicd
    skill: ci-cd-wizard
    inputs:
      platform: github-actions

  - name: init-git
    skill: git-initializer
    inputs:
      remote: ${inputs.git_remote}
```

**Usage:**
```
User: "Set up a new Python project called 'my-api'"

Workflow Composer:
- Prompts for project type, git remote
- Executes all steps
- Shows progress
- Reports results
```

### Example 2: Pre-Commit Workflow

```yaml
name: pre-commit-checks
description: Run all quality checks before committing code

steps:
  - name: quality-checks
    parallel:
      - name: lint
        skill: code-linter

      - name: format
        skill: code-formatter

      - name: security
        skill: security-auditor

      - name: tests
        skill: test-runner

  - name: fix-issues
    when: ${steps.quality-checks.lint.issues > 0}
    skill: auto-fixer
    inputs:
      auto_fix: true

  - name: verify
    when: ${steps.fix-issues.ran}
    skill: test-runner
```

### Example 3: Deploy Pipeline

```yaml
name: deploy-pipeline
description: Complete deployment with tests, build, and monitoring setup

steps:
  - name: pre-deploy-checks
    parallel:
      - skill: test-runner
      - skill: security-auditor
      - skill: dependency-checker

  - name: build
    when: ${steps.pre-deploy-checks.all_passed}
    skill: build-optimizer

  - name: deploy
    skill: deploy-orchestrator
    inputs:
      environment: ${inputs.environment}
      strategy: blue-green
    retry:
      max_attempts: 3

  - name: post-deploy
    parallel:
      - skill: health-checker
      - skill: performance-tester
      - skill: log-analyzer

  - name: notify
    skill: slack-bridge
    inputs:
      channel: deployments
      message: "✅ Deployed ${inputs.environment}"
```

## Built-in Workflow Templates

### 1. New Feature Workflow
- Create feature branch
- Generate boilerplate
- Create tests
- Setup documentation
- Create PR

### 2. Code Review Workflow
- Run linters
- Check security
- Verify tests
- Review architecture
- Post review comments

### 3. Bug Fix Workflow
- Reproduce bug
- Generate failing test
- Apply fix
- Verify fix
- Update changelog

### 4. Release Workflow
- Update version
- Generate changelog
- Run full test suite
- Build artifacts
- Create release notes
- Tag release

## Tool Requirements

- **Read**: Load workflow definitions
- **Write**: Save workflows and reports
- **Bash**: Execute commands, git operations
- **Task**: Launch skills as agents
- **TodoWrite**: Track workflow progress

## Advanced Features

### 1. Workflow Variables

```yaml
variables:
  project_name: my-app
  version: 1.0.0
  deploy_env: production

steps:
  - name: deploy
    inputs:
      version: ${variables.version}
      env: ${variables.deploy_env}
```

### 2. Conditional Execution

```yaml
- name: deploy-prod
  when: |
    ${inputs.environment == 'production'} &&
    ${steps.tests.coverage >= 80} &&
    ${steps.security.issues == 0}
```

### 3. Loop Execution

```yaml
- name: process-files
  for_each: ${inputs.files}
  skill: file-processor
  inputs:
    file: ${item}
```

### 4. Error Recovery

```yaml
- name: critical-step
  skill: deployment
  on_failure:
    - skill: rollback
    - skill: notify-team
  continue_on_error: false
```

## Workflow Library

Share workflows with the community:

```bash
# Publish workflow
claude workflow publish my-workflow.yml

# Install community workflow
claude workflow install feature-complete-setup

# List available workflows
claude workflow browse
```

## Best Practices

1. **Start simple**: Begin with 2-3 steps, then expand
2. **Add error handling**: Always plan for failures
3. **Use parallel execution**: Speed up independent tasks
4. **Document inputs**: Clear descriptions for all parameters
5. **Test incrementally**: Run each step individually first
6. **Version your workflows**: Track changes over time

## Limitations

- Maximum 20 steps per workflow (performance)
- Parallel execution limited to 5 concurrent skills
- Workflow execution timeout: 30 minutes
- Cannot nest workflows (yet - coming in v2.0)

## Related Skills

This skill works with ANY other skill in the marketplace!

Popular combinations:
- [pr-reviewer](../../devops/pr-reviewer/SKILL.md) + [unit-test-generator](../../testing/unit-test-generator/SKILL.md)
- [readme-generator](../../documentation/readme-generator/SKILL.md) + [api-doc-generator](../../documentation/api-doc-generator/SKILL.md)
- [security-auditor](../../security/security-auditor/SKILL.md) + [dependency-checker](../../security/dependency-checker/SKILL.md)

## Changelog

### Version 1.0.0 (2025-01-13)
- Initial release
- Sequential and parallel execution
- Conditional logic support
- Error handling and retry
- Progress tracking
- Workflow templates
- Community sharing

## Contributing

This is a cornerstone skill for the marketplace. Help us improve:
- Add new workflow templates
- Improve error handling
- Add visualization features
- Create workflow examples

## License

Apache License 2.0 - See [LICENSE](../../../LICENSE)

## Author

**GLINCKER Team**
- GitHub: [@GLINCKER](https://github.com/GLINCKER)
- Repository: [claude-code-marketplace](https://github.com/GLINCKER/claude-code-marketplace)

---

**🌟 WORLD'S FIRST skill composition framework for Claude Code!**
