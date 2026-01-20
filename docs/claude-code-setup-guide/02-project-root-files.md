# Part 2: Project Root Configuration Files

## Overview

The project root contains two critical files that Claude Code automatically reads when starting a session:

- **CLAUDE.md** - Primary configuration and rules
- **AGENTS.md** - Quick reference for AI assistants

## CLAUDE.md: The Project Constitution

CLAUDE.md is the authoritative source of truth for Claude Code. It should be concise, concrete, and actionable. LLMs have limited instruction-following capacity, so every word matters.

### Best Practices for CLAUDE.md

1. **Be Specific**: "Use 2-space indentation" not "format code properly"
2. **Keep It Concise**: Aim for essential information only
3. **Use Concrete Examples**: Show patterns, not just describe them
4. **Prioritize Information**: Most critical rules first
5. **Reference External Docs**: Use imports (`@docs/auth.md`) rather than duplicating

### CLAUDE.md Structure

```markdown
# Project: [Project Name]

## Tech Stack
- **Runtime**: [e.g., Bun 1.x]
- **Backend**: [Framework, language]
- **Frontend**: [Framework, language]
- **Architecture**: [Pattern name]
- **Testing**: [Frameworks]
- **Package Manager**: [Tool]

## Key Commands
```bash
# Development
bun run dev              # Start all services
bun run dev:api          # Start API only

# Testing
bun run test             # Run all tests
bun run test:unit        # Unit tests only

# Build & Quality
bun run build            # Build all packages
bun run lint             # Linting
bun run typecheck        # Type checking
```

## Directory Structure
```
apps/
  api/                   # Backend
    src/
      domain/            # Pure business logic
      ports/             # Interfaces
      adapters/          # Implementations
  web/                   # Frontend
packages/
  shared-types/          # Shared TypeScript types
  shared-utils/          # Shared utilities
```

## Architecture Rules (STRICTLY ENFORCED)
1. **Domain Isolation**: `domain/` NEVER imports from `adapters/`
2. **Port Contracts**: All external communication through `ports/` interfaces
3. **Dependency Injection**: Adapters injected at startup
4. **No File > 500 lines**: Split large files
5. **No Function > 50 lines**: Keep functions small

## Code Style
- TypeScript strict mode enabled
- Named exports preferred
- 2-space indentation
- Single quotes for strings
- Trailing commas in multi-line

## Testing Strategy
- Write tests BEFORE implementation
- Test WHAT, not HOW (behavior, not implementation)
- Use Fakes, not Mocks for domain testing
- Target 80%+ coverage

## Git Commit Format
- `feat:` New features
- `fix:` Bug fixes
- `test:` Test additions
- `docs:` Documentation
- `refactor:` Code restructuring

## Quality Gates (MANDATORY)
```bash
bun run gate:fast    # After EVERY code change
bun run gate:unit    # After each feature unit
bun run gate:commit  # Before EVERY commit
bun run gate:full    # Before PR/merge
```

## Agents
- `orchestrator` - Coordinates workflows
- `architect` - System design
- `coder` - Implementation
- `tester` - Quality assurance
- `reviewer` - Code review
- `security-auditor` - Security analysis

## Important Notes
- NEVER commit `.env` files or secrets
- Run `bun run lint && bun run test` before committing
```

## AGENTS.md: Quick Reference for AI Assistants

AGENTS.md provides a shorter overview specifically for AI agents, pointing them to detailed resources.

### AGENTS.md Structure

```markdown
# AGENTS.md

## Project Overview
[One-line description of the project]

## For AI Assistants

### Tech Stack
[Brief list of technologies]

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

### Workflow
Follow the SPARC methodology:
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

### Available Commands
- `/sparc-full` - Full workflow
- `/sparc-spec` - Specification phase
- `/sparc-refine` - TDD implementation

### Code Style
- TypeScript strict mode
- Named exports
- 2-space indentation
- Single quotes

### Commit Format
```
type: description
Types: feat, fix, test, docs, refactor, chore
```
```

## File Location Strategy

Both files should be at the project root:

```
project-root/
├── CLAUDE.md          # Primary configuration
├── AGENTS.md          # AI quick reference
├── .claude/           # Detailed configurations
└── ...
```

Claude Code will automatically read CLAUDE.md when starting a session. AGENTS.md serves as supplementary context.
