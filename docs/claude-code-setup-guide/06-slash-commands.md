# Part 6: Slash Commands

## Overview

Slash commands are reusable workflows defined in `.claude/commands/`. They provide standardized entry points for common development tasks, ensuring consistency and enforcing quality gates.

## Command Structure

Commands are markdown files that define:
- **Trigger**: When to use this command
- **Prerequisites**: What must exist before running
- **Inputs/Outputs**: Files consumed and produced
- **Steps**: What the command does
- **Quality Gates**: Verification requirements

## The SPARC Commands

### /sparc-full

The complete SPARC workflow command.

```markdown
# /sparc-full - Full SPARC Workflow

## Trigger
Run to execute all SPARC phases for a complete feature.

## Prerequisites
- Clear feature request or requirements
- Project memory bank initialized
- Development environment ready

## Flags
- `--skip-research` - Skip Phase 0
- `--auto` - Autonomous mode (no pauses)
- `--parallel` - Enable parallel execution

## Execution Flow
Phase 0: Research (optional) → /sparc-research
Phase 1: Specification       → /sparc-spec
Phase 2: Pseudocode          → /sparc-pseudo
Phase 3: Architecture        → /sparc-arch
Phase 4: Refinement          → /sparc-refine
Phase 5: Completion          → /sparc-complete

## Quality Gates
| Phase | Gate | Must Pass |
|-------|------|-----------|
| Specification | Scenarios defined | All acceptance criteria as Gherkin |
| Refinement | Tests pass | 100% property, 100% BDD, 80% coverage |
| Completion | All audits pass | Security, review, E2E |
```

### /sparc-spec

Specification phase command.

```markdown
# /sparc-spec - Specification Phase

## Trigger
Run to analyze requirements and create specifications.

## Inputs
- `projectBrief.md` - Project requirements
- User story or feature request

## Outputs
- `docs/specification.md` - Requirements document
- `tests/features/*.feature` - BDD scenarios
- Property invariants documented

## Process
1. Read project brief and requirements
2. Identify:
   - Core features and behaviors
   - Edge cases and error conditions
   - Integration points
3. Write user stories with acceptance criteria
4. Create BDD/Gherkin scenarios
5. Define domain invariants for property testing
6. Document in specification.md
```

### /sparc-pseudo

Pseudocode design phase.

```markdown
# /sparc-pseudo - Pseudocode Phase

## Trigger
Run after specification to design the solution.

## Inputs
- `docs/specification.md`
- BDD scenarios

## Outputs
- `docs/pseudocode.md`
- Algorithm outlines
- Function signatures
- Test strategy plan

## Process
1. Review specification
2. Design solution approach for each requirement
3. Define function/method signatures
4. Plan data flows
5. Identify fakes needed for testing
6. Document pseudocode
```

### /sparc-arch

Architecture design phase.

```markdown
# /sparc-arch - Architecture Phase

## Trigger
Run after pseudocode to create system structure.

## Inputs
- `docs/specification.md`
- `docs/pseudocode.md`

## Outputs
- Folder structure
- Port interfaces
- Fake implementations
- Database schema (if applicable)

## Process
1. Create hexagonal folder structure
2. Define port interfaces (driven and driver)
3. Create stub implementations
4. Create fake implementations for testing
5. Design database schema
6. Update systemPatterns.md
```

### /sparc-refine

Implementation phase with specification-driven testing.

```markdown
# /sparc-refine - Refinement Phase

## Trigger
Run after architecture to implement the solution.

## Inputs
- `docs/specification.md`
- `docs/architecture.md`
- Port interface definitions
- BDD feature scenarios

## MANDATORY: Quality Gate Enforcement

Every code change MUST pass gates:

After EVERY Write/Edit:  bun run gate:fast
After each feature unit: bun run gate:unit
Before any commit:       bun run gate:commit

## Implementation Process

1. Define Property Invariants
   - Domain rules that must ALWAYS hold
   - Write property tests first

2. Write BDD Scenarios (Before Code)
   - Gherkin files as specifications
   - Step definitions using fakes

3. Create Fakes for Dependencies
   - In-memory implementations
   - Working test doubles

4. Write Black-Box Unit Tests
   - Test through ports
   - Use fakes, NOT mocks

5. Implement Domain Logic
   - Minimal code to satisfy specs
   - Pure business logic

6. Implement Adapters
   - HTTP handlers
   - Database implementations

7. Contract Tests
   - Validate fakes match real implementations
```

### /sparc-complete

Completion and validation phase.

```markdown
# /sparc-complete - Completion Phase

## Trigger
Run after implementation to validate and finalize.

## Inputs
- Implementation code
- All tests

## Outputs
- Verified, documented code
- Security audit report
- Code review summary
- API documentation

## Process
1. Run full test suite
2. Run security scan
3. Code review
4. Generate documentation
5. Update decisionLog.md
6. Run gate:full

## Parallel Audits
- Code Review (reviewer agent)
- Security Scan (security-auditor agent)
- E2E Tests (tester agent)
```

### /sparc-research

Research phase command.

```markdown
# /sparc-research - Research Phase

## Trigger
Run when external knowledge is needed.

## Outputs
- Technology documentation summaries
- Best practices research
- Decision rationale
- Updated techContext.md

## Process
1. Identify research topics
2. Fetch official documentation
3. Research best practices
4. Analyze similar implementations
5. Synthesize findings
6. Update memory bank
```

## Command File Template

Use this template for creating new commands:

```markdown
# /command-name - [Brief Description]

## Trigger
[When to use this command]

## Prerequisites
[What must exist before running]

## Flags
- `--flag-name` - [Description]

## Inputs
[Files or context consumed]

## Outputs
[Files or artifacts produced]

## Process
1. [Step 1]
2. [Step 2]
...

## Quality Gates
[Verification requirements]

## Error Recovery
[What to do if something fails]

## Examples
```bash
/command-name [arguments]
```
```

## Invoking Commands

Commands are invoked by typing their name:

```
User: /sparc-full Implement user authentication

Claude: Starting SPARC workflow...

## Phase 1: Specification
[Analyzes requirements, creates spec]

Ready to proceed to Pseudocode? [Y/n]
```

With flags:
```
User: /sparc-full --auto --parallel Implement checkout flow

Claude: Running SPARC in autonomous mode with parallel tracks...
```

## Command Best Practices

1. **Keep commands focused**: One clear purpose per command
2. **Define clear inputs/outputs**: What goes in, what comes out
3. **Include quality gates**: Verification at key checkpoints
4. **Document error recovery**: What to do when things fail
5. **Provide examples**: Show typical usage patterns
