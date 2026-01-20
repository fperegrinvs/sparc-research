# BDD Testing Skill: Executable Specifications

## Core Principle: Gherkin IS the Specification

> "Gherkin scenarios are not tests that verify code — they are specifications that the code must satisfy."

BDD bridges the gap between business requirements and technical implementation. The Gherkin file IS the acceptance criteria, written BEFORE implementation.

## The Specification Workflow

```
┌─────────────────────────────────────────────────────────────────┐
│  1. DISCOVERY: Collaborate with stakeholders                    │
│     - "What should the system do?"                              │
│     - Identify examples and edge cases                          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  2. FORMULATION: Write Gherkin scenarios                        │
│     - Express examples as Given/When/Then                       │
│     - Use domain language, NOT technical terms                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  3. AUTOMATION: Implement step definitions                      │
│     - Connect Gherkin to code through thin adapter layer        │
│     - Step definitions call domain use cases via ports          │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│  4. IMPLEMENTATION: Write code to pass scenarios                │
│     - Scenarios drive development (outside-in)                  │
│     - Done when all scenarios pass                              │
└─────────────────────────────────────────────────────────────────┘
```

## Writing Good Specifications

### Focus on WHAT, Not HOW

```gherkin
# BAD: Technical implementation details
Scenario: User registration
  Given the POST endpoint /api/users exists
  When I send JSON {"email": "test@example.com", "password": "hash123"}
  Then the response status should be 201
  And the database should contain a user record

# GOOD: Business behavior
Scenario: New user can register with valid credentials
  Given no account exists for "alice@example.com"
  When Alice registers with email "alice@example.com" and a valid password
  Then Alice should have an active account
  And Alice should receive a welcome email
```

### Declarative Over Imperative

```gherkin
# BAD: Step-by-step UI instructions
Scenario: Purchase product
  Given I navigate to the home page
  When I click the "Products" link
  And I click on "Widget A"
  And I click "Add to Cart"
  And I click the cart icon
  And I click "Checkout"
  And I fill in "Card Number" with "4111111111111111"
  And I click "Pay Now"
  Then I should see "Order confirmed"

# GOOD: Intent-focused
Scenario: Customer completes purchase
  Given "Widget A" is in stock
  And customer has a valid payment method
  When customer purchases "Widget A"
  Then the order should be confirmed
  And inventory should decrease by 1
```

## Gherkin Syntax Reference

### Feature Structure
```gherkin
@billing @priority-high
Feature: Subscription Billing
  As a subscription customer
  I want my payment to be processed automatically
  So that my service continues without interruption

  Background:
    Given today is the 1st of the month
    And the billing system is operational

  # Happy path - most common scenario
  Scenario: Successful monthly charge
    Given customer "acme-corp" has an active subscription
    And their payment method is valid
    When the billing cycle runs
    Then the customer should be charged $99.00
    And their subscription should remain active
    And they should receive an invoice email

  # Edge case
  Scenario: Payment failure triggers retry
    Given customer "failing-corp" has an active subscription
    But their payment method has expired
    When the billing cycle runs
    Then the charge should fail
    And the system should schedule a retry in 3 days
    And the customer should receive a payment failure notification

  # Parameterized specification
  Scenario Outline: Pricing tiers
    Given customer has a "<plan>" subscription
    When they are billed
    Then they should be charged <amount>

    Examples:
      | plan       | amount  |
      | starter    | $29.00  |
      | pro        | $99.00  |
      | enterprise | $299.00 |
```

### Keywords

| Keyword | Purpose | Notes |
|---------|---------|-------|
| **Feature** | Groups related scenarios | One feature per file |
| **Background** | Common setup for all scenarios | Runs before each scenario |
| **Scenario** | Single specification example | Independent and isolated |
| **Scenario Outline** | Parameterized specification | Use with Examples table |
| **Given** | Precondition (arrange) | Sets up initial state |
| **When** | Action (act) | The thing being specified |
| **Then** | Outcome (assert) | Observable result |
| **And/But** | Continuation | Same type as previous step |
| **@tag** | Categorization | For filtering and organization |

## Step Definitions: The Thin Adapter

Step definitions should be **thin adapters** that translate Gherkin to domain operations. They should NOT contain business logic.

### Structure
```typescript
// tests/features/steps/billing.steps.ts
import { Given, When, Then, Before, After } from '@cucumber/cucumber'
import { expect } from 'chai'
import { createTestDependencies } from '@tests/support/test-dependencies'
import type { World } from '@tests/support/world'

// Setup test dependencies (fakes)
Before(async function(this: World) {
  this.deps = await createTestDependencies()
})

// Cleanup
After(async function(this: World) {
  await this.deps.cleanup()
})

// GIVEN: Setup preconditions
Given('customer {string} has an active subscription', async function(
  this: World,
  customerId: string
) {
  // Thin adapter: call domain operation, store result for later steps
  this.customer = await this.deps.customerService.createWithSubscription({
    id: customerId,
    status: 'active'
  })
})

Given('their payment method is valid', async function(this: World) {
  await this.deps.paymentService.addPaymentMethod(this.customer.id, {
    type: 'card',
    valid: true
  })
})

// WHEN: Execute the action
When('the billing cycle runs', async function(this: World) {
  // Call the use case being specified
  this.billingResult = await this.deps.billingUseCase.runBillingCycle({
    customerId: this.customer.id,
    date: this.currentDate
  })
})

// THEN: Verify outcomes
Then('the customer should be charged {currency}', async function(
  this: World,
  expectedAmount: string
) {
  // Assert on observable outcome
  expect(this.billingResult.chargedAmount).to.equal(parseFloat(expectedAmount))
})

Then('their subscription should remain active', async function(this: World) {
  // Verify state through ports (not implementation)
  const subscription = await this.deps.subscriptionRepo.findByCustomer(
    this.customer.id
  )
  expect(subscription?.status).to.equal('active')
})

Then('they should receive an invoice email', async function(this: World) {
  // Verify behavior through fake (not mock)
  const sentEmails = this.deps.emailService.getSentEmails()
  const invoiceEmail = sentEmails.find(
    e => e.to === this.customer.email && e.type === 'invoice'
  )
  expect(invoiceEmail).to.exist
})
```

### World Object (Test Context)
```typescript
// tests/support/world.ts
import type { TestDependencies } from './test-dependencies'

export interface World {
  deps: TestDependencies
  customer?: Customer
  billingResult?: BillingResult
  currentDate: Date
}

// Custom parameter types
import { defineParameterType } from '@cucumber/cucumber'

defineParameterType({
  name: 'currency',
  regexp: /\$[\d,]+\.?\d*/,
  transformer: (s) => s.replace('$', '').replace(',', '')
})
```

## Tagging Strategy

```gherkin
@smoke           # Critical path tests (run on every commit)
@regression      # Full test suite (run nightly)
@wip             # Work in progress (skip in CI)
@manual          # Requires manual verification
@slow            # Long-running tests (run separately)

# Domain tags
@billing @payments
@auth @security
@orders @inventory

# Priority
@priority-high
@priority-low
```

### Running Tagged Tests
```bash
# Run only smoke tests
bun run test:features --tags "@smoke"

# Run billing tests except WIP
bun run test:features --tags "@billing and not @wip"

# Run high priority OR smoke
bun run test:features --tags "@priority-high or @smoke"
```

## Specification Templates

### User Story Feature
```gherkin
Feature: [User Story Title]
  As a [role]
  I want [goal]
  So that [benefit]

  # Acceptance Criteria as Scenarios
  Scenario: [Criterion 1 - Happy Path]
    Given [precondition]
    When [action]
    Then [expected outcome]

  Scenario: [Criterion 2 - Edge Case]
    ...

  Scenario: [Criterion 3 - Error Case]
    ...
```

### API Contract Feature
```gherkin
Feature: Users API Contract
  As an API consumer
  I want consistent user endpoints
  So that I can integrate reliably

  Scenario: Get user by ID returns user data
    Given user "user-123" exists
    When I request user "user-123"
    Then the response should include:
      | field     | type   |
      | id        | string |
      | email     | string |
      | createdAt | date   |

  Scenario: Get non-existent user returns not found
    Given user "fake-id" does not exist
    When I request user "fake-id"
    Then the response should indicate not found
    And the error should include "User not found"
```

### Business Rule Feature
```gherkin
Feature: Discount Rules
  As a pricing manager
  I want discounts applied correctly
  So that customers are charged fairly

  Rule: Bulk discounts apply to orders over $100

    Scenario: Order qualifies for bulk discount
      Given product "Widget" costs $20
      When customer orders 6 units
      Then subtotal should be $120
      And bulk discount of 10% should apply
      And total should be $108

    Scenario: Order does not qualify for bulk discount
      Given product "Widget" costs $20
      When customer orders 4 units
      Then subtotal should be $80
      And no discount should apply
      And total should be $80

  Rule: Member discounts stack with bulk discounts

    Scenario: Member with bulk order
      Given customer is a premium member (5% discount)
      And product "Widget" costs $20
      When customer orders 6 units
      Then bulk discount of 10% should apply first
      Then member discount of 5% should apply
      And total should be $102.60
```

## Anti-Patterns to Avoid

### 1. Testing UI, Not Behavior
```gherkin
# BAD
Then the button should be green
And the success message should appear in div.alert

# GOOD
Then the operation should succeed
And the user should be notified
```

### 2. Technical Language
```gherkin
# BAD
Given the Redis cache is cleared
When I POST to /api/v1/orders with JSON body

# GOOD
Given no pending orders exist
When customer places an order
```

### 3. Too Many Steps
```gherkin
# BAD: 15 steps describing every micro-action

# GOOD: 3-5 steps focusing on the essence
Scenario: Complete checkout
  Given customer has items in cart
  When customer completes checkout
  Then order should be placed
```

### 4. Dependent Scenarios
```gherkin
# BAD: Scenario 2 depends on state from Scenario 1

# GOOD: Each scenario is completely independent
```

## Best Practices Checklist

- [ ] Scenarios describe behavior, not implementation
- [ ] Language is domain-specific, not technical
- [ ] Each scenario is independent (no shared state)
- [ ] Steps are declarative (what), not imperative (how)
- [ ] Background only contains truly common setup
- [ ] Step definitions are thin adapters to domain
- [ ] Scenarios are written BEFORE implementation
- [ ] Stakeholders can read and understand scenarios
