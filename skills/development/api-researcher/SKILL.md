---
name: api-researcher
description: Expert API research including discovery, evaluation, integration analysis, and documentation review
version: 1.0.0
author: USER
tags: [api, research, integration, evaluation, documentation]
---

# API Researcher

## Purpose
Research, evaluate, and document APIs including feature analysis, integration complexity, and usage recommendations.

## Activation Keywords
- api research, find api
- api evaluation, api comparison
- integration complexity, api docs
- rest api, graphql api
- api features, api pricing

## Core Capabilities

### 1. API Discovery
- Public API catalogs
- Category search
- Feature matching
- Alternative finding
- Emerging APIs

### 2. Documentation Analysis
- Completeness assessment
- Example quality
- SDK availability
- Error documentation
- Rate limit clarity

### 3. Integration Evaluation
- Authentication methods
- Request/response format
- Rate limits
- Error handling
- Versioning strategy

### 4. Quality Assessment
- Reliability (uptime)
- Performance (latency)
- Support quality
- Community size
- Update frequency

### 5. Comparison
- Feature matrices
- Pricing comparison
- Limit comparison
- DX comparison
- Migration complexity

## API Evaluation Template

```markdown
## API Research: [API Name]

### Overview
- **Provider**: [Company]
- **Type**: [REST/GraphQL/gRPC]
- **Version**: [Current version]
- **Status**: [GA/Beta/Deprecated]

### Authentication
- **Methods**: [API Key/OAuth/JWT]
- **Complexity**: [Simple/Moderate/Complex]
- **Setup Time**: [Estimate]

### Endpoints Summary
| Category | Endpoints | Notes |
|----------|-----------|-------|
| [Category 1] | N | [Notes] |
| [Category 2] | N | [Notes] |

### Rate Limits
| Tier | Requests | Period |
|------|----------|--------|
| Free | X | per minute |
| Pro | Y | per minute |

### Pricing
| Tier | Price | Includes |
|------|-------|----------|
| Free | $0 | X calls/month |
| Pro | $X/mo | Y calls/month |
| Enterprise | Custom | Unlimited |

### SDK Availability
- JavaScript/Node: ✅/❌
- Python: ✅/❌
- Go: ✅/❌
- Ruby: ✅/❌

### Documentation Quality
| Aspect | Score | Notes |
|--------|-------|-------|
| Completeness | X/10 | [Assessment] |
| Examples | X/10 | [Assessment] |
| Error docs | X/10 | [Assessment] |
| Search | X/10 | [Assessment] |

### Integration Complexity
- **Overall**: [Easy/Moderate/Complex]
- **Time Estimate**: [X hours/days]
- **Gotchas**: [List any known issues]

### Recommendation
[Use/Consider/Avoid] for [use case]
```

## API Discovery Sources

| Source | URL | Best For |
|--------|-----|----------|
| RapidAPI | rapidapi.com | General discovery |
| Public APIs | public-apis.io | Free APIs |
| ProgrammableWeb | programmableweb.com | Directory |
| API List | apilist.fun | Curated list |
| Postman | postman.com/explore | Collections |

## Example Usage

```
User: "Research payment APIs for a SaaS application"

API Researcher Response:

## API Research: Payment APIs for SaaS

### Candidates Evaluated
1. Stripe
2. Paddle
3. LemonSqueezy

### Quick Comparison
| Feature | Stripe | Paddle | LemonSqueezy |
|---------|--------|--------|------------|
| MoR | No | Yes | Yes |
| Subscriptions | ✅ | ✅ | ✅ |
| Tax handling | Add-on | Included | Included |
| Fees | 2.9%+30¢ | 5%+50¢ | 5%+50¢ |
| Setup complexity | Medium | Low | Low |

### Detailed: Stripe
- **Auth**: API Keys + Webhooks
- **SDK**: Excellent (all languages)
- **Docs**: 10/10
- **Integration time**: 1-2 days
- **Best for**: Full control, international

### Detailed: Paddle
- **Auth**: API Keys + Webhooks
- **SDK**: Good (JS, Python)
- **Docs**: 8/10
- **Integration time**: 4-8 hours
- **Best for**: SaaS, MoR needed

### Detailed: LemonSqueezy
- **Auth**: API Keys + Webhooks
- **SDK**: Basic (JS)
- **Docs**: 7/10
- **Integration time**: 2-4 hours
- **Best for**: Simple SaaS, indie

### Recommendation
- **Enterprise SaaS**: Stripe (flexibility)
- **Solo/Indie**: LemonSqueezy (simplicity)
- **Tax concerns**: Paddle (MoR included)
```
