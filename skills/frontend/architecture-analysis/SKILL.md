---
name: architecture-analysis
description: Comprehensive frontend architecture analyzer that identifies technology stacks, build tools, and architectural patterns. Use when you need to quickly understand a project's structure, dependencies, and technical configuration. Provides analysis for Vue/React/Angular frameworks, Node.js environments, package managers, TypeScript usage, linters, and architecture patterns with multiple output formats including executive summaries and visualizations.
---

# Frontend Architecture Analyzer

This skill analyzes frontend project architecture and provides comprehensive insights about technology stacks, build tools, and architectural patterns.

## Quick Start

Analyze any frontend project with a single command:

```javascript
const result = await analyzeProject("/path/to/project", {
  format: "markdown",  // json, markdown, summary, scorecard
  includeRecommendations: true
});
```

## Core Capabilities

The analyzer detects:
- **Frameworks**: Vue, React, Angular, Svelte, Solid.js
- **Meta-frameworks**: Nuxt, Next.js, Remix, Gatsby, Astro
- **Build Tools**: Vite, Webpack, Rollup, Parcel, esbuild
- **Package Managers**: pnpm, yarn, npm
- **Architecture Patterns**: Monorepo, microservices, modular, layered
- **Quality Metrics**: TypeScript coverage, linters, code quality tools

## Output Formats

Choose the format that best suits your audience:

### Technical Analysis (JSON)
```json
{
  "success": true,
  "data": {
    "framework": { "name": "vue", "metaFramework": "nuxt" },
    "buildTool": { "name": "vite", "version": "5.0.0" },
    "architecturePatterns": ["modular", "layered"]
  }
}
```

### Markdown Report
Human-readable report with sections for stakeholders

### Executive Summary
High-level overview for decision makers

### Scorecard
Quantitative assessment with scores

## Usage Examples

### Basic Analysis
```bash
node scripts/analyze-project.js /path/to/project
```

### With Options
```bash
node scripts/analyze-project.js /path/to/project '{"format": "markdown", "depth": 2}'
```

### Generate Executive Summary
```javascript
const analyzer = new ProjectAnalyzer("./my-project");
const result = await analyzer.analyze();
const report = new ReportGenerator(result);
const summary = report.generate("summary");
```

## Advanced Features

### Pattern Recognition
- **Monorepo Detection**: Identifies workspace configurations
- **Microservices**: Service-based architecture detection
- **Modular Design**: Feature-based organization analysis
- **Layered Architecture**: Controller-service-repository patterns

### Quality Assessment
- TypeScript adoption and coverage calculation
- Code quality tool detection (ESLint, Prettier, Stylelint)
- Architectural complexity evaluation
- Maintainability scoring

### Recommendations Engine
Provides actionable recommendations based on:
- Missing tooling (testing, linting)
- Architecture improvements
- Best practice adoption
- Technology debt

## Implementation Details

### Detector Modules
- `framework-detector.js` - Framework and meta-framework detection
- `build-tool-detector.js` - Build tool and bundler identification
- `architecture-detector.js` - Pattern recognition and scoring

### Report Generator
Supports multiple output formats:
- Technical JSON for API integration
- Markdown for documentation
- Executive summaries for presentations
- Scorecards for metrics tracking

## Reference Documentation

Detailed implementation guides and patterns:

- **Framework Detection**: See [framework-patterns.md](references/framework-patterns.md)
- **Build Tools**: See [build-tool-patterns.md](references/build-tool-patterns.md)
- **Architecture Patterns**: See [architecture-patterns.md](references/architecture-patterns.md)
- **Output Formats**: See [output-formats.md](references/output-formats.md)
