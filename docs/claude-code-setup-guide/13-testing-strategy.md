# Part 13: Testing Strategy for AI-Generated Code

## Overview

Testing AI-generated code requires a multi-layered approach that verifies behavior without coupling to implementation details. The key insight: **test WHAT the system does, not HOW it does it**.

## Beyond the Pyramid: Modern Testing Shapes

The traditional testing pyramid emphasizes unit test volume, but modern strategies recognize that **integration tests provide optimal confidence-per-effort** for AI-generated code.

**Testing Trophy** (Kent C. Dodds): Static analysis foundation, integration tests primary, unit tests for complex logic only.

**Testing Honeycomb** (Spotify): The microservice IS the unit. Integration tests validate through edges (API, database). Some services have zero implementation-detail tests.

**Testing Diamond**: Integration tests widest, unit tests only for parsing, calculations, complex transformations.

### Recommended Shape for AI-Generated Code

```
          ╱╲            E2E: Critical paths only (minimize)
         ╱──╲
        ╱    ╲          BDD/Gherkin: Executable specifications
       ╱──────╲
      ╱        ╲        Integration: Test through ports
     ╱ ════════ ╲       ← WIDEST LAYER (fakes AND real)
    ╱            ╲      Property Tests: Domain invariants
   ╱──────────────╲
  ╱                ╲    Unit: Complex algorithms only
 ╱                  ╲
```

**Key insight**: Minimize tests that lock in implementation details, maximize tests that validate observable behavior.

## Layer 1: Property-Based Tests

Define invariants that must hold for ALL inputs.

### Purpose
- Catch edge cases humans miss
- Define domain rules mathematically
- Generate thousands of test cases automatically

### Example
```typescript
import { test, fc } from '@fast-check/vitest'

// Domain invariant
test.prop([fc.array(orderItemArbitrary)])
  ('order total is never negative', (items) => {
    const order = createOrder(items)
    expect(order.total).toBeGreaterThanOrEqual(0)
  })

// Round-trip property
test.prop([userArbitrary])
  ('user serialization round-trip', (user) => {
    const serialized = serializeUser(user)
    const deserialized = deserializeUser(serialized)
    expect(deserialized).toEqual(user)
  })

// Commutativity
test.prop([fc.array(fc.integer()), fc.array(fc.integer())])
  ('set union is commutative', (a, b) => {
    const setA = new Set(a)
    const setB = new Set(b)
    expect(union(setA, setB)).toEqual(union(setB, setA))
  })
```

### Custom Arbitraries
```typescript
const userArbitrary = fc.record({
  id: fc.uuid(),
  email: fc.emailAddress(),
  firstName: fc.string({ minLength: 1, maxLength: 50 }),
  age: fc.integer({ min: 0, max: 150 }),
  role: fc.constantFrom('admin', 'user', 'guest')
})

const orderItemArbitrary = fc.record({
  productId: fc.uuid(),
  price: fc.integer({ min: 1, max: 1000000 }), // cents
  quantity: fc.integer({ min: 1, max: 100 })
})
```

## Layer 2: Black-Box Unit Tests

Test domain logic through ports using fakes.

### Key Principle
```typescript
// GOOD: Tests behavior through ports with fakes
describe('RegisterUser', () => {
  let deps: TestDependencies

  beforeEach(() => {
    deps = createTestDependencies() // Uses fakes, NOT mocks
  })

  it('creates user with valid credentials', async () => {
    const result = await registerUser({
      email: 'new@example.com',
      password: 'ValidPass123!'
    }, deps)

    expect(result.isOk()).toBe(true)

    // Verify observable outcome: user exists
    const found = await deps.userRepository.findByEmail('new@example.com')
    expect(found).not.toBeNull()
  })
})
```

### Anti-Pattern
```typescript
// BAD: Tests implementation, not behavior
it('calls repository and hasher', async () => {
  const mockRepo = { create: vi.fn() }
  const mockHasher = vi.fn().mockReturnValue('hashed')

  await registerUser(input, { repo: mockRepo, hasher: mockHasher })

  // These assertions couple to HOW, not WHAT
  expect(mockHasher).toHaveBeenCalledWith('password')
  expect(mockRepo.create).toHaveBeenCalledTimes(1)
})
```

## Layer 3: Contract Tests

Ensure fakes behave like real implementations.

### Contract Definition
```typescript
function userRepositoryContract(
  createRepo: () => UserRepository,
  cleanup: () => Promise<void>
) {
  describe('UserRepository Contract', () => {
    let repo: UserRepository

    beforeEach(() => { repo = createRepo() })
    afterEach(cleanup)

    it('returns null for non-existent user', async () => {
      const result = await repo.findById('non-existent')
      expect(result).toBeNull()
    })

    it('creates and retrieves user', async () => {
      const created = await repo.create({
        email: 'test@example.com',
        passwordHash: 'hash'
      })
      const found = await repo.findById(created.id)
      expect(found?.email).toBe('test@example.com')
    })
  })
}
```

### Apply to Both Fake and Real
```typescript
// Run against fake
describe('InMemoryUserRepository', () => {
  userRepositoryContract(
    () => createInMemoryUserRepository(),
    async () => {}
  )
})

// Run against real (integration test)
describe('PostgresUserRepository', () => {
  userRepositoryContract(
    () => new PostgresUserRepository(testDb),
    async () => { await testDb.exec('DELETE FROM users') }
  )
})
```

## Layer 4: BDD Feature Tests

Executable specifications in Gherkin.

### Gherkin Example
```gherkin
Feature: User Registration
  As a new visitor
  I want to create an account
  So that I can access the application

  Scenario: Successful registration with valid credentials
    Given no account exists for "alice@example.com"
    When Alice registers with email "alice@example.com" and password "SecurePass123!"
    Then Alice should have an active account
    And Alice should receive a welcome email

  Scenario: Registration rejected for existing email
    Given "bob@example.com" is already registered
    When someone tries to register with "bob@example.com"
    Then registration should fail with "Email already registered"
```

### Step Definitions
```typescript
Given('no account exists for {string}', async function(email: string) {
  // Verify user doesn't exist (using fake)
  const existing = await this.deps.userRepository.findByEmail(email)
  expect(existing).toBeNull()
})

When('Alice registers with email {string} and password {string}', async function(
  email: string,
  password: string
) {
  this.result = await registerUser({ email, password }, this.deps)
})

Then('Alice should have an active account', async function() {
  expect(this.result.isOk()).toBe(true)
  const user = await this.deps.userRepository.findByEmail('alice@example.com')
  expect(user?.status).toBe('active')
})
```

## Layer 5: E2E Tests

Critical user journeys only.

### Purpose
- Verify complete system integration
- Test "money paths" (critical business flows)
- Keep minimal to avoid flakiness

### Example
```typescript
describe('E2E: Checkout Flow', () => {
  it('completes purchase with valid payment', async () => {
    // Setup
    const user = await createTestUser()
    await addItemToCart(user, testProduct)

    // Execute full flow
    const result = await checkout(user, testPaymentMethod)

    // Verify final state
    expect(result.order.status).toBe('confirmed')
    expect(result.payment.status).toBe('captured')
  })
})
```

## Test Doubles Strategy

| Double | Use When | Example |
|--------|----------|---------|
| **Fake** | Testing domain logic | In-memory repository |
| **Stub** | Isolating from slow/flaky services | Hardcoded API response |
| **Mock** | Verifying adapter boundaries ONLY | HTTP client called external API |

### Creating Good Fakes
```typescript
export function createInMemoryUserRepository(): UserRepository {
  const users = new Map<string, User>()

  return {
    async findById(id) {
      return users.get(id) ?? null
    },
    async findByEmail(email) {
      return [...users.values()].find(u => u.email === email) ?? null
    },
    async create(data) {
      const user = {
        ...data,
        id: crypto.randomUUID(),
        createdAt: new Date(),
        updatedAt: new Date(),
      }
      users.set(user.id, user)
      return user
    },
    // Test helpers
    _clear() { users.clear() },
    _seed(seedUsers: User[]) {
      seedUsers.forEach(u => users.set(u.id, u))
    }
  }
}
```

## Test Organization

```
tests/
├── properties/              # Property-based invariant tests
│   ├── order-invariants.test.ts
│   └── user-invariants.test.ts
├── unit/                    # Black-box domain tests (fakes)
│   └── use-cases/
│       └── register-user.test.ts
├── contracts/               # Fake vs Real validation
│   └── user-repository.contract.ts
├── features/                # BDD Gherkin scenarios
│   ├── auth.feature
│   └── steps/
│       └── auth.steps.ts
├── integration/             # Real adapter tests
│   └── db/
└── e2e/                     # Critical user journeys
    └── checkout.test.ts
```

## Coverage Targets

| Layer | Target | Metric |
|-------|--------|--------|
| Property tests | 100% of domain invariants | All critical rules tested |
| BDD scenarios | 100% of acceptance criteria | All features specified |
| Domain unit tests | 80%+ line coverage | Black-box via ports |
| Contract tests | All fakes validated | Fake == Real behavior |
| E2E tests | Critical paths only | ~10 scenarios max |

## Checklist: Is This Test Good?

- [ ] Does it test observable behavior (outputs/state)?
- [ ] Would it survive a refactoring that preserves behavior?
- [ ] Does it use fakes (not mocks) for domain dependencies?
- [ ] Is the assertion about WHAT happened, not HOW?
- [ ] Could a stakeholder understand what's being verified?
- [ ] Does it document a specification?
