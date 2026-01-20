# Decision Log

## Architecture Decisions

### ADR-001: Bun as Backend Runtime
**Date**: Initial setup
**Status**: Accepted

**Context**: Need a fast, TypeScript-native runtime for the backend.

**Decision**: Use Bun instead of Node.js.

**Rationale**:
- Native TypeScript support (no transpilation step)
- Built-in bundler and test runner
- Faster startup and execution
- Built-in SQLite support
- Native workspace support for monorepo

**Consequences**:
- (+) Simplified build process
- (+) Faster development experience
- (-) Smaller ecosystem than Node.js
- (-) Some npm packages may have compatibility issues

---

### ADR-002: Hexagonal Architecture for Backend
**Date**: Initial setup
**Status**: Accepted

**Context**: Need maintainable, testable architecture that supports AI-assisted development.

**Decision**: Implement hexagonal (ports & adapters) architecture.

**Rationale**:
- Clear separation of concerns
- Domain logic isolated and easily testable
- Adapters can be swapped without affecting domain
- AI agents can reason about boundaries clearly
- Enforces dependency injection naturally

**Consequences**:
- (+) High testability
- (+) Technology flexibility
- (+) Clear boundaries for AI reasoning
- (-) More boilerplate code
- (-) Learning curve for new developers

---

### ADR-003: Vue 3 Composition API for Frontend
**Date**: Initial setup
**Status**: Accepted

**Context**: Need a reactive frontend framework with TypeScript support.

**Decision**: Use Vue 3 with Composition API exclusively.

**Rationale**:
- Better TypeScript integration
- More flexible code organization
- Reusable logic via composables
- Better tree-shaking
- `<script setup>` reduces boilerplate

**Consequences**:
- (+) Type-safe templates
- (+) Reusable logic patterns
- (+) Smaller bundle size
- (-) Different from Options API (learning curve)

---

### ADR-004: Monorepo with Bun Workspaces
**Date**: Initial setup
**Status**: Accepted

**Context**: Need to share code between frontend and backend.

**Decision**: Use Bun workspaces for monorepo management.

**Rationale**:
- Native Bun support
- Shared packages for types and utilities
- Single dependency lockfile
- Easier cross-package testing
- TypeScript project references support

**Consequences**:
- (+) Code reuse across packages
- (+) Consistent versioning
- (+) Simplified CI/CD
- (-) Build complexity
- (-) Requires careful package boundaries

---

### ADR-005: BDD with Cucumber for Acceptance Testing
**Date**: Initial setup
**Status**: Accepted

**Context**: Need executable specifications that bridge business and technical requirements.

**Decision**: Use Cucumber.js with Gherkin syntax for BDD tests.

**Rationale**:
- Human-readable specifications
- Stakeholder collaboration
- Living documentation
- AI can generate scenarios from requirements
- Clear acceptance criteria

**Consequences**:
- (+) Specification as documentation
- (+) Business-developer communication
- (+) Regression safety net
- (-) Additional test layer to maintain
- (-) Step definition maintenance

---

## Technical Decisions

### TD-001: Drizzle ORM for Database Access
**Decision**: Use Drizzle ORM for database operations.

**Rationale**: Type-safe, lightweight, SQL-first approach aligns with hexagonal architecture.

---

### TD-002: Zod for Validation
**Decision**: Use Zod for runtime validation and type inference.

**Rationale**: Works at compile-time and runtime, integrates with TypeScript.

---

### TD-003: Vitest for Unit Testing
**Decision**: Use Vitest as the primary test runner.

**Rationale**: Fast, native ESM, Jest-compatible API, works with Bun.

---

## Pending Decisions

### PD-001: Production Database Choice
**Status**: Pending
**Options**: PostgreSQL, MySQL, SQLite
**Notes**: PostgreSQL preferred for production, decision pending requirements finalization.

---

### PD-002: Authentication Strategy Details
**Status**: Pending
**Options**: Session-based, JWT with refresh tokens, OAuth integration
**Notes**: JWT with refresh tokens baseline, OAuth may be added later.
