# SPARC Methodology Skill

## Overview
SPARC (Specification, Pseudocode, Architecture, Refinement, Completion) is a structured workflow for AI-assisted software development that emphasizes **specification-driven testing** and **hexagonal architecture**.

## Core Philosophy

> "Think before coding. Specify before implementing. Test behavior, not implementation."

## Phases

### Phase 0: Research (Optional)
**Objective**: Gather external knowledge to inform design decisions before writing specifications.

**When to Use**: Run when you need information about technologies, frameworks, best practices, or similar implementations. Skip with `--skip-research` if requirements are already clear.

**Deliverables**:
- Technology documentation summaries
- Best practices and patterns
- Similar implementation analysis
- Decision rationale

**Agent**: `researcher` (see `.claude/agents/researcher.md`)
**Command**: `/sparc-research`

**Actions**:
1. Identify research topics from project brief
2. Fetch official documentation and tutorials
3. Search for best practices and patterns
4. Synthesize findings into decision rationale
5. Update memory bank with technology decisions

### Phase 1: Specification
**Objective**: Define clear, testable requirements before any code is written.

**Deliverables**:
- Functional requirements document
- Non-functional requirements (performance, security, scalability)
- User stories with acceptance criteria
- **BDD/Gherkin scenarios** (executable specifications)
- **Domain invariants** (properties that must ALWAYS hold)

**Actions**:
1. Read project brief from `.claude/memory-bank/projectBrief.md`
2. Analyze requirements and extract:
   - Core features and behaviors
   - Edge cases and error conditions
   - Integration points
3. Document specification in `/docs/specification.md`
4. **Write Gherkin scenarios BEFORE any implementation**
5. **Define property invariants for domain logic**

**Specification Artifacts**:
```
docs/
├── specification.md           # Requirements document
└── features/
    ├── auth.feature           # BDD scenarios
    └── orders.feature
```

### Phase 2: Pseudocode
**Objective**: Create high-level solution design without implementation.

**Deliverables**:
- Algorithm outlines for core logic
- Data flow diagrams (text-based)
- Function/method signatures
- **Test strategy outline** (properties, BDD, contracts)

**Actions**:
1. Review specification from Phase 1
2. Design solution approach for each requirement
3. Document pseudocode in `/docs/pseudocode.md`
4. **Identify domain invariants to test with property testing**
5. **Plan fake implementations for dependencies**

### Phase 3: Architecture
**Objective**: Design system structure following hexagonal architecture.

**Deliverables**:
- Component/module definitions
- Port interfaces (driven and driver)
- Data models and schemas
- File structure with stubs
- **Fake implementations for testing**

**Hexagonal Architecture Rules**:
```
┌────────────────────────────────────────────────────────────┐
│                        ADAPTERS                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │   HTTP   │  │    DB    │  │  Email   │  │  Queue   │   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘   │
│       │             │             │             │          │
│       ▼             ▼             ▼             ▼          │
│  ┌─────────────────────────────────────────────────────┐  │
│  │                     PORTS                            │  │
│  │  (Technology-agnostic interfaces)                    │  │
│  └─────────────────────────┬───────────────────────────┘  │
│                            │                               │
│                            ▼                               │
│       ┌────────────────────────────────────────┐          │
│       │              DOMAIN                     │          │
│       │  (Pure business logic, NO external     │          │
│       │   imports, fully testable)             │          │
│       └────────────────────────────────────────┘          │
└────────────────────────────────────────────────────────────┘
```

- Domain logic in `/domain` - NO external dependencies
- Interfaces in `/ports` - Technology-agnostic contracts
- Implementations in `/adapters` - Concrete technical code
- All dependencies point INWARD

**Actions**:
1. Create folder structure per architecture rules
2. Define port interfaces first
3. **Create fake implementations for testing**
4. Create stub implementations
5. Update `.claude/memory-bank/systemPatterns.md` with decisions

### Phase 4: Refinement (Specification-Driven Implementation)
**Objective**: Implement code driven by specifications, not the other way around.

**Testing Strategy** (Specification-Driven, NOT London School TDD):

```
┌─────────────────────────────────────────────────────────────┐
│                    SPECIFICATION PYRAMID                     │
│                                                              │
│            /\                                                │
│           /  \         E2E: Critical paths only              │
│          /────\                                              │
│         /      \       BDD: Executable specifications        │
│        /────────\                                            │
│       /          \     Black-Box: Behavior via ports         │
│      /────────────\                                          │
│     /              \   Property: Domain invariants           │
│    /────────────────\                                        │
└─────────────────────────────────────────────────────────────┘
```

**Implementation Order**:
1. **Property Tests** - Define invariants that must ALWAYS hold
2. **BDD Scenarios** - Write Gherkin before code
3. **Fakes** - Create working test implementations
4. **Domain Logic** - Implement to satisfy specifications
5. **Adapters** - Implement real dependencies
6. **Contract Tests** - Validate fakes match real implementations

**Test Principles**:
- Test WHAT (behavior), not HOW (implementation)
- Use **Fakes** for domain testing, not mocks
- Mocks ONLY at adapter boundaries (external APIs)
- Tests should survive refactoring

**Quality Gates**:
- No file > 500 lines
- No function > 50 lines
- No hard-coded secrets
- Property tests define all domain invariants
- BDD scenarios cover all acceptance criteria
- Contract tests validate all fakes
- 80%+ test coverage (secondary metric)

**Actions**:
1. Write property tests for domain invariants
2. Implement BDD step definitions using fakes
3. Implement domain logic to pass specifications
4. Implement adapters
5. Write contract tests (fake vs real)
6. Run linting and type checks
7. Commit with semantic message (feat:, test:, fix:)

### Phase 5: Completion
**Objective**: Finalize and prepare for deployment.

**Deliverables**:
- Passing test suite (properties, BDD, contracts, e2e)
- API documentation
- Deployment configuration
- Updated decision log

**Actions**:
1. Run full test suite
2. Run security scan
3. **Verify all property invariants hold**
4. **Verify all BDD scenarios pass**
5. **Verify all contract tests pass**
6. Generate/update documentation
7. Update `decisionLog.md` with final decisions
8. Create deployment checklist

## Test File Organization

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

## Parallel Tracks
When appropriate, work on multiple tracks concurrently:
- **Backend Track**: API implementation
- **Frontend Track**: UI implementation
- **Integration Track**: API contracts and e2e tests

Synchronization points:
- After Architecture phase (shared types)
- Before Completion (integration testing)

## Commands
- `/sparc-research` - Run Research phase (Phase 0, optional)
- `/sparc-spec` - Run Specification phase (Phase 1)
- `/sparc-pseudo` - Run Pseudocode phase (Phase 2)
- `/sparc-arch` - Run Architecture phase (Phase 3)
- `/sparc-refine` - Run Refinement phase (Phase 4)
- `/sparc-complete` - Run Completion phase (Phase 5)
- `/sparc-full` - Run all phases sequentially (with parallel execution support)

## Agents
- `orchestrator` - Coordinates multi-agent workflows and parallel execution
- `researcher` - Gathers external knowledge (Phase 0)
- `architect` - System design and architecture (Phases 1-3)
- `coder` - Implementation (Phase 4)
- `tester` - Testing and quality assurance (Phases 4-5)
- `reviewer` - Code review (Phase 5)
- `security-auditor` - Security analysis (Phase 5)

## Related Skills
- `hexagonal-architecture` - Ports & Adapters pattern implementation
- `tdd-workflow` - Specification-driven testing (Fakes over Mocks)
- `bdd-testing` - Gherkin/Cucumber workflow
- `property-testing` - Domain invariants with fast-check
- `metamorphic-testing` - Testing without oracle (AI-generated code validation)
- `arch-linting` - Architecture constraint enforcement

## Anti-Patterns to Avoid

### Testing Anti-Patterns
```typescript
// BAD: Mock-based interaction testing
expect(mockRepo.create).toHaveBeenCalledTimes(1)  // Tests HOW

// GOOD: Behavior verification
const found = await deps.userRepository.findByEmail(email)
expect(found).not.toBeNull()  // Tests WHAT
```

### Architecture Anti-Patterns
```typescript
// BAD: Domain importing from adapters
import { PostgresUserRepository } from '../adapters/db'

// GOOD: Domain depends only on ports
import type { UserRepository } from '../ports/user-repository'
```
