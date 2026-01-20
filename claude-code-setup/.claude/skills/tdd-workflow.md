# Specification-Driven Testing Skill

## Core Principle: Test Behavior, Not Implementation

> "A test that breaks when you refactor (without changing behavior) is a bad test."

Tests should validate **WHAT** the system does (observable outcomes), not **HOW** it does it (internal mechanics).

## The Specification Testing Mindset

### Wrong: Implementation-Coupled Testing
```typescript
// BAD: Tests implementation details
test('registerUser calls repository and hashes password', async () => {
  const mockRepo = { create: vi.fn() }
  const mockHasher = vi.fn().mockReturnValue('hashed')

  await registerUser(input, { repo: mockRepo, hasher: mockHasher })

  // These assertions couple to implementation:
  expect(mockHasher).toHaveBeenCalledWith('password123')
  expect(mockRepo.create).toHaveBeenCalledTimes(1)
  expect(mockRepo.create).toHaveBeenCalledWith({
    email: 'test@example.com',
    passwordHash: 'hashed'
  })
})
// Problem: Refactoring internals breaks this test even if behavior is correct
```

### Right: Specification-Based Testing
```typescript
// GOOD: Tests observable behavior through ports
test('registered user can authenticate with provided credentials', async () => {
  // Arrange: Use a FAKE (working implementation), not a mock
  const userStore = createInMemoryUserStore()
  const deps = createTestDependencies({ userStore })

  // Act: Exercise through the port interface
  const user = await registerUser({
    email: 'test@example.com',
    password: 'SecurePass123'
  }, deps)

  // Assert: Verify observable outcomes
  expect(user.email).toBe('test@example.com')
  expect(user.id).toBeDefined()

  // Verify the BEHAVIOR: user can now authenticate
  const authenticated = await authenticateUser({
    email: 'test@example.com',
    password: 'SecurePass123'
  }, deps)
  expect(authenticated).toBeTruthy()
})
// This test survives refactoring as long as behavior is preserved
```

## Test Doubles: Fakes Over Mocks

### The Test Double Spectrum
```
Mocks ◄────────────────────────────────────────► Fakes
(Verify interactions)                    (Verify state/behavior)

More coupled to implementation ◄──────► Less coupled, more realistic
```

### When to Use What

| Double | Use When | Example |
|--------|----------|---------|
| **Fake** | Testing domain logic | In-memory repository |
| **Stub** | Isolating from slow/flaky services | Hardcoded API response |
| **Mock** | Verifying adapter boundaries ONLY | HTTP client called external API |

### Fake Example: In-Memory Repository
```typescript
// This fake behaves like the real thing, just in memory
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
      const user: User = {
        ...data,
        id: crypto.randomUUID(),
        createdAt: new Date(),
        updatedAt: new Date(),
      }
      users.set(user.id, user)
      return user
    },

    async update(id, data) {
      const existing = users.get(id)
      if (!existing) return null
      const updated = { ...existing, ...data, updatedAt: new Date() }
      users.set(id, updated)
      return updated
    },

    async delete(id) {
      return users.delete(id)
    },

    // Test helper: reset between tests
    _reset() {
      users.clear()
    }
  }
}
```

### Contract Testing: Validate Fakes Match Reality
```typescript
// The SAME test suite runs against both Fake and Real
function userRepositoryContractTests(
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

    it('creates and retrieves user by id', async () => {
      const created = await repo.create({
        email: 'test@example.com',
        passwordHash: 'hash'
      })

      const found = await repo.findById(created.id)
      expect(found?.email).toBe('test@example.com')
    })

    it('finds user by email', async () => {
      await repo.create({ email: 'find@example.com', passwordHash: 'hash' })

      const found = await repo.findByEmail('find@example.com')
      expect(found).not.toBeNull()
    })

    // ... more contract tests
  })
}

// Run against fake
describe('InMemoryUserRepository', () => {
  userRepositoryContractTests(
    () => createInMemoryUserRepository(),
    async () => {}
  )
})

// Run against real (integration test)
describe('PostgresUserRepository', () => {
  userRepositoryContractTests(
    () => new PostgresUserRepository(testDb),
    async () => { await testDb.exec('DELETE FROM users') }
  )
})
```

## Property-Based Testing

### Define Invariants, Not Examples
```typescript
import { test, fc } from '@fast-check/vitest'

// Instead of testing specific cases, define properties that ALWAYS hold
test.prop([fc.integer(), fc.integer()])
  ('addition is commutative', (a, b) => {
    expect(a + b).toBe(b + a)
  })

// Domain example: Order total invariants
test.prop([
  fc.array(fc.record({
    price: fc.integer({ min: 0, max: 100000 }),
    quantity: fc.integer({ min: 1, max: 100 })
  }), { minLength: 1 })
])('order total is never negative', (items) => {
  const order = createOrder(items)
  expect(order.total).toBeGreaterThanOrEqual(0)
})

test.prop([
  fc.array(fc.record({
    price: fc.integer({ min: 0 }),
    quantity: fc.integer({ min: 1 })
  }))
])('order total equals sum of line items', (items) => {
  const order = createOrder(items)
  const expectedTotal = items.reduce(
    (sum, item) => sum + item.price * item.quantity,
    0
  )
  expect(order.total).toBe(expectedTotal)
})
```

## Testing Through Ports (Black-Box Domain Testing)

```typescript
// Domain use case tests go through the PORT interface
describe('RegisterUser Use Case', () => {
  let deps: TestDependencies

  beforeEach(() => {
    deps = createTestDependencies() // Uses fakes
  })

  describe('successful registration', () => {
    it('creates user with valid email and password', async () => {
      const result = await registerUser({
        email: 'new@example.com',
        password: 'ValidPass123!'
      }, deps)

      expect(result.isOk()).toBe(true)
      expect(result.value.email).toBe('new@example.com')
    })

    it('stores user retrievable by email', async () => {
      await registerUser({
        email: 'stored@example.com',
        password: 'ValidPass123!'
      }, deps)

      // Verify through another port operation
      const found = await deps.userRepository.findByEmail('stored@example.com')
      expect(found).not.toBeNull()
    })
  })

  describe('validation failures', () => {
    it('rejects invalid email format', async () => {
      const result = await registerUser({
        email: 'not-an-email',
        password: 'ValidPass123!'
      }, deps)

      expect(result.isErr()).toBe(true)
      expect(result.error.type).toBe('VALIDATION_ERROR')
    })

    it('rejects weak password', async () => {
      const result = await registerUser({
        email: 'valid@example.com',
        password: '123'
      }, deps)

      expect(result.isErr()).toBe(true)
      expect(result.error.type).toBe('VALIDATION_ERROR')
    })
  })

  describe('business rules', () => {
    it('prevents duplicate email registration', async () => {
      // First registration succeeds
      await registerUser({
        email: 'taken@example.com',
        password: 'ValidPass123!'
      }, deps)

      // Second registration fails
      const result = await registerUser({
        email: 'taken@example.com',
        password: 'DifferentPass456!'
      }, deps)

      expect(result.isErr()).toBe(true)
      expect(result.error.type).toBe('CONFLICT_ERROR')
    })
  })
})
```

## When Mocks ARE Appropriate

Use mocks ONLY at adapter boundaries to verify external interactions:

```typescript
// Testing that the HTTP adapter correctly calls the external payment API
describe('StripePaymentAdapter', () => {
  it('sends correct payload to Stripe API', async () => {
    const mockHttpClient = {
      post: vi.fn().mockResolvedValue({ id: 'ch_123', status: 'succeeded' })
    }

    const adapter = new StripePaymentAdapter(mockHttpClient, 'sk_test_xxx')

    await adapter.charge({
      amount: 1000,
      currency: 'usd',
      customerId: 'cus_123'
    })

    // This IS appropriate: verifying adapter translates domain to external API
    expect(mockHttpClient.post).toHaveBeenCalledWith(
      'https://api.stripe.com/v1/charges',
      expect.objectContaining({
        amount: 1000,
        currency: 'usd'
      })
    )
  })
})
```

## Test Organization

```
tests/
├── properties/              # Property-based invariant tests
│   └── order-invariants.test.ts
├── unit/                    # Black-box domain tests (use fakes)
│   └── use-cases/
│       └── register-user.test.ts
├── contracts/               # Fake vs Real adapter validation
│   └── user-repository.contract.ts
├── integration/             # Real adapters with test DB
│   └── repositories/
│       └── postgres-user-repository.test.ts
├── features/                # BDD Gherkin scenarios
│   ├── auth.feature
│   └── steps/
│       └── auth.steps.ts
└── e2e/                     # Critical user journeys only
    └── checkout-flow.test.ts
```

## Checklist: Is This Test Good?

- [ ] Does it test observable behavior (outputs/state), not internal calls?
- [ ] Would it survive a refactoring that preserves behavior?
- [ ] Does it use fakes (not mocks) for domain dependencies?
- [ ] Is the assertion about WHAT happened, not HOW?
- [ ] Could a stakeholder understand what's being verified?
- [ ] Does it document a specification, not an implementation?
