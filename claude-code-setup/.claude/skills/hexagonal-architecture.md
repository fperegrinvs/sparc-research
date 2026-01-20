# Hexagonal Architecture Skill

## Overview
Hexagonal Architecture (Ports & Adapters) isolates domain logic from external concerns, enabling testability and flexibility.

## Core Principles

### The Hexagon
```
                    ┌──────────────────────────────┐
                    │        Driver Adapters        │
                    │   (HTTP, CLI, Events, etc.)   │
                    └──────────────┬───────────────┘
                                   │
                    ┌──────────────▼───────────────┐
                    │        Driver Ports           │
                    │   (Use Case Interfaces)       │
                    └──────────────┬───────────────┘
                                   │
         ┌─────────────────────────▼─────────────────────────┐
         │                    DOMAIN                          │
         │  • Entities (Business Objects)                     │
         │  • Use Cases (Business Logic)                      │
         │  • Domain Events                                   │
         │  • Domain Errors                                   │
         │                                                    │
         │  ★ NO EXTERNAL DEPENDENCIES ★                      │
         └─────────────────────────┬─────────────────────────┘
                                   │
                    ┌──────────────▼───────────────┐
                    │        Driven Ports           │
                    │   (Repository Interfaces)     │
                    └──────────────┬───────────────┘
                                   │
                    ┌──────────────▼───────────────┐
                    │       Driven Adapters         │
                    │   (DB, APIs, Services, etc.)  │
                    └──────────────────────────────┘
```

### Dependency Rule
- Dependencies point INWARD
- Domain knows nothing about adapters
- Adapters depend on ports
- Ports are owned by the domain

## Folder Structure

### Backend (Bun/Hono)
```
src/
├── domain/
│   ├── entities/
│   │   ├── user.ts           # User entity + validation
│   │   └── order.ts          # Order entity
│   ├── use-cases/
│   │   ├── register-user.ts  # Use case implementation
│   │   └── create-order.ts
│   ├── events/
│   │   └── user-registered.ts
│   └── errors.ts             # Domain-specific errors
│
├── ports/
│   ├── driver/               # Entry points (optional folder)
│   │   └── auth-service.ts   # Auth use case interface
│   └── driven/               # Dependencies
│       ├── repositories/
│       │   └── user-repository.ts
│       └── services/
│           └── email-service.ts
│
├── adapters/
│   ├── http/                 # Driver adapter
│   │   ├── router.ts
│   │   ├── middleware/
│   │   └── routes/
│   ├── db/                   # Driven adapter
│   │   ├── sqlite/
│   │   │   └── user-repository.ts
│   │   └── postgres/
│   │       └── user-repository.ts
│   └── external/             # Driven adapter
│       └── sendgrid-email.ts
│
└── config/
    ├── index.ts              # Configuration loading
    └── dependencies.ts       # DI container
```

### Frontend (Vue)
```
src/
├── domain/                   # Business logic (if any)
│   ├── models/
│   └── validation/
│
├── ports/
│   └── api/                  # API contracts
│       ├── auth-api.ts
│       └── user-api.ts
│
├── adapters/
│   └── api/                  # API implementations
│       ├── http-client.ts
│       └── auth-api-impl.ts
│
├── components/
│   ├── base/                 # Pure UI (adapters to user)
│   └── business/             # Domain-aware components
│
└── composables/              # Reactive business logic
```

## Port Patterns

### Repository Port
```typescript
// ports/driven/repositories/user-repository.ts
export interface UserRepository {
  findById(id: string): Promise<User | null>
  findByEmail(email: string): Promise<User | null>
  create(user: CreateUserInput): Promise<User>
  update(id: string, data: UpdateUserInput): Promise<User | null>
  delete(id: string): Promise<boolean>
}
```

### Service Port
```typescript
// ports/driven/services/email-service.ts
export interface EmailService {
  sendWelcomeEmail(to: string, name: string): Promise<void>
  sendPasswordReset(to: string, token: string): Promise<void>
}
```

### Use Case Port
```typescript
// ports/driver/auth-service.ts
export interface AuthService {
  register(input: RegisterInput): Promise<User>
  login(input: LoginInput): Promise<AuthTokens>
  logout(userId: string): Promise<void>
}
```

## Adapter Patterns

### Repository Adapter
```typescript
// adapters/db/sqlite/user-repository.ts
import type { UserRepository } from '@ports/driven/repositories/user-repository'

export class SqliteUserRepository implements UserRepository {
  constructor(private db: Database) {}

  async findById(id: string): Promise<User | null> {
    return this.db.query('SELECT * FROM users WHERE id = ?', [id]).first()
  }
  // ... other methods
}
```

### HTTP Adapter
```typescript
// adapters/http/routes/auth.ts
import type { AuthService } from '@ports/driver/auth-service'

export function createAuthRoutes(authService: AuthService) {
  const router = new Hono()

  router.post('/register', async (c) => {
    const body = await c.req.json()
    const user = await authService.register(body)
    return c.json(user, 201)
  })

  return router
}
```

## Testing with Hexagonal

### Unit Test (Domain)
```typescript
// Test domain in isolation with fake ports
const fakeRepo: UserRepository = {
  users: new Map(),
  async findByEmail(email) { return this.users.get(email) ?? null },
  async create(data) {
    const user = { ...data, id: '123' }
    this.users.set(user.email, user)
    return user
  },
}

test('registerUser creates user', async () => {
  const result = await registerUser(input, { userRepository: fakeRepo })
  expect(result.email).toBe(input.email)
})
```

### Integration Test (Adapter)
```typescript
// Test adapter with real implementation
test('SqliteUserRepository creates user', async () => {
  const db = new Database(':memory:')
  const repo = new SqliteUserRepository(db)

  const user = await repo.create({ email: 'test@example.com' })
  expect(user.id).toBeDefined()
})
```

## Anti-Patterns to Avoid
1. Domain importing from adapters
2. Hard-coded dependencies in domain
3. Business logic in adapters
4. Leaking infrastructure types to domain
5. Circular dependencies between layers
