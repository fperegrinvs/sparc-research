# Agents Overview

This directory contains specialized agent definitions for the SPARC methodology workflow.

## Agent Roles

### Orchestrator (`orchestrator.md`)
**Purpose**: Coordinates multi-agent workflows and parallel execution.
**Key Responsibilities**:
- Decompose complex tasks into parallel tracks
- Manage synchronization points between agents
- Implement boomerang pattern for parallel execution
- Monitor progress and handle failures

### Researcher (`researcher.md`)
**Purpose**: Gathers external knowledge for Phase 0 (Research).
**Key Responsibilities**:
- Fetch official documentation and tutorials
- Research best practices and patterns
- Analyze similar implementations
- Document technology decisions with rationale

### Architect (`architect.md`)
**Purpose**: System design and architecture (Phases 1-3).
**Key Responsibilities**:
- Analyze requirements and create specifications
- Design algorithms and data flows (pseudocode)
- Define hexagonal architecture structure
- Create port interfaces and fake implementations

### Coder (`coder.md`)
**Purpose**: Implementation following specifications (Phase 4).
**Key Responsibilities**:
- Implement domain logic to satisfy specifications
- Create adapters for external dependencies
- Follow hexagonal architecture strictly
- Write clean, maintainable code

### Tester (`tester.md`)
**Purpose**: Testing and quality assurance (Phases 4-5).
**Key Responsibilities**:
- Write property-based tests for domain invariants
- Implement BDD step definitions
- Create contract tests (fake vs real)
- Validate all specifications are covered

### Reviewer (`reviewer.md`)
**Purpose**: Code quality and standards review (Phase 5).
**Key Responsibilities**:
- Review code for quality and maintainability
- Check adherence to architecture rules
- Verify testing coverage and quality
- Suggest improvements

### Security Auditor (`security-auditor.md`)
**Purpose**: Security vulnerability analysis (Phase 5).
**Key Responsibilities**:
- Analyze code for security vulnerabilities
- Check for common security anti-patterns
- Verify authentication and authorization
- Review data handling practices

## Agent Selection by Phase

| Phase | Primary Agent | Supporting Agents |
|-------|--------------|-------------------|
| 0: Research | researcher | - |
| 1: Specification | architect | - |
| 2: Pseudocode | architect | - |
| 3: Architecture | architect | - |
| 4: Refinement | coder, tester | orchestrator (parallel) |
| 5: Completion | tester, reviewer, security-auditor | orchestrator (parallel) |

## Parallel Execution

When using `--parallel` flag, the orchestrator coordinates:

**Phase 4 Parallel Tracks**:
- Backend Track (coder) - Domain logic, API routes, DB adapters
- Frontend Track (coder) - Components, state management, API client
- Test Track (tester) - Property tests, contract tests, BDD scenarios

**Phase 5 Parallel Audits**:
- Code Review (reviewer)
- Security Scan (security-auditor)
- E2E Tests (tester)

## Usage

Agents are invoked via the Task tool or within SPARC commands:

```bash
# Individual phase (implicitly uses appropriate agent)
/sparc-spec          # Uses: architect
/sparc-refine        # Uses: coder, tester

# Full workflow with orchestration
/sparc-full --parallel   # Uses: orchestrator to coordinate all agents
```
