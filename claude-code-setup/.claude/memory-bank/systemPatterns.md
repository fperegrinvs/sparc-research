# System Patterns

## Architectural Pattern: Hexagonal Architecture

### Core Principle
The domain (business logic) is isolated at the center, communicating with the outside world only through ports (interfaces). Adapters implement these ports to connect to specific technologies.

### Layer Structure
```
┌─────────────────────────────────────────────────┐
│                   Adapters                       │
│  ┌─────────────────────────────────────────┐    │
│  │              Ports (Interfaces)          │    │
│  │  ┌─────────────────────────────────┐    │    │
│  │  │         Domain Logic            │    │    │
│  │  │   (Pure Business Rules)         │    │    │
│  │  └─────────────────────────────────┘    │    │
│  └─────────────────────────────────────────┘    │
└─────────────────────────────────────────────────┘
```

### Port Types
1. **Driver Ports** (Primary): Entry points into the application
   - HTTP handlers
   - CLI commands
   - Message consumers

2. **Driven Ports** (Secondary): Exit points from the application
   - Repository interfaces
   - External service clients
   - Notification services

### Dependency Rule
- Dependencies point INWARD only
- Domain knows nothing about adapters
- Adapters depend on ports
- Ports define contracts

## Frontend Patterns

### Component Classification
1. **Base Components** (`components/base/`)
   - Reusable UI primitives
   - No business logic
   - Props-driven, emit events

2. **Business Components** (`components/business/`)
   - Domain-specific UI
   - May use stores directly
   - Composed from base components

### Composables Pattern
```typescript
// Pattern: useXxx
export function useAuth() {
  const user = ref(null)
  const isAuthenticated = computed(() => !!user.value)

  async function login(credentials) { /* ... */ }
  function logout() { /* ... */ }

  return { user, isAuthenticated, login, logout }
}
```

### Store Pattern (Pinia)
```typescript
// Composition API style preferred
export const useUserStore = defineStore('user', () => {
  // State
  const users = ref<User[]>([])

  // Getters
  const activeUsers = computed(() => users.value.filter(u => u.active))

  // Actions
  async function fetchUsers() { /* ... */ }

  return { users, activeUsers, fetchUsers }
})
```

## Testing Patterns

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

### Test Doubles Strategy
1. **Fakes**: For repositories (in-memory implementations)
2. **Stubs**: For external services (predictable responses)
3. **Mocks**: For verification of interactions (use sparingly)

### BDD Scenario Structure
```gherkin
Scenario: User logs in successfully
  Given a registered user with email "test@example.com"
  When the user submits valid credentials
  Then the user should be authenticated
  And the user should be redirected to the dashboard
```

## Error Handling Pattern

### Domain Errors
```typescript
// Custom error types for domain logic
class DomainError extends Error {
  constructor(
    message: string,
    public code: string,
    public details?: Record<string, unknown>
  ) {
    super(message)
    this.name = 'DomainError'
  }
}
```

### API Error Response
```typescript
interface ApiError {
  status: number
  code: string
  message: string
  details?: Record<string, unknown>
}
```

## Dependency Injection Pattern

### Backend DI
```typescript
// Create dependencies at startup
const dependencies = {
  userRepository: new PostgresUserRepository(db),
  emailService: new SendGridEmailService(config),
  // ... other adapters
}

// Inject into use cases
const createUser = new CreateUserUseCase(
  dependencies.userRepository,
  dependencies.emailService
)
```

### Frontend DI
- Use Vue's provide/inject for cross-cutting concerns
- Pinia stores for global state
- Props/events for component communication

## Code Organization Principles

1. **Colocation**: Keep related code together
2. **Single Responsibility**: One reason to change per module
3. **Open/Closed**: Extend via interfaces, not modification
4. **Interface Segregation**: Small, focused interfaces
5. **Dependency Inversion**: Depend on abstractions

## Development Workflow: SPARC Methodology

### Phases
1. **Phase 0: Research** (optional) - Gather documentation, best practices
2. **Phase 1: Specification** - Define requirements, BDD scenarios, domain invariants
3. **Phase 2: Pseudocode** - Algorithm design, function signatures, test strategy
4. **Phase 3: Architecture** - Create structure, port interfaces, fakes
5. **Phase 4: Refinement** - Implement driven by specifications
6. **Phase 5: Completion** - Review, audit, document

### Key Principles
- Write specifications BEFORE implementation
- Test WHAT (behavior), not HOW (implementation)
- Use Fakes for domain testing, Mocks only at adapter boundaries
- Property tests define domain invariants
- BDD scenarios are executable acceptance criteria

### Commands
- `/sparc-full` - Run complete workflow
- `/sparc-research`, `/sparc-spec`, `/sparc-pseudo`, `/sparc-arch`, `/sparc-refine`, `/sparc-complete`

See `.claude/skills/sparc-methodology.md` for detailed guidance.
