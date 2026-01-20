# Background: Architecture, Testing, and SPARC

This section provides the conceptual foundation for the setup guide. Understanding these principles will help you make better use of Claude Code for autonomous development.

---

## Why This Matters

Research shows that **40-62% of AI-generated code contains security vulnerabilities**, and AI-generated pull requests wait **4.6x longer for review** than human-written code. The solution isn't to avoid AI—it's to build infrastructure that makes AI contributions trustworthy.

The key insight: **treat AI-generated code like submissions from a junior developer**. Verify everything, automate quality gates, and invest in architecture that makes verification possible.

---

## Hexagonal Architecture: The Foundation

Hexagonal Architecture (Ports & Adapters) is essential for AI-assisted development because it makes code **testable without external dependencies**.

```
┌─────────────────────────────────────────────────────────────┐
│                        ADAPTERS                              │
│   HTTP handlers, database clients, external APIs             │
│                           │                                  │
│                           ▼                                  │
│   ┌─────────────────────────────────────────────────────┐   │
│   │                      PORTS                           │   │
│   │   Technology-agnostic interfaces (contracts)         │   │
│   └─────────────────────────┬───────────────────────────┘   │
│                             │                                │
│                             ▼                                │
│          ┌────────────────────────────────────┐             │
│          │              DOMAIN                 │             │
│          │   Pure business logic               │             │
│          │   NO external dependencies          │             │
│          │   Fully testable in isolation       │             │
│          └────────────────────────────────────┘             │
└─────────────────────────────────────────────────────────────┘
```

**The Dependency Rule**: All dependencies point INWARD. Domain never imports from adapters.

**Why this matters for AI**: When business logic is isolated from infrastructure, AI can generate domain code that's immediately testable. Without this structure, AI tends to produce tightly-coupled "spaghetti code" that's hard to verify.

---

## Testing Strategy: Specification-Driven, Not Implementation-Coupled

Traditional unit testing often tests *how* code works internally. This creates tests that break when you refactor, even if behavior is preserved. For AI-generated code, this is especially problematic—the AI may implement things differently than you expect.

**The solution**: Test *what* the system does (behavior), not *how* it does it.

### Beyond the Pyramid: Modern Testing Shapes

The traditional testing pyramid emphasizes unit test volume, but modern strategies recognize that **integration tests provide optimal confidence-per-effort**. Three alternative shapes have emerged:

**Testing Trophy** (Kent C. Dodds): Static analysis as foundation, integration tests as primary focus, unit tests only for complex logic. *"The more your tests resemble the way your software is used, the more confidence they can give you."*

**Testing Honeycomb** (Spotify): Treats the microservice as the unit of testing. Integration tests validate the service through its edges (API, database, queues). Some Spotify services have **zero implementation detail tests**.

**Testing Diamond**: Integration tests form the widest layer, with unit tests only for "critical parts"—parsing, calculations, complex transformations.

### Recommended Shape for AI-Generated Code

```
          ╱╲            E2E: Critical paths only
         ╱──╲
        ╱    ╲          BDD/Gherkin: Executable specifications
       ╱──────╲
      ╱        ╲        Integration: Test through ports (fakes/real)
     ╱ ════════ ╲       ← WIDEST LAYER
    ╱            ╲      Property Tests: Domain invariants
   ╱──────────────╲
  ╱                ╲    Unit: Complex algorithms only
 ╱                  ╲
```

**Choose by architecture**:
- **Microservices/APIs**: Testing Honeycomb—treat the service as the unit
- **Frontend applications**: Testing Trophy—integration with Testing Library patterns
- **Domain-heavy applications**: Testing Diamond—integration at boundaries, units for algorithms

All patterns converge on one insight: **minimize tests that lock in implementation details, maximize tests that validate observable behavior**.

### Key Testing Principles

1. **Use Fakes, Not Mocks** for domain testing
   - Fakes are working implementations (e.g., in-memory database)
   - Mocks verify method calls (couples tests to implementation)
   - Reserve mocks for adapter boundaries only

2. **Write Specifications Before Code**
   - Gherkin scenarios define acceptance criteria
   - Property tests define domain invariants
   - Code is written to satisfy specifications

3. **Contract Tests Validate Fakes**
   - Run same tests against fake AND real implementation
   - Ensures your test doubles actually behave correctly

### Example: Good vs. Bad Tests

```typescript
// BAD: Tests implementation details
it('calls repository.create once', () => {
  const mockRepo = { create: vi.fn() }
  await registerUser(input, { repo: mockRepo })
  expect(mockRepo.create).toHaveBeenCalledTimes(1)  // Tests HOW
})

// GOOD: Tests observable behavior
it('registered user can authenticate', async () => {
  const deps = createTestDependencies()  // Uses fakes
  await registerUser({ email: 'test@example.com', password: 'Pass123!' }, deps)

  // Verify WHAT: the user can now authenticate
  const result = await authenticate({ email: 'test@example.com', password: 'Pass123!' }, deps)
  expect(result.isOk()).toBe(true)
})
```

---

## SPARC: A Structured Workflow for AI-Assisted Development

SPARC (Specification, Pseudocode, Architecture, Refinement, Completion) provides a structured workflow that maximizes AI effectiveness while maintaining quality.

### The Phases

| Phase | Purpose | Key Output |
|-------|---------|------------|
| **0. Research** | Gather knowledge before designing | Technology decisions, best practices |
| **1. Specification** | Define WHAT the system does | Gherkin scenarios, property invariants |
| **2. Pseudocode** | Design HOW to solve it | Algorithm outlines, function signatures |
| **3. Architecture** | Structure WHERE code lives | Folder structure, port interfaces, fakes |
| **4. Refinement** | Implement driven by specs | Working code that passes all specifications |
| **5. Completion** | Verify and document | Passing tests, security audit, documentation |

### Why SPARC Works for AI

1. **Specification-first** means AI has clear targets to satisfy
2. **Hexagonal architecture** enables isolated, testable code generation
3. **Quality gates** at each phase catch issues early
4. **Parallel execution** lets multiple agents work on backend/frontend/tests simultaneously

### The Core Philosophy

> "Think before coding. Specify before implementing. Test behavior, not implementation."

---

## Quality Gates: Non-Negotiable Checkpoints

Quality gates are mandatory automated checks that MUST pass before proceeding. They're not optional—they're the enforcement mechanism that makes AI autonomy safe.

| Gate | When | Time | Checks |
|------|------|------|--------|
| `gate:fast` | After EVERY code change | < 10s | TypeScript, ESLint |
| `gate:unit` | After completing a feature | < 60s | + Unit tests, property tests |
| `gate:commit` | Before EVERY commit | < 2min | + Contracts, BDD, architecture |
| `gate:full` | Before PR/merge | < 10min | + E2E, coverage, security scan |

**The rule**: No code proceeds without passing gates. No exceptions.

---

## Putting It Together

The setup in this guide implements these principles through:

1. **CLAUDE.md** - Communicates architecture rules and quality expectations to AI
2. **Specialized Agents** - Role-specific configurations (coder, tester, reviewer, etc.)
3. **Memory Bank** - Persistent context so AI understands the project deeply
4. **Slash Commands** - Automate SPARC workflow phases
5. **Skills** - Reference documentation for patterns and practices
6. **Hooks & CI** - Enforce quality gates automatically

The goal: enable AI to work autonomously while guaranteeing the output meets your standards.

---

## Further Reading

For deeper dives into these concepts, see:
- Part 4: SPARC Methodology (detailed phase breakdown)
- Part 7: Skills Reference (hexagonal architecture, testing patterns)
- Part 9: Quality Gates (implementation details)
- Part 13: Testing Strategy (complete testing approach)
