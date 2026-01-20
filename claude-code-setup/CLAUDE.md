# Project: Full-Stack TypeScript Application

## Tech Stack
- **Runtime**: Bun 1.x (backend)
- **Backend**: Hono framework, TypeScript strict mode
- **Frontend**: Vue 3 (Composition API), Pinia, Vue Router, TypeScript
- **Architecture**: Hexagonal (Ports & Adapters) for backend, Feature-based for frontend
- **Testing**: Vitest (unit), Playwright (e2e), Cucumber/Gherkin (BDD)
- **Package Manager**: Bun workspaces (monorepo)

## Key Commands
```bash
# Development
bun run dev              # Start all services
bun run dev:api          # Start API only (port 3000)
bun run dev:web          # Start frontend only (port 5173)

# Testing
bun run test             # Run all tests
bun run test:unit        # Run unit tests
bun run test:e2e         # Run e2e tests
bun run test:features    # Run BDD feature tests

# Build & Quality
bun run build            # Build all packages
bun run lint             # ESLint check
bun run typecheck        # TypeScript check
bun run arch:test        # Run architecture tests
```

## Directory Structure
```
apps/
  api/                   # Bun + Hono backend
    src/
      domain/            # Pure business logic (NO external imports)
      ports/             # Interfaces (driven & driver ports)
      adapters/          # Implementations (http, db, external)
      config/            # Environment configuration
      utils/             # Internal utilities
    tests/
      unit/              # Unit tests (domain logic)
      integration/       # Integration tests (adapters)
      e2e/               # End-to-end API tests
      features/          # Gherkin/Cucumber BDD tests

  web/                   # Vue 3 frontend
    src/
      api/               # API client layer
      components/        # base/ (UI) + business/ (domain)
      composables/       # Reusable reactive logic
      layouts/           # Page layouts
      router/            # Vue Router config
      stores/            # Pinia state management
      types/             # TypeScript definitions
      utils/             # Pure utility functions
      views/             # Page components

packages/
  shared-types/          # Shared TypeScript types
  shared-utils/          # Shared utility functions
  eslint-config/         # Shared ESLint configuration
  typescript-config/     # Shared TypeScript configuration
```

## Architecture Rules (STRICTLY ENFORCED)
1. **Domain Isolation**: `domain/` NEVER imports from `adapters/` or external libraries
2. **Port Contracts**: All external communication through `ports/` interfaces
3. **Dependency Injection**: Adapters injected at startup, never hard-coded
4. **No File > 500 lines**: Split large files into focused modules
5. **No Function > 50 lines**: Keep functions small and testable
6. **No Hard-coded Secrets**: Use environment variables via config/

## Code Style
- TypeScript strict mode enabled (no `any`, no implicit returns)
- Named exports preferred over default exports
- 2-space indentation
- Single quotes for strings
- Trailing commas in multi-line
- Vue: `<script setup>` with Composition API
- Tests: Arrange-Act-Assert pattern

## Testing Strategy (Specification-Driven)

### Philosophy: Test WHAT, Not HOW
Tests validate observable behavior against specifications, NOT implementation details.
Never test that "method X called method Y" — test that "given input A, output is B".

### Test Pyramid (Specification Focus)
```
        ╱╲
       ╱  ╲        E2E: Critical user journeys only ("money paths")
      ╱────╲
     ╱      ╲      BDD/Feature: Executable specifications (Gherkin)
    ╱────────╲
   ╱          ╲    Component: Domain logic through port interfaces (black-box)
  ╱────────────╲
 ╱              ╲  Property: Invariants that must hold for ALL inputs
╱────────────────╲
```

### Layer Guidelines

1. **Property-Based Tests** (Foundation)
   - Define invariants: "Total price is never negative"
   - Use Hypothesis/fast-check to generate random inputs
   - Catches edge cases humans miss

2. **Component/Unit Tests** (Black-Box)
   - Test domain logic through ports, NOT internal methods
   - Use Fakes (working in-memory implementations), NOT mocks
   - Verify contract: same tests run against Fake AND real adapter

3. **BDD Feature Tests** (Specifications)
   - Gherkin scenarios ARE the acceptance criteria
   - Written BEFORE implementation
   - Stakeholder-readable documentation

4. **E2E Tests** (Smoke/Critical Paths)
   - Only test critical user flows
   - Avoid testing every edge case at this layer
   - Keep fast to prevent flakiness

### Anti-Patterns (AVOID)
- Mocking internal collaborators within domain
- Testing private methods directly
- Asserting on method call counts
- Tests that break when refactoring (without behavior change)

### Coverage Targets
- Domain invariants: 100% (via property tests)
- Feature scenarios: 100% acceptance criteria covered
- Overall line coverage: 80%+ (secondary metric)

## BDD Feature Format
```gherkin
Feature: [Feature Name]
  As a [role]
  I want [goal]
  So that [benefit]

  Background:
    Given [common preconditions]

  Scenario: [Happy Path]
    Given [specific precondition]
    When [action]
    Then [expected observable outcome]

  Scenario Outline: [Parameterized Scenario]
    Given <input>
    When [action]
    Then <expected_output>

    Examples:
      | input | expected_output |
      | A     | X               |
      | B     | Y               |
```

## Git Commit Format
- `feat:` New features
- `fix:` Bug fixes
- `test:` Test additions/changes
- `docs:` Documentation
- `refactor:` Code restructuring
- `chore:` Maintenance tasks

## SPARC Workflow Commands
```bash
# Full workflow (all phases)
/sparc-full                    # Interactive mode with checkpoints
/sparc-full --auto             # Autonomous mode (no pauses)
/sparc-full --parallel         # Enable parallel backend/frontend tracks
/sparc-full --skip-research    # Skip Phase 0 (research)

# Individual phases
/sparc-research                # Phase 0: Gather documentation/best practices
/sparc-spec                    # Phase 1: Create specifications (BDD scenarios)
/sparc-pseudo                  # Phase 2: Design algorithms and signatures
/sparc-arch                    # Phase 3: Create hexagonal architecture
/sparc-refine                  # Phase 4: Implement (specification-driven)
/sparc-complete                # Phase 5: Review, audit, document
```

## Agents
- `orchestrator` - Coordinates multi-agent workflows (boomerang pattern)
- `researcher` - Gathers external knowledge for Phase 0
- `architect` - System design, specifications, pseudocode, architecture
- `coder` - Implementation following specifications
- `tester` - Property tests, BDD, contract tests
- `reviewer` - Code quality and standards review
- `security-auditor` - Security vulnerability analysis

## Skills Reference
- `sparc-methodology` - Complete SPARC workflow guide
- `hexagonal-architecture` - Ports & Adapters patterns
- `tdd-workflow` - Specification-driven testing (Fakes over Mocks)
- `bdd-testing` - Gherkin/Cucumber with executable specifications
- `property-testing` - Domain invariants with fast-check
- `metamorphic-testing` - Testing without oracle (for AI-generated code)
- `arch-linting` - Architecture constraint enforcement

## Important Notes
- NEVER commit `.env` files or secrets
- Run `bun run lint && bun run test` before committing
- Use shared types from `@app/shared-types` for API contracts
- Backend responses follow JSON:API specification
- All API endpoints require authentication except health checks
