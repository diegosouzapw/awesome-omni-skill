---
name: acceptance-criteria-authoring
description: Write clear, testable acceptance criteria in Given-When-Then format following INVEST principles and BDD best practices.
allowed-tools: Read, Write, Glob, Grep, Task
---

# Acceptance Criteria Authoring

## When to Use This Skill

Use this skill when:

- **Acceptance Criteria Authoring tasks** - Working on write clear, testable acceptance criteria in given-when-then format following invest principles and bdd best practices
- **Planning or design** - Need guidance on Acceptance Criteria Authoring approaches
- **Best practices** - Want to follow established patterns and standards

## Overview

Acceptance criteria define the conditions that must be met for a user story to be considered complete. Well-written acceptance criteria enable clear communication between stakeholders and drive automated acceptance tests.

## Given-When-Then Format

```gherkin
Given [precondition/context]
When [action/event]
Then [expected outcome]
```

### Components

| Component | Purpose | Example |
|-----------|---------|---------|
| **Given** | Set up initial context | "Given a logged-in premium user" |
| **When** | Trigger action/event | "When they click 'Download Report'" |
| **Then** | Assert expected outcome | "Then a PDF report downloads" |
| **And** | Chain multiple conditions | "And the report includes all orders" |
| **But** | Negative assertion | "But archived orders are excluded" |

## INVEST Principles for User Stories

| Letter | Principle | Description |
|--------|-----------|-------------|
| **I** | Independent | Can be developed in any order |
| **N** | Negotiable | Details can be discussed |
| **V** | Valuable | Delivers user/business value |
| **E** | Estimable | Can be sized by team |
| **S** | Small | Fits in one sprint |
| **T** | Testable | Has clear acceptance criteria |

## Acceptance Criteria Best Practices

### Do ✅

- Use business language, not technical jargon
- Be specific and measurable
- Include happy path and edge cases
- Keep scenarios focused and atomic
- Write from user perspective
- Include error scenarios

### Don't ❌

- Include implementation details
- Make criteria too broad
- Use ambiguous terms ("fast", "user-friendly")
- Combine multiple behaviors in one scenario
- Skip error handling scenarios

## Example: E-commerce Checkout

### User Story

```text
As a registered customer
I want to checkout with a saved payment method
So that I can complete purchases quickly
```

### Acceptance Criteria

```gherkin
Feature: Checkout with Saved Payment

  Background:
    Given I am logged in as a registered customer
    And I have a saved Visa card ending in 4242

  Scenario: Successful checkout with saved card
    Given I have items in my cart totaling $50.00
    When I proceed to checkout
    And I select my saved Visa card
    And I click "Place Order"
    Then I see an order confirmation page
    And I receive a confirmation email
    And my card is charged $50.00

  Scenario: Checkout with expired saved card
    Given my saved card has expired
    And I have items in my cart
    When I proceed to checkout
    And I select my expired card
    Then I see a message "This card has expired"
    And I am prompted to update the card or add a new one

  Scenario: Checkout when card is declined
    Given I have items in my cart
    When I proceed to checkout
    And I select my saved card
    And I click "Place Order"
    And the payment is declined
    Then I see a message "Payment declined. Please try another payment method."
    And the order is not created
    And my cart is preserved

  Scenario: Checkout with insufficient inventory
    Given I have 3 units of "Widget X" in my cart
    And only 2 units are in stock
    When I proceed to checkout
    Then I see a message "Widget X: Only 2 available"
    And I am prompted to update quantity
```

## Scenario Patterns

### Happy Path

```gherkin
Scenario: User successfully [action]
  Given [valid preconditions]
  When [correct action]
  Then [expected positive outcome]
```

### Error Handling

```gherkin
Scenario: [Action] fails due to [reason]
  Given [preconditions that lead to failure]
  When [action that will fail]
  Then [appropriate error message]
  And [system state is preserved/recovered]
```

### Edge Cases

```gherkin
Scenario: [Action] with boundary condition
  Given [boundary condition setup]
  When [action at boundary]
  Then [expected behavior at boundary]
```

### Security

```gherkin
Scenario: Unauthorized user attempts [action]
  Given I am not logged in
  When I try to access [protected resource]
  Then I am redirected to login page
  And I see "Please log in to continue"
```

## Scenario Outlines

Use for testing multiple data variations:

```gherkin
Scenario Outline: Discount applied based on order value
  Given I have items in my cart totaling <order_total>
  When I proceed to checkout
  Then I see a discount of <discount>
  And my final total is <final_total>

  Examples:
    | order_total | discount | final_total |
    | $50.00      | $0.00    | $50.00      |
    | $100.00     | $5.00    | $95.00      |
    | $200.00     | $20.00   | $180.00     |
```

## .NET SpecFlow Example

```csharp
[Binding]
public class CheckoutSteps
{
    private readonly CheckoutContext _context;

    public CheckoutSteps(CheckoutContext context)
    {
        _context = context;
    }

    [Given(@"I am logged in as a registered customer")]
    public void GivenIAmLoggedInAsARegisteredCustomer()
    {
        _context.Customer = TestCustomers.CreateRegistered();
        _context.Session = _context.AuthService.Login(_context.Customer);
    }

    [Given(@"I have items in my cart totaling \$(.*)")]
    public void GivenIHaveItemsInMyCartTotaling(decimal total)
    {
        _context.Cart = TestCart.WithTotal(total);
    }

    [When(@"I click ""(.*)""")]
    public void WhenIClick(string button)
    {
        _context.Result = _context.CheckoutPage.Click(button);
    }

    [Then(@"I see an order confirmation page")]
    public void ThenISeeAnOrderConfirmationPage()
    {
        Assert.IsType<OrderConfirmationPage>(_context.Result);
    }

    [Then(@"I receive a confirmation email")]
    public void ThenIReceiveAConfirmationEmail()
    {
        var emails = _context.EmailService.GetEmailsFor(_context.Customer.Email);
        Assert.Contains(emails, e => e.Subject.Contains("Order Confirmation"));
    }
}
```

## Coverage Checklist

For each user story, ensure coverage of:

- [ ] **Happy path**: Main success scenario
- [ ] **Validation errors**: Invalid input handling
- [ ] **Business rule violations**: Domain constraint failures
- [ ] **Authorization failures**: Access control
- [ ] **External service failures**: Third-party integration errors
- [ ] **Boundary conditions**: Min/max values, empty states
- [ ] **Concurrency**: Multiple users, race conditions
- [ ] **State transitions**: Valid and invalid state changes

## Acceptance Criteria Template

```markdown
## User Story
As a [role]
I want to [action]
So that [benefit]

## Acceptance Criteria

### Scenario 1: [Happy path description]
Given [precondition]
When [action]
Then [expected outcome]

### Scenario 2: [Error case description]
Given [error-inducing condition]
When [action]
Then [error handling behavior]

### Scenario 3: [Edge case description]
Given [edge condition]
When [action]
Then [boundary behavior]

## Out of Scope
- [Explicitly excluded scenarios]

## Notes
- [Implementation hints or business context]
```

## Integration Points

**Inputs from**:

- Requirements → Story context
- `jtbd-analysis` skill → Job steps
- `test-case-design` skill → Test techniques

**Outputs to**:

- SpecFlow/Cucumber automation
- `test-strategy-planning` skill → Acceptance test scope
- Definition of Done
