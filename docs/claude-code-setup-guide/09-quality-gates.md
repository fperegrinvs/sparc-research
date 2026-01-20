# Part 9: Quality Gates and Automated Verification

## Overview

Quality gates are **mandatory automated checkpoints** that MUST pass before proceeding. This is NOT a checklist—it's an enforcement mechanism that ensures code quality throughout the development process.

## Core Principle

> "No code proceeds without passing gates. No exceptions."

After EVERY code change, the agent MUST run verification before moving on. Gates are the specification—code that doesn't pass gates doesn't exist.

## Gate Levels

### Level 1: Fast Gate (`gate:fast`)

**When**: After EVERY file modification
**Time Limit**: < 10 seconds
**Purpose**: Immediate feedback on syntax and style

```bash
bun run gate:fast
```

**Equivalent to**:
```bash
bun run typecheck --incremental && bun run lint --cache
```

**Checks**:
- TypeScript compilation (incremental)
- ESLint errors (cached)
- Import violations (architecture)

**On Failure**: Fix immediately before any other work.

### Level 2: Unit Gate (`gate:unit`)

**When**: After completing a logical unit of work
**Time Limit**: < 60 seconds
**Purpose**: Verify feature correctness

```bash
bun run gate:unit
```

**Equivalent to**:
```bash
bun run gate:fast && bun run test:unit --changed && bun run test:properties
```

**Checks**:
- All Level 1 checks
- Unit tests for changed files
- Property tests (all - they're fast)

**On Failure**: Fix tests before proceeding to next feature.

### Level 3: Integration Gate (`gate:commit`)

**When**: Before every commit
**Time Limit**: < 2 minutes
**Purpose**: Ensure commit-worthy quality

```bash
bun run gate:commit
```

**Equivalent to**:
```bash
bun run gate:unit && bun run test:contracts && bun run test:features && bun run arch:test
```

**Checks**:
- All Level 2 checks
- Contract tests (fake vs real)
- BDD feature scenarios
- Architecture constraint tests

**On Failure**: Fix before committing. No `--no-verify` allowed.

### Level 4: Full Gate (`gate:full`)

**When**: Before creating PR or merging to main
**Time Limit**: < 10 minutes
**Purpose**: Complete quality assurance

```bash
bun run gate:full
```

**Equivalent to**:
```bash
bun run gate:commit && bun run test:e2e && bun run test:coverage && bun run security:scan
```

**Checks**:
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

### Agent Behavior (REQUIRED)

```typescript
// After EVERY code change
async function afterCodeChange() {
  const result = await Bash('bun run gate:fast')
  if (!result.success) {
    await fixIssues(result.errors)
    await afterCodeChange() // Retry
  }
}

// After completing a feature
async function afterFeatureComplete() {
  const result = await Bash('bun run gate:unit')
  if (!result.success) {
    await fixTests(result.failures)
    await afterFeatureComplete() // Retry
  }
}

// Before committing
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

# Common fixes:
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

## Non-Negotiable Rules

1. `gate:fast` runs after EVERY code change
2. `gate:unit` runs after completing each feature unit
3. `gate:commit` runs before EVERY commit (enforced by hook)
4. `gate:full` runs in CI (blocks merge on failure)

**No workarounds allowed**:
- No `--no-verify` on commits
- No skipping gates "just this once"
- No manual overrides in CI

The gates ARE the specification. Code that doesn't pass gates doesn't exist.
