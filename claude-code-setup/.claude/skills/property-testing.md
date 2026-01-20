# Property-Based Testing Skill

## Core Principle: Define Invariants, Not Examples

> "A single property test can replace hundreds of example-based tests by defining what must ALWAYS be true."

Property-based testing (PBT) generates random inputs to verify that invariants hold across the entire input domain, not just cherry-picked examples.

## Why Property-Based Testing?

### Example-Based Testing Limitations
```typescript
// You test the cases you think of...
test('adds two numbers', () => {
  expect(add(1, 2)).toBe(3)
  expect(add(0, 0)).toBe(0)
  expect(add(-1, 1)).toBe(0)
})
// But what about add(Number.MAX_SAFE_INTEGER, 1)?
// Or add(0.1, 0.2)?
```

### Property-Based Testing Strength
```typescript
// Define what's ALWAYS true
test.prop([fc.integer(), fc.integer()])
  ('addition is commutative', (a, b) => {
    expect(add(a, b)).toBe(add(b, a))
  })

test.prop([fc.integer()])
  ('adding zero is identity', (n) => {
    expect(add(n, 0)).toBe(n)
  })
// Frameworks generate thousands of random inputs to find counterexamples
```

## Property Categories

### 1. Invariants
Properties that must ALWAYS hold, regardless of input.

```typescript
// Business invariant: Order total is never negative
test.prop([
  fc.array(fc.record({
    price: fc.integer({ min: 0, max: 1000000 }),
    quantity: fc.integer({ min: 1, max: 1000 })
  }))
])('order total is never negative', (items) => {
  const order = createOrder(items)
  expect(order.total).toBeGreaterThanOrEqual(0)
})

// Domain invariant: User email is always lowercase after creation
test.prop([fc.emailAddress()])
  ('user email is normalized to lowercase', (email) => {
    const user = createUser({ email })
    expect(user.email).toBe(email.toLowerCase())
  })
```

### 2. Idempotence
Operations that can be applied multiple times with same result.

```typescript
// Parsing then serializing should be idempotent
test.prop([fc.json()])
  ('JSON round-trip is idempotent', (data) => {
    const once = JSON.parse(JSON.stringify(data))
    const twice = JSON.parse(JSON.stringify(once))
    expect(twice).toEqual(once)
  })

// Normalizing a path twice gives same result
test.prop([fc.string()])
  ('path normalization is idempotent', (path) => {
    const once = normalizePath(path)
    const twice = normalizePath(once)
    expect(twice).toBe(once)
  })
```

### 3. Symmetry / Round-Trip
Encoding then decoding returns original value.

```typescript
// Encryption round-trip
test.prop([fc.string(), fc.string({ minLength: 16 })])
  ('encrypt then decrypt returns original', (plaintext, key) => {
    const encrypted = encrypt(plaintext, key)
    const decrypted = decrypt(encrypted, key)
    expect(decrypted).toBe(plaintext)
  })

// Serialization round-trip
test.prop([userArbitrary])
  ('user serializes and deserializes correctly', (user) => {
    const serialized = serializeUser(user)
    const deserialized = deserializeUser(serialized)
    expect(deserialized).toEqual(user)
  })
```

### 4. Commutativity / Associativity
Order of operations doesn't matter.

```typescript
// Set operations are commutative
test.prop([fc.array(fc.integer()), fc.array(fc.integer())])
  ('set union is commutative', (a, b) => {
    const setA = new Set(a)
    const setB = new Set(b)
    const unionAB = union(setA, setB)
    const unionBA = union(setB, setA)
    expect(unionAB).toEqual(unionBA)
  })

// Filter order doesn't change result (for independent filters)
test.prop([fc.array(fc.integer())])
  ('filter order is irrelevant', (numbers) => {
    const evenThenPositive = numbers.filter(isEven).filter(isPositive)
    const positiveThenEven = numbers.filter(isPositive).filter(isEven)
    expect(evenThenPositive).toEqual(positiveThenEven)
  })
```

### 5. Monotonicity
Increasing input leads to predictable output change.

```typescript
// Adding items increases or maintains total
test.prop([
  fc.array(orderItemArbitrary),
  orderItemArbitrary
])('adding item never decreases order total', (items, newItem) => {
  const order1 = createOrder(items)
  const order2 = createOrder([...items, newItem])
  expect(order2.total).toBeGreaterThanOrEqual(order1.total)
})

// Higher credit score → better (or equal) loan terms
test.prop([
  fc.integer({ min: 300, max: 850 }),
  fc.integer({ min: 300, max: 850 })
])('higher credit score gives better terms', (score1, score2) => {
  fc.pre(score2 > score1) // Precondition
  const terms1 = calculateLoanTerms(score1)
  const terms2 = calculateLoanTerms(score2)
  expect(terms2.interestRate).toBeLessThanOrEqual(terms1.interestRate)
})
```

### 6. Oracle / Reference Implementation
Compare against known-correct (possibly slow) implementation.

```typescript
// Fast algorithm matches slow but correct reference
test.prop([fc.array(fc.integer())])
  ('optimized sort matches reference sort', (arr) => {
    const optimized = quickSort([...arr])
    const reference = arr.slice().sort((a, b) => a - b)
    expect(optimized).toEqual(reference)
  })

// New parser matches old parser
test.prop([fc.string()])
  ('new parser matches legacy parser', (input) => {
    const newResult = newParser(input)
    const legacyResult = legacyParser(input)
    expect(newResult).toEqual(legacyResult)
  })
```

## Framework: fast-check with Vitest

### Setup
```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    // Enable property testing
  }
})

// Install
// bun add -D @fast-check/vitest fast-check
```

### Basic Usage
```typescript
import { test, fc } from '@fast-check/vitest'

// Simple property
test.prop([fc.string()])('string has non-negative length', (s) => {
  expect(s.length).toBeGreaterThanOrEqual(0)
})

// Multiple arbitraries
test.prop([fc.integer(), fc.integer(), fc.integer()])
  ('addition is associative', (a, b, c) => {
    expect((a + b) + c).toBe(a + (b + c))
  })

// With preconditions
test.prop([fc.integer(), fc.integer()])
  ('division then multiplication returns original', (a, b) => {
    fc.pre(b !== 0)  // Skip when b is 0
    expect((a / b) * b).toBeCloseTo(a)
  })
```

### Custom Arbitraries

```typescript
// Domain object arbitrary
const userArbitrary = fc.record({
  id: fc.uuid(),
  email: fc.emailAddress(),
  firstName: fc.string({ minLength: 1, maxLength: 50 }),
  lastName: fc.string({ minLength: 1, maxLength: 50 }),
  age: fc.integer({ min: 0, max: 150 }),
  role: fc.constantFrom('admin', 'user', 'guest'),
  createdAt: fc.date({ min: new Date('2020-01-01') })
})

// Money arbitrary (avoid floating point)
const moneyArbitrary = fc.record({
  cents: fc.integer({ min: 0, max: 100000000 }), // Store as cents
  currency: fc.constantFrom('USD', 'EUR', 'GBP')
})

// Order item arbitrary
const orderItemArbitrary = fc.record({
  productId: fc.uuid(),
  name: fc.string({ minLength: 1, maxLength: 100 }),
  price: fc.integer({ min: 1, max: 1000000 }), // cents
  quantity: fc.integer({ min: 1, max: 100 })
})

// Use in tests
test.prop([userArbitrary])('user validation works', (user) => {
  const result = validateUser(user)
  expect(result.isValid).toBe(true)
})
```

### Shrinking
fast-check automatically finds minimal failing examples:

```typescript
// If this fails for [1000, 500, -1], fast-check will shrink to [-1]
test.prop([fc.array(fc.integer())])
  ('all numbers are positive', (numbers) => {
    expect(numbers.every(n => n >= 0)).toBe(true)
  })
// Output: Counterexample: [-1]
// (Shrunk from original random array)
```

## Domain-Specific Properties

### E-Commerce
```typescript
// Cart invariants
test.prop([fc.array(orderItemArbitrary)])
  ('cart total equals sum of line items', (items) => {
    const cart = createCart(items)
    const expectedTotal = items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    )
    expect(cart.total).toBe(expectedTotal)
  })

// Discount never exceeds original price
test.prop([
  fc.integer({ min: 100, max: 1000000 }),
  fc.integer({ min: 0, max: 100 })
])('discount never exceeds original price', (price, discountPercent) => {
  const discounted = applyDiscount(price, discountPercent)
  expect(discounted).toBeGreaterThanOrEqual(0)
  expect(discounted).toBeLessThanOrEqual(price)
})
```

### Authentication
```typescript
// Password hashing properties
test.prop([fc.string({ minLength: 8, maxLength: 128 })])
  ('same password produces same verification result', async (password) => {
    const hash = await hashPassword(password)
    const result1 = await verifyPassword(password, hash)
    const result2 = await verifyPassword(password, hash)
    expect(result1).toBe(result2)
    expect(result1).toBe(true)
  })

test.prop([
  fc.string({ minLength: 8 }),
  fc.string({ minLength: 8 })
])('different passwords produce different hashes', async (p1, p2) => {
  fc.pre(p1 !== p2)
  const hash1 = await hashPassword(p1)
  const hash2 = await hashPassword(p2)
  expect(hash1).not.toBe(hash2)
})
```

### Data Transformation
```typescript
// CSV parsing properties
test.prop([fc.array(fc.array(fc.string()))])
  ('CSV round-trip preserves data', (rows) => {
    const csv = toCsv(rows)
    const parsed = parseCsv(csv)
    expect(parsed).toEqual(rows)
  })

// Date formatting
test.prop([fc.date()])
  ('date format round-trip', (date) => {
    const formatted = formatDate(date, 'YYYY-MM-DD')
    const parsed = parseDate(formatted, 'YYYY-MM-DD')
    // Compare dates (ignoring time)
    expect(parsed.toDateString()).toBe(date.toDateString())
  })
```

### State Machines
```typescript
// State transition properties
const stateArbitrary = fc.constantFrom('idle', 'loading', 'success', 'error')
const eventArbitrary = fc.constantFrom('fetch', 'succeed', 'fail', 'reset')

test.prop([stateArbitrary, fc.array(eventArbitrary)])
  ('reset always returns to idle', (initialState, events) => {
    let state = initialState
    for (const event of events) {
      state = transition(state, event)
    }
    state = transition(state, 'reset')
    expect(state).toBe('idle')
  })
```

## Test Organization

```
tests/
└── properties/
    ├── domain/
    │   ├── order-invariants.test.ts
    │   ├── user-invariants.test.ts
    │   └── pricing-rules.test.ts
    ├── serialization/
    │   ├── json-roundtrip.test.ts
    │   └── api-contracts.test.ts
    └── algorithms/
        ├── sorting.test.ts
        └── search.test.ts
```

## When to Use Property Testing

| Use Property Testing | Use Example Testing |
|---------------------|---------------------|
| Mathematical operations | UI interactions |
| Data transformations | Specific bug reproductions |
| Serialization/parsing | Integration with mocks |
| Algorithms with known properties | Configuration validation |
| Domain invariants | Error message formatting |
| Security-critical code | Specific edge cases |

## Best Practices

1. **Start with invariants**: "What must ALWAYS be true?"
2. **Use domain arbitraries**: Model your actual data shapes
3. **Leverage shrinking**: Let the framework find minimal cases
4. **Combine with examples**: Use both for comprehensive coverage
5. **Test at boundaries**: Include min/max values in generators
6. **Watch for flakiness**: Seed random tests in CI for reproducibility

## CI Configuration

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    // Seed for reproducibility in CI
    seed: process.env.CI ? 12345 : undefined,

    // More iterations in CI
    fuzz: {
      numRuns: process.env.CI ? 1000 : 100
    }
  }
})
```

## Checklist: Is This a Good Property?

- [ ] Does it describe something that's ALWAYS true?
- [ ] Is it independent of specific examples?
- [ ] Would finding a counterexample reveal a real bug?
- [ ] Is the property understandable to domain experts?
- [ ] Does it exercise interesting edge cases?
