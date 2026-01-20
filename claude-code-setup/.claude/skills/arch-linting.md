# Architecture Linting Skill

## Overview
Automated enforcement of architectural rules using static analysis.

## Architecture Tests (TypeScript)

### Setup
Create `tests/arch/` directory for architecture tests using custom rules or ts-arch.

### Architecture Rules Implementation

```typescript
// tests/arch/hexagonal.test.ts
import { describe, it, expect } from 'bun:test'
import { glob } from 'glob'
import * as fs from 'fs'
import * as path from 'path'

describe('Hexagonal Architecture Rules', () => {
  const srcDir = path.join(__dirname, '../../src')

  it('domain should not import from adapters', async () => {
    const domainFiles = await glob('**/domain/**/*.ts', { cwd: srcDir })

    for (const file of domainFiles) {
      const content = fs.readFileSync(path.join(srcDir, file), 'utf-8')

      // Check for adapter imports
      const adapterImport = content.match(/from\s+['"].*adapters.*['"]/g)
      expect(adapterImport).toBeNull(
        `Domain file ${file} imports from adapters`
      )
    }
  })

  it('domain should not import external libraries except allowed', async () => {
    const allowedImports = [
      '@app/shared-types',
      '@app/shared-utils',
      'zod', // Validation is domain concern
    ]

    const domainFiles = await glob('**/domain/**/*.ts', { cwd: srcDir })

    for (const file of domainFiles) {
      const content = fs.readFileSync(path.join(srcDir, file), 'utf-8')

      // Find all external imports (not starting with . or @domain/@ports)
      const imports = content.matchAll(/from\s+['"]([^'"]+)['"]/g)

      for (const [, importPath] of imports) {
        if (importPath.startsWith('.')) continue
        if (importPath.startsWith('@domain')) continue
        if (importPath.startsWith('@ports')) continue

        const isAllowed = allowedImports.some((allowed) =>
          importPath.startsWith(allowed)
        )

        expect(isAllowed).toBe(true,
          `Domain file ${file} has unauthorized import: ${importPath}`
        )
      }
    }
  })

  it('adapters should implement port interfaces', async () => {
    const adapterFiles = await glob('**/adapters/**/*.ts', { cwd: srcDir })

    for (const file of adapterFiles) {
      if (file.includes('index.ts') || file.includes('router.ts')) continue

      const content = fs.readFileSync(path.join(srcDir, file), 'utf-8')

      // Check if it implements a port interface
      const implementsPattern = /implements\s+\w+/
      const hasExport = /export\s+(class|function|const)/

      if (hasExport.test(content) && !implementsPattern.test(content)) {
        // Warn but don't fail - some adapters are just wiring
        console.warn(`Adapter ${file} may not implement a port interface`)
      }
    }
  })

  it('files should not exceed 500 lines', async () => {
    const tsFiles = await glob('**/*.ts', { cwd: srcDir })

    for (const file of tsFiles) {
      const content = fs.readFileSync(path.join(srcDir, file), 'utf-8')
      const lines = content.split('\n').length

      expect(lines).toBeLessThanOrEqual(500,
        `File ${file} has ${lines} lines (max 500)`
      )
    }
  })

  it('functions should not exceed 50 lines', async () => {
    const tsFiles = await glob('**/*.ts', { cwd: srcDir })

    for (const file of tsFiles) {
      const content = fs.readFileSync(path.join(srcDir, file), 'utf-8')

      // Simple regex to find functions (not perfect but catches most)
      const functionPattern = /(?:function\s+\w+|(?:async\s+)?(?:\w+\s*=\s*)?(?:async\s+)?\([^)]*\)\s*(?:=>|{))/g

      // This is a simplified check - in production use AST parsing
      // For now, check that there are no massive function blocks
      const lines = content.split('\n')
      let bracketDepth = 0
      let functionLines = 0
      let inFunction = false

      for (const line of lines) {
        if (line.match(/(?:function|=>)/)) {
          inFunction = true
          functionLines = 0
        }

        if (inFunction) {
          functionLines++
          bracketDepth += (line.match(/{/g) || []).length
          bracketDepth -= (line.match(/}/g) || []).length

          if (bracketDepth === 0 && functionLines > 0) {
            expect(functionLines).toBeLessThanOrEqual(50,
              `Function in ${file} has ${functionLines} lines (max 50)`
            )
            inFunction = false
            functionLines = 0
          }
        }
      }
    }
  })
})
```

## ESLint Rules for Architecture

### ESLint Config
```javascript
// eslint.config.js
export default [
  {
    rules: {
      // Prevent imports from adapters in domain
      'no-restricted-imports': ['error', {
        patterns: [
          {
            group: ['**/adapters/**'],
            message: 'Domain cannot import from adapters'
          }
        ]
      }],

      // Enforce max file length
      'max-lines': ['error', {
        max: 500,
        skipBlankLines: true,
        skipComments: true
      }],

      // Enforce max function length
      'max-lines-per-function': ['error', {
        max: 50,
        skipBlankLines: true,
        skipComments: true
      }],

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
        patterns: [
          '**/adapters/**',
          'hono',
          'drizzle-orm',
          'better-sqlite3',
          // Add other infrastructure packages
        ]
      }]
    }
  }
]
```

## Dependency Checking Script

```typescript
// tools/check-deps.ts
import * as fs from 'fs'
import * as path from 'path'

interface DependencyRule {
  source: string
  canImport: string[]
  cannotImport: string[]
}

const rules: DependencyRule[] = [
  {
    source: 'domain',
    canImport: ['domain', 'shared-types', 'shared-utils', 'zod'],
    cannotImport: ['adapters', 'config', 'hono', 'drizzle'],
  },
  {
    source: 'ports',
    canImport: ['domain', 'shared-types'],
    cannotImport: ['adapters', 'config'],
  },
  {
    source: 'adapters',
    canImport: ['domain', 'ports', 'shared-types', 'shared-utils', 'config'],
    cannotImport: [],
  },
]

export function checkDependencies(srcDir: string): string[] {
  const violations: string[] = []

  // Implementation: walk files and check imports
  // Return list of violations

  return violations
}
```

## CI Integration

```yaml
# .github/workflows/arch-test.yml
name: Architecture Tests

on: [push, pull_request]

jobs:
  arch-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v1

      - name: Install dependencies
        run: bun install

      - name: Run architecture tests
        run: bun test tests/arch

      - name: Check dependencies
        run: bun run tools/check-deps.ts
```

## Commands
```bash
bun run arch:test           # Run architecture tests
bun run arch:check-deps     # Check dependency rules
bun lint                    # Run ESLint with arch rules
```

## Best Practices
1. Run arch tests in CI before merge
2. Add new rules as patterns emerge
3. Document exceptions when necessary
4. Use AST parsing for complex rules
5. Keep rules simple and understandable
