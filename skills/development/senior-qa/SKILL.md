---

# === CORE IDENTITY ===
name: senior-qa
title: Senior QA Skill Package
description: Comprehensive QA and testing skill for quality assurance, test automation, and testing strategies for ReactJS, NextJS, NodeJS applications. Includes test suite generation, coverage analysis, E2E testing setup, and quality metrics. Use when designing test strategies, writing test cases, implementing test automation, performing manual testing, or analyzing test coverage.
domain: engineering
subdomain: quality-assurance

# === WEBSITE DISPLAY ===
difficulty: advanced
time-saved: "TODO: Quantify time savings"
frequency: "TODO: Estimate usage frequency"
use-cases:
  - Designing comprehensive test strategies and test plans
  - Implementing automated testing frameworks
  - Setting up continuous testing in CI/CD pipelines
  - Conducting performance and load testing

# === RELATIONSHIPS ===
related-agents: [qa-engineer]
related-skills: [engineering-team/code-reviewer]
related-commands: [/generate.tests, /generate.tdd]
orchestrated-by: [qa-engineer]

# === TECHNICAL ===
dependencies:
  scripts:
    - test_suite_generator.py
    - coverage_analyzer.py
    - e2e_test_scaffolder.py
    - tdd_workflow.py
    - fixture_generator.py
    - format_detector.py
    - test_spec_generator.py
    - refactor_analyzer.py
  references:
    - testing_strategies.md
    - test_automation_patterns.md
    - qa_best_practices.md
    - tdd_methodology.md
  assets: []
compatibility:
  python-version: 3.8+
  platforms: [macos, linux, windows]
tech-stack: [Python 3.8+, Markdown]

# === EXAMPLES ===
examples:
  -
    title: Example Usage
    input: "TODO: Add example input for senior-qa"
    output: "TODO: Add expected output"

# === ANALYTICS ===
stats:
  downloads: 0
  stars: 0
  rating: 0.0
  reviews: 0

# === VERSIONING ===
version: v1.0.0
author: Claude Skills Team
contributors: []
created: 2025-10-19
updated: 2025-11-23
license: MIT

# === DISCOVERABILITY ===
tags: [analysis, automation, design, engineering, senior, testing]
featured: false
verified: true
---


# Senior QA

Complete toolkit for senior QA engineers with comprehensive testing strategies, automation frameworks, and quality assurance best practices.

## Overview

The Senior QA skill provides world-class testing frameworks, automation tools, and quality assurance methodologies for modern software development. This skill covers unit testing, integration testing, E2E testing, test coverage analysis, and quality metrics used by leading engineering teams.

Designed for senior QA engineers and test automation specialists, this skill includes proven patterns for React/Next.js/Node.js applications, test suite generation, coverage optimization, and E2E test scaffolding. All content focuses on production-grade testing with industry best practices.

**Core Value:** Build comprehensive test suites that catch 90%+ of bugs before production while maintaining fast CI/CD pipelines and high developer productivity.

## Quick Start

### Main Capabilities

This skill provides eight core capabilities through automated scripts:

```bash
# Test Suite Generator
python scripts/test_suite_generator.py --input <path> [options]

# Coverage Analyzer
python scripts/coverage_analyzer.py --input <path> [options]

# E2E Test Scaffolder
python scripts/e2e_test_scaffolder.py --input <path> [options]

# TDD Workflow (Red-Green-Refactor)
python scripts/tdd_workflow.py --input <path> --phase red|green|refactor [options]

# Fixture Generator (Test Data)
python scripts/fixture_generator.py --input <path> [options]

# Format Detector (Framework Detection)
python scripts/format_detector.py --input <path> [options]

# Test Spec Generator (Given-When-Then)
python scripts/test_spec_generator.py --input <path> --requirement "..." [options]

# Refactor Analyzer (Safety Validation)
python scripts/refactor_analyzer.py --input <path> [options]
```

## Core Capabilities

### 1. Test Suite Generator

Automated tool for test suite generator tasks.

**Features:**
- Automated scaffolding
- Best practices built-in
- Configurable templates
- Quality checks

**Usage:**
```bash
python scripts/test_suite_generator.py <project-path> [options]
```

### 2. Coverage Analyzer

Comprehensive analysis and optimization tool.

**Features:**
- Deep analysis
- Performance metrics
- Recommendations
- Automated fixes

**Usage:**
```bash
python scripts/coverage_analyzer.py <target-path> [--verbose]
```

### 3. E2E Test Scaffolder

Advanced tooling for specialized tasks.

**Features:**
- Expert-level automation
- Custom configurations
- Integration ready
- Production-grade output

**Usage:**
```bash
python scripts/e2e_test_scaffolder.py [arguments] [options]
```

## Key Workflows

### 1. Design Test Strategy for New Feature

**Time:** 2-3 hours

1. **Analyze Feature Requirements** - Understand what needs testing
2. **Identify Test Scenarios** - Map out happy path, edge cases, error conditions
3. **Generate Test Suite** - Use test_suite_generator.py to scaffold tests
4. **Implement Tests** - Write unit, integration, and E2E tests
5. **Verify Coverage** - Run coverage_analyzer.py to ensure adequate coverage

**Expected Output:** Comprehensive test suite with >80% coverage

### 2. Improve Test Coverage

**Time:** 4-6 hours

1. **Run Coverage Analysis** - Identify gaps with coverage_analyzer.py
2. **Prioritize Coverage Gaps** - Focus on critical paths first
3. **Write Missing Tests** - Add tests for uncovered code
4. **Refactor for Testability** - Break down complex functions
5. **Verify Improvement** - Re-run coverage analysis

**Expected Output:** Test coverage increased by 15-30%

### 3. Set Up E2E Testing

**Time:** 1-2 days

1. **Design E2E Test Scenarios** - Identify critical user journeys
2. **Scaffold E2E Framework** - Use e2e_test_scaffolder.py
3. **Implement Test Cases** - Write Cypress or Playwright tests
4. **Integrate with CI/CD** - Add E2E tests to pipeline
5. **Monitor and Maintain** - Keep tests stable and fast

**Expected Output:** Production E2E test suite running in CI/CD

### 4. TDD Feature Development (Red-Green-Refactor)

**Time:** 2-4 hours per feature

1. **Detect Framework** - Run format_detector.py to identify test framework
2. **Generate Test Specs** - Use test_spec_generator.py with feature requirement
3. **RED Phase** - Run tdd_workflow.py --phase red, write failing tests
4. **Generate Fixtures** - Use fixture_generator.py for test data
5. **GREEN Phase** - Run tdd_workflow.py --phase green, implement minimal code
6. **REFACTOR Phase** - Run refactor_analyzer.py, then tdd_workflow.py --phase refactor
7. **Verify** - Run coverage_analyzer.py to confirm coverage

**Expected Output:** Feature with 90%+ test coverage following TDD methodology

### 5. Legacy Code Refactoring with TDD Safety

**Time:** 4-8 hours

1. **Analyze Refactoring Safety** - Run refactor_analyzer.py to check readiness
2. **Write Characterization Tests** - Capture existing behavior with test_suite_generator.py
3. **RED Phase** - Add failing tests for desired changes
4. **GREEN Phase** - Implement minimal changes to pass tests
5. **REFACTOR Phase** - Use refactor_analyzer.py suggestions
6. **Verify Safety** - Re-run refactor_analyzer.py to confirm all checks pass

**Expected Output:** Safely refactored code with test coverage as safety net

### 6. TDD Project Setup

**Time:** 1-2 hours

1. **Detect Existing Framework** - Run format_detector.py
2. **Generate Initial Fixtures** - Use fixture_generator.py
3. **Create Test Spec Template** - Run test_spec_generator.py with sample requirement
4. **Configure TDD Workflow** - Set up tdd_workflow.py for team use
5. **Document Process** - Reference tdd_methodology.md for team onboarding

**Expected Output:** Project configured for TDD with team documentation

## Python Tools

### test_suite_generator.py

Generates comprehensive test suites from source code analysis with unit, integration, and edge case coverage.

**Key Features:**
- Automated test scaffolding for React/Next.js/Node.js
- Unit test generation with Jest/React Testing Library patterns
- Integration test templates with database and API mocking
- Edge case identification and test generation
- Test naming conventions and file structure automation

**Common Usage:**
```bash
# Generate test suite for component
python scripts/test_suite_generator.py src/components/Button.tsx

# Generate tests for entire directory
python scripts/test_suite_generator.py src/components/ --recursive

# Help
python scripts/test_suite_generator.py --help
```

**Use Cases:**
- Scaffolding tests for new components or modules
- Ensuring consistent test structure across codebase
- Accelerating test development for large codebases
- Training junior engineers on testing patterns

### coverage_analyzer.py

Analyzes test coverage with detailed gap identification and actionable recommendations for improvement.

**Key Features:**
- Line, branch, and function coverage metrics
- Coverage gap identification by file and function
- Priority scoring (critical paths vs. utility functions)
- Historical trend analysis
- Integration with Jest, Istanbul, and NYC coverage tools
- Visual coverage reports with heat maps

**Common Usage:**
```bash
# Analyze coverage for project
python scripts/coverage_analyzer.py .

# Detailed report with recommendations
python scripts/coverage_analyzer.py . --verbose

# JSON output for CI/CD integration
python scripts/coverage_analyzer.py . --output json

# Help
python scripts/coverage_analyzer.py --help
```

**Use Cases:**
- Identifying untested code before production
- Setting coverage targets and tracking progress
- Prioritizing test development efforts
- Generating coverage reports for stakeholders

### e2e_test_scaffolder.py

Scaffolds end-to-end test infrastructure with Cypress or Playwright including page objects, test data, and CI/CD integration.

**Key Features:**
- E2E framework setup (Cypress or Playwright)
- Page Object Model pattern generation
- Test data management and fixtures
- CI/CD pipeline integration (GitHub Actions, CircleCI)
- Visual regression testing setup
- Parallelization and sharding configuration

**Common Usage:**
```bash
# Scaffold Cypress E2E tests
python scripts/e2e_test_scaffolder.py --framework cypress

# Scaffold Playwright tests
python scripts/e2e_test_scaffolder.py --framework playwright

# With CI/CD integration
python scripts/e2e_test_scaffolder.py --framework cypress --ci github-actions

# Help
python scripts/e2e_test_scaffolder.py --help
```

**Use Cases:**
- Setting up E2E testing for new applications
- Migrating from Selenium to modern frameworks
- Implementing visual regression testing
- Establishing E2E testing standards across teams

### tdd_workflow.py

Orchestrates TDD Red-Green-Refactor cycles with phase tracking, checklists, and guidance for test-driven development.

**Key Features:**
- Red-Green-Refactor cycle management
- Phase-specific checklists and validation
- Guidance and best practices per phase
- Test file metrics and tracking
- Next steps recommendations

**Common Usage:**
```bash
# Start RED phase - write failing test
python scripts/tdd_workflow.py --input . --phase red

# Move to GREEN phase - make tests pass
python scripts/tdd_workflow.py --input . --phase green

# Move to REFACTOR phase - improve code
python scripts/tdd_workflow.py --input . --phase refactor --test-file tests/test_feature.py

# Help
python scripts/tdd_workflow.py --help
```

**Use Cases:**
- Following strict TDD methodology
- Training teams on Red-Green-Refactor cycles
- Tracking TDD cycle metrics
- Enforcing TDD best practices

### fixture_generator.py

Generates comprehensive test fixtures with boundary values, edge cases, and realistic test data for various data types.

**Key Features:**
- Boundary value generation (min, max, zero, negative)
- Edge case generation (null, empty, special chars, XSS, SQL injection)
- Factory patterns for common entities (user, product, order)
- Multiple data type support (integer, string, array, email, url, date)
- Configurable fixture counts

**Common Usage:**
```bash
# Generate fixtures for all types
python scripts/fixture_generator.py --input .

# Generate email-specific fixtures
python scripts/fixture_generator.py --input . --type email --count 10

# Generate with edge cases
python scripts/fixture_generator.py --input . --output json

# Help
python scripts/fixture_generator.py --help
```

**Use Cases:**
- Creating test data for unit tests
- Generating boundary condition test cases
- Security testing with injection patterns
- Building comprehensive test fixtures

### format_detector.py

Auto-detects test framework and coverage format from project configuration files.

**Key Features:**
- Framework detection (Jest, Vitest, Mocha, Pytest, JUnit, RSpec)
- Coverage format detection (Istanbul, LCOV, Cobertura, JaCoCo, coverage.py)
- Language detection (JavaScript, TypeScript, Python, Java, Ruby, Go)
- Config file discovery
- Framework-specific recommendations

**Common Usage:**
```bash
# Detect framework for project
python scripts/format_detector.py --input /path/to/project

# JSON output for CI/CD
python scripts/format_detector.py --input . --output json

# Verbose detection
python scripts/format_detector.py --input . -v

# Help
python scripts/format_detector.py --help
```

**Use Cases:**
- Automatic test framework setup
- CI/CD pipeline configuration
- Multi-project testing standardization
- Coverage tool selection

### test_spec_generator.py

Generates Given-When-Then test specifications from feature requirements with framework-specific templates.

**Key Features:**
- Requirement parsing and analysis
- Given-When-Then specification generation
- Edge case spec generation
- Framework-specific test templates (Jest, Pytest, Vitest, Mocha)
- Priority and category assignment

**Common Usage:**
```bash
# Generate specs from requirement
python scripts/test_spec_generator.py --input . --requirement "User can login with email"

# Generate Pytest template
python scripts/test_spec_generator.py --input . --requirement "API validates input" --framework pytest

# Output test template only
python scripts/test_spec_generator.py --input . --requirement "Calculate total" --output template

# Help
python scripts/test_spec_generator.py --help
```

**Use Cases:**
- Converting requirements to test specifications
- BDD-style test planning
- Test scaffolding for new features
- Training teams on Given-When-Then format

### refactor_analyzer.py

Validates refactoring safety by analyzing code smells, test coverage, and providing improvement suggestions during TDD refactor phase.

**Key Features:**
- Code smell detection (long methods, deep nesting, complexity)
- Test coverage safety checks
- Refactoring suggestions with steps
- Safety scoring and readiness assessment
- CI/CD pipeline detection

**Common Usage:**
```bash
# Analyze project for refactoring
python scripts/refactor_analyzer.py --input /path/to/src

# JSON output for reporting
python scripts/refactor_analyzer.py --input . --output json

# Skip test safety checks
python scripts/refactor_analyzer.py --input . --no-test-check

# Help
python scripts/refactor_analyzer.py --help
```

**Use Cases:**
- Pre-refactoring safety validation
- Code smell identification
- Technical debt assessment
- TDD refactor phase guidance

## Reference Documentation

### Testing Strategies

Comprehensive guide available in `references/testing_strategies.md`:

- Detailed patterns and practices
- Code examples
- Best practices
- Anti-patterns to avoid
- Real-world scenarios

### Test Automation Patterns

Complete workflow documentation in `references/test_automation_patterns.md`:

- Step-by-step processes
- Optimization strategies
- Tool integrations
- Performance tuning
- Troubleshooting guide

### Qa Best Practices

Technical reference guide in `references/qa_best_practices.md`:

- Technology stack details
- Configuration examples
- Integration patterns
- Security considerations
- Scalability guidelines

### TDD Methodology

Comprehensive TDD guide in `references/tdd_methodology.md`:

- Red-Green-Refactor cycle detailed phases
- TDD best practices and conventions
- Anti-patterns to avoid
- Framework-specific patterns (Jest, Pytest)
- TDD metrics and quality indicators

## Tech Stack

**Languages:** TypeScript, JavaScript, Python, Go, Swift, Kotlin
**Frontend:** React, Next.js, React Native, Flutter
**Backend:** Node.js, Express, GraphQL, REST APIs
**Database:** PostgreSQL, Prisma, NeonDB, Supabase
**DevOps:** Docker, Kubernetes, Terraform, GitHub Actions, CircleCI
**Cloud:** AWS, GCP, Azure

## Development Workflow

### 1. Setup and Configuration

```bash
# Install dependencies
npm install
# or
pip install -r requirements.txt

# Configure environment
cp .env.example .env
```

### 2. Run Quality Checks

```bash
# Use the analyzer script
python scripts/coverage_analyzer.py .

# Review recommendations
# Apply fixes
```

### 3. Implement Best Practices

Follow the patterns and practices documented in:
- `references/testing_strategies.md`
- `references/test_automation_patterns.md`
- `references/qa_best_practices.md`

## Best Practices Summary

### Code Quality
- Follow established patterns
- Write comprehensive tests
- Document decisions
- Review regularly

### Performance
- Measure before optimizing
- Use appropriate caching
- Optimize critical paths
- Monitor in production

### Security
- Validate all inputs
- Use parameterized queries
- Implement proper authentication
- Keep dependencies updated

### Maintainability
- Write clear code
- Use consistent naming
- Add helpful comments
- Keep it simple

## Common Commands

```bash
# Development
npm run dev
npm run build
npm run test
npm run lint

# Analysis
python scripts/coverage_analyzer.py .
python scripts/e2e_test_scaffolder.py --analyze

# Deployment
docker build -t app:latest .
docker-compose up -d
kubectl apply -f k8s/
```

## Troubleshooting

### Common Issues

Check the comprehensive troubleshooting section in `references/qa_best_practices.md`.

### Getting Help

- Review reference documentation
- Check script output messages
- Consult tech stack documentation
- Review error logs

## Resources

- Pattern Reference: `references/testing_strategies.md`
- Workflow Guide: `references/test_automation_patterns.md`
- Technical Guide: `references/qa_best_practices.md`
- Tool Scripts: `scripts/` directory
