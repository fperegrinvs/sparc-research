# /sparc-refine - Refinement Phase (Specification-Driven Implementation)

## Trigger
Run after architecture to implement code driven by specifications.

## Inputs
- `docs/specification.md`
- `docs/architecture.md`
- Port interface definitions
- BDD feature scenarios
- Property invariants

## Philosophy

> "Specifications drive implementation, not the other way around."

Tests should validate WHAT the system does (behavior), not HOW it does it (implementation).

## MANDATORY: Quality Gate Enforcement

**Every code change MUST pass gates before proceeding. No exceptions.**

See `.claude/skills/quality-gates.md` for full details.

### Gate Checkpoints

```
┌────────────────────────────────────────────────────────────────┐
│  After EVERY Write/Edit:  bun run gate:fast                   │
│  ──────────────────────────────────────────────────────────── │
│  After each feature unit: bun run gate:unit                   │
│  ──────────────────────────────────────────────────────────── │
│  Before any commit:       bun run gate:commit                 │
└────────────────────────────────────────────────────────────────┘
```

### Agent Behavior (REQUIRED)

```typescript
// After EVERY code modification
await Write(file, content)
await Bash('bun run gate:fast')  // MUST pass before continuing

// After completing a use case or component
await Bash('bun run gate:unit')  // MUST pass before next feature

// Before committing
await Bash('bun run gate:commit')  // MUST pass to commit
```

## Implementation Process

### Step 1: Define Property Invariants
Start by identifying domain rules that must ALWAYS hold.

```typescript
// tests/properties/order-invariants.test.ts
import { test, fc } from '@fast-check/vitest'

test.prop([fc.array(orderItemArbitrary)])
  ('order total is never negative', (items) => {
    const order = createOrder(items)
    expect(order.total).toBeGreaterThanOrEqual(0)
  })

test.prop([fc.array(orderItemArbitrary)])
  ('order total equals sum of line items', (items) => {
    const order = createOrder(items)
    const expected = items.reduce((sum, i) => sum + i.price * i.quantity, 0)
    expect(order.total).toBe(expected)
  })
```

### Step 2: Write BDD Scenarios (Before Code)
```gherkin
# tests/features/user-registration.feature
Feature: User Registration
  As a visitor
  I want to create an account
  So that I can access the application

  Scenario: Successful registration
    Given no account exists for "alice@example.com"
    When Alice registers with valid credentials
    Then Alice should have an active account
    And Alice should receive a welcome email
```

### Step 3: Create Fakes for Dependencies
```typescript
// tests/support/fakes/in-memory-user-repository.ts
export function createInMemoryUserRepository(): UserRepository {
  const users = new Map<string, User>()

  return {
    async findByEmail(email) {
      return [...users.values()].find(u => u.email === email) ?? null
    },
    async create(data) {
      const user = { ...data, id: crypto.randomUUID() }
      users.set(user.id, user)
      return user
    },
    _reset() { users.clear() }
  }
}
```

### Step 4: Write Black-Box Unit Tests
Test domain logic through ports using fakes (NOT mocks).

```typescript
// tests/unit/use-cases/register-user.test.ts
describe('RegisterUser', () => {
  let deps: TestDependencies

  beforeEach(() => {
    deps = createTestDependencies() // Uses fakes
  })

  it('creates user with valid credentials', async () => {
    const result = await registerUser({
      email: 'new@example.com',
      password: 'ValidPass123!'
    }, deps)

    expect(result.isOk()).toBe(true)

    // Verify observable outcome (not method calls)
    const found = await deps.userRepository.findByEmail('new@example.com')
    expect(found).not.toBeNull()
  })

  it('rejects duplicate email', async () => {
    // Setup: create existing user
    await deps.userRepository.create({ email: 'taken@example.com', passwordHash: 'x' })

    // Test the behavior
    const result = await registerUser({
      email: 'taken@example.com',
      password: 'ValidPass123!'
    }, deps)

    expect(result.isErr()).toBe(true)
    expect(result.error.type).toBe('CONFLICT')
  })
})
```

### Step 5: Implement Domain Logic
Write minimal code to satisfy specifications.

```typescript
// src/domain/use-cases/register-user.ts
export async function registerUser(
  input: RegisterInput,
  deps: Dependencies
): Promise<Result<User, DomainError>> {
  // Check for existing user
  const existing = await deps.userRepository.findByEmail(input.email)
  if (existing) {
    return err({ type: 'CONFLICT', message: 'Email already registered' })
  }

  // Create user
  const user = await deps.userRepository.create({
    email: input.email.toLowerCase(),
    passwordHash: await deps.hashPassword(input.password),
  })

  // Send welcome email
  await deps.emailService.sendWelcome(user.email)

  return ok(user)
}
```

### Step 6: Implement Adapters
```typescript
// src/adapters/db/postgres-user-repository.ts
export class PostgresUserRepository implements UserRepository {
  constructor(private db: Database) {}

  async findByEmail(email: string): Promise<User | null> {
    return this.db.query('SELECT * FROM users WHERE email = $1', [email])
  }
  // ...
}
```

### Step 7: Contract Tests (Validate Fakes)
```typescript
// tests/contracts/user-repository.contract.ts
function userRepositoryContract(createRepo: () => UserRepository) {
  it('returns null for non-existent user', async () => {
    const repo = createRepo()
    expect(await repo.findByEmail('fake@example.com')).toBeNull()
  })

  it('creates and retrieves user', async () => {
    const repo = createRepo()
    const created = await repo.create({ email: 'test@example.com', passwordHash: 'x' })
    const found = await repo.findByEmail('test@example.com')
    expect(found?.id).toBe(created.id)
  })
}

// Apply to both fake and real
describe('InMemoryUserRepository', () => {
  userRepositoryContract(() => createInMemoryUserRepository())
})

describe('PostgresUserRepository', () => {
  userRepositoryContract(() => new PostgresUserRepository(testDb))
})
```

## Order of Implementation (With Gates)

```
1. Property Tests       → Define domain invariants         → gate:fast
2. BDD Scenarios        → Define acceptance criteria       → gate:fast
3. Fakes                → Create test dependencies         → gate:fast + gate:unit
4. Domain Entities      → Value objects, entities          → gate:fast
5. Domain Use Cases     → Business logic (through ports)   → gate:fast + gate:unit
6. Port Interfaces      → Define contracts                 → gate:fast
7. Adapters             → HTTP, DB, external services      → gate:fast + gate:unit
8. Contract Tests       → Validate fakes match reality     → gate:fast + gate:unit
9. Integration Tests    → Test real adapters               → gate:commit
```

**Gate enforcement is not optional.** If a gate fails, fix it before proceeding.

## Quality Gates (AUTOMATED - NOT A CHECKLIST)

Gates are **automatically enforced**, not manually checked.

### After Every Code Change
```bash
bun run gate:fast  # TypeScript + ESLint (< 10 seconds)
```

### After Each Feature Unit
```bash
bun run gate:unit  # gate:fast + unit tests + property tests
```

### Before Every Commit
```bash
bun run gate:commit  # gate:unit + contracts + BDD + arch tests
```

### What Gates Check
- TypeScript compilation (strict mode)
- ESLint rules (no warnings allowed)
- Architecture constraints (no domain→adapter imports)
- Property tests (domain invariants)
- Unit tests (behavior through ports)
- Contract tests (fakes match real)
- BDD scenarios (acceptance criteria)
- File size limits (< 500 lines)
- Function size limits (< 50 lines)

## Anti-Patterns to Avoid

```typescript
// BAD: Mock-based interaction testing
it('calls repository and hasher', async () => {
  const mockRepo = { create: vi.fn() }
  await registerUser(input, { repo: mockRepo })
  expect(mockRepo.create).toHaveBeenCalledTimes(1) // Tests HOW, not WHAT
})

// GOOD: Behavior verification with fakes
it('creates retrievable user', async () => {
  await registerUser(input, deps)
  const found = await deps.userRepository.findByEmail(input.email)
  expect(found).not.toBeNull() // Tests WHAT happened
})
```

## Commands
```bash
bun test                    # Run all tests
bun test:properties         # Property tests
bun test:unit               # Unit tests
bun test:contracts          # Contract tests
bun test:features           # BDD scenarios
bun test:watch              # Watch mode
bun lint                    # Check linting
bun typecheck               # Type check
```

## Commit Pattern
```
feat: Add user registration use case
test: Add user registration property tests
test: Add user registration BDD scenarios
fix: Handle empty email validation
refactor: Extract email validation to value object
```

## Next Phase
After implementation complete, run `/sparc-complete` for final validation.
