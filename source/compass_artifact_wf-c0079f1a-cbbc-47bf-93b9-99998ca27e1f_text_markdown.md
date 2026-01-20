# Enforcing code quality with agentic AI assistants

The bottleneck in AI-assisted development is no longer code generation—it's verification. Teams using Claude Code and similar agentic tools must build **layered enforcement infrastructure** combining AI-native hooks, traditional git hooks, CI/CD pipelines, and containerized sandboxing to maintain code quality and security at scale.

The emerging consensus: treat AI-generated code as you would submissions from a junior developer. Verify everything, automate quality gates at multiple points, and invest in isolation proportional to the autonomy you grant the AI agent.

## Claude Code hooks complement—but don't replace—traditional enforcement

Claude Code provides **9 hook events** that intercept agent actions at critical moments: `PreToolUse` (blocks dangerous operations before they execute), `PostToolUse` (runs formatters after edits), `PermissionRequest` (auto-approves safe operations), and others. These hooks are configured in `.claude/settings.json` and receive rich JSON context about the operation being performed.

The key distinction: **Claude Code hooks intercept intent** while git hooks intercept actual operations. For comprehensive enforcement, teams need both:

| Enforcement Point | Best For | Limitations |
|-------------------|----------|-------------|
| Claude Code `PreToolUse` hooks | Blocking dangerous commands before AI executes them, auto-formatting after edits | Only runs when Claude Code is the committer |
| Git pre-commit hooks | Fast linting, formatting, secret detection (**< 2 seconds total**) | Bypassable with `--no-verify` |
| Git pre-push hooks | Unit test subset, type checking | Still bypassable |
| CI/CD pipelines | Full test suites, security scanning, **authoritative quality gate** | Slower feedback loop |

**The practical recommendation**: CI is your source of truth, everything else is developer convenience. Design your system assuming some developers (and AI agents) will bypass local hooks—branch protection with required CI checks is the only reliable enforcement mechanism.

For Claude Code specifically, configure a `PreToolUse` hook that triggers pre-commit checks when the agent attempts `git commit`:

```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "if echo \"$CLAUDE_TOOL_INPUT\" | jq -r '.command' | grep -q '^git commit'; then pre-commit run --all-files; fi",
        "timeout": 180
      }]
    }]
  }
}
```

## Containerize integration tests, run fast linters locally

The performance overhead of containerization makes blanket Docker-based linting counterproductive. Modern linters like **Ruff** execute in milliseconds—containerizing them adds seconds of overhead that encourages developers to skip checks entirely.

The optimal split follows execution time: anything under **2 seconds** runs locally; anything slower or requiring complex dependencies runs containerized.

**Run locally** (no container overhead):
- Fast linters: Ruff (10-100x faster than Flake8), ESLint, oxlint
- Formatters: Prettier, Black, Ruff format
- Pre-commit framework hooks for these tools

**Containerize** (reproducibility matters more than speed):
- Integration tests with real databases (use Testcontainers)
- Complex multi-language builds
- Full CI validation runs
- **AI agent sandboxing**—always isolate autonomous AI execution

For reproducibility, structure Dockerfiles with dependencies before code so cache invalidation is minimized:

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./          # Changes rarely - cached
RUN npm ci --only=production   # Cached until package.json changes
COPY . .                       # Changes frequently - always rebuilt
```

## Dev containers provide the safest Claude Code execution environment

Running Claude Code **inside a dev container** is the recommended configuration for teams that want autonomous AI operation. The container provides filesystem and network isolation, enabling safe use of Claude Code's `--dangerously-skip-permissions` flag for unattended operation.

Anthropic's official documentation states that container isolation allows bypassing permission prompts because the AI can only affect files in the mounted workspace, not your host system, SSH keys, or shell configuration.

A minimal secure dev container for Claude Code:

```json
{
  "name": "Claude Code Environment",
  "build": { "dockerfile": "Dockerfile" },
  "features": {
    "ghcr.io/devcontainers/features/node:1": { "version": "20" },
    "ghcr.io/devcontainers/features/github-cli:1": {}
  },
  "mounts": [
    "source=${localEnv:HOME}/.claude,target=/home/vscode/.claude,type=bind"
  ],
  "postCreateCommand": "npm install -g @anthropic-ai/claude-code",
  "remoteUser": "vscode"
}
```

The `~/.claude` mount persists API credentials across container rebuilds. For maximum security, Anthropic's reference devcontainer includes a firewall initialization script that whitelists only required domains (npm registry, GitHub, Claude API) while blocking all other outbound traffic.

**What to standardize in devcontainer.json** (commit to repo): language runtimes, linters, formatters, test runners, required VS Code extensions, git hooks setup.

**What to leave flexible** (personal dotfiles): editor themes, keybindings, shell customizations.

GitHub's guidance is explicit: "Things like linters are good to standardize on... things like user interface decorators or themes are personal choices."

## Teams should commit shared configuration, not individual preferences

Team standardization for AI-assisted development requires clear separation between shared infrastructure and personal preferences.

**Commit to the repository**:
- `.claude/settings.json` — Team hook configurations, permission rules
- `CLAUDE.md` or `.claude/CLAUDE.md` — Project context, architecture, coding standards
- `.pre-commit-config.yaml` — Shared pre-commit hook definitions
- `devcontainer.json` — Standardized development environment
- CI/CD workflow definitions

**Keep individual** (gitignored):
- `.claude/settings.local.json` — Personal Claude Code preferences
- `CLAUDE.local.md` — Personal sandbox URLs, test credentials
- User-level `~/.claude/settings.json` — Personal defaults

For **CLAUDE.md**, keep instructions concise—LLMs have limited instruction-following capacity. Specify concrete rules ("use 2-space indentation") rather than vague directives ("format code properly"). Reference external documentation with imports (`@docs/authentication.md`) rather than duplicating content.

**Enterprise teams** can deploy `managed-settings.json` via MDM tools to enforce organization-wide policies that cannot be overridden:

```json
{
  "permissions": {
    "deny": ["Read(**/.env)", "Bash(sudo:*)", "Bash(rm -rf:*)"]
  },
  "allowManagedHooksOnly": true
}
```

The `allowManagedHooksOnly: true` setting blocks all user, project, and plugin hooks—only organization-approved managed hooks execute.

## Security requires both filesystem and network isolation

Research indicates **40-62% of AI-generated code contains security vulnerabilities**, including SQL injection, XSS, hardcoded secrets, and cryptographic weaknesses. Additionally, **19.7% of AI-suggested package dependencies don't exist**—attackers can register malicious packages matching these hallucinated names ("slopsquatting").

Anthropic's sandboxing architecture (released October 2025) addresses these risks through dual boundaries:

1. **Filesystem isolation**: Read/write restricted to the current working directory; blocks modification of system files (`~/.bashrc`, `~/.ssh`)
2. **Network isolation**: All traffic routed through a proxy; only approved domains permitted

The documentation emphasizes that **both are required**—without network isolation, a compromised agent could exfiltrate SSH keys; without filesystem isolation, the agent could modify system configuration to bypass network restrictions.

For running tests and linters on AI-generated code:
- Use ephemeral containers for each CI job
- Apply network isolation by default
- Scan code with SAST tools before any execution
- Verify dependencies against known-good lists before installation
- Never run privileged containers

Claude Code's native sandbox mode achieves an **84% reduction in permission prompts** while maintaining security. Enable it with the `/sandbox` command or configure boundaries in settings:

```json
{
  "permissions": {
    "allow": ["Bash(npm test)", "Bash(npm run lint)"],
    "deny": ["WebFetch", "Bash(curl:*)", "Read(./secrets/**)"]
  }
}
```

The defense-in-depth strategy layers protection: code review catches logic flaws, SAST catches vulnerability patterns, container isolation limits blast radius, network restrictions prevent exfiltration, and monitoring detects anomalies.

## Conclusion

Enforcing code quality with agentic AI assistants requires rethinking traditional workflows. The core insight is that **verification infrastructure must scale with AI autonomy**—granting an AI agent permission to write code autonomously without robust quality gates creates technical debt faster than it creates value.

The recommended architecture layers Claude Code hooks (catching issues during AI operation), fast local pre-commit hooks (immediate feedback), containerized dev environments (isolation and reproducibility), and authoritative CI/CD pipelines (guaranteed enforcement). Security demands both filesystem and network sandboxing—Claude Code's native sandbox mode provides this with minimal friction.

Teams should commit shared configuration (hooks, CLAUDE.md, devcontainer.json) to repositories while allowing personal preferences to remain individual. Enterprise deployments can enforce organization-wide policies through managed settings that cannot be overridden. The goal is standardized quality enforcement that applies equally to human and AI-generated code, treating every AI contribution with the same scrutiny you'd apply to code from a new team member.