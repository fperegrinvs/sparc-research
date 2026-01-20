# Part 12: Code Quality Enforcement with Agentic AI Assistants

## Overview

Claude Code hooks **complement—but don't replace—traditional enforcement mechanisms**. For comprehensive quality enforcement, teams need multiple layers working together.

## The Enforcement Hierarchy

| Enforcement Point | Best For | Limitations |
|-------------------|----------|-------------|
| Claude Code `PreToolUse` hooks | Blocking dangerous operations before AI executes them | Only runs when Claude Code is the committer |
| Git pre-commit hooks | Fast linting, formatting, secret detection (< 2 seconds) | Bypassable with `--no-verify` |
| Git pre-push hooks | Unit test subset, type checking | Still bypassable |
| CI/CD pipelines | Full test suites, security scanning, **authoritative quality gate** | Slower feedback loop |

**The practical recommendation**: CI is your source of truth, everything else is developer convenience.

## Claude Code Hooks

Claude Code provides 9 hook events that intercept agent actions:

### Available Hook Events

- `PreToolUse` - Before tool execution (can block)
- `PostToolUse` - After tool execution
- `PermissionRequest` - Auto-approve safe operations
- `Notification` - Observe without blocking

### Hook Configuration

Hooks are configured in `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "if echo \"$CLAUDE_TOOL_INPUT\" | jq -r '.command' | grep -q '^git commit'; then bun run gate:commit; fi",
        "timeout": 180
      }]
    }],
    "PostToolUse": [{
      "matcher": "Write",
      "hooks": [{
        "type": "command",
        "command": "bun run lint --fix ${CLAUDE_FILE_PATH}",
        "timeout": 30
      }]
    }]
  }
}
```

### Pre-Commit Hook via Claude Code

Run pre-commit checks when the agent attempts `git commit`:

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "if echo \"$CLAUDE_TOOL_INPUT\" | jq -r '.command' | grep -q '^git commit'; then pre-commit run --all-files; fi",
        "timeout": 180
      }]
    }]
  }
}
```

### Auto-Format After Edits

```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Edit",
      "hooks": [{
        "type": "command",
        "command": "bun run prettier --write ${CLAUDE_FILE_PATH}",
        "timeout": 10
      }]
    }]
  }
}
```

## Git Hooks with Husky

### Setup

```bash
bun add -D husky
bunx husky init
```

### Pre-commit Hook

```bash
#!/bin/sh
# .husky/pre-commit

echo "🔍 Running pre-commit checks..."

# Run fast gate
bun run gate:commit

if [ $? -ne 0 ]; then
  echo "❌ Pre-commit checks failed"
  exit 1
fi

echo "✅ Pre-commit checks passed"
```

### Pre-push Hook

```bash
#!/bin/sh
# .husky/pre-push

echo "🚀 Running pre-push checks..."

# Run full gate
bun run gate:full

if [ $? -ne 0 ]; then
  echo "❌ Pre-push checks failed"
  exit 1
fi

echo "✅ Pre-push checks passed"
```

## Linting Strategy

### Fast Linters Run Locally

Modern linters execute in milliseconds—containerizing them adds unnecessary overhead.

**Run locally** (no container):
- **Ruff** (Python) - 10-100x faster than Flake8
- **ESLint** (JavaScript/TypeScript)
- **oxlint** - Faster ESLint alternative
- **Prettier** - Code formatting
- **Black** - Python formatting

**Containerize** (for reproducibility):
- Integration tests with databases
- Complex multi-language builds
- Full CI validation runs
- AI agent sandboxing

### ESLint Configuration

```javascript
// eslint.config.js
export default [
  {
    rules: {
      // Enforce max file length
      'max-lines': ['error', { max: 500, skipBlankLines: true, skipComments: true }],

      // Enforce max function length
      'max-lines-per-function': ['error', { max: 50, skipBlankLines: true }],

      // No any type
      '@typescript-eslint/no-explicit-any': 'error',

      // Explicit return types
      '@typescript-eslint/explicit-function-return-type': 'warn',
    }
  },
  // Domain-specific rules
  {
    files: ['**/domain/**/*.ts'],
    rules: {
      'no-restricted-imports': ['error', {
        patterns: ['**/adapters/**', 'hono', 'drizzle-orm']
      }]
    }
  }
]
```

### Ruff Configuration (Python)

```toml
# pyproject.toml
[tool.ruff]
line-length = 100
select = ["E", "F", "W", "I", "N", "D", "UP", "B", "C4", "SIM"]
ignore = ["D100", "D104"]

[tool.ruff.per-file-ignores]
"tests/**" = ["D"]
```

## Pre-commit Framework

### Configuration

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
      - id: detect-private-key

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.9
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: local
    hooks:
      - id: typescript-check
        name: TypeScript
        entry: bun run typecheck
        language: system
        types: [typescript]
        pass_filenames: false

      - id: eslint
        name: ESLint
        entry: bun run lint --fix
        language: system
        types: [typescript]
```

### Installation

```bash
pip install pre-commit
pre-commit install
```

## CI/CD Pipeline

### GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v1

      - name: Install dependencies
        run: bun install

      - name: TypeScript check
        run: bun run typecheck

      - name: ESLint
        run: bun run lint

      - name: Architecture tests
        run: bun run arch:test

  test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v1

      - name: Install dependencies
        run: bun install

      - name: Unit tests
        run: bun run test:unit

      - name: Property tests
        run: bun run test:properties

      - name: Contract tests
        run: bun run test:contracts

      - name: BDD tests
        run: bun run test:features

      - name: Coverage check
        run: bun run test:coverage --check

  e2e:
    runs-on: ubuntu-latest
    needs: test
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v1

      - name: Install dependencies
        run: bun install

      - name: Install Playwright
        run: bunx playwright install --with-deps

      - name: E2E tests
        run: bun run test:e2e

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Dependency audit
        run: bun audit

      - name: Secret scanning
        uses: trufflesecurity/trufflehog@v3
```

### Branch Protection

Configure branch protection rules in GitHub:

1. **Require status checks to pass**:
   - lint
   - test
   - e2e
   - security

2. **Require branches to be up to date**

3. **Require pull request reviews**

4. **Do not allow bypassing the above settings**

## Containerized Testing

### Docker Compose for Integration Tests

```yaml
# docker-compose.test.yml
version: '3.8'

services:
  test-db:
    image: postgres:15
    environment:
      POSTGRES_DB: test
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
    healthcheck:
      test: pg_isready -U test
      interval: 5s
      timeout: 5s
      retries: 5

  app:
    build:
      context: .
      dockerfile: Dockerfile.test
    depends_on:
      test-db:
        condition: service_healthy
    environment:
      DATABASE_URL: postgresql://test:test@test-db:5432/test
    command: bun run test:integration
```

### Optimized Dockerfile

```dockerfile
FROM oven/bun:1 as builder
WORKDIR /app
COPY package*.json bun.lockb ./
RUN bun install --frozen-lockfile

FROM oven/bun:1
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .
```

## Summary: The Layered Approach

1. **Claude Code Hooks**: Intercept intent before execution
2. **Git Pre-commit Hooks**: Fast linting and formatting (< 2 seconds)
3. **Git Pre-push Hooks**: Unit tests and type checking
4. **CI/CD Pipelines**: Full test suites, security scans (authoritative)
5. **Branch Protection**: Cannot bypass CI checks

**Design your system assuming some developers (and AI agents) will bypass local hooks**—branch protection with required CI checks is the only reliable enforcement mechanism.
