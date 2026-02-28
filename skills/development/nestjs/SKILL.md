---
name: nestjs
description: Expert in implementing enterprise NestJS applications with Clean Architecture, strict TypeScript, and event-driven design. Supports Yarn build system and production-ready patterns.
---

# Skill: NestJS Backend Development

## Overview

Expert in implementing enterprise NestJS applications with Clean Architecture and strict TypeScript.

## Rule Categories

| Priority | Category | Impact |
| :--- | :--- | :--- |
| 1 | [TypeScript Standards](rules/typescript.md) | HIGH |
| 2 | [Error Handling](rules/errors.md) | MEDIUM |

## Core Expertise

- **Clean Architecture**: Presentation → Application → Domain → Infrastructure.
- **Strict TypeScript**: No `any`, explicit types, and concrete interfaces.
- **Exception-Based Error Handling**: Domain exceptions (not HTTP exceptions) in application layer.

## Instructions for the Agent

- **Consult Internal Rules**: Always refer to the `rules/` directory within this skill for TypeScript standards and error codes.
- **Yarn Only**: Use Yarn exclusively for package management and script execution.
- **Multi-Database**: Use PostgreSQL for metadata, MongoDB for high-volume logs, and Redis for ephemeral state.

