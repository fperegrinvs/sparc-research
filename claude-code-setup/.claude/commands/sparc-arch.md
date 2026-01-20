# /sparc-arch - Architecture Phase

## Trigger
Run after pseudocode to design system structure.

## Inputs
- `docs/specification.md`
- `docs/pseudocode.md`
- `systemPatterns.md`

## Process
1. **Define Components**
   - Identify bounded contexts
   - Map to hexagonal architecture layers
   - Define module boundaries

2. **Design Ports (Interfaces)**
   - Driver ports (entry points)
   - Driven ports (dependencies)

3. **Plan Adapters**
   - HTTP/REST adapters
   - Database adapters
   - External service adapters

4. **Create Folder Structure**
   ```
   src/
     domain/           # Pure business logic
       entities/       # Domain models
       use-cases/      # Business operations
       errors.ts       # Domain errors
     ports/            # Interfaces
       repositories/   # Data access contracts
       services/       # External service contracts
     adapters/         # Implementations
       http/           # REST API
       db/             # Database access
       external/       # Third-party services
     config/           # Configuration
   ```

5. **Document Decisions**
   - Update `decisionLog.md`
   - Record architectural trade-offs

## Output
- Folder structure created
- Port interface definitions (TypeScript interfaces)
- `docs/architecture.md` - Architecture documentation
- Updated `systemPatterns.md`

## Architecture Rules
```
RULE 1: Domain NEVER imports from adapters
RULE 2: Ports define technology-agnostic contracts
RULE 3: Adapters implement port interfaces
RULE 4: All dependencies injected at startup
RULE 5: Configuration through environment only
```

## Next Phase
After architecture is designed, run `/sparc-refine` for TDD implementation.
