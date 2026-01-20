# Part 7: Skills Reference Documentation

## Overview

Skills are reusable knowledge documents in `.claude/skills/` that provide detailed reference material for patterns, practices, and methodologies. They serve as on-demand documentation that agents can reference during workflows.

## Core Skills

### Hexagonal Architecture

Provides patterns for Ports & Adapters architecture.

**Key Concepts**:
- **Domain**: Pure business logic with NO external dependencies
- **Ports**: Technology-agnostic interfaces (contracts)
- **Adapters**: Concrete implementations connecting to infrastructure
- **Dependency Rule**: All dependencies point INWARD

**Structure**:
```
src/
├── domain/           # Pure business logic
│   ├── entities/     # Domain objects
│   ├── use-cases/    # Business operations
│   └── errors.ts     # Domain errors
├── ports/
│   ├── driver/       # Entry points (use case interfaces)
│   └── driven/       # Dependencies (repository interfaces)
└── adapters/
    ├── http/         # HTTP handlers
    ├── db/           # Database implementations
    └── external/     # Third-party integrations
```

**Port Example**:
```typescript
// ports/driven/repositories/user-repository.ts
export interface UserRepository {
  findById(id: string): Promise<User | null>
  findByEmail(email: string): Promise<User | null>
  create(user: CreateUserInput): Promise<User>
}
```

**Adapter Example**:
```typescript
// adapters/db/postgres/user-repository.ts
export class PostgresUserRepository implements UserRepository {
  constructor(private db: Database) {}

  async findById(id: string): Promise<User | null> {
    return this.db.query('SELECT * FROM users WHERE id = $1', [id])
  }
}
```

### Quality Gates

Mandatory automated checkpoints that MUST pass before proceeding.

**Gate Levels**:

| Gate | When | Checks | Time |
|------|------|--------|------|
| `gate:fast` | After EVERY code change | TypeScript, ESLint | < 10s |
| `gate:unit` | After feature completion | + Unit tests, property tests | < 60s |
| `gate:commit` | Before EVERY commit | + Contracts, BDD, arch tests | < 2min |
| `gate:full` | Before PR/merge | + E2E, coverage, security | < 10min |

**Enforcement Flow**:
```
Write Code ──► gate:fast ──► PASS? ──► Continue
                   │
                   ▼
                FAIL? ──► Fix ──► Retry gate:fast
```

**Package.json Scripts**:
```json
{
  "scripts": {
    "gate:fast": "bun run typecheck --incremental && bun run lint --cache",
    "gate:unit": "bun run gate:fast && bun run test:unit --changed && bun run test:properties",
    "gate:commit": "bun run gate:unit && bun run test:contracts && bun run test:features && bun run arch:test",
    "gate:full": "bun run gate:commit && bun run test:e2e && bun run test:coverage --check && bun run security:scan"
  }
}
```

### Specification-Driven Testing (TDD Workflow)

Test behavior, not implementation. Modern testing shapes (Trophy, Honeycomb, Diamond) all emphasize **integration tests over unit tests**. The pyramid is outdated.

**Core Principles**:
> "A test that breaks when you refactor (without changing behavior) is a bad test."

> "The more your tests resemble the way your software is used, the more confidence they can give you." — Kent C. Dodds

**Test Doubles Strategy**:

| Double | Use When | Example |
|--------|----------|---------|
| **Fake** | Testing domain logic | In-memory repository |
| **Stub** | Isolating from slow services | Hardcoded API response |
| **Mock** | Adapter boundaries ONLY | HTTP client verification |

**Good Test (Behavior)**:
```typescript
it('registered user can authenticate', async () => {
  const deps = createTestDependencies() // Uses fakes

  await registerUser({ email: 'test@example.com', password: 'Pass123!' }, deps)

  // Verify BEHAVIOR: user can authenticate
  const authenticated = await authenticateUser({
    email: 'test@example.com',
    password: 'Pass123!'
  }, deps)
  expect(authenticated).toBeTruthy()
})
```

**Bad Test (Implementation)**:
```typescript
it('calls repository and hasher', async () => {
  const mockRepo = { create: vi.fn() }
  await registerUser(input, { repo: mockRepo })
  expect(mockRepo.create).toHaveBeenCalledTimes(1) // Tests HOW, not WHAT
})
```

### BDD Testing

Executable specifications using Gherkin.

**Workflow**:
1. **Discovery**: Collaborate with stakeholders on examples
2. **Formulation**: Express examples as Given/When/Then
3. **Automation**: Connect Gherkin to code via step definitions
4. **Implementation**: Write code to pass scenarios

**Good Gherkin (Business Behavior)**:
```gherkin
Scenario: Customer completes purchase
  Given "Widget A" is in stock
  And customer has a valid payment method
  When customer purchases "Widget A"
  Then the order should be confirmed
  And inventory should decrease by 1
```

**Bad Gherkin (Technical Details)**:
```gherkin
Scenario: Purchase
  Given I POST to /api/orders with JSON body
  When the response status is 201
  Then the database contains an order record
```

### Property-Based Testing

Define invariants, not examples.

**Property Categories**:

1. **Invariants**: Must ALWAYS hold
```typescript
test.prop([fc.array(orderItemArbitrary)])
  ('order total is never negative', (items) => {
    const order = createOrder(items)
    expect(order.total).toBeGreaterThanOrEqual(0)
  })
```

2. **Symmetry/Round-Trip**: Encode then decode returns original
```typescript
test.prop([userArbitrary])
  ('user serializes and deserializes correctly', (user) => {
    const serialized = serializeUser(user)
    const deserialized = deserializeUser(serialized)
    expect(deserialized).toEqual(user)
  })
```

3. **Commutativity**: Order doesn't matter
```typescript
test.prop([fc.array(fc.integer()), fc.array(fc.integer())])
  ('set union is commutative', (a, b) => {
    expect(union(new Set(a), new Set(b)))
      .toEqual(union(new Set(b), new Set(a)))
  })
```

4. **Monotonicity**: Increasing input → predictable change
```typescript
test.prop([fc.array(orderItemArbitrary), orderItemArbitrary])
  ('adding item never decreases order total', (items, newItem) => {
    const order1 = createOrder(items)
    const order2 = createOrder([...items, newItem])
    expect(order2.total).toBeGreaterThanOrEqual(order1.total)
  })
```

### Architecture Linting

Automated enforcement of architectural rules.

**Architecture Test Example**:
```typescript
describe('Hexagonal Architecture Rules', () => {
  it('domain should not import from adapters', async () => {
    const domainFiles = await glob('**/domain/**/*.ts', { cwd: srcDir })

    for (const file of domainFiles) {
      const content = fs.readFileSync(path.join(srcDir, file), 'utf-8')
      const adapterImport = content.match(/from\s+['"].*adapters.*['"]/g)
      expect(adapterImport).toBeNull(`Domain file ${file} imports from adapters`)
    }
  })

  it('files should not exceed 500 lines', async () => {
    const tsFiles = await glob('**/*.ts', { cwd: srcDir })

    for (const file of tsFiles) {
      const lines = fs.readFileSync(path.join(srcDir, file), 'utf-8').split('\n').length
      expect(lines).toBeLessThanOrEqual(500)
    }
  })
})
```

**ESLint Rules**:
```javascript
// Domain-specific rules
{
  files: ['**/domain/**/*.ts'],
  rules: {
    'no-restricted-imports': ['error', {
      patterns: ['**/adapters/**', 'hono', 'drizzle-orm']
    }]
  }
}
```

## Creating Custom Skills

Skills follow this template:

```markdown
# [Skill Name] Skill

## Overview
[What this skill covers and why it matters]

## Core Principle
> "[Key insight or philosophy]"

## Key Concepts
### [Concept 1]
[Explanation with examples]

### [Concept 2]
[Explanation with examples]

## Code Examples
```typescript
// Good example
[code]

// Bad example (anti-pattern)
[code]
```

## Best Practices
1. [Practice 1]
2. [Practice 2]

## Anti-Patterns to Avoid
1. [Anti-pattern 1]
2. [Anti-pattern 2]

## Related Skills
- [skill-1] - [how it relates]
- [skill-2] - [how it relates]

## Commands
```bash
[relevant commands]
```
```

## Referencing Skills

Skills can be referenced in CLAUDE.md:
```markdown
## Skills Reference
- `sparc-methodology` - Complete SPARC workflow guide
- `quality-gates` - Automated verification at every checkpoint
- `hexagonal-architecture` - Ports & Adapters patterns
```

Or imported directly:
```markdown
See @.claude/skills/quality-gates.md for implementation details.
```
