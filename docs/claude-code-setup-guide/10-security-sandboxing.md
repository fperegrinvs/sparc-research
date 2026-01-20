# Part 10: Security and Sandboxing

## Overview

Research indicates **40-62% of AI-generated code contains security vulnerabilities**, including SQL injection, XSS, hardcoded secrets, and cryptographic weaknesses. Additionally, **19.7% of AI-suggested package dependencies don't exist**—attackers can register malicious packages matching these hallucinated names ("slopsquatting").

Robust security infrastructure is essential when granting AI agents autonomous coding capabilities.

## The Security Landscape

### AI-Generated Code Risks

1. **Common Vulnerabilities**:
   - SQL injection
   - Cross-site scripting (XSS)
   - Hardcoded secrets
   - Cryptographic weaknesses
   - Path traversal

2. **Dependency Risks**:
   - Hallucinated packages (don't exist)
   - Outdated vulnerable versions
   - Typosquatted packages

3. **Operational Risks**:
   - Unauthorized file system access
   - Network exfiltration
   - System modification

## Defense in Depth Strategy

### Layer 1: Filesystem Isolation

Restrict what the AI can read and write.

**Claude Code Permissions**:
```json
{
  "permissions": {
    "allow": [
      "Read(./src/**)",
      "Read(./tests/**)",
      "Write(./src/**)",
      "Write(./tests/**)"
    ],
    "deny": [
      "Read(**/.env)",
      "Read(./**/secrets/**)",
      "Read(~/.ssh/**)",
      "Write(~/.bashrc)",
      "Write(~/.zshrc)",
      "Bash(sudo:*)",
      "Bash(rm -rf:*)"
    ]
  }
}
```

**Key Protections**:
- Read/write restricted to project directories
- Blocks modification of system files
- Prevents access to secrets and credentials

### Layer 2: Network Isolation

Control what external resources can be accessed.

**Dev Container Network Rules**:
```json
{
  "runArgs": ["--network=host"],
  "postCreateCommand": "npm install -g @anthropic-ai/claude-code && ./setup-firewall.sh"
}
```

**Firewall Setup Script**:
```bash
#!/bin/bash
# setup-firewall.sh

# Allow only required domains
iptables -A OUTPUT -d registry.npmjs.org -j ACCEPT
iptables -A OUTPUT -d api.anthropic.com -j ACCEPT
iptables -A OUTPUT -d github.com -j ACCEPT
iptables -A OUTPUT -d *.githubusercontent.com -j ACCEPT

# Block everything else
iptables -A OUTPUT -j DROP
```

### Layer 3: Dev Container Sandboxing

Running Claude Code inside a dev container provides the safest execution environment.

**Minimal Secure Dev Container**:
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

**Benefits**:
- Filesystem and network isolation
- Safe use of `--dangerously-skip-permissions` flag
- AI can only affect files in the mounted workspace
- Protects host system, SSH keys, and shell configuration

### Layer 4: Claude Code Hooks

Intercept AI actions at critical moments.

**Hook Configuration**:
```json
{
  "hooks": {
    "PreToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "if echo \"$CLAUDE_TOOL_INPUT\" | jq -r '.command' | grep -q '^git commit'; then bun run gate:commit; fi",
        "timeout": 180
      }]
    }],
    "PostToolUse": [{
      "matcher": "Write",
      "hooks": [{
        "type": "command",
        "command": "bun run lint --fix ${CLAUDE_FILE_PATH}",
        "timeout": 30
      }]
    }]
  }
}
```

**Available Hook Events**:
- `PreToolUse` - Before tool execution (can block)
- `PostToolUse` - After tool execution (can modify)
- `PermissionRequest` - Auto-approve safe operations

### Layer 5: Git Hooks

Traditional enforcement that applies to all commits.

**Pre-commit Hook**:
```bash
#!/bin/sh
# .husky/pre-commit

echo "🔒 Running security checks..."

# Check for secrets
if git diff --cached --name-only | xargs grep -l -E "(password|secret|api_key|token)\s*=" 2>/dev/null; then
  echo "❌ Potential secrets detected in staged files"
  exit 1
fi

# Run quality gates
bun run gate:commit

echo "✅ Pre-commit checks passed"
```

**Pre-push Hook**:
```bash
#!/bin/sh
# .husky/pre-push

echo "🔒 Running security scan..."
bun audit
bun run lint:security

echo "✅ Pre-push checks passed"
```

### Layer 6: CI/CD Enforcement

The authoritative quality gate that cannot be bypassed.

```yaml
# .github/workflows/security.yml
name: Security Scan

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Dependency audit
        run: bun audit

      - name: SAST scan
        uses: github/codeql-action/analyze@v2

      - name: Secret scanning
        uses: trufflesecurity/trufflehog@v3
        with:
          path: ./

      - name: License compliance
        run: npx license-checker --failOn "GPL"
```

## Claude Code Sandbox Mode

Claude Code provides native sandbox mode for reduced friction while maintaining security.

**Enable via Command**:
```
/sandbox
```

**Configure Boundaries**:
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
  }
}
```

**Benefits**:
- 84% reduction in permission prompts
- Maintains security boundaries
- Enables more autonomous operation

## Enterprise Configuration

For organizations, use managed settings that cannot be overridden.

**Deploy via MDM** (`managed-settings.json`):
```json
{
  "permissions": {
    "deny": [
      "Read(**/.env)",
      "Bash(sudo:*)",
      "Bash(rm -rf:*)",
      "WebFetch(*)"
    ]
  },
  "allowManagedHooksOnly": true
}
```

**`allowManagedHooksOnly: true`** blocks all user, project, and plugin hooks—only organization-approved managed hooks execute.

## What to Standardize vs. Personalize

### Team Standardization (Commit to Repo)
- `.claude/settings.json` - Hook configurations, permission rules
- `CLAUDE.md` - Project context, coding standards
- `.pre-commit-config.yaml` - Shared hook definitions
- `devcontainer.json` - Development environment
- CI/CD workflow definitions

### Individual Preferences (Gitignored)
- `.claude/settings.local.json` - Personal preferences
- `CLAUDE.local.md` - Personal sandbox URLs, test credentials
- User-level `~/.claude/settings.json` - Personal defaults

## Security Checklist

Before enabling autonomous AI operation:

- [ ] Filesystem permissions configured
- [ ] Network isolation enabled (for sensitive projects)
- [ ] Dev container setup for sandboxing
- [ ] Claude Code hooks configured
- [ ] Git pre-commit hook active
- [ ] CI security scans enabled
- [ ] Dependency auditing configured
- [ ] Secret scanning active
- [ ] Branch protection with required checks

## The Key Insight

**Verification infrastructure must scale with AI autonomy.** Granting an AI agent permission to write code autonomously without robust quality gates creates technical debt faster than it creates value.

The recommended architecture layers:
1. Claude Code hooks (catching issues during AI operation)
2. Fast local pre-commit hooks (immediate feedback)
3. Containerized dev environments (isolation and reproducibility)
4. Authoritative CI/CD pipelines (guaranteed enforcement)

Security demands both filesystem AND network sandboxing—Claude Code's native sandbox mode provides this with minimal friction.
