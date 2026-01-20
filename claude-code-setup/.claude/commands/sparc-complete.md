# /sparc-complete - Completion Phase

## Trigger
Run after refinement to finalize and validate the implementation.

## Inputs
- All implementation code
- Test suite
- Documentation

## Process

### 1. Run Full Test Suite
```bash
bun test              # All unit tests
bun test:integration  # Integration tests
bun test:e2e          # End-to-end tests
bun test:features     # BDD acceptance tests
bun test:coverage     # Coverage report
```

### 2. Quality Validation
```bash
bun lint              # ESLint check
bun typecheck         # TypeScript check
bun run arch:test     # Architecture tests
```

### 3. Security Audit
```bash
bun audit             # Dependency vulnerabilities
bun lint:security     # Security-focused linting
```

### 4. Documentation
- [ ] API endpoints documented
- [ ] README updated
- [ ] Environment variables documented
- [ ] Architecture diagrams updated

### 5. Final Review
- [ ] All acceptance criteria met
- [ ] No critical issues
- [ ] Coverage thresholds met (80%+)
- [ ] No security vulnerabilities

### 6. Update Decision Log
Document final decisions and trade-offs in `decisionLog.md`

## Output Checklist
- [ ] All tests passing
- [ ] Coverage >= 80%
- [ ] No linting errors
- [ ] No type errors
- [ ] No security vulnerabilities
- [ ] Documentation complete
- [ ] Decision log updated

## Deployment Preparation
```bash
bun build             # Build for production
```

## Commit
```
docs: Update documentation for feature X
chore: Prepare release
```

## Summary Report
Generate completion summary:
```markdown
## SPARC Completion Report

### Features Implemented
- [List of features]

### Test Coverage
- Unit: XX%
- Integration: XX%
- Overall: XX%

### Quality Metrics
- Linting: PASS/FAIL
- Type Check: PASS/FAIL
- Security: PASS/FAIL

### Documentation
- [Links to docs]

### Known Issues
- [Any remaining issues]

### Next Steps
- [Recommendations]
```
