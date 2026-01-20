# Part 3: The .claude Directory Structure

## Overview

The `.claude/` directory contains all the detailed configuration that extends beyond the root CLAUDE.md file. This hierarchical structure keeps the root file concise while providing deep context when needed.

## Directory Layout

```
.claude/
├── agents/              # Specialized agent definitions
│   ├── AGENTS.md        # Overview of all agents
│   ├── orchestrator.md  # Multi-agent coordination
│   ├── researcher.md    # Research and documentation
│   ├── architect.md     # System design
│   ├── coder.md         # Implementation
│   ├── tester.md        # Quality assurance
│   ├── reviewer.md      # Code review
│   └── security-auditor.md
│
├── commands/            # Slash commands (workflows)
│   ├── sparc-full.md    # Full SPARC workflow
│   ├── sparc-spec.md    # Specification phase
│   ├── sparc-pseudo.md  # Pseudocode phase
│   ├── sparc-arch.md    # Architecture phase
│   ├── sparc-refine.md  # Refinement phase
│   ├── sparc-complete.md # Completion phase
│   └── sparc-research.md # Research phase
│
├── memory-bank/         # Persistent project context
│   ├── projectBrief.md  # Core requirements
│   ├── techContext.md   # Technical stack details
│   ├── systemPatterns.md # Architecture patterns
│   └── decisionLog.md   # Decision record
│
├── skills/              # Reusable knowledge and patterns
│   ├── sparc-methodology.md
│   ├── quality-gates.md
│   ├── hexagonal-architecture.md
│   ├── tdd-workflow.md
│   ├── bdd-testing.md
│   ├── property-testing.md
│   ├── metamorphic-testing.md
│   └── arch-linting.md
│
└── settings.json        # Claude Code settings (optional)
```

## Purpose of Each Directory

### agents/

Contains role-specific instructions for different development tasks. Each agent has:
- A defined **role** and **responsibilities**
- **Allowed tools** (some agents shouldn't write code)
- **Constraints** and rules to follow
- **Output formats** for consistency

### commands/

Slash commands are invokable workflows that can be triggered with `/command-name`. They define:
- **Trigger conditions** - When to use this command
- **Prerequisites** - What must exist before running
- **Steps** - What the command does
- **Outputs** - What gets created
- **Quality gates** - Verification requirements

### memory-bank/

Persistent context files that provide project-specific knowledge:
- **projectBrief.md** - What the project is and its goals
- **techContext.md** - Technology stack and configuration
- **systemPatterns.md** - Architectural decisions and patterns
- **decisionLog.md** - Record of key decisions and rationale

### skills/

Reference documentation for patterns, practices, and methodologies:
- Detailed explanations of concepts
- Code examples and templates
- Best practices and anti-patterns
- Can be imported into CLAUDE.md with `@.claude/skills/skill-name.md`

## Configuration Files

### settings.json (Project-Level)

```json
{
  "permissions": {
    "allow": [
      "Bash(npm test)",
      "Bash(npm run lint)",
      "Bash(bun test)"
    ],
    "deny": [
      "WebFetch",
      "Bash(curl:*)",
      "Read(./secrets/**)"
    ]
  },
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "if echo \"$CLAUDE_TOOL_INPUT\" | jq -r '.command' | grep -q '^git commit'; then bun run gate:commit; fi",
        "timeout": 180
      }]
    }]
  }
}
```

### .mcp.json (MCP Server Configuration)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "mcpServers": {
    "notes": {
      "This file configures MCP servers for Claude Code": true
    }
  },
  "settings": {
    "allowedTools": [
      "Read",
      "Write",
      "Edit",
      "Glob",
      "Grep",
      "Bash",
      "WebFetch",
      "Task",
      "BatchTool"
    ]
  }
}
```

## File Naming Conventions

- Use lowercase with hyphens: `sparc-methodology.md`
- Prefix commands with their workflow: `sparc-spec.md`, `sparc-arch.md`
- Agent files match agent names: `coder.md`, `tester.md`
- Keep names short but descriptive

## What to Commit vs. Gitignore

### Commit to Repository
- `.claude/settings.json` - Team hook configurations
- `.claude/agents/` - Agent definitions
- `.claude/commands/` - Workflow commands
- `.claude/skills/` - Shared knowledge
- `.claude/memory-bank/projectBrief.md` - Project requirements
- `.claude/memory-bank/systemPatterns.md` - Architecture patterns

### Keep Individual (Gitignore)
- `.claude/settings.local.json` - Personal preferences
- `CLAUDE.local.md` - Personal sandbox URLs, credentials
- User-level `~/.claude/settings.json` - Personal defaults

## Example .gitignore Entries

```gitignore
# Claude Code local files
.claude/settings.local.json
CLAUDE.local.md

# Don't commit decision log if it contains sensitive info
# .claude/memory-bank/decisionLog.md
```
