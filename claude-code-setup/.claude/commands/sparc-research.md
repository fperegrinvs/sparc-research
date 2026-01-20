# /sparc-research - Research Phase (Phase 0)

## Trigger
Run before specification when you need to gather information about:
- Technologies and frameworks
- Best practices and patterns
- API documentation
- Similar implementations

This phase is **optional** - skip with `--skip-research` if requirements are already clear.

## Objective
Gather external knowledge to inform design decisions before writing specifications.

## Tools Used
- **WebFetch**: Fetch documentation, tutorials, best practices
- **Grep/Glob**: Search existing codebase for patterns
- **Read**: Examine existing code and documentation

## Research Areas

### 1. Technology Documentation
```markdown
## Research: [Technology Name]

### Official Documentation
- URL: [link]
- Key concepts: [list]
- API reference: [notes]

### Best Practices
- [Practice 1]: [why it matters]
- [Practice 2]: [why it matters]

### Gotchas / Pitfalls
- [Issue 1]: [how to avoid]
- [Issue 2]: [how to avoid]
```

### 2. Pattern Research
```markdown
## Research: [Pattern Name]

### When to Use
- [scenario 1]
- [scenario 2]

### Implementation
- [step 1]
- [step 2]

### Examples
- [example 1 with link]
- [example 2 with link]

### Trade-offs
- Pros: [list]
- Cons: [list]
```

### 3. Similar Implementations
```markdown
## Research: Similar Projects

### [Project Name 1]
- Repository: [link]
- Relevant patterns: [list]
- What to adopt: [list]
- What to avoid: [list]

### [Project Name 2]
- Repository: [link]
- Relevant patterns: [list]
- What to adopt: [list]
- What to avoid: [list]
```

## Process

### Step 1: Identify Research Topics
Based on project brief, list topics needing research:
- [ ] Framework/runtime specifics (e.g., Bun, Hono)
- [ ] Architecture patterns (e.g., hexagonal)
- [ ] Testing approaches
- [ ] Database/ORM options
- [ ] Authentication strategies
- [ ] Deployment targets

### Step 2: Gather Information
For each topic:
1. Fetch official documentation
2. Search for best practices articles
3. Find example implementations
4. Note any gotchas or pitfalls

### Step 3: Synthesize Findings
Create summary document with:
- Technology choices (with rationale)
- Patterns to follow
- Anti-patterns to avoid
- Open questions for specification phase

### Step 4: Update Memory Bank
Add key decisions to:
- `.claude/memory-bank/techContext.md` - Technology details
- `.claude/memory-bank/decisionLog.md` - Decision records

## Parallel Research (Using BatchTool)

When researching multiple topics, use parallel fetching:

```typescript
// Conceptual parallel research
const researchTasks = [
  { topic: 'Bun runtime', queries: ['bun documentation', 'bun best practices'] },
  { topic: 'Hono framework', queries: ['hono documentation', 'hono middleware'] },
  { topic: 'Drizzle ORM', queries: ['drizzle setup', 'drizzle migrations'] },
]

// Execute in parallel via BatchTool
await Promise.all(researchTasks.map(task =>
  fetchAndSummarize(task.topic, task.queries)
))
```

## Output Location
Save to `/docs/research.md`

## Research Template

```markdown
# Research Summary

## Date: [YYYY-MM-DD]
## Topics Researched: [list]

---

## 1. [Topic 1]

### Sources
- [Source 1 URL]
- [Source 2 URL]

### Key Findings
- [Finding 1]
- [Finding 2]

### Recommendations
- [Recommendation 1]
- [Recommendation 2]

### Open Questions
- [Question 1]
- [Question 2]

---

## 2. [Topic 2]
...

---

## Decision Summary

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Runtime | Bun | Native TS, fast startup, built-in test runner |
| Framework | Hono | Lightweight, edge-compatible, good DX |
| ORM | Drizzle | Type-safe, SQL-first, good performance |

## Next Steps
- [ ] Update techContext.md with choices
- [ ] Add decisions to decisionLog.md
- [ ] Proceed to /sparc-spec
```

## Quality Checklist
- [ ] All identified topics researched
- [ ] Official documentation consulted
- [ ] Best practices documented
- [ ] Gotchas and pitfalls noted
- [ ] Decisions documented with rationale
- [ ] Memory bank updated

## Next Phase
After research complete, run `/sparc-spec` to create the specification.
