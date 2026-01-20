# Metamorphic Testing Skill

## Core Principle: Test Without an Oracle

> "When you don't know the expected output, test the RELATIONSHIP between inputs and outputs."

Metamorphic testing addresses the **oracle problem** — situations where you can't easily determine if a specific output is correct, but you CAN verify that relationships between outputs hold.

## Why Metamorphic Testing?

### The Oracle Problem
```typescript
// How do you test this without manually computing the answer?
function calculateOptimalRoute(graph: Graph, start: Node, end: Node): Path {
  // Complex algorithm...
}

// You don't know if the result is optimal, but you CAN verify:
// - A longer route shouldn't cost less than a shorter one
// - Adding an edge shouldn't make the optimal path longer
// - Reversing start/end shouldn't change path length (in undirected graphs)
```

### Metamorphic Relations (MRs)
Instead of checking `f(x) === expected`, check relationships:
- `f(x) relation f(transform(x))`
- Example: `sort(reverse(arr)) === sort(arr)`

## Metamorphic Relation Categories

### 1. Invariance
Output remains the same despite input transformation.

```typescript
// Sorting is invariant to input order
test('sorting is invariant to shuffle', () => {
  const original = [3, 1, 4, 1, 5, 9]
  const shuffled = shuffle([...original])

  expect(sort(shuffled)).toEqual(sort(original))
})

// Search results invariant to query case
test('search is case-insensitive', () => {
  const result1 = search('Hello World')
  const result2 = search('hello world')
  const result3 = search('HELLO WORLD')

  expect(result1).toEqual(result2)
  expect(result2).toEqual(result3)
})
```

### 2. Symmetry
Swapping inputs produces predictable output change.

```typescript
// Distance is symmetric
test('distance is symmetric', () => {
  const pointA = { x: 0, y: 0 }
  const pointB = { x: 3, y: 4 }

  expect(distance(pointA, pointB)).toBe(distance(pointB, pointA))
})

// Comparison is anti-symmetric
test('comparison reverses when operands swap', () => {
  const a = 5, b = 10
  const cmp1 = compare(a, b)  // -1 (a < b)
  const cmp2 = compare(b, a)  // +1 (b > a)

  expect(cmp1).toBe(-cmp2)
})
```

### 3. Monotonicity
Increasing input leads to predictable output direction.

```typescript
// Price increases with quantity (before bulk discounts)
test('price increases with quantity', () => {
  const price1 = calculatePrice({ quantity: 1, unitPrice: 10 })
  const price2 = calculatePrice({ quantity: 2, unitPrice: 10 })
  const price3 = calculatePrice({ quantity: 3, unitPrice: 10 })

  expect(price2).toBeGreaterThan(price1)
  expect(price3).toBeGreaterThan(price2)
})

// Credit score increase shouldn't worsen loan terms
test('better credit score gives equal or better terms', () => {
  const terms1 = getLoanTerms({ creditScore: 650 })
  const terms2 = getLoanTerms({ creditScore: 750 })

  expect(terms2.interestRate).toBeLessThanOrEqual(terms1.interestRate)
  expect(terms2.maxAmount).toBeGreaterThanOrEqual(terms1.maxAmount)
})
```

### 4. Additivity / Composition
Combining inputs combines outputs predictably.

```typescript
// Total of merged orders equals sum of individual totals
test('order totals are additive', () => {
  const order1 = createOrder([{ price: 100, qty: 1 }])
  const order2 = createOrder([{ price: 50, qty: 2 }])
  const merged = mergeOrders(order1, order2)

  expect(merged.total).toBe(order1.total + order2.total)
})

// String concat length is additive
test('concat length is sum of lengths', () => {
  const s1 = 'hello'
  const s2 = 'world'

  expect(concat(s1, s2).length).toBe(s1.length + s2.length)
})
```

### 5. Invertibility / Round-Trip
Applying inverse operation returns to original.

```typescript
// Encrypt/decrypt round-trip
test('encryption is invertible', () => {
  const plaintext = 'secret message'
  const key = 'secure-key-123'

  const encrypted = encrypt(plaintext, key)
  const decrypted = decrypt(encrypted, key)

  expect(decrypted).toBe(plaintext)
})

// Serialize/deserialize round-trip
test('serialization is invertible', () => {
  const user = { id: '123', name: 'Alice', age: 30 }

  const serialized = serialize(user)
  const deserialized = deserialize(serialized)

  expect(deserialized).toEqual(user)
})
```

### 6. Subset / Inclusion
Subset of input produces subset of output.

```typescript
// Filtering then searching ⊆ searching all
test('filtered search is subset of full search', () => {
  const allResults = search('query', { filter: null })
  const filteredResults = search('query', { filter: 'category-a' })

  // Every filtered result should appear in all results
  filteredResults.forEach(result => {
    expect(allResults).toContainEqual(result)
  })
})

// Permissions: subset of roles gets subset of permissions
test('fewer roles means fewer or equal permissions', () => {
  const perms1 = getPermissions(['admin', 'editor'])
  const perms2 = getPermissions(['editor'])

  perms2.forEach(perm => {
    expect(perms1).toContain(perm)
  })
})
```

## Metamorphic Prompt Testing (for AI-Generated Code)

A critical technique for validating LLM-generated code by testing across paraphrased prompts.

### The Technique
```typescript
// Generate code from multiple paraphrased prompts
const prompts = [
  'Write a function that checks if a number is prime',
  'Create a primality test function',
  'Implement isPrime that returns true for prime numbers',
  'Write code to determine if an integer is a prime number',
]

// Generate code from each prompt
const implementations = await Promise.all(
  prompts.map(prompt => llm.generateCode(prompt))
)

// All implementations should behave identically
test('paraphrased prompts produce consistent behavior', () => {
  const testInputs = [2, 3, 4, 5, 10, 17, 100, 101]

  testInputs.forEach(input => {
    const results = implementations.map(impl => impl(input))
    const firstResult = results[0]

    // All implementations should agree
    results.forEach((result, i) => {
      expect(result).toBe(firstResult,
        `Implementation ${i} disagrees on input ${input}`)
    })
  })
})
```

### Why This Works
- If prompts describe the same behavior, correct code should behave identically
- Disagreement indicates at least one implementation is wrong
- 75% recall rate in detecting erroneous LLM-generated code (per research)

### Implementation Pattern
```typescript
import { test, fc } from '@fast-check/vitest'

// Combine with property-based testing for comprehensive coverage
test.prop([fc.integer({ min: 1, max: 10000 })])
  ('all isPrime implementations agree', async (n) => {
    const results = await Promise.all(
      isPrimeImplementations.map(impl => impl(n))
    )

    const allSame = results.every(r => r === results[0])
    expect(allSame).toBe(true)
  })
```

## Domain-Specific Metamorphic Relations

### E-Commerce
```typescript
// Adding item never decreases cart total
MR: cart.add(item) => total' >= total

// Removing item never increases cart total
MR: cart.remove(item) => total' <= total

// Empty cart has zero total
MR: cart.clear() => total === 0

// Applying then removing coupon returns to original price
MR: removeCoupon(applyCoupon(cart, code), code) => cart
```

### Search / Filtering
```typescript
// More specific query returns subset of results
MR: search(query + ' extra') ⊆ search(query)

// AND filters reduce results
MR: count(filter(A) AND filter(B)) <= count(filter(A))

// OR filters increase results
MR: count(filter(A) OR filter(B)) >= count(filter(A))
```

### Authentication
```typescript
// Same credentials always produce same auth result
MR: auth(user, pass) === auth(user, pass)

// Wrong password always fails (regardless of which wrong password)
MR: auth(user, wrong1).failed === auth(user, wrong2).failed

// Logout then login with valid creds succeeds
MR: login(logout(session), validCreds).success === true
```

### Data Processing
```typescript
// Parsing valid format always succeeds
MR: parse(serialize(data)).success === true

// Double transformation is idempotent (for normalization)
MR: normalize(normalize(data)) === normalize(data)

// Chunking preserves total count
MR: sum(chunks.map(c => c.length)) === original.length
```

## Test Organization

```
tests/
└── metamorphic/
    ├── search-relations.test.ts
    ├── cart-relations.test.ts
    ├── auth-relations.test.ts
    └── prompt-consistency.test.ts
```

## Combining with Property-Based Testing

```typescript
import { test, fc } from '@fast-check/vitest'

// Metamorphic relation as a property
test.prop([fc.array(fc.integer())])
  ('sorting is invariant to shuffle', (arr) => {
    const sorted1 = sort([...arr])
    const sorted2 = sort(shuffle([...arr]))

    expect(sorted1).toEqual(sorted2)
  })

// Symmetry as a property
test.prop([fc.integer(), fc.integer()])
  ('addition is commutative', (a, b) => {
    expect(add(a, b)).toBe(add(b, a))
  })

// Monotonicity as a property
test.prop([
  fc.integer({ min: 0, max: 100 }),
  fc.integer({ min: 0, max: 100 })
])('quantity increase raises or maintains price', (q1, q2) => {
  fc.pre(q2 > q1)
  const price1 = calculatePrice(q1)
  const price2 = calculatePrice(q2)
  expect(price2).toBeGreaterThanOrEqual(price1)
})
```

## When to Use Metamorphic Testing

| Use Case | Why It Helps |
|----------|--------------|
| Complex algorithms (ML, optimization) | No oracle for "correct" output |
| Data transformations | Verify relationships without checking every value |
| Search/ranking systems | Can't verify absolute correctness, but can verify consistency |
| AI-generated code | Cross-validate across paraphrased specifications |
| Security/crypto | Verify properties (invertibility, uniqueness) |
| Numerical computing | Verify mathematical properties |

## Checklist: Defining Good Metamorphic Relations

- [ ] Does the relation describe a fundamental property of the system?
- [ ] Is it independent of specific input values?
- [ ] Would violating it indicate a real bug?
- [ ] Can it be combined with random input generation?
- [ ] Is it testable without knowing the "correct" output?

## Integration with Test Suite

```typescript
// In vitest.config.ts or test setup
describe('Metamorphic Relations: Cart', () => {
  describe('MR1: Adding items', () => {
    test.prop([cartArbitrary, itemArbitrary])
      ('adding item never decreases total', (cart, item) => {
        const before = cart.total
        cart.add(item)
        expect(cart.total).toBeGreaterThanOrEqual(before)
      })
  })

  describe('MR2: Removing items', () => {
    test.prop([nonEmptyCartArbitrary])
      ('removing item never increases total', (cart) => {
        const item = cart.items[0]
        const before = cart.total
        cart.remove(item)
        expect(cart.total).toBeLessThanOrEqual(before)
      })
  })

  describe('MR3: Round-trip', () => {
    test.prop([cartArbitrary, itemArbitrary])
      ('add then remove returns to original total', (cart, item) => {
        const before = cart.total
        cart.add(item)
        cart.remove(item)
        expect(cart.total).toBe(before)
      })
  })
})
```
