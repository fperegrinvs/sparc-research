# /sparc-full - Full SPARC Workflow

## Trigger
Run to execute all SPARC phases sequentially for a complete feature.

## Overview
This command orchestrates the full SPARC methodology:

```
Phase 0: Research (optional)     → /sparc-research
Phase 1: Specification           → /sparc-spec
Phase 2: Pseudocode              → /sparc-pseudo
Phase 3: Architecture            → /sparc-arch
Phase 4: Refinement              → /sparc-refine
Phase 5: Completion              → /sparc-complete
```

## Prerequisites
- Clear feature request or requirements
- Project memory bank initialized (`.claude/memory-bank/`)
- Development environment ready

## Flags
- `--skip-research` - Skip Phase 0 (research)
- `--auto` - Run without human checkpoints (autonomous mode)
- `--parallel` - Enable parallel execution for Phase 4

## MANDATORY: Quality Gate Enforcement

**Gates run automatically at every checkpoint. Cannot be bypassed.**

```
┌─────────────────────────────────────────────────────────────────┐
│  GATE LEVELS (enforced throughout workflow)                    │
├─────────────────────────────────────────────────────────────────┤
│  gate:fast    → After EVERY code change (< 10s)               │
│  gate:unit    → After each feature unit completion            │
│  gate:commit  → Before EVERY commit (blocks on failure)       │
│  gate:full    → Before PR/merge (runs in CI)                  │
└─────────────────────────────────────────────────────────────────┘
```

See `.claude/skills/quality-gates.md` for implementation details.

## Execution Flow

### Phase 0: Research (Optional)
**Agent**: `researcher`
**Command**: `/sparc-research`

- Gather documentation and best practices
- Research similar implementations
- Document technology decisions
- **Output**: `docs/research.md`, updated memory bank
- **Skip**: Use `--skip-research` if requirements are clear

### Phase 1: Specification
**Agent**: `architect` (or `specifier` if available)
**Command**: `/sparc-spec`

- Analyze requirements from `projectBrief.md`
- Write user stories with acceptance criteria
- Create BDD/Gherkin scenarios
- Define domain invariants for property testing
- **Output**: `docs/specification.md`, `tests/features/*.feature`
- **Checkpoint**: Review spec before continuing

### Phase 2: Pseudocode
**Agent**: `architect`
**Command**: `/sparc-pseudo`

- Design solution approach (algorithms, data flows)
- Define function/method signatures
- Plan test strategy (properties, BDD, contracts)
- Identify fakes needed for testing
- **Output**: `docs/pseudocode.md`
- **Checkpoint**: Review design before continuing

### Phase 3: Architecture
**Agent**: `architect`
**Command**: `/sparc-arch`

- Create hexagonal folder structure
- Define port interfaces
- Create stub/fake implementations
- Design database schema (if applicable)
- **Output**: Folder structure, port interfaces, fakes
- **Checkpoint**: Review architecture before continuing

### Phase 4: Refinement (Parallel Capable)
**Agents**: `coder`, `tester`
**Command**: `/sparc-refine`

- Implement using specification-driven approach
- Write property tests for invariants
- Implement BDD step definitions
- Create domain logic and adapters
- Write contract tests (fake vs real)
- **Output**: Implementation code, tests
- **Gate Enforcement**:
  - `gate:fast` after EVERY Write/Edit
  - `gate:unit` after each use case/component
  - `gate:commit` before any commit
- **Checkpoint**: All tests passing (verified by gates)

### Phase 5: Completion
**Agents**: `tester`, `reviewer`, `security-auditor`
**Command**: `/sparc-complete`

- Run full test suite
- Code review
- Security audit
- Generate documentation
- **Output**: Verified, documented, deployable code
- **Gate Enforcement**: `gate:full` MUST pass
- **Final checkpoint**: Ready for deployment (verified by CI)

## Parallel Execution (Boomerang Pattern)

### When to Use
Enable with `--parallel` flag for full-stack features where backend and frontend can be developed independently.

### Parallel Workflow Diagram
```
┌─────────────────────────────────────────────────────────────┐
│                 Phase 0: Research (optional)                 │
│                    Agent: researcher                         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   Phase 1: Specification                     │
│                    Agent: architect                          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Phase 2: Pseudocode                       │
│                    Agent: architect                          │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   Phase 3: Architecture                      │
│                    Agent: architect                          │
│          (Defines shared types, port interfaces)             │
└─────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              │     SYNC POINT: Shared        │
              │     types defined             │
              └───────────────┬───────────────┘
                              │
          ┌───────────────────┼───────────────────┐
          ▼                   │                   ▼
┌─────────────────┐           │         ┌─────────────────┐
│  Backend Track  │           │         │ Frontend Track  │
│  Agent: coder   │           │         │  Agent: coder   │
│                 │           │         │                 │
│  - Domain logic │           │         │  - Components   │
│  - API routes   │           │         │  - State mgmt   │
│  - DB adapters  │           │         │  - API client   │
│  - Unit tests   │           │         │  - Unit tests   │
└────────┬────────┘           │         └────────┬────────┘
         │                    │                  │
         │      ┌─────────────┴─────────────┐    │
         │      │   Test Track (parallel)   │    │
         │      │     Agent: tester         │    │
         │      │                           │    │
         │      │   - Property tests        │    │
         │      │   - Contract tests        │    │
         │      │   - BDD scenarios         │    │
         │      └─────────────┬─────────────┘    │
         │                    │                  │
         └────────────────────┼──────────────────┘
                              │
              ┌───────────────┴───────────────┐
              │     SYNC POINT: All tracks    │
              │     complete                  │
              └───────────────┬───────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Integration Tests                         │
│                     Agent: tester                            │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   Phase 5: Completion                        │
│            Agents: reviewer, security-auditor                │
│                                                              │
│     ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│     │  Code Review │  │Security Scan │  │  E2E Tests   │   │
│     │   (parallel) │  │  (parallel)  │  │  (parallel)  │   │
│     └──────────────┘  └──────────────┘  └──────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │  Deploy Ready   │
                    └─────────────────┘
```

### Synchronization Points

| Sync Point | Required Tracks | Action |
|------------|-----------------|--------|
| `post-architecture` | architecture | Shared types must be defined |
| `pre-integration` | backend, frontend, tests | All unit tests passing |
| `pre-completion` | integration tests | Integration verified |

### Using BatchTool for Parallel Execution
```typescript
// Orchestrator dispatches parallel tracks
const results = await BatchTool([
  Task({ agent: 'coder', task: 'implement-backend' }),
  Task({ agent: 'coder', task: 'implement-frontend' }),
  Task({ agent: 'tester', task: 'write-property-tests' }),
])

// Wait for sync point
await waitForSync('pre-integration')

// Run integration tests
await Task({ agent: 'tester', task: 'integration-tests' })
```

## Usage Examples

### Standard Workflow (Interactive)
```
User: /sparc-full Implement user authentication feature

Claude: Starting SPARC workflow...

## Phase 1: Specification
[Analyzes requirements, creates spec]

Ready to proceed to Pseudocode? [Y/n]
```

### Autonomous Mode
```
User: /sparc-full --auto Implement user authentication

Claude: Running SPARC in autonomous mode...
[Executes all phases without pauses]
[Self-corrects if tests fail]
[Completes when all quality gates pass]
```

### With Parallel Execution
```
User: /sparc-full --parallel Implement checkout flow

Claude: Running SPARC with parallel tracks...
[Spec, Pseudo, Arch run sequentially]
[Backend, Frontend, Tests run in parallel]
[Integration and Completion run after sync]
```

### Skip Research
```
User: /sparc-full --skip-research Add logging middleware

Claude: Skipping research phase...
Starting with Specification...
```

## Quality Gates (Per Phase)

| Phase | Gate | Must Pass |
|-------|------|-----------|
| Specification | Scenarios defined | All acceptance criteria as Gherkin |
| Pseudocode | Invariants identified | Property tests planned |
| Architecture | Ports defined | All interfaces documented |
| Refinement | Tests pass | 100% property, 100% BDD, 80% coverage |
| Completion | All audits pass | Security, review, E2E |

## Error Recovery

If a phase fails:
1. **Interactive mode**: Pause and report error
2. **Auto mode**: Attempt self-correction up to 3 times
3. If still failing: Pause and request human intervention

## Commit Strategy

Commits are made at logical checkpoints:
```
feat(spec): Add user authentication specification
feat(arch): Add authentication ports and adapters scaffold
feat(domain): Implement user registration use case
test(auth): Add authentication property and BDD tests
feat(api): Implement authentication HTTP adapter
docs(auth): Add API documentation for auth endpoints
```

## Next Steps After Completion
- Create pull request
- Deploy to staging
- Run smoke tests
- Monitor for issues
