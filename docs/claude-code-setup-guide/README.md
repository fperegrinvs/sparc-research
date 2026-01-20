# Claude Code Setup Guide for Autonomous Coding Agents

## A Comprehensive Guide to Maximizing AI-Assisted Development Effectiveness

---

## Table of Contents

### [Part 0: Background - Architecture, Testing, and SPARC](./00-background.md)
- Why This Matters (AI code risks and the solution)
- Hexagonal Architecture: The Foundation
- Testing Strategy: Specification-Driven, Not Implementation-Coupled
- SPARC: A Structured Workflow for AI-Assisted Development
- Quality Gates: Non-Negotiable Checkpoints

### [Part 1: Introduction](./01-introduction.md)
- Why Project Setup Matters for AI-Assisted Development
- The Challenge of Limited Context Windows
- The Solution: Structured Project Configuration
- What This Guide Covers

### [Part 2: Project Root Configuration Files](./02-project-root-files.md)
- CLAUDE.md: The Project Constitution
- AGENTS.md: Quick Reference for AI Assistants
- File Location Strategy

### [Part 3: The .claude Directory Structure](./03-claude-directory-structure.md)
- Directory Layout
- Purpose of Each Directory (agents, commands, memory-bank, skills)
- Configuration Files (settings.json, .mcp.json)
- What to Commit vs. Gitignore

### [Part 4: The SPARC Methodology](./04-sparc-methodology.md)
- Phase 0: Research (Optional)
- Phase 1: Specification
- Phase 2: Pseudocode
- Phase 3: Architecture
- Phase 4: Refinement
- Phase 5: Completion
- Parallel Execution (Boomerang Pattern)

### [Part 5: Specialized Agents](./05-specialized-agents.md)
- The Orchestrator Agent
- The Researcher Agent
- The Architect Agent
- The Coder Agent
- The Tester Agent
- The Reviewer Agent
- The Security Auditor Agent
- Creating Custom Agents

### [Part 6: Slash Commands](./06-slash-commands.md)
- The SPARC Commands (/sparc-full, /sparc-spec, etc.)
- Command File Template
- Command Best Practices

### [Part 7: Skills Reference Documentation](./07-skills-reference.md)
- Hexagonal Architecture
- Quality Gates
- Specification-Driven Testing
- BDD Testing
- Property-Based Testing
- Architecture Linting
- Creating Custom Skills

### [Part 8: Memory Bank for Persistent Context](./08-memory-bank.md)
- projectBrief.md
- techContext.md
- systemPatterns.md
- decisionLog.md
- Updating the Memory Bank

### [Part 9: Quality Gates and Automated Verification](./09-quality-gates.md)
- Gate Levels (fast, unit, commit, full)
- Enforcement in SPARC Workflow
- Package.json Scripts
- Pre-commit Hooks
- CI/CD Integration

### [Part 10: Security and Sandboxing](./10-security-sandboxing.md)
- Defense in Depth Strategy
- Filesystem Isolation
- Network Isolation
- Dev Container Sandboxing
- Claude Code Hooks
- Enterprise Configuration

### [Part 11: MCP Server Configuration](./11-mcp-configuration.md)
- Basic Configuration
- Adding MCP Servers
- Tool Permissions
- Security Considerations

### [Part 12: Code Quality Enforcement](./12-code-quality-enforcement.md)
- The Enforcement Hierarchy
- Claude Code Hooks
- Git Hooks with Husky
- Linting Strategy
- Pre-commit Framework
- CI/CD Pipeline

### [Part 13: Testing Strategy for AI-Generated Code](./13-testing-strategy.md)
- The Specification Testing Pyramid
- Property-Based Tests
- Black-Box Unit Tests
- Contract Tests
- BDD Feature Tests
- E2E Tests
- Test Organization

### [Part 14: Quick Start Guide](./14-quick-start.md)
- 30-Minute Setup Process
- Verification Checklist
- Quick Reference Commands

---

## How to Use This Guide

**For new projects**: Start with [Part 14 (Quick Start Guide)](./14-quick-start.md) to get a basic setup running, then customize using the detailed sections.

**For existing projects**: Review [Part 1-3](./01-introduction.md) to understand the structure, then add components incrementally based on your needs.

**For specific topics**: Each part is self-contained and can be read independently.

---

## Key Principles

1. **Verify everything**: Treat AI-generated code like submissions from a junior developer
2. **Automate quality gates**: Make quality enforcement unavoidable
3. **Layer your defenses**: Claude Code hooks, git hooks, and CI all serve different purposes
4. **Test behavior, not implementation**: Use fakes, not mocks, for domain testing
5. **Keep context concise**: LLMs have limited instruction-following capacity

---

## Source Materials

This guide synthesizes information from:
- The example claude-code-setup configuration in `/source/claude-code-setup/`
- Research on SPARC methodology and AI-assisted development
- Multi-layered quality assurance strategies for AI-generated code
- Best practices for enforcing code quality with agentic AI assistants

---

## Merging All Files

To create a single comprehensive document, run:

```bash
cat 01-introduction.md 02-project-root-files.md 03-claude-directory-structure.md 04-sparc-methodology.md 05-specialized-agents.md 06-slash-commands.md 07-skills-reference.md 08-memory-bank.md 09-quality-gates.md 10-security-sandboxing.md 11-mcp-configuration.md 12-code-quality-enforcement.md 13-testing-strategy.md 14-quick-start.md > COMPLETE-GUIDE.md
```
