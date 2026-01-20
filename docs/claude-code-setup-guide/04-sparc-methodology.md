# Part 4: The SPARC Methodology

## Overview

SPARC (Specification, Pseudocode, Architecture, Refinement, Completion) is a structured workflow for AI-assisted software development that emphasizes **specification-driven testing** and **hexagonal architecture**.

The core philosophy:

> "Think before coding. Specify before implementing. Test behavior, not implementation."

## The SPARC Phases

```
Phase 0: Research (Optional)     → Gather knowledge before designing
Phase 1: Specification           → Define WHAT the system does
Phase 2: Pseudocode              → Design HOW to solve it
Phase 3: Architecture            → Structure WHERE code lives
Phase 4: Refinement              → Implement driven by specs
Phase 5: Completion              → Verify, audit, document
```

## Phase 0: Research (Optional)

**Objective**: Gather external knowledge to inform design decisions.

**When to Use**: Run when you need information about technologies, frameworks, or best practices. Skip with `--skip-research` if requirements are clear.

**Agent**: `researcher`

**Deliverables**:
- Technology documentation summaries
- Best practices and patterns research
- Similar implementation analysis
- Decision rationale documentation

**Actions**:
1. Identify research topics from project brief
2. Fetch official documentation and tutorials
3. Search for best practices and patterns
4. Synthesize findings into decision rationale
5. Update memory bank with technology decisions

## Phase 1: Specification

**Objective**: Define clear, testable requirements before any code.

**Agent**: `architect`

**Deliverables**:
- Functional requirements document
- Non-functional requirements (performance, security)
- User stories with acceptance criteria
- **BDD/Gherkin scenarios** (executable specifications)
- **Domain invariants** (properties that must ALWAYS hold)

**Actions**:
1. Read project brief from memory bank
2. Analyze requirements and extract:
   - Core features and behaviors
   - Edge cases and error conditions
   - Integration points
3. Document specification in `/docs/specification.md`
4. **Write Gherkin scenarios BEFORE implementation**
5. **Define property invariants for domain logic**

**Example Gherkin Scenario**:
```gherkin
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

## Phase 2: Pseudocode

**Objective**: Create high-level solution design without implementation.

**Agent**: `architect`

**Deliverables**:
- Algorithm outlines for core logic
- Data flow diagrams (text-based)
- Function/method signatures
- Test strategy outline (properties, BDD, contracts)

**Actions**:
1. Review specification from Phase 1
2. Design solution approach for each requirement
3. Document pseudocode in `/docs/pseudocode.md`
4. **Identify domain invariants for property testing**
5. **Plan fake implementations for dependencies**

## Phase 3: Architecture

**Objective**: Design system structure following hexagonal architecture.

**Agent**: `architect`

**Deliverables**:
- Component/module definitions
- Port interfaces (driven and driver)
- Data models and schemas
- File structure with stubs
- **Fake implementations for testing**

**Hexagonal Architecture Structure**:
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

**Key Rules**:
- Domain NEVER imports from adapters
- Domain NEVER imports external libraries (except shared types)
- All dependencies point INWARD
- Adapters implement port interfaces

## Phase 4: Refinement

**Objective**: Implement code driven by specifications, not the other way around.

**Agents**: `coder`, `tester`

**Implementation Order**:
1. **Property Tests** - Define invariants that must ALWAYS hold
2. **BDD Scenarios** - Write Gherkin before code
3. **Fakes** - Create working test implementations
4. **Domain Logic** - Implement to satisfy specifications
5. **Adapters** - Implement real dependencies
6. **Contract Tests** - Validate fakes match real implementations

**Quality Gates** (MANDATORY):
```bash
gate:fast    → After EVERY code change (< 10s)
gate:unit    → After each feature unit completion
gate:commit  → Before EVERY commit
```

**Test Principles**:
- Test WHAT (behavior), not HOW (implementation)
- Use **Fakes** for domain testing, not mocks
- Mocks ONLY at adapter boundaries (external APIs)
- Tests should survive refactoring

## Phase 5: Completion

**Objective**: Finalize and prepare for deployment.

**Agents**: `tester`, `reviewer`, `security-auditor`

**Deliverables**:
- Passing test suite (properties, BDD, contracts, E2E)
- API documentation
- Deployment configuration
- Updated decision log

**Actions**:
1. Run full test suite
2. Run security scan
3. Verify all property invariants hold
4. Verify all BDD scenarios pass
5. Verify all contract tests pass
6. Generate/update documentation
7. Update decisionLog.md
8. Create deployment checklist

## Running SPARC Workflows

### Full Workflow (Interactive)
```bash
/sparc-full Implement user authentication
```

### Autonomous Mode
```bash
/sparc-full --auto Implement user authentication
```

### With Parallel Execution
```bash
/sparc-full --parallel Implement checkout flow
```

### Individual Phases
```bash
/sparc-research    # Phase 0
/sparc-spec        # Phase 1
/sparc-pseudo      # Phase 2
/sparc-arch        # Phase 3
/sparc-refine      # Phase 4
/sparc-complete    # Phase 5
```

## Parallel Execution (Boomerang Pattern)

For full-stack features, Phase 4 can run parallel tracks:

```
┌──────────────────┐
│  Backend Track   │
│  (domain, API)   │
└────────┬─────────┘
         │
         │         ┌──────────────────┐
         ├────────►│  Frontend Track  │
         │         │  (UI, state)     │
         │         └────────┬─────────┘
         │                  │
         │         ┌────────┴─────────┐
         └────────►│   Test Track     │
                   │  (properties,    │
                   │   BDD, contracts)│
                   └────────┬─────────┘
                            │
                   ┌────────▼─────────┐
                   │  SYNC: Integrate │
                   └──────────────────┘
```

Enable with `--parallel` flag:
```bash
/sparc-full --parallel Implement checkout flow
```
