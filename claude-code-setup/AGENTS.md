# AGENTS.md

## Project Overview
Full-stack TypeScript application with Bun backend and Vue.js frontend, following hexagonal architecture and SPARC methodology.

## For AI Assistants

### Tech Stack
- Backend: Bun, Hono, TypeScript
- Frontend: Vue 3, Pinia, TypeScript
- Architecture: Hexagonal (Ports & Adapters)
- Testing: Vitest, Playwright, Cucumber

### Directory Structure
```
.claude/
├── agents/          # Specialized agent definitions
├── commands/        # Slash commands for workflows
├── memory-bank/     # Persistent context files
└── skills/          # Reusable knowledge/patterns

docs/                # Generated documentation
```

### Key Files to Read First
1. `CLAUDE.md` - Main configuration and rules
2. `.claude/memory-bank/projectBrief.md` - Project requirements
3. `.claude/memory-bank/techContext.md` - Technical details
4. `.claude/memory-bank/systemPatterns.md` - Architectural patterns

### SPARC Workflow
Follow the SPARC methodology for feature development:
1. Specification - Define requirements
2. Pseudocode - Design solution
3. Architecture - Structure components
4. Refinement - TDD implementation
5. Completion - Validation

### Architecture Rules (CRITICAL)
1. Domain NEVER imports from adapters
2. All external access through port interfaces
3. Dependency injection at startup
4. No file > 500 lines
5. No function > 50 lines

### Testing Requirements
- Write tests BEFORE implementation (TDD)
- Unit tests for domain logic
- Integration tests for adapters
- BDD scenarios for acceptance criteria
- Target 80%+ coverage

### Available Commands
- `/sparc-spec` - Specification phase
- `/sparc-arch` - Architecture phase
- `/sparc-refine` - TDD implementation
- `/sparc-complete` - Finalization
- `/sparc-full` - Full workflow

### Code Style
- TypeScript strict mode
- Named exports
- 2-space indentation
- Single quotes
- Trailing commas

### Commit Format
```
type: description

Types: feat, fix, test, docs, refactor, chore
```
