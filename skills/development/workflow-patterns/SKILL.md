---
name: workflow-patterns
description: "Industry-specific workflow patterns and templates for finance, healthcare, logistics, manufacturing, retail, and common use cases like AI document processing, API integration, business rules, ETL, RAG, security, and project management. Use when asking about 'workflow examples', 'workflow templates', 'industry workflows', 'finance workflows', 'healthcare workflows', 'logistics workflows', 'manufacturing workflows', 'retail workflows', 'ETL workflows', 'RAG workflows', 'API workflows', 'document processing', 'business rules', or 'workflow patterns'."
---

# Kailash Workflows - Industry Patterns & Templates

Production-ready workflow patterns and templates for industry-specific use cases and common application patterns.

## Overview

Complete workflow patterns for:
- Industry-specific applications
- Common use case templates
- Production-ready patterns
- Best practice implementations

## Industry-Specific Patterns

### Finance
- **[workflow-industry-finance](workflow-industry-finance.md)** - Financial services workflows
  - Payment processing
  - Fraud detection
  - Risk assessment
  - Compliance reporting
  - Trade settlement
  - Credit scoring

### Healthcare
- **[workflow-industry-healthcare](workflow-industry-healthcare.md)** - Healthcare workflows
  - Patient data processing
  - Medical record management
  - Clinical decision support
  - Insurance claims
  - Lab result processing
  - HIPAA compliance

### Logistics
- **[workflow-industry-logistics](workflow-industry-logistics.md)** - Logistics workflows
  - Order fulfillment
  - Inventory management
  - Route optimization
  - Shipment tracking
  - Warehouse automation
  - Supply chain coordination

### Manufacturing
- **[workflow-industry-manufacturing](workflow-industry-manufacturing.md)** - Manufacturing workflows
  - Production planning
  - Quality control
  - Equipment monitoring
  - Maintenance scheduling
  - Supply chain management
  - Defect tracking

### Retail
- **[workflow-industry-retail](workflow-industry-retail.md)** - Retail workflows
  - Order processing
  - Inventory management
  - Customer service automation
  - Pricing optimization
  - Promotional campaigns
  - Returns processing

## Common Use Case Patterns

### AI & Document Processing
- **[workflow-pattern-ai-document](workflow-pattern-ai-document.md)** - AI document processing
  - Document classification
  - Entity extraction
  - Document summarization
  - OCR and parsing
  - Form processing
  - Multi-document analysis

### API Integration
- **[workflow-pattern-api](workflow-pattern-api.md)** - API integration patterns
  - API orchestration
  - Multi-API coordination
  - API retry logic
  - Rate limiting
  - Response transformation
  - Error handling

### Business Rules
- **[workflow-pattern-business-rules](workflow-pattern-business-rules.md)** - Business rule engines
  - Rule evaluation
  - Decision tables
  - Policy enforcement
  - Conditional processing
  - Dynamic routing
  - Approval workflows

### Cyclic Workflows
- **[workflow-pattern-cyclic](workflow-pattern-cyclic.md)** - Cyclic workflow patterns
  - Iterative processing
  - Feedback loops
  - Retry mechanisms
  - State machines
  - Multi-cycle patterns
  - Convergence detection

### Data Processing
- **[workflow-pattern-data](workflow-pattern-data.md)** - Data processing patterns
  - Data validation
  - Data enrichment
  - Data aggregation
  - Data normalization
  - Data quality checks
  - Master data management

### ETL (Extract, Transform, Load)
- **[workflow-pattern-etl](workflow-pattern-etl.md)** - ETL workflows
  - Data extraction
  - Transformation pipelines
  - Data loading
  - Incremental updates
  - Error handling
  - Performance optimization

### File Processing
- **[workflow-pattern-file](workflow-pattern-file.md)** - File processing patterns
  - Bulk file processing
  - File monitoring
  - File transformation
  - Archive management
  - File validation
  - Multi-format handling

### Project Management
- **[workflow-pattern-project-mgmt](workflow-pattern-project-mgmt.md)** - Project workflows
  - Task automation
  - Status tracking
  - Resource allocation
  - Deadline monitoring
  - Report generation
  - Approval workflows

### RAG (Retrieval-Augmented Generation)
- **[workflow-pattern-rag](workflow-pattern-rag.md)** - RAG workflows
  - Document indexing
  - Vector search
  - Context retrieval
  - Answer generation
  - Source attribution
  - Multi-source RAG

### Security
- **[workflow-pattern-security](workflow-pattern-security.md)** - Security workflows
  - Access control
  - Audit logging
  - Threat detection
  - Incident response
  - Compliance checking
  - Security scanning

## Pattern Usage

### How to Use Patterns

1. **Select Pattern**: Choose relevant industry or use case
2. **Review Template**: Study the pattern structure
3. **Customize**: Adapt to your specific needs
4. **Test**: Follow testing best practices
5. **Deploy**: Use production deployment patterns

### Pattern Structure

Each pattern includes:
- **Overview**: Use case description
- **Architecture**: Workflow design
- **Nodes Used**: Required nodes
- **Configuration**: Parameter setup
- **Example Code**: Working implementation
- **Best Practices**: Production tips
- **Testing**: Test strategies

## When to Use This Skill

Use this skill when you need:
- Industry-specific workflow templates
- Production-ready starting points
- Common use case implementations
- Best practice examples
- Workflow design inspiration
- Pattern-based development

## Implementation Tips

### Starting from Patterns

```python
# 1. Copy pattern template
workflow = WorkflowBuilder()

# 2. Add nodes from pattern
workflow.add_node("NodeType", "id", {...})

# 3. Customize parameters
# 4. Add industry-specific logic
# 5. Test with real data
```

### Combining Patterns

- Mix patterns for complex workflows
- Use common patterns as building blocks
- Adapt industry patterns to your domain
- Layer security patterns on all workflows

## Quick Patterns

### ETL Workflow
```python
workflow.add_node("Extract", "extract", {"source": "..."})
workflow.add_node("Transform", "transform", {"logic": "..."})
workflow.add_node("Load", "load", {"destination": "..."})
workflow.add_connection("extract", "data", "transform", "input")
workflow.add_connection("transform", "output", "load", "data")
```

### RAG Workflow
```python
workflow.add_node("Embed", "embed", {"model": "text-embedding-ada-002"})
workflow.add_node("Search", "search", {"index": "vectors"})
workflow.add_node("Generate", "generate", {"model": "gpt-4"})
```

## CRITICAL Warnings

| Rule | Reason |
|------|--------|
| ❌ NEVER hardcode secrets | Use environment variables |
| ✅ ALWAYS validate inputs | At workflow boundaries |
| ❌ NEVER skip error handling | Required in production |

## Related Skills

- **[01-core-sdk](../../01-core-sdk/SKILL.md)** - Core workflow creation
- **[06-cheatsheets](../cheatsheets/SKILL.md)** - Pattern quick reference
- **[08-nodes-reference](../nodes/SKILL.md)** - Node reference
- **[02-dataflow](../../02-dataflow/SKILL.md)** - Database workflows
- **[03-nexus](../../03-nexus/SKILL.md)** - Workflow deployment
- **[17-gold-standards](../../17-gold-standards/SKILL.md)** - Best practices

## Support

For workflow pattern help, invoke:
- `pattern-expert` - Workflow pattern selection and design
- `framework-advisor` - Architecture decisions
- `testing-specialist` - Pattern testing strategies
