# /sparc-full - Full SPARC Workflow

## Trigger
Run to execute all SPARC phases sequentially for a complete feature.

## Overview
This command orchestrates the full SPARC methodology:
1. Specification
2. Pseudocode
3. Architecture
4. Refinement (TDD)
5. Completion

## Prerequisites
- Clear feature request or requirements
- Project memory bank initialized
- Development environment ready

## Execution Flow

### Phase 1: Specification
- Analyze requirements
- Write user stories
- Define acceptance criteria
- Create Gherkin scenarios
- **Checkpoint**: Review spec before continuing

### Phase 2: Pseudocode
- Design solution approach
- Outline algorithms
- Plan data structures
- Define test strategy
- **Checkpoint**: Review design before continuing

### Phase 3: Architecture
- Design component structure
- Define port interfaces
- Plan adapter implementations
- Create folder scaffold
- **Checkpoint**: Review architecture before continuing

### Phase 4: Refinement
- Implement using TDD cycle
- Write tests first (RED)
- Implement code (GREEN)
- Refactor (REFACTOR)
- **Checkpoint**: All tests passing

### Phase 5: Completion
- Run full test suite
- Validate quality gates
- Security audit
- Complete documentation
- **Final checkpoint**: Ready for deployment

## Parallel Tracks (Optional)
When implementing full-stack features:
```
┌─────────────────────────────────────────────┐
│              Specification                   │
└─────────────────────────────────────────────┘
                    │
┌─────────────────────────────────────────────┐
│               Pseudocode                     │
└─────────────────────────────────────────────┘
                    │
┌─────────────────────────────────────────────┐
│               Architecture                   │
└─────────────────────────────────────────────┘
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
┌───────────────┐       ┌───────────────┐
│  Backend TDD  │       │  Frontend TDD │
└───────────────┘       └───────────────┘
        │                       │
        └───────────┬───────────┘
                    ▼
┌─────────────────────────────────────────────┐
│     Integration & Completion                 │
└─────────────────────────────────────────────┘
```

## Usage
```
User: /sparc-full Implement user authentication feature

Claude: Starting SPARC workflow...

## Phase 1: Specification
[Analyzes requirements, creates spec]

Ready to proceed to Pseudocode? [Y/n]
```

## Human Checkpoints
By default, pause at each phase for human review.
Use `/sparc-full --auto` to run without pauses (for experienced users).
