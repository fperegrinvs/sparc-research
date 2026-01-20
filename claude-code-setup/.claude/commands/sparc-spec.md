# /sparc-spec - Specification Phase

## Trigger
Run when starting a new feature or project to define requirements.

## Inputs
- User requirements or feature request
- Existing projectBrief.md (if available)

## Process
1. **Analyze Requirements**
   - Extract functional requirements
   - Identify non-functional requirements
   - List constraints and dependencies

2. **Define User Stories**
   ```
   As a [role]
   I want [goal]
   So that [benefit]
   ```

3. **Write Acceptance Criteria**
   - Use Gherkin Given/When/Then format
   - Cover happy paths and error cases

4. **Document in Specification**
   - Create or update `docs/specification.md`
   - Link to user stories

## Output
- `docs/specification.md` - Complete specification document
- `tests/features/*.feature` - Gherkin scenarios
- Updated `projectBrief.md` - If new requirements

## Example
```markdown
## Feature: User Authentication

### Requirements
- FR-001: Users can register with email/password
- FR-002: Users can log in with credentials
- FR-003: Users can reset forgotten password

### Acceptance Criteria
Given a new user
When they submit valid registration data
Then their account should be created
And they should receive a confirmation email
```

## Next Phase
After specification is complete, run `/sparc-pseudo` for pseudocode design.
