---
name: ai-development-governance
description: AI-augmented development controls, GitHub Copilot governance, LLM security, AI-generated code review per Hack23 Secure Development Policy
license: MIT
---

# AI Development Governance Skill

## Context

This skill applies when:
- Using GitHub Copilot or other AI coding assistants
- Reviewing AI-generated code
- Implementing AI-augmented development workflows
- Creating custom Copilot agents and skills
- Securing AI/LLM integrations
- Documenting AI usage in development process

This skill enforces **[Hack23 Secure Development Policy Section 🤖](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md#ai-augmented-development-controls)** for responsible AI-assisted development.

## Rules

### 1. AI as Proposal Generator, Not Authority (Policy Section 🤖.1)

1. **Human Review Required**: All AI-generated code MUST be reviewed by human developer
2. **Understanding Required**: Developer MUST understand what AI-generated code does
3. **No Blind Acceptance**: Never merge AI suggestions without verification
4. **Test AI Code**: All AI-generated code requires tests with 80%+ coverage
5. **Security Review**: AI-generated security code requires additional security review

### 2. PR Review Requirements (Policy Section 🤖.2)

6. **AI Disclosure**: PRs with significant AI-generated code MUST be labeled
7. **Code Ownership**: Developer submitting PR owns the code, regardless of AI generation
8. **Review Standards**: Same review standards apply to AI-generated and human-written code
9. **Security Validation**: Security-critical AI code requires security team approval
10. **No Bypass**: AI assistance does NOT reduce review requirements

### 3. Curator-Agent as Tooling Change (Policy Section 🤖.3)

11. **Agent Purpose**: Custom agents provide specialized expertise, not replace human judgment
12. **Skill Integration**: Skills teach patterns, developers validate appropriateness
13. **Agent Transparency**: Document which agents/skills were used
14. **Agent Limitations**: Understand agent limitations and potential biases
15. **Agent Testing**: Test agent-generated code as rigorously as any other code

### 4. Security Requirements (Policy Section 🤖.4)

16. **No Secrets in Prompts**: Never include secrets, API keys, or credentials in AI prompts
17. **Data Classification**: Don't share classified or sensitive data with AI
18. **Code Review**: AI-generated security controls require expert security review
19. **Vulnerability Scanning**: Run CodeQL/SAST on all AI-generated code
20. **ISMS Compliance**: AI-generated code MUST comply with all ISMS policies

## Examples

### ✅ Good Pattern: AI-Generated Code with Proper Review

```typescript
/**
 * Search MEPs by country
 * 
 * AI-GENERATED: GitHub Copilot assisted with initial implementation
 * REVIEWED-BY: developer@hack23.com
 * SECURITY-REVIEWED-BY: security@hack23.com
 * TEST-COVERAGE: 95%
 * 
 * ISMS Policy: Secure Development Policy Section 🤖.1-🤖.4
 * Evidence: https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md#ai-augmented-development-controls
 * 
 * @param country - ISO 3166-1 alpha-2 country code
 * @returns Array of MEPs from specified country
 */
export async function searchMEPsByCountry(country: string): Promise<MEP[]> {
  // AI suggested this validation - REVIEWED AND ENHANCED
  const CountrySchema = z.string()
    .length(2, 'Country code must be 2 characters')
    .regex(/^[A-Z]{2}$/, 'Country code must be uppercase')
    .refine(
      (code) => EU_MEMBER_STATES.has(code),
      { message: 'Not a valid EU member state' }
    );
  
  // Human added: Additional security validation
  const validatedCountry = CountrySchema.parse(country);
  
  // AI suggested this caching logic - REVIEWED AND APPROVED
  const cacheKey = `meps:country:${validatedCountry}`;
  const cached = mepCache.get(cacheKey);
  if (cached) {
    auditLog.record({
      eventType: AuditEventType.CACHE_HIT,
      subject: cacheKey,
      outcome: 'success',
    });
    return cached;
  }
  
  // Human added: Rate limiting (AI didn't suggest this)
  await rateLimiter.waitForSlot();
  
  // AI suggested this API call - REVIEWED
  const response = await fetch(
    `https://data.europarl.europa.eu/api/v2/meps?country=${validatedCountry}`,
    {
      headers: {
        'Accept': 'application/json',
        'User-Agent': 'European-Parliament-MCP-Server/1.0',
      },
    }
  );
  
  // Human added: Comprehensive error handling
  if (!response.ok) {
    auditLog.recordAPIError(response.status, validatedCountry);
    throw new APIError(`EP API error: ${response.status}`);
  }
  
  const meps = await response.json();
  
  // AI suggested caching - REVIEWED AND APPROVED
  mepCache.set(cacheKey, meps);
  
  // Human added: GDPR audit logging
  auditLog.recordPersonalDataAccess(
    'system',
    `meps:country:${validatedCountry}`,
    'Country-based MEP search'
  );
  
  return meps;
}

// AI-GENERATED TESTS - REVIEWED AND ENHANCED
describe('searchMEPsByCountry', () => {
  it('should validate country code format', async () => {
    // AI suggested this test
    await expect(searchMEPsByCountry('USA')).rejects.toThrow('Not a valid EU member state');
  });
  
  it('should reject lowercase country codes', async () => {
    // Human added this test (AI missed it)
    await expect(searchMEPsByCountry('de')).rejects.toThrow('Country code must be uppercase');
  });
  
  it('should use cache on subsequent requests', async () => {
    // AI suggested this test - ENHANCED
    vi.mock('./api', () => ({
      fetchMEPs: vi.fn().mockResolvedValue([{ id: 1, country: 'DE' }])
    }));
    
    await searchMEPsByCountry('DE');
    await searchMEPsByCountry('DE');
    
    const apiMock = await import('./api');
    expect(apiMock.fetchMEPs).toHaveBeenCalledTimes(1);
  });
  
  it('should log GDPR access', async () => {
    // Human added (security-critical test)
    const logSpy = vi.spyOn(auditLog, 'recordPersonalDataAccess');
    
    await searchMEPsByCountry('DE');
    
    expect(logSpy).toHaveBeenCalledWith(
      'system',
      'meps:country:DE',
      'Country-based MEP search'
    );
  });
});
```

**Pull Request Description:**
```markdown
## AI-Generated Code Disclosure

**AI Tool Used**: GitHub Copilot  
**AI Contribution**: ~60% (initial implementation, test structure)  
**Human Review**: All code reviewed, enhanced security, added GDPR logging  
**Security Review**: ✅ Approved by security@hack23.com  
**Test Coverage**: 95% (4 passing tests)

### Human Enhancements
- Added EU member state validation
- Added rate limiting
- Enhanced error handling
- Added GDPR audit logging
- Added security-focused test case

### Review Checklist
- [x] Code understood and validated
- [x] Security controls verified
- [x] Tests pass with 80%+ coverage
- [x] ISMS compliance validated
- [x] No secrets or sensitive data
- [x] Documentation complete
```

**Policy Reference**: [Secure Development Policy Section 🤖](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md#ai-augmented-development-controls)

### ✅ Good Pattern: Custom Copilot Agent with Governance

```markdown
---
name: example-agent
description: Example agent with proper governance
tools: ["view", "edit", "create"]
---

# Example Agent

**AI Governance Notice**: This agent uses AI to generate code suggestions. All suggestions MUST be reviewed by human developers per [Secure Development Policy Section 🤖](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md#ai-augmented-development-controls).

## Core Expertise

You specialize in X, Y, Z.

## Rules

1. **Security First**: All code must follow security-by-design principles
2. **ISMS Compliance**: Reference appropriate ISMS policies
3. **Test Coverage**: Generate tests achieving 80%+ coverage
4. **Human Review**: Remind users to review all generated code
5. **Evidence Links**: Provide evidence links to policy compliance

## Remember

**ALWAYS:**
- ✅ Generate code that follows ISMS policies
- ✅ Include comprehensive tests
- ✅ Document security considerations
- ✅ Reference evidence in Hack23 repos
- ✅ Remind users to review AI-generated code

**NEVER:**
- ❌ Include secrets or credentials
- ❌ Skip input validation
- ❌ Generate code without tests
- ❌ Bypass security controls
- ❌ Claim AI-generated code is perfect

---

**Governance**: This agent's output requires human review per AI Development Governance policy.
```

### ✅ Good Pattern: AI Security Validation Checklist

```markdown
# AI-Generated Code Security Review Checklist

**Project**: European Parliament MCP Server  
**PR**: #123  
**AI Tool**: GitHub Copilot  
**Date**: 2026-02-16

## Pre-Merge Security Validation

### Code Understanding
- [ ] Developer understands all AI-generated code
- [ ] Code purpose and logic documented
- [ ] Edge cases identified and handled

### Security Controls
- [ ] Input validation with Zod schemas
- [ ] No hardcoded secrets or credentials
- [ ] Error handling doesn't expose sensitive data
- [ ] Audit logging for security events
- [ ] Rate limiting implemented where needed

### ISMS Compliance
- [ ] Follows security-by-design principles
- [ ] Aligns with threat model
- [ ] Implements defense-in-depth
- [ ] Complies with GDPR (if handling personal data)
- [ ] References appropriate ISMS policies

### Testing
- [ ] Unit tests achieve 80%+ coverage
- [ ] Security test cases included
- [ ] Edge cases tested
- [ ] Error paths tested
- [ ] Integration tests passing

### Code Quality
- [ ] TypeScript strict mode compliant
- [ ] ESLint passing with no warnings
- [ ] Code follows project conventions
- [ ] Documentation complete (JSDoc)
- [ ] No TODO or FIXME comments

### Vulnerability Scanning
- [ ] CodeQL scan passing
- [ ] npm audit shows zero vulnerabilities
- [ ] Dependency licenses approved
- [ ] SAST tools report no issues

## Security Review Sign-Off

- [ ] Reviewed by: ___________________
- [ ] Date: ___________________
- [ ] Security concerns: None / [Document concerns]
- [ ] Approval: ✅ Approved / ❌ Requires changes

## Post-Merge Monitoring

- [ ] Monitor for runtime errors
- [ ] Track performance metrics
- [ ] Review audit logs for anomalies
- [ ] Schedule follow-up review in 30 days
```

**Policy Reference**: [Secure Development Policy Section 🤖.2](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md#pr-review-requirements)

### ✅ Good Pattern: AI-Generated Code Documentation

```typescript
/**
 * AI-GENERATED CODE DOCUMENTATION
 * 
 * Generation Details:
 * - Tool: GitHub Copilot
 * - Generation Date: 2026-02-16
 * - Prompt: "Create a rate limiter for European Parliament API"
 * - AI Contribution: 70% (algorithm, structure)
 * - Human Contribution: 30% (security enhancements, tests)
 * 
 * Review Details:
 * - Reviewed By: developer@hack23.com
 * - Security Review: security@hack23.com
 * - Review Date: 2026-02-16
 * - Review Notes: Enhanced with audit logging, added comprehensive tests
 * 
 * ISMS Compliance:
 * - Policy: Secure Development Policy Section 🤖
 * - Controls: Rate limiting (DoS prevention), Audit logging
 * - Evidence: https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md
 * 
 * Test Coverage: 95%
 * Security Tests: 3 test cases
 * Performance Tests: 2 test cases
 */
export class EPAPIRateLimiter {
  // Implementation...
}
```

## Anti-Patterns

### ❌ Bad: Blindly Accepting AI Code

```typescript
// GitHub Copilot suggested this - looks good, merging!
export async function handleUserData(data: any) {
  return await processData(data);  // No validation!
}
```

**Why**: Violates AI Development Governance - no review, no validation, no tests

### ❌ Bad: Including Secrets in AI Prompts

```
Prompt to Copilot: "Create API client with API key abc123xyz456"
```

**Why**: Violates Policy Section 🤖.4 - Never share secrets with AI

### ❌ Bad: No AI Disclosure

```markdown
# Pull Request

Changed the search function.
```

**Why**: Violates transparency - significant AI-generated code must be disclosed

## Evidence Portfolio

### Reference Implementations

1. **European Parliament MCP Server**
   - Custom Agents: https://github.com/Hack23/European-Parliament-MCP-Server/tree/main/.github/agents
   - AI Skills: https://github.com/Hack23/European-Parliament-MCP-Server/tree/main/.github/skills
   - Agent Guide: https://github.com/Hack23/European-Parliament-MCP-Server/blob/main/AGENTS_AND_SKILLS_GUIDE.md

2. **Citizen Intelligence Agency (CIA)**
   - AI Governance: https://github.com/Hack23/cia/blob/master/AI_DEVELOPMENT_GOVERNANCE.md

3. **Black Trigram Game**
   - Copilot Agents: https://github.com/Hack23/blacktrigram/tree/main/.github/agents

### Policy Documents

- **Secure Development Policy Section 🤖**: https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md#ai-augmented-development-controls
- **AI Development Guidelines**: Best practices for AI-assisted development

## GitHub Copilot Integration

### Custom Agents Best Practices

1. **Clear Purpose**: Each agent has specific expertise
2. **Security Focus**: Security controls in all agent instructions
3. **Evidence Links**: Reference Hack23 repos for patterns
4. **Human Review Reminder**: Always remind users to review
5. **ISMS Alignment**: All agents reference ISMS policies

### Skills Best Practices

1. **Pattern-Based**: Teach patterns, not specific solutions
2. **Auto-Activation**: Skills activate based on context
3. **Evidence-Backed**: Include working examples from real repos
4. **Compliance-Focused**: Map patterns to ISMS policies
5. **Test-Driven**: Include testing patterns

## AI Development Checklist

- [ ] AI tool disclosed in PR description
- [ ] AI-generated code percentage estimated
- [ ] All AI code reviewed and understood
- [ ] Security controls validated
- [ ] Tests achieve 80%+ coverage
- [ ] ISMS compliance verified
- [ ] No secrets in code or prompts
- [ ] Documentation includes AI disclosure
- [ ] Security review completed (if security-critical)
- [ ] CodeQL/SAST passing
- [ ] Appropriate ISMS policies referenced

## ISMS Compliance

This skill enforces:
- **AI-001**: AI as proposal generator principle
- **AI-002**: PR review requirements for AI code
- **AI-003**: Custom agent governance
- **AI-004**: AI security requirements

**Policy Reference**: [Hack23 Secure Development Policy Section 🤖](https://github.com/Hack23/ISMS-PUBLIC/blob/main/Secure_Development_Policy.md#ai-augmented-development-controls)
