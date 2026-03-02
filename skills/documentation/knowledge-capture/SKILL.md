---
name: knowledge-capture
description: "Capture and organize business rules, technical patterns, and service interfaces discovered during analysis or implementation into structured documentation"
license: MIT
compatibility: opencode
metadata:
  category: documentation
  version: "1.0"
---

# Knowledge Capture

Roleplay as a documentation specialist that captures and organizes knowledge discovered during development work.

KnowledgeCapture {
  Activation {
    - Document business rules discovered during analysis or implementation
    - Capture technical patterns found in or applied to the codebase
    - Record service interfaces for external API contracts and integrations
    - Organize domain knowledge for team reference and onboarding
    - Deduplicate documentation to prevent fragmentation
  }

  Constraints {
    1. Deduplication is critical - Always check first
    2. Categories matter - Business vs Technical vs External
    3. Names are discoverable - Use full, descriptive names
    4. Templates ensure consistency - Follow the structure
    5. Cross-reference liberally - Connect related knowledge
    6. Maintain freshness - Update docs when code changes
  }

  ReferenceMaterials {
    See `reference/` directory for detailed methodology:
    - [knowledge-capture.md](reference/knowledge-capture.md) -- Advanced categorization, naming conventions, decision matrices, quality standards

    See `templates/` directory for consistent formatting:
    - [pattern-template.md](templates/pattern-template.md) -- Technical patterns
    - [interface-template.md](templates/interface-template.md) -- External integrations
    - [domain-template.md](templates/domain-template.md) -- Business rules
  }

  DocumentationStructure {
    All documentation follows this hierarchy:

    ```
    docs/
      domain/          # Business rules, domain logic, workflows, validation rules
      patterns/        # Technical patterns, architectural solutions, code patterns
      interfaces/      # External API contracts, service integrations, webhooks
    ```
  }

  DecisionTree {
    DocsDomain {
      Business rules and domain logic:
      - User permissions and authorization rules
      - Workflow state machines
      - Business validation rules
      - Domain entity behaviors
      - Industry-specific logic

      Examples:
      - `user-permissions.md` -- Who can do what
      - `order-workflow.md` -- Order state transitions
      - `pricing-rules.md` -- How prices are calculated
    }

    DocsPatterns {
      Technical and architectural patterns:
      - Code structure patterns
      - Architectural approaches
      - Design patterns in use
      - Data modeling strategies
      - Error handling patterns

      Examples:
      - `repository-pattern.md` -- Data access abstraction
      - `caching-strategy.md` -- How caching is implemented
      - `error-handling.md` -- Standardized error responses
    }

    DocsInterfaces {
      External service contracts:
      - Third-party API integrations
      - Webhook specifications
      - External service authentication
      - Data exchange formats
      - Partner integrations

      Examples:
      - `stripe-api.md` -- Payment processing integration
      - `sendgrid-webhooks.md` -- Email event handling
      - `oauth-providers.md` -- Authentication integrations
    }
  }

  Workflow {
    Step0_Deduplication {
      REQUIRED - DO THIS FIRST

      Always check for existing documentation before creating new files:

      ```bash
      # Search for existing documentation
      grep -ri "main keyword" docs/domain/ docs/patterns/ docs/interfaces/
      find docs -name "*topic-keyword*"
      ```

      DecisionTree:
      - Found similar documentation --> Use edit to UPDATE existing file instead
      - Found NO similar documentation --> Proceed to Step 1 (Determine Category)

      Critical: Always prefer updating existing files over creating new ones.
      Deduplication prevents documentation fragmentation.
    }

    Step1_DetermineCategory {
      Ask yourself:
      - Is this about business logic? --> `docs/domain/`
      - Is this about how we build? --> `docs/patterns/`
      - Is this about external services? --> `docs/interfaces/`
    }

    Step2_ChooseCreateOrUpdate {
      Create new if:
      - No related documentation exists
      - Topic is distinct enough to warrant separation
      - Would create confusion to merge with existing doc

      Update existing if:
      - Related documentation already exists
      - New info enhances existing document
      - Same category and closely related topic
    }

    Step3_UseDescriptiveSearchableNames {
      Good names:
      - `authentication-flow.md` (clear, searchable)
      - `database-migration-strategy.md` (specific)
      - `stripe-payment-integration.md` (exact)

      Bad names:
      - `auth.md` (too vague)
      - `db.md` (unclear)
      - `api.md` (which API?)
    }

    Step4_FollowTemplateStructure {
      Use the templates in `templates/` for consistent formatting:
      - `pattern-template.md` -- For technical patterns
      - `interface-template.md` -- For external integrations
      - `domain-template.md` -- For business rules
    }
  }

  DocumentStructureStandards {
    Every document should include:

    1. **Title and Purpose** -- What this documents
    2. **Context** -- When/why this applies
    3. **Details** -- The actual content (patterns, rules, contracts)
    4. **Examples** -- Code snippets or scenarios
    5. **References** -- Related docs or external links
  }

  DeduplicationProtocol {
    Before creating any documentation:

    1. **Search by topic**: `grep -ri "topic" docs/`
    2. **Check category**: List files in target category
    3. **Read related files**: Verify no overlap
    4. **Decide**: Create new vs enhance existing
    5. **Cross-reference**: Link between related docs
  }

  GrayAreasAndEdgeCases {
    WhenBusinessAndTechnicalOverlap {
      Authentication Example:
      - `docs/domain/user-roles.md` -- WHO can access WHAT (business rule)
      - `docs/patterns/authentication-flow.md` -- HOW authentication works (technical)
      - `docs/interfaces/oauth-providers.md` -- EXTERNAL services used (integration)

      Guideline: If it affects WHAT users can do, it belongs in domain.
      If it affects HOW we build it, it belongs in patterns.
    }

    WhenMultipleCategoriesApply {
      Create separate documents for each perspective.
      Cross-reference heavily.
    }
  }

  CrossReferencing {
    When documentation relates to other docs:

    ```markdown
    ## Related Documentation

    - [Authentication Flow](../patterns/authentication-flow.md) - Technical implementation
    - [OAuth Providers](../interfaces/oauth-providers.md) - External integrations
    - [User Permissions](../domain/user-permissions.md) - Business rules
    ```
  }

  DocumentationMaintenance {
    StalenessDetection {
      Check for stale documentation when modifying code:

      1. Git-based staleness: Compare doc and code modification times
         - If source file changed after related doc, flag for review

      2. Reference validation: Verify documented items still exist
         - Function names, API endpoints, configuration options

      3. Example validation: Confirm code examples still work
    }

    StalenessCategories {
      | Category | Indicator | Action |
      |----------|-----------|--------|
      | Critical | Code changed, doc not updated | Update immediately |
      | Warning | > 90 days since doc update | Review needed |
      | Info | > 180 days since update | Consider refresh |
    }

    SyncDuringImplementation {
      When modifying code, proactively check documentation impact:

      Function signature changes --> Update JSDoc/docstrings and API docs
      New public API --> Create documentation before PR
      Breaking changes --> Update all references, add migration notes
    }
  }

  QualityChecklist {
    Before finalizing any documentation:

    - [ ] Checked for existing related documentation
    - [ ] Chosen correct category (domain/patterns/interfaces)
    - [ ] Used descriptive, searchable filename
    - [ ] Included title, context, details, examples
    - [ ] Added cross-references to related docs
    - [ ] Used appropriate template structure
    - [ ] Verified no duplicate content
  }

  IntegrationWithOtherSkills {
    Works alongside:
    - `skill({ name: "documentation-sync" })` -- Keeping documentation in sync with code changes
    - `skill({ name: "codebase-analysis" })` -- Discovering patterns and knowledge during analysis
    - `skill({ name: "specification-management" })` -- Referencing specifications from documentation
  }

  OutputFormat {
    After documenting, always report:

    ```
    Documentation Created/Updated:
    - docs/[category]/[filename].md
      Purpose: [Brief description]
      Action: [Created new / Updated existing / Merged with existing]
    ```
  }
}
