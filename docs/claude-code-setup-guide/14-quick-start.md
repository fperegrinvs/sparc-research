# Part 14: Quick Start Guide

## Overview

This quick start guide helps you set up a new project for optimal Claude Code effectiveness in under 30 minutes.

## Step 1: Create Directory Structure (2 minutes)

```bash
mkdir -p .claude/{agents,commands,memory-bank,skills}
touch CLAUDE.md AGENTS.md .mcp.json
```

## Step 2: Create CLAUDE.md (5 minutes)

Copy and customize this template:

```markdown
# Project: [Your Project Name]

## Tech Stack
- **Runtime**: [e.g., Bun 1.x, Node 20]
- **Backend**: [e.g., Hono, Express]
- **Frontend**: [e.g., Vue 3, React]
- **Testing**: [e.g., Vitest, Playwright]

## Key Commands
```bash
# Development
bun run dev              # Start all services

# Testing
bun run test             # Run all tests
bun run test:unit        # Unit tests

# Quality
bun run lint             # Linting
bun run typecheck        # Type checking
```

## Directory Structure
```
src/
  domain/            # Business logic
  ports/             # Interfaces
  adapters/          # Implementations
tests/
  unit/              # Unit tests
  features/          # BDD tests
```

## Architecture Rules
1. Domain NEVER imports from adapters
2. All external access through port interfaces
3. No file > 500 lines
4. No function > 50 lines

## Code Style
- TypeScript strict mode
- Named exports
- 2-space indentation

## Quality Gates (MANDATORY)
```bash
bun run gate:fast    # After EVERY code change
bun run gate:commit  # Before EVERY commit
```
```

## Step 3: Create Memory Bank (5 minutes)

### projectBrief.md
```markdown
# Project Brief

## Overview
[One paragraph describing your project]

## Core Requirements
1. [Requirement 1]
2. [Requirement 2]

## Success Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]
```

### techContext.md
```markdown
# Technical Context

## Runtime
- [Runtime]: [Version]

## Key Libraries
- [Library 1]: [Purpose]
- [Library 2]: [Purpose]

## Environment
```
NODE_ENV=development
PORT=3000
```
```

### systemPatterns.md
```markdown
# System Patterns

## Architecture
[Describe your architecture pattern]

## Testing
- Fakes for domain testing
- Mocks only at adapter boundaries

## Error Handling
[Describe error handling pattern]
```

## Step 4: Create Core Agents (5 minutes)

### .claude/agents/coder.md
```markdown
# Coder Agent

## Role
Implementation following specification-driven practices.

## Tools Allowed
- Read, Glob, Grep (code exploration)
- Write, Edit (code modification)
- Bash (run tests, linting)

## Constraints
- NEVER skip quality gates
- NEVER exceed 500 lines per file
- ALWAYS use TypeScript strict mode

## After EVERY Code Change
```bash
bun run gate:fast  # MUST PASS
```
```

### .claude/agents/tester.md
```markdown
# Tester Agent

## Role
Quality assurance through specification-driven testing.

## Philosophy
Test WHAT the system does, not HOW it does it.

## Tools Allowed
- Read, Glob, Grep (exploration)
- Write, Edit (test files)
- Bash (run tests)

## Test Priorities
1. Property tests (invariants)
2. BDD scenarios (specifications)
3. Black-box unit tests (via fakes)
```

## Step 5: Create Quality Gates Scripts (5 minutes)

Add to `package.json`:

```json
{
  "scripts": {
    "gate:fast": "bun run typecheck && bun run lint",
    "gate:unit": "bun run gate:fast && bun test --changed",
    "gate:commit": "bun run gate:unit && bun test",
    "gate:full": "bun run gate:commit && bun run test:e2e",

    "typecheck": "tsc --noEmit",
    "lint": "eslint . --ext .ts",

    "test": "vitest run",
    "test:unit": "vitest run tests/unit",
    "test:e2e": "playwright test"
  }
}
```

## Step 6: Create Git Hooks (3 minutes)

```bash
# Install husky
bun add -D husky
bunx husky init

# Create pre-commit hook
echo '#!/bin/sh
bun run gate:commit' > .husky/pre-commit
chmod +x .husky/pre-commit
```

## Step 7: Create .mcp.json (2 minutes)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "settings": {
    "allowedTools": [
      "Read",
      "Write",
      "Edit",
      "Glob",
      "Grep",
      "Bash",
      "Task"
    ]
  }
}
```

## Step 8: Create Settings (3 minutes)

### .claude/settings.json
```json
{
  "permissions": {
    "allow": [
      "Bash(bun test)",
      "Bash(bun run lint)"
    ],
    "deny": [
      "Read(**/.env)",
      "Bash(rm -rf:*)"
    ]
  }
}
```

## Final Directory Structure

```
project/
├── CLAUDE.md
├── AGENTS.md
├── .mcp.json
├── .claude/
│   ├── agents/
│   │   ├── coder.md
│   │   └── tester.md
│   ├── memory-bank/
│   │   ├── projectBrief.md
│   │   ├── techContext.md
│   │   └── systemPatterns.md
│   └── settings.json
├── .husky/
│   └── pre-commit
├── src/
│   ├── domain/
│   ├── ports/
│   └── adapters/
└── tests/
    ├── unit/
    └── features/
```

## Verification Checklist

- [ ] CLAUDE.md created with project rules
- [ ] Memory bank files populated
- [ ] At least coder and tester agents defined
- [ ] Quality gate scripts in package.json
- [ ] Pre-commit hook installed
- [ ] .mcp.json configured
- [ ] .claude/settings.json created

## Next Steps

1. **Add more agents** as needed (architect, reviewer, security-auditor)
2. **Add slash commands** for your workflows
3. **Add skills** for patterns you use frequently
4. **Customize quality gates** for your project
5. **Consider containerization** for more autonomous operation

## Quick Reference Commands

Start Claude Code:
```bash
claude
```

Run with autonomous mode (requires container):
```bash
claude --dangerously-skip-permissions
```

Common slash commands:
```
/sparc-full    # Full development workflow
/sparc-spec    # Write specifications
/sparc-refine  # TDD implementation
```
