---
name: stacks
description: Stack-specific skills organized by technology category (backend, frontend, infrastructure, mobile)
category: stacks
allowed-tools: Read, Glob, Grep
user-invocable: true
context: inject
---

# Stack Skills

Stack-specific patterns, best practices, and implementation guides organized by technology category.

## Categories

### Backend (175 skills)
Server-side frameworks and patterns.

| Stack | Skills | Description |
|-------|--------|-------------|
| [nestjs](backend/nestjs/SKILL.md) | 17 | Progressive Node.js framework |
| [spring](backend/spring/SKILL.md) | 20 | Java enterprise framework |
| [fastapi](backend/fastapi/SKILL.md) | 20 | Modern Python web framework |
| [django](backend/django/SKILL.md) | 16 | Python web framework |
| [laravel](backend/laravel/SKILL.md) | 20 | PHP web framework |
| [rails](backend/rails/SKILL.md) | 19 | Ruby web framework |
| [go](backend/go/SKILL.md) | 20 | Go backend patterns |
| [rust](backend/rust/SKILL.md) | 20 | Rust backend patterns |
| [dotnet](backend/dotnet/SKILL.md) | 23 | .NET Core patterns |

### Frontend (138 skills)
Client-side frameworks and patterns.

| Stack | Skills | Description |
|-------|--------|-------------|
| [react](frontend/react/SKILL.md) | 25 | React patterns and hooks |
| [angular](frontend/angular/SKILL.md) | 32 | Angular enterprise patterns |
| [vue](frontend/vue/SKILL.md) | 28 | Vue.js patterns |
| [nextjs](frontend/nextjs/SKILL.md) | 27 | Next.js SSR/SSG patterns |
| [nuxt](frontend/nuxt/SKILL.md) | 26 | Nuxt.js patterns |

### Infrastructure (100 skills)
DevOps and infrastructure patterns.

| Stack | Skills | Description |
|-------|--------|-------------|
| [docker](infrastructure/docker/SKILL.md) | 29 | Container patterns |
| [kubernetes](infrastructure/kubernetes/SKILL.md) | 34 | K8s orchestration |
| [terraform](infrastructure/terraform/SKILL.md) | 37 | IaC patterns |

### Mobile (58 skills)
Mobile development patterns.

| Stack | Skills | Description |
|-------|--------|-------------|
| [flutter](mobile/flutter/SKILL.md) | 29 | Flutter/Dart patterns |
| [react-native](mobile/react-native/SKILL.md) | 29 | React Native patterns |

## Auto-Detection

Stacks are auto-detected based on:
- **Files**: Configuration files (e.g., `nest-cli.json`, `package.json`)
- **Packages**: Dependencies in manifest files
- **Patterns**: Code patterns in source files

See [_index.yaml](_index.yaml) for detection rules.

## Usage

Skills are automatically loaded when a stack is detected in the project.

Manual loading:
```bash
/f5-load --stack nestjs
```

Check detected stacks:
```bash
/f5-status stacks
```
