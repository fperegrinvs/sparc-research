# Quality Gates Skill

## Overview
Quality gates are **mandatory automated checkpoints** that MUST pass before proceeding to the next step. This is NOT a checklist - it's an enforcement mechanism.

## Core Principle

> "No code proceeds without passing gates. No exceptions."

After EVERY code change (write, edit, refactor), the agent MUST run verification before moving on.

## Gate Levels

### Level 1: Fast Gate (After Every Change)
Run after EVERY file modification. Must complete in < 10 seconds.

```bash
# MANDATORY after every code change
bun run gate:fast
```

**Equivalent to:**
```bash
bun run typecheck --incremental && bun run lint --cache
```

**What it checks:**
- TypeScript compilation (incremental)
- ESLint errors (cached)
- Import violations (architecture)

**On failure:** Fix immediately before any other work.

### Level 2: Unit Gate (After Feature Completion)
Run after completing a logical unit of work (function, use case, component).

```bash
# MANDATORY after completing a feature unit
bun run gate:unit
```

**Equivalent to:**
```bash
bun run gate:fast && bun run test:unit --changed && bun run test:properties
```

**What it checks:**
- All Level 1 checks
- Unit tests for changed files
- Property tests (all - they're fast)

**On failure:** Fix tests before proceeding to next feature.

### Level 3: Integration Gate (Before Commits)
Run before every commit. This is the pre-commit hook.

```bash
# MANDATORY before any commit
bun run gate:commit
```

**Equivalent to:**
```bash
bun run gate:unit && bun run test:contracts && bun run test:features && bun run arch:test
```

**What it checks:**
- All Level 2 checks
- Contract tests (fake vs real)
- BDD feature scenarios
- Architecture constraint tests

**On failure:** Fix before committing. No `--no-verify` allowed.

### Level 4: Full Gate (Before PR/Merge)
Run before creating PR or merging to main.

```bash
# MANDATORY before PR
bun run gate:full
```

**Equivalent to:**
```bash
bun run gate:commit && bun run test:e2e && bun run test:coverage && bun run security:scan
```

**What it checks:**
- All Level 3 checks
- E2E tests (critical paths)
- Coverage thresholds (80%+)
- Security vulnerability scan

## Enforcement in SPARC Workflow

### During /sparc-refine

```
┌─────────────────────────────────────────────────────────────┐
│                    REFINEMENT LOOP                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Write Code ──► gate:fast ──► PASS? ──► Continue          │
│                      │                                      │
│                      ▼                                      │
│                   FAIL? ──► Fix ──► Retry gate:fast        │
│                                                             │
│   Complete Feature ──► gate:unit ──► PASS? ──► Next Feature│
│                            │                                │
│                            ▼                                │
│                         FAIL? ──► Fix ──► Retry gate:unit  │
│                                                             │
│   Ready to Commit ──► gate:commit ──► PASS? ──► Commit     │
│                            │                                │
│                            ▼                                │
│                         FAIL? ──► Fix ──► Retry gate:commit│
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Agent Behavior

The `coder` agent MUST:
1. Run `gate:fast` after EVERY Write or Edit tool call
2. Run `gate:unit` after completing each use case/component
3. Run `gate:commit` before any git commit

```typescript
// Pseudo-code for agent behavior
async function afterCodeChange() {
  const result = await Bash('bun run gate:fast')
  if (!result.success) {
    // MUST fix before proceeding
    await fixIssues(result.errors)
    await afterCodeChange() // Retry
  }
}

async function afterFeatureComplete() {
  const result = await Bash('bun run gate:unit')
  if (!result.success) {
    // MUST fix tests before next feature
    await fixTests(result.failures)
    await afterFeatureComplete() // Retry
  }
}

async function beforeCommit() {
  const result = await Bash('bun run gate:commit')
  if (!result.success) {
    throw new Error('Cannot commit with failing gates')
  }
}
```

## Package.json Scripts

```json
{
  "scripts": {
    "gate:fast": "bun run typecheck --incremental && bun run lint --cache",
    "gate:unit": "bun run gate:fast && bun run test:unit --changed && bun run test:properties",
    "gate:commit": "bun run gate:unit && bun run test:contracts && bun run test:features && bun run arch:test",
    "gate:full": "bun run gate:commit && bun run test:e2e && bun run test:coverage --check && bun run security:scan",

    "typecheck": "tsc --noEmit",
    "lint": "eslint . --ext .ts,.tsx,.vue",
    "arch:test": "bun run test:arch",
    "security:scan": "bun audit && bun run lint:security",

    "test": "vitest run",
    "test:unit": "vitest run tests/unit",
    "test:properties": "vitest run tests/properties",
    "test:contracts": "vitest run tests/contracts",
    "test:features": "cucumber-js tests/features",
    "test:e2e": "playwright test",
    "test:coverage": "vitest run --coverage"
  }
}
```

## Pre-commit Hook

```bash
#!/bin/sh
# .husky/pre-commit

echo "🚦 Running commit gate..."
bun run gate:commit

if [ $? -ne 0 ]; then
  echo "❌ Commit gate failed. Fix issues before committing."
  exit 1
fi

echo "✅ Commit gate passed."
```

## CI/CD Integration

```yaml
# .github/workflows/ci.yml
name: CI

on: [push, pull_request]

jobs:
  gate-full:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v1

      - name: Install dependencies
        run: bun install

      - name: Run full gate
        run: bun run gate:full

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        if: always()
```

## Gate Failure Recovery

### TypeScript Errors
```bash
# See all errors
bun run typecheck 2>&1 | head -50

# Fix common issues
# - Missing types: Add type annotations
# - Import errors: Check paths
# - Strict null: Add null checks
```

### Lint Errors
```bash
# Auto-fix what's possible
bun run lint --fix

# See remaining issues
bun run lint
```

### Test Failures
```bash
# Run specific failing test
bun test tests/unit/use-cases/register-user.test.ts

# Run with verbose output
bun test --reporter=verbose

# Debug specific test
bun test --inspect-brk tests/unit/...
```

### Architecture Violations
```bash
# See violations
bun run arch:test

# Common fixes:
# - Domain importing adapter: Move to port interface
# - Circular dependency: Extract shared types
# - File too large: Split into modules
```

## Gate Metrics

Track gate pass rates to identify problem areas:

| Gate | Target Pass Rate | Acceptable Retry Count |
|------|------------------|----------------------|
| gate:fast | 95%+ first try | 1-2 retries |
| gate:unit | 90%+ first try | 2-3 retries |
| gate:commit | 85%+ first try | 2-3 retries |
| gate:full | 95%+ (CI) | 0 retries (must pass) |

## Summary

**Non-negotiable rules:**
1. `gate:fast` runs after EVERY code change
2. `gate:unit` runs after completing each feature unit
3. `gate:commit` runs before EVERY commit (enforced by hook)
4. `gate:full` runs in CI (blocks merge on failure)

**No workarounds allowed:**
- No `--no-verify` on commits
- No skipping gates "just this once"
- No manual overrides in CI

The gates are the specification. Code that doesn't pass gates doesn't exist.
