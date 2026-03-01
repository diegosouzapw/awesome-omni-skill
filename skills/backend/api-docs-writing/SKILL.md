---
name: api-docs-writing
description: Update or create API documentation after making changes to the public interface of an API. Use when modifying existing endpoints, introducing new endpoints, or when API implementation changes are complete and tested.
---

# API Documentation Writer

Specialist guidance for creating clear, accurate, and developer-friendly API documentation.

## Workflow

### 1. Analyze Changes

- Review actual code changes to understand modifications
- Identify affected endpoints, parameters, request/response formats, and authentication requirements
- Note breaking changes affecting backward compatibility
- Consider API version and ensure versioning is properly documented

### 2. Follow Existing Patterns

- Study current documentation structure and style before making updates
- Maintain consistency in formatting, terminology, and organization
- Use the same level of detail and examples as existing documentation
- Preserve project-specific documentation conventions

### 3. Document with Precision

- Describe each endpoint's purpose and functionality
- List all parameters with types, requirements (required/optional), and constraints
- Provide accurate request and response examples with realistic data
- Document all possible response codes and their meanings
- Include authentication and authorization requirements
- Note rate limiting or usage restrictions

### 4. Handle API Versioning

- Indicate which API version the documentation applies to
- Document differences between API versions
- Highlight deprecated features and migration paths
- Ensure version-specific documentation is properly segregated

### 5. Enhance Developer Experience

- Include practical examples for common use cases
- Provide clear error response formats and troubleshooting guidance
- Add notes about best practices and performance considerations
- Include links to related endpoints or resources when relevant

### 6. Quality Assurance

- Verify documented endpoints match actual implementation
- Ensure examples are syntactically correct and would work in practice
- Check that all new parameters and fields are documented
- Confirm removed features are marked as deprecated or removed
- Validate documentation remains internally consistent

## Handling Ambiguity

When encountering ambiguity or needing clarification:
- Ask specific questions about intended behavior
- Request examples of expected usage patterns
- Seek confirmation on backward compatibility implications

Documentation should be comprehensive enough that a developer unfamiliar with recent changes can successfully integrate with the API using only the documentation.
