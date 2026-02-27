---
name: interviewing-stakeholders
description: Conducts comprehensive stakeholder interviews to gather detailed requirements and generate Gherkin (Given/When/Then) acceptance criteria for test-driven development. Use this skill when you need to: (1) Clarify ambiguities or gaps identified in the synthesizing-requirements output, (2) Interview users to understand functional requirements and system behaviors, (3) Explore non-functional requirements like performance, security, scalability, and reliability constraints, (4) Understand user personas, use cases, and user journeys in detail, (5) Identify technical constraints, integration requirements, and technology preferences, (6) Define data requirements including structures, relationships, and validation rules, (7) Generate comprehensive Gherkin scenarios for each feature ready for test implementation, or (8) Start from scratch when no prior synthesis exists and you need to gather requirements through conversation. This skill typically follows synthesizing-requirements in the Pact workflow but can be used independently.
---

# Interviewing Stakeholders

## Overview

This skill conducts structured interviews with stakeholders to gather detailed requirements and generate comprehensive Gherkin acceptance criteria. It's designed to transform product intent into testable specifications through guided conversation.

**Primary Goal**: Through systematic questioning, extract complete requirements and produce Given/When/Then scenarios that developers can immediately convert into tests for test-driven development.

---

## When to Use This Skill

Use `interviewing-stakeholders` when you need to:

- **After synthesis**: Follow up on gaps and ambiguities from `synthesizing-requirements`
- **For clarification**: Get specific details about features, behaviors, or constraints
- **Before implementation**: Ensure you have testable acceptance criteria before coding
- **For new features**: Explore requirements for a new feature through conversation
- **To validate assumptions**: Confirm your understanding with stakeholders
- **Starting from scratch**: No synthesis exists yet, and you need to gather requirements

**This skill works best when**: You have access to a stakeholder (the user) who can answer questions about the system's intended behavior.

---

## Step-by-Step Workflow

### Phase 1: Prepare for the Interview

#### 1.1: Review Existing Context

If following `synthesizing-requirements`:

1. **Read the synthesis document** - Understand what's already known
2. **Identify priority gaps** - Focus on the "Gaps & Ambiguities" section
3. **Note ambiguous requirements** - List unclear or contradictory items
4. **Prioritize questions** - Start with must-haves, then should-haves

If starting from scratch:

1. **Understand the domain** - Ask for basic project context
2. **Identify stakeholders** - Who are the primary users?
3. **Define scope** - What's the high-level purpose of the system?

#### 1.2: Plan Your Interview Strategy

Organize questions into logical sections:

1. **System overview** (if not already clear)
2. **User personas and use cases**
3. **Functional requirements** (core features and behaviors)
4. **Non-functional requirements** (performance, security, etc.)
5. **Data requirements** (what data, how it's structured, validation)
6. **Technical constraints** (technology, integrations, limitations)
7. **Edge cases and error handling**

### Phase 2: Conduct the Interview

Use the `AskUserQuestion` tool to systematically gather information. Ask questions in logical groups, building from general to specific.

#### 2.1: Understand the System Context

**If starting from scratch or synthesis is incomplete:**

Example questions:

- "What is the primary purpose of this system?"
- "Who are the main users or user types?"
- "What problem does this solve for users?"
- "What are the top 3 most important features?"

**Use the AskUserQuestion tool:**

```markdown
Question: "What is the primary user persona for this system?"
Header: "User Type"
Options:
- End consumer (Individual users accessing a product or service)
- Business admin (Internal staff managing operations)
- Developer/Technical (Engineers integrating via API)
- Multiple personas (System serves different user types)
```

#### 2.2: Explore Functional Requirements

For each feature or user story, dig into specifics:

**Questions to ask:**

- What triggers this functionality?
- What's the expected outcome?
- What data is involved?
- What are the validation rules?
- What happens in error cases?
- Are there any preconditions?
- What permissions are required?

**Use the AskUserQuestion tool strategically:**

```markdown
Question: "For the user registration feature, what authentication method should we use?"
Header: "Auth Method"
Options:
- Email/password (Traditional email and password authentication)
- OAuth/Social (Google, GitHub, etc.)
- Magic link (Passwordless email link authentication)
- Multi-factor (Combination of methods with 2FA)
```

**Pro tip**: Ask follow-up questions based on answers. If they choose "Email/password", next ask about password requirements, reset flow, etc.

#### 2.3: Clarify Non-Functional Requirements

These are often overlooked but critical for implementation:

**Performance questions:**

- What's the expected load (concurrent users, requests/second)?
- What are acceptable response times?
- What's the target uptime/availability?

**Security questions:**

- What data is sensitive and needs encryption?
- What compliance requirements exist (GDPR, HIPAA, etc.)?
- What are the authentication/authorization requirements?

**Scalability questions:**

- How many users at launch? In 1 year? In 5 years?
- What's the data growth expectation?
- What are the peak usage patterns?

**Use targeted questions:**

```markdown
Question: "What are the performance requirements for API response times?"
Header: "Performance"
Options:
- Fast (<100ms) (Real-time or interactive features)
- Standard (<500ms) (Typical web application)
- Relaxed (<2s) (Less time-critical operations)
- Varies by endpoint (Different requirements for different features)
```

#### 2.4: Define Data Requirements

For each entity or data model:

**Questions to ask:**

- What fields/attributes are needed?
- What are the data types?
- What validations apply (required, format, range, etc.)?
- What are the relationships to other entities?
- What's the lifecycle (create, update, delete, archive)?
- What business rules govern this data?

**Use structured questions:**

```markdown
Question: "For the User entity, what fields are required vs optional?"
Header: "User Fields"
Options:
- Minimal (email, password, name only)
- Standard (+ profile photo, bio, preferences)
- Extended (+ address, phone, demographics)
- Custom (I'll specify exactly what's needed)
```

#### 2.5: Identify Technical Constraints

Understanding technical boundaries helps create realistic acceptance criteria:

**Questions to explore:**

- What systems must this integrate with?
- What technologies are required or prohibited?
- What are the deployment constraints?
- What are the browser/device support requirements?
- What are the data storage/retention requirements?

#### 2.6: Explore Edge Cases and Error Handling

This is critical for complete acceptance criteria:

**For each feature, ask:**

- What if the user provides invalid input?
- What if an external service is unavailable?
- What if there's a network failure?
- What if the user lacks permissions?
- What if data is missing or corrupted?
- What are the timeout scenarios?

**Example question:**

```markdown
Question: "When a payment fails, what should happen?"
Header: "Payment Fail"
Options:
- Retry automatically (Attempt payment again after delay)
- Show error to user (User manually retries)
- Queue for later (Store and retry in background)
- Notify admin (Escalate to support team)
```

### Phase 3: Generate Gherkin Acceptance Criteria

After gathering information through interviews, create comprehensive Gherkin scenarios for each feature.

#### 3.1: Structure Your Gherkin Document

Create a markdown file with all acceptance criteria:

```markdown
# Acceptance Criteria - [System Name]

**Generated**: [Date]
**Based on**: [Synthesis doc reference if applicable]
**Status**: Ready for test implementation

---

## Feature: [Feature Name]

**Epic**: [Epic name if applicable]
**Priority**: [Must Have / Should Have / Could Have]
**User Story**: As a [persona], I want to [action], so that [benefit]

---

### Scenario: [Scenario Name - Happy Path]

**Given** [precondition 1]
**And** [precondition 2]
**When** [action performed]
**And** [additional action]
**Then** [expected outcome]
**And** [additional outcome]

---

### Scenario: [Scenario Name - Alternative Path]

**Given** [different precondition]
**When** [action performed]
**Then** [different expected outcome]

---

### Scenario: [Scenario Name - Error Case]

**Given** [precondition]
**When** [action that triggers error]
**Then** [error handling behavior]
**And** [user feedback shown]

---

## Feature: [Feature Name 2]

[Same structure...]
```

#### 3.2: Write High-Quality Gherkin Scenarios

Follow these best practices:

**Good Gherkin characteristics:**

1. **Use business language**, not technical implementation details
   - Good: "Given the user is logged in"
   - Bad: "Given the JWT token is valid in the session storage"

2. **One scenario tests one behavior**
   - Don't combine multiple unrelated assertions
   - Keep scenarios focused and atomic

3. **Use concrete examples**
   - Good: "When the user enters '<john@example.com>'"
   - Bad: "When the user enters an email"

4. **State preconditions clearly**
   - Always use Given to establish context
   - Don't assume starting state

5. **Describe actions, not UI elements**
   - Good: "When the user submits the registration form"
   - Bad: "When the user clicks the submit button"

6. **Assert on observable outcomes**
   - Good: "Then the user sees a success message"
   - Bad: "Then the database contains the user record"

**Example - Good Gherkin:**

```gherkin
Scenario: User successfully registers with valid email and password

Given the user is on the registration page
And no account exists for "john@example.com"
When the user enters email "john@example.com"
And the user enters password "SecureP@ss123"
And the user enters password confirmation "SecureP@ss123"
And the user submits the registration form
Then the user sees a "Registration successful" message
And the user is redirected to the dashboard
And the user receives a welcome email at "john@example.com"

Scenario: User registration fails with weak password

Given the user is on the registration page
When the user enters email "john@example.com"
And the user enters password "123"
And the user submits the registration form
Then the user sees an error message "Password must be at least 8 characters"
And the user remains on the registration page
And no account is created

Scenario: User registration fails with duplicate email

Given an account already exists for "john@example.com"
And the user is on the registration page
When the user enters email "john@example.com"
And the user enters password "SecureP@ss123"
And the user submits the registration form
Then the user sees an error message "Email already registered"
And the user is offered a "Reset password" link
```

#### 3.3: Cover All Scenario Types

For each feature, create scenarios for:

1. **Happy path** - Everything works as expected
2. **Alternative paths** - Valid but different ways to accomplish the goal
3. **Edge cases** - Boundary conditions, unusual but valid inputs
4. **Error cases** - Invalid inputs, missing data, system failures
5. **Permission cases** - Unauthorized access attempts
6. **State-dependent cases** - Behavior varies based on system state

#### 3.4: Organize Scenarios Logically

Group related scenarios:

```markdown
## Feature: User Authentication

### Login Scenarios

#### Scenario: Successful login with valid credentials
[Gherkin...]

#### Scenario: Failed login with incorrect password
[Gherkin...]

#### Scenario: Failed login with non-existent email
[Gherkin...]

### Logout Scenarios

#### Scenario: User logs out successfully
[Gherkin...]

### Session Management Scenarios

#### Scenario: Session expires after timeout
[Gherkin...]
```

### Phase 4: Validate and Finalize

#### 4.1: Review Completeness

Check that you have:

- [ ] Scenarios for all user stories
- [ ] Happy paths for all features
- [ ] Error cases for all failure modes
- [ ] Edge cases identified during interview
- [ ] Non-functional requirements captured (where applicable)
- [ ] Data validation scenarios
- [ ] Permission/authorization scenarios

#### 4.2: Verify Testability

Every scenario should be:

- **Implementable** as an automated test
- **Deterministic** (same inputs = same outputs)
- **Independent** (doesn't depend on other scenarios)
- **Clear** (no ambiguity in Given/When/Then)

#### 4.3: Get Stakeholder Confirmation

Before finalizing:

1. **Summarize the scenarios** - Give an overview of what was created
2. **Highlight key scenarios** - Point out critical or complex ones
3. **Ask for validation** - "Do these scenarios capture your requirements?"
4. **Iterate if needed** - Refine based on feedback

### Phase 5: Deliver the Output

#### 5.1: Save the Acceptance Criteria

Save to an appropriate location:

- `docs/acceptance-criteria/[feature-name]-acceptance-criteria.md`
- Or organize by epic/module if applicable

#### 5.2: Create an Implementation Guide

Along with the Gherkin file, provide a summary:

```markdown
# Implementation Guide

## Overview
[Brief description of what was defined]

## Statistics
- **Total features**: X
- **Total scenarios**: Y
- **Must-have scenarios**: Z
- **Should-have scenarios**: N

## Priority Order for Implementation

1. **Feature: [Name]** (Must Have)
   - Scenarios: X
   - Estimated complexity: [Simple/Medium/Complex]
   - Dependencies: [None/List dependencies]

2. **Feature: [Name]** (Must Have)
   [Same structure...]

## Technical Notes

- **Testing framework recommended**: [e.g., Jest + Cucumber]
- **Key integrations needed**: [List external dependencies]
- **Data setup requirements**: [Test data needed]

## Next Steps

1. Set up testing framework
2. Implement scenarios in priority order using TDD
3. Write tests first (from Gherkin scenarios)
4. Implement features to make tests pass
5. Refactor and iterate

## Open Questions

[Any remaining questions that came up during the interview]
```

#### 5.3: Communicate Results

Tell the stakeholder:

- **What was created**: Number of features and scenarios
- **Key findings**: Important requirements or constraints discovered
- **Priorities**: What should be built first
- **Next steps**: Ready to begin TDD implementation
- **Questions**: Any remaining ambiguities

---

## Best Practices

### Asking Effective Questions

1. **Start broad, then narrow**
   - Begin with open-ended questions
   - Follow up with specific details

2. **Ask "why" not just "what"**
   - Understand the reasoning behind requirements
   - This helps identify better solutions

3. **Use examples**
   - Ask "Can you give me an example of when this would happen?"
   - Concrete scenarios reveal hidden requirements

4. **Validate your understanding**
   - Summarize what you heard and ask "Is this correct?"
   - Prevents miscommunication

5. **Don't ask yes/no questions**
   - Bad: "Should users be able to delete their account?"
   - Good: "What should happen when a user wants to remove their account?"

### Using AskUserQuestion Effectively

The `AskUserQuestion` tool is powerful but has limits. Use it strategically:

**Do:**

- Group related questions (up to 4 per call)
- Provide clear, distinct options
- Include helpful descriptions for each option
- Use multiSelect when appropriate (non-exclusive choices)

**Don't:**

- Ask too many questions at once (cognitive overload)
- Make options too similar
- Use technical jargon in questions unless speaking with technical stakeholders
- Forget that users can always select "Other" to provide custom input

**Example - Good usage:**

```markdown
[First question set - System context]
1. User persona
2. Primary use case
3. Deployment environment

[Second question set - Feature details]
1. Authentication method
2. Data storage approach
3. Frontend framework

[Third question set - Non-functional requirements]
1. Performance targets
2. Security requirements
3. Scalability needs
```

### Writing Gherkin for TDD

Remember, these scenarios will become tests:

1. **Be specific enough to test**
   - "Then the user sees a success message" is testable
   - "Then the user is happy" is not

2. **Avoid implementation details**
   - Focus on behavior, not code
   - Tests should survive refactoring

3. **Use data tables for similar scenarios**

   ```gherkin
   Scenario Outline: Password validation

   Given the user is registering
   When the user enters password "<password>"
   Then the user sees "<message>"

   Examples:
   | password | message |
   | 123 | Password too short |
   | password | Password needs number |
   | Pass123! | Password accepted |
   ```

4. **Keep scenarios independent**
   - Each scenario should work alone
   - Don't rely on execution order

### Handling Different Stakeholder Types

**Non-technical stakeholders:**

- Use business language
- Avoid technical terms
- Focus on user value and outcomes
- Use examples from their domain

**Technical stakeholders:**

- Can go deeper on technical constraints
- Can discuss implementation approaches
- Can explore integration details
- Can define technical acceptance criteria

**Product managers:**

- Focus on user stories and value
- Prioritize features together
- Discuss trade-offs
- Explore MVP vs. future enhancements

**End users:**

- Focus on workflows and pain points
- Use real-world scenarios
- Explore edge cases from their experience
- Validate assumptions about usage patterns

---

## Integration with Pact Workflow

This skill is **Step 2** in the Pact workflow:

1. **synthesizing-requirements** ← Run first to gather baseline
2. **interviewing-stakeholders** (this skill) ← You are here
3. **Test-driven implementation** ← Build using generated Gherkin scenarios

**The output flows into implementation:**

```bash
Gherkin Scenario (from this skill)
    ↓
Jest/Cucumber Test (written first)
    ↓
Implementation (to make test pass)
    ↓
Refactor (improve code while tests stay green)
```

---

## Example Interview Session

**Context**: Following up on a synthesis document that identified gaps in the user authentication feature.

**You**: I've reviewed the requirements synthesis document. I see we need to clarify the authentication approach and password requirements. Let me ask a few questions to fill in these gaps.

[Uses AskUserQuestion tool]

**Question 1**: "What authentication method should we implement for user login?"

- Email/password
- OAuth/Social login
- Magic link
- Multiple options

**User selects**: Email/password, OAuth/Social login

**You**: Got it, we'll support both email/password and OAuth. Let me ask about the specifics for each.

[Uses AskUserQuestion tool]

**Question 2**: "For email/password authentication, what are the password requirements?"

- Basic (8+ characters)
- Standard (8+ chars, uppercase, number)
- Strong (12+ chars, uppercase, lowercase, number, special character)
- Custom requirements

**User selects**: Strong

**Question 3**: "Which OAuth providers should we support?"

- Google
- GitHub
- Microsoft
- Facebook

**User selects** (multiSelect): Google, GitHub

**You**: Perfect. Now let me ask about error handling and security.

[Continues with more questions about password reset, account lockout, session management, etc.]

**After gathering all information:**

**You**: Great! Based on our conversation, I've generated comprehensive Gherkin acceptance criteria. Here's what I've created:

**Authentication Feature**:

- 12 scenarios for email/password login (happy path, wrong password, account lockout, etc.)
- 8 scenarios for OAuth login (Google and GitHub flows)
- 6 scenarios for password reset
- 4 scenarios for session management
- 5 scenarios for security (rate limiting, failed attempts, etc.)

**Total**: 35 testable scenarios ready for implementation.

I've saved this to [docs/acceptance-criteria/authentication-acceptance-criteria.md](docs/acceptance-criteria/authentication-acceptance-criteria.md).

**Next steps**:

1. Review the scenarios to ensure they match your expectations
2. Set up Jest + Cucumber for BDD testing
3. Implement scenarios in priority order using TDD (write tests first, then code)

Would you like to review the scenarios now, or should we move on to the next feature?

---

## Common Pitfalls to Avoid

1. **Asking too many questions at once** - Breaks flow, causes cognitive overload
2. **Not confirming understanding** - Assumptions lead to incorrect criteria
3. **Skipping error cases** - Real systems fail, scenarios must cover failures
4. **Being too technical** - Match language to stakeholder's expertise
5. **Not prioritizing** - Can't build everything at once, focus on must-haves
6. **Forgetting "why"** - Understanding intent helps you write better scenarios
7. **Making scenarios too long** - Keep them focused on one behavior
8. **Ignoring non-functional requirements** - Performance and security matter
9. **Not validating testability** - Every scenario must be implementable as a test
10. **Stopping too early** - Push for edge cases and error handling

---

## Output Checklist

Before completing the interview process:

- [ ] All gaps from synthesis document are addressed (if applicable)
- [ ] Functional requirements are fully explored
- [ ] Non-functional requirements are defined
- [ ] Data requirements are specified with validation rules
- [ ] Technical constraints are documented
- [ ] User personas and use cases are clear
- [ ] Gherkin scenarios are written for all features
- [ ] Happy path scenarios exist for each feature
- [ ] Error and edge case scenarios are included
- [ ] Scenarios follow Gherkin best practices
- [ ] All scenarios are testable and specific
- [ ] Acceptance criteria file is saved appropriately
- [ ] Implementation guide is created
- [ ] Stakeholder has reviewed and approved the scenarios

---

## Tips for Success

- **Take your time** - Rushing leads to missing requirements
- **Build on previous answers** - Each answer should inform the next question
- **Don't assume** - Always verify your understanding
- **Think like a tester** - Ask "How could this break?"
- **Think like a user** - Ask "What would frustrate a user here?"
- **Document everything** - Even small details can be important
- **Prioritize relentlessly** - Not everything is must-have
- **Stay organized** - Group related scenarios together
- **Be thorough** - Missing one edge case can cause bugs later

---

**Remember**: The quality of your acceptance criteria directly impacts the quality of the final system. Invest time in getting this right, and the implementation phase will be smooth and test-driven.
