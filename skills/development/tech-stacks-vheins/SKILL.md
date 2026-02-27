---
name: tech-stacks
description: A skill for managing and applying technology-specific architectural patterns and best practices using predefined templates.
---

# Tech Stacks Skill

This skill provides a collection of architectural blueprints and best practices for various technology stacks. It is designed to ensure that new features follow the established patterns of the project's ecosystem.

## Resources

The templates are located in `.agent/skills/tech-stacks/resources/`. Available tech stacks include:

- **PHP**: `laravel.md`, `laravel-filament.md`
- **Frontend**: `vuejs.md`, `react.md`, `nextjs.md`, `nuxtjs.md`, `flutter.md`
- **Backend API**: `express.md`, `fastapi.md`, `golang.md`, `rust.md`, `spring-boot.md`
- **Enterprise**: `dotnet.md`, `odoo.md`

## Usage Instructions

1. **Identify the Stack**: Determine which technology stack is relevant to the task (e.g., if working on the admin dashboard, refer to `laravel-filament.md`).
2. **Review Best Practices**: Read the corresponding template in `resources/` to understand the data structures, directory patterns, and naming conventions.
3. **Apply Patterns**: Implement code, migrations, or services following the specific rules outlined in the template.
4. **Validation**: Ensure that any new code adheres to the architectural rules of the selected stack.

## Best Practices
- **Consistency**: Always prefer project-specific patterns over generic coding styles.
- **Reference**: Link to the relevant tech stack documentation when explaining architectural decisions.
- **Maintenance**: If the project's architecture evolves, update the templates in `resources/` to reflect the new standards.
