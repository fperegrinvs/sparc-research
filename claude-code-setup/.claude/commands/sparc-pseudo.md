# /sparc-pseudo - Pseudocode Phase

## Trigger
Run after specification to design the solution approach before implementation.

## Inputs
- `docs/specification.md` (from Phase 1)
- `.claude/memory-bank/projectBrief.md`
- BDD/Gherkin scenarios

## Objective
Create a high-level solution design without writing implementation code. This phase forces "thinking before coding" and ensures the AI has a clear plan.

## Deliverables

### 1. Algorithm Outlines
```markdown
## Register User Algorithm

1. Validate input
   - Check email format (RFC 5322)
   - Check password strength (min 8 chars, mixed case, number)
2. Check for existing user
   - Query userRepository.findByEmail(email)
   - If exists → return CONFLICT error
3. Hash password
   - Use bcrypt with cost factor 12
4. Create user record
   - Generate UUID for id
   - Store email (lowercase), passwordHash, timestamps
5. Send welcome email
   - Queue async email via emailService
6. Return created user (without password hash)
```

### 2. Data Flow Diagrams (Text-Based)
```
┌──────────┐     ┌──────────────┐     ┌────────────┐
│  HTTP    │────▶│  Use Case    │────▶│ Repository │
│ Adapter  │     │ (Domain)     │     │   (Port)   │
└──────────┘     └──────────────┘     └────────────┘
     │                  │                    │
     │                  ▼                    ▼
     │           ┌──────────────┐     ┌────────────┐
     │           │   Domain     │     │  Database  │
     │           │   Entity     │     │  Adapter   │
     │           └──────────────┘     └────────────┘
     │                  │
     ▼                  ▼
┌──────────┐     ┌──────────────┐
│ Response │◀────│   Result     │
│   DTO    │     │  (Ok/Err)    │
└──────────┘     └──────────────┘
```

### 3. Function/Method Signatures
```typescript
// Domain Use Cases
function registerUser(
  input: RegisterUserInput,
  deps: { userRepository: UserRepository; hashPassword: HashFn; emailService: EmailService }
): Promise<Result<User, DomainError>>

function authenticateUser(
  input: AuthenticateInput,
  deps: { userRepository: UserRepository; verifyPassword: VerifyFn; tokenService: TokenService }
): Promise<Result<AuthTokens, DomainError>>

// Port Interfaces
interface UserRepository {
  findById(id: string): Promise<User | null>
  findByEmail(email: string): Promise<User | null>
  create(data: CreateUserData): Promise<User>
  update(id: string, data: UpdateUserData): Promise<User | null>
  delete(id: string): Promise<boolean>
}

// Domain Entities
interface User {
  id: string
  email: string
  passwordHash: string
  createdAt: Date
  updatedAt: Date
}
```

### 4. Test Strategy Outline
```markdown
## Property Tests (Invariants)
- User email is always lowercase after creation
- Password hash is never equal to plain password
- User ID is always valid UUID format
- Created timestamp is never in the future

## BDD Scenarios (from specification)
- Successful registration with valid credentials
- Registration rejected for invalid email
- Registration rejected for weak password
- Registration rejected for duplicate email

## Contract Tests (Fakes vs Real)
- UserRepository: InMemory vs PostgreSQL
- EmailService: InMemory vs SendGrid
- TokenService: InMemory vs JWT

## Integration Tests
- Full registration flow with real DB
- Authentication flow with real token generation
```

### 5. Domain Invariants
```typescript
// Invariants to verify with property-based testing
const userInvariants = {
  emailAlwaysLowercase: (user: User) => user.email === user.email.toLowerCase(),
  idIsValidUUID: (user: User) => isValidUUID(user.id),
  timestampsAreValid: (user: User) => user.createdAt <= user.updatedAt,
  passwordNeverStored: (user: User) => !user.hasOwnProperty('password'),
}

const orderInvariants = {
  totalNeverNegative: (order: Order) => order.total >= 0,
  totalEqualsLineItems: (order: Order) =>
    order.total === order.items.reduce((sum, i) => sum + i.price * i.quantity, 0),
  quantityAlwaysPositive: (order: Order) =>
    order.items.every(i => i.quantity > 0),
}
```

## Process

### Step 1: Analyze Specification
Read the specification and extract:
- Core behaviors to implement
- Business rules and constraints
- Error conditions and edge cases

### Step 2: Design Algorithms
For each use case:
1. Write step-by-step algorithm in plain language
2. Identify decision points and branches
3. Note error conditions at each step

### Step 3: Define Data Flows
Create text diagrams showing:
- How data enters the system (adapters)
- How it flows through domain logic
- How it exits (responses, side effects)

### Step 4: Draft Signatures
Write function/interface signatures for:
- Use case functions (domain)
- Port interfaces
- Entity types

### Step 5: Plan Testing Strategy
Document:
- Property invariants to test
- BDD scenarios (reference from spec)
- Contract test pairings (fake vs real)
- Integration test scope

## Output Location
Save to `/docs/pseudocode.md`

## Quality Checklist
- [ ] Every use case from spec has an algorithm
- [ ] Data flow covers all entry/exit points
- [ ] All port interfaces are defined
- [ ] Domain invariants are identified
- [ ] Test strategy covers all layers

## Next Phase
After pseudocode approved, run `/sparc-arch` to create the architecture.
