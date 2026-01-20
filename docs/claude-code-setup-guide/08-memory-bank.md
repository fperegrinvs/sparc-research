# Part 8: Memory Bank for Persistent Context

## Overview

The Memory Bank (`.claude/memory-bank/`) provides persistent project context that survives across sessions. It stores information that Claude Code needs to understand the project deeply and make informed decisions.

## Memory Bank Structure

```
.claude/memory-bank/
├── projectBrief.md      # Core project requirements
├── techContext.md       # Technical stack and configuration
├── systemPatterns.md    # Architectural patterns and decisions
└── decisionLog.md       # Record of key decisions
```

## Core Memory Files

### projectBrief.md

Defines what the project is and what it aims to achieve.

**Contents**:
- Project overview and goals
- Core functional requirements
- Non-functional requirements (performance, security)
- Success criteria and constraints
- Stakeholder information

**Example**:
```markdown
# Project Brief

## Overview
Full-stack TypeScript application built with Bun backend and Vue.js
frontend, following hexagonal architecture principles.

## Core Requirements

### Functional Requirements
1. RESTful API backend with authentication
2. Reactive frontend with state management
3. Type-safe communication between frontend and backend
4. Comprehensive testing at all levels

### Non-Functional Requirements
1. **Performance**: API response time < 200ms
2. **Scalability**: Horizontal scaling support
3. **Security**: JWT authentication, input validation
4. **Maintainability**: Modular architecture, documentation
5. **Testability**: 80%+ code coverage, BDD acceptance tests

## Success Criteria
- [ ] All SPARC phases completed with documentation
- [ ] Architecture tests pass (no layering violations)
- [ ] Unit test coverage >= 80%
- [ ] BDD scenarios for all user stories pass
- [ ] No critical security vulnerabilities

## Constraints
- Bun runtime required for backend
- Vue 3 with Composition API for frontend
- TypeScript strict mode enabled
- Monorepo structure with shared packages
```

### techContext.md

Details the technical stack, configuration, and tooling.

**Contents**:
- Runtime and build tools
- Backend stack details
- Frontend stack details
- Shared packages
- Testing tools
- Environment configuration

**Example**:
```markdown
# Technical Context

## Runtime & Build Tools

### Bun (Backend)
- Version: 1.x (latest stable)
- Features: HTTP server, test runner, bundler, workspaces
- Configuration: `bunfig.toml`

## Backend Stack

### Hono Framework
- Lightweight, fast web framework
- Native TypeScript support
- Middleware-based architecture

### Database
- Development: SQLite
- Production: PostgreSQL
- ORM: Drizzle ORM (type-safe)

### Authentication
- JWT-based token authentication
- Refresh token rotation
- Secure cookie storage

## Frontend Stack

### Vue 3
- Composition API with `<script setup>`
- Reactive primitives: `ref`, `reactive`, `computed`

### Pinia
- Composition API-style stores
- DevTools integration

### Vue Router
- History mode routing
- Route guards for authentication

## Testing Tools
- **Vitest**: Unit and integration testing
- **Playwright**: E2E browser testing
- **Cucumber.js**: BDD testing with Gherkin

## Environment Configuration
```
NODE_ENV=development|production|test
API_PORT=3000
DATABASE_URL=sqlite://./dev.db
JWT_SECRET=<from-env>
```
```

### systemPatterns.md

Documents architectural decisions and established patterns.

**Contents**:
- Architectural patterns (hexagonal, etc.)
- Frontend patterns (components, composables)
- Testing patterns
- Error handling patterns
- Dependency injection patterns
- Code organization principles

**Example**:
```markdown
# System Patterns

## Architectural Pattern: Hexagonal Architecture

### Core Principle
The domain (business logic) is isolated at the center, communicating
with the outside world only through ports (interfaces).

### Layer Structure
```
┌─────────────────────────────────────────────────┐
│                   Adapters                       │
│  ┌─────────────────────────────────────────┐    │
│  │              Ports (Interfaces)          │    │
│  │  ┌─────────────────────────────────┐    │    │
│  │  │         Domain Logic            │    │    │
│  │  └─────────────────────────────────┘    │    │
│  └─────────────────────────────────────────┘    │
└─────────────────────────────────────────────────┘
```

### Dependency Rule
- Dependencies point INWARD only
- Domain knows nothing about adapters
- Adapters depend on ports

## Testing Patterns

### Test Doubles Strategy
1. **Fakes**: For repositories (in-memory implementations)
2. **Stubs**: For external services (predictable responses)
3. **Mocks**: For verification of interactions (use sparingly)

### Arrange-Act-Assert
```typescript
test('should calculate total', () => {
  // Arrange
  const cart = new Cart()
  cart.addItem({ price: 100, quantity: 2 })

  // Act
  const total = cart.calculateTotal()

  // Assert
  expect(total).toBe(200)
})
```

## Error Handling Pattern
```typescript
class DomainError extends Error {
  constructor(
    message: string,
    public code: string,
    public details?: Record<string, unknown>
  ) {
    super(message)
  }
}
```
```

### decisionLog.md

Records key decisions and their rationale.

**Contents**:
- Dated decision entries
- Context and problem statement
- Decision made
- Rationale and trade-offs
- Consequences

**Example**:
```markdown
# Decision Log

## DEC-001: Use Bun as Backend Runtime
**Date**: 2024-01-15
**Status**: Accepted

### Context
Need to choose a JavaScript runtime for the backend.

### Decision
Use Bun 1.x as the backend runtime.

### Rationale
- Native TypeScript support (no transpilation)
- Fast startup and execution
- Built-in test runner
- Compatible with Node.js packages

### Trade-offs
- Newer ecosystem (fewer packages)
- Some Node.js APIs not fully supported

---

## DEC-002: Hexagonal Architecture
**Date**: 2024-01-15
**Status**: Accepted

### Context
Need to establish an architectural pattern for the backend.

### Decision
Implement hexagonal (ports & adapters) architecture.

### Rationale
- Clear separation of concerns
- Domain logic is fully testable
- Easy to swap implementations
- Supports TDD workflow

### Trade-offs
- More initial boilerplate
- Requires discipline to maintain boundaries
```

## Updating the Memory Bank

### When to Update

| File | Update When |
|------|-------------|
| projectBrief.md | Requirements change |
| techContext.md | Technology decisions made |
| systemPatterns.md | New patterns established |
| decisionLog.md | Any significant decision |

### Update Process

1. **Research Phase**: Updates techContext.md with findings
2. **Architecture Phase**: Updates systemPatterns.md
3. **After Decisions**: Updates decisionLog.md
4. **Scope Changes**: Updates projectBrief.md

## Best Practices

1. **Keep Files Focused**: Each file has a single purpose
2. **Be Concise**: Essential information only
3. **Use Examples**: Show patterns, not just describe
4. **Date Entries**: Especially in decisionLog.md
5. **Cross-Reference**: Link related information
6. **Version Control**: Commit changes with meaningful messages

## Reading Priority for AI Agents

When starting a new session, agents should read in this order:
1. `CLAUDE.md` - Project rules (auto-read)
2. `projectBrief.md` - What we're building
3. `techContext.md` - How we're building it
4. `systemPatterns.md` - Established patterns
5. `decisionLog.md` - Past decisions (as needed)
