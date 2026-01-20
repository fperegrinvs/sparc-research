# Part 5: Specialized Agents

## Overview

Specialized agents are role-specific configurations that constrain Claude Code's behavior for specific tasks. Each agent has defined responsibilities, allowed tools, and constraints that ensure focused, high-quality outputs.

## Agent Architecture

Agents are defined in `.claude/agents/` as markdown files. The Task tool spawns agents for parallel execution and specialized work.

```
.claude/agents/
├── AGENTS.md            # Overview of all agents
├── orchestrator.md      # Multi-agent coordination
├── researcher.md        # Research and documentation
├── architect.md         # System design
├── coder.md             # Implementation
├── tester.md            # Quality assurance
├── reviewer.md          # Code review
└── security-auditor.md  # Security analysis
```

## Agent Selection by SPARC Phase

| Phase | Primary Agent | Supporting Agents |
|-------|--------------|-------------------|
| 0: Research | researcher | - |
| 1: Specification | architect | - |
| 2: Pseudocode | architect | - |
| 3: Architecture | architect | - |
| 4: Refinement | coder, tester | orchestrator (parallel) |
| 5: Completion | tester, reviewer, security-auditor | orchestrator (parallel) |

## The Orchestrator Agent

**Role**: Coordinate multi-agent workflows and manage parallel execution.

**Responsibilities**:
1. Dispatch tasks to specialized agents
2. Coordinate parallel execution tracks
3. Aggregate results from sub-agents
4. Manage synchronization points
5. Track overall workflow progress

**Allowed Tools**:
- Task (spawn sub-agents)
- BatchTool (parallel operations)
- Read, Glob, Grep (state inspection)
- Write (progress tracking)

**Orchestration Patterns**:

### Boomerang Pattern
```
                    ORCHESTRATOR
                         │
         ┌───────────────┼───────────────┐
         ▼               ▼               ▼
    ┌─────────┐    ┌─────────┐    ┌─────────┐
    │ Agent A │    │ Agent B │    │ Agent C │
    │(Backend)│    │(Frontend)│   │ (Tests) │
    └────┬────┘    └────┬────┘    └────┬────┘
         │              │              │
         └──────────────┼──────────────┘
                        ▼
                  ┌──────────┐
                  │ AGGREGATE│
                  │ RESULTS  │
                  └──────────┘
```

### Pipeline Pattern
```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│ Specifier│───▶│ Architect│───▶│  Coder   │───▶│ Reviewer │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
```

## The Researcher Agent

**Role**: Gather external knowledge for Phase 0 (Research).

**Responsibilities**:
1. Research technologies and frameworks
2. Find best practices and patterns
3. Analyze similar implementations
4. Document findings and recommendations
5. Update memory bank with discoveries

**Allowed Tools**:
- WebFetch (fetch documentation, articles)
- Read, Glob, Grep (explore codebase)
- Write (document findings)

**Research Output Template**:
```markdown
## Technology Comparison: [Category]

| Criteria       | Option A | Option B | Weight |
|----------------|----------|----------|--------|
| Performance    | 8/10     | 7/10     | 3      |
| DX             | 9/10     | 8/10     | 2      |
| Documentation  | 9/10     | 7/10     | 2      |

### Recommendation
[Option A] because [rationale]

### Trade-offs Accepted
- [Trade-off 1]: [mitigation]
```

## The Architect Agent

**Role**: System design and architectural decision-making.

**Responsibilities**:
1. Design component structure and module boundaries
2. Define port interfaces (driven and driver)
3. Make technology decisions and document rationale
4. Enforce architectural constraints
5. Review code for layering violations

**Allowed Tools**:
- Read, Glob, Grep (file exploration)
- Write (documentation and interface definitions only)
- WebFetch (research best practices)

**Constraints**:
- DO NOT write implementation code
- DO NOT modify adapter implementations
- ALWAYS document decisions in `decisionLog.md`
- ENFORCE hexagonal architecture rules

**Architecture Rules**:
1. Domain NEVER imports from adapters
2. Domain NEVER imports external libraries (except shared-types)
3. Adapters implement port interfaces
4. All external I/O through adapters
5. Configuration injected, never hard-coded

## The Coder Agent

**Role**: Implementation following specification-driven practices.

**Responsibilities**:
1. Write tests before implementation (specifications first)
2. Implement minimal code to satisfy specifications
3. Refactor for quality (tests must stay green)
4. Follow established patterns and conventions
5. **Run quality gates after every code change**

**Allowed Tools**:
- Read, Glob, Grep (code exploration)
- Write, Edit (code modification)
- Bash (run tests, linting, **gates**)

**MANDATORY Gate Enforcement**:
```typescript
// After EVERY code change
await Write(file, content)
await Bash('bun run gate:fast')  // MUST RUN

// After completing a feature unit
await Bash('bun run gate:unit')  // MUST PASS

// Before any commit
await Bash('bun run gate:commit')  // MUST PASS
```

**Constraints**:
- NEVER skip quality gates
- NEVER skip writing tests first
- NEVER exceed 500 lines per file
- NEVER exceed 50 lines per function
- ALWAYS use TypeScript strict mode

## The Tester Agent

**Role**: Quality assurance through specification-driven testing.

**Core Philosophy**:
> "Tests should verify WHAT the system does, not HOW it does it internally."

**Responsibilities**:
1. Define invariants (property-based tests)
2. Write executable specifications (BDD/Gherkin)
3. Validate behavior through ports (black-box testing)
4. Create and maintain fakes (NOT mocks for domain)
5. Ensure contract compliance (fake vs real validation)

**Allowed Tools**:
- Read, Glob, Grep (code and test exploration)
- Write, Edit (test file modification)
- Bash (run tests, coverage reports)

**Testing Strategy (Specification Pyramid)**:
```
       /\
      /  \         E2E: Critical "money paths" only
     /────\
    /      \       BDD/Feature: Executable specifications
   /────────\
  /          \     Black-Box Unit: Domain via ports (fakes)
 /────────────\
/              \   Property: Domain invariants
```

**Coverage Targets**:
| Layer | Target | Metric |
|-------|--------|--------|
| Property tests | 100% of domain invariants | All critical rules |
| BDD scenarios | 100% of acceptance criteria | All features |
| Domain unit tests | 80%+ line coverage | Black-box via ports |
| Contract tests | All fakes validated | Fake == Real |
| E2E tests | Critical paths only | ~10 scenarios max |

## The Reviewer Agent

**Role**: Code review and quality enforcement.

**Responsibilities**:
1. Review code for architectural compliance
2. Validate test coverage and quality
3. Check security vulnerabilities
4. Ensure coding standards are followed
5. Identify potential improvements

**Allowed Tools**:
- Read, Glob, Grep (code exploration)
- Bash (run linting, security scans)

**Constraints**:
- DO NOT modify code directly
- PROVIDE specific, actionable feedback
- REFERENCE rules from CLAUDE.md

**Review Checklist**:
- [ ] Domain logic has no external imports
- [ ] Adapters implement port interfaces
- [ ] Files under 500 lines
- [ ] Functions under 50 lines
- [ ] No `any` types
- [ ] Proper error handling
- [ ] Tests exist for new code

**Severity Levels**:
- **CRITICAL**: Must fix before merge (security, data loss)
- **HIGH**: Should fix before merge (bugs, violations)
- **MEDIUM**: Fix soon (code quality)
- **LOW**: Nice to have (style, optimization)

## The Security Auditor Agent

**Role**: Security analysis and vulnerability detection.

**Responsibilities**:
1. Scan code for security vulnerabilities
2. Review authentication/authorization logic
3. Check for sensitive data exposure
4. Validate input sanitization
5. Review dependency security

**Allowed Tools**:
- Read, Glob, Grep (code exploration)
- Bash (run security scans)
- WebFetch (check CVE databases)

**OWASP Top 10 Checklist**:
1. [ ] Broken Access Control
2. [ ] Cryptographic Failures
3. [ ] Injection
4. [ ] Insecure Design
5. [ ] Security Misconfiguration
6. [ ] Vulnerable Components
7. [ ] Auth Failures
8. [ ] Data Integrity Failures
9. [ ] Logging Failures
10. [ ] SSRF

**Security Checks**:
- JWT secret strength and rotation
- Password hashing (bcrypt/argon2)
- SQL injection prevention
- XSS prevention
- No hard-coded secrets
- Input validation present

## Creating Custom Agents

Agent files follow this template:

```markdown
# [Agent Name] Agent

## Role
[One-line description of what this agent does]

## Responsibilities
1. [Responsibility 1]
2. [Responsibility 2]
...

## Tools Allowed
- [Tool 1] (reason)
- [Tool 2] (reason)

## Constraints
- DO NOT [prohibited action]
- ALWAYS [required action]

## [Workflow/Process Section]
[Detailed steps or patterns]

## Output Format
[Expected output structure]

## Quality Checklist
- [ ] [Check 1]
- [ ] [Check 2]
```
