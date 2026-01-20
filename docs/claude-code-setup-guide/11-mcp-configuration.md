# Part 11: MCP Server Configuration

## Overview

The Model Context Protocol (MCP) allows Claude Code to connect to external tools and services. Configuration is done through `.mcp.json` files at the project root.

## Basic MCP Configuration

### File Location

```
project-root/
├── .mcp.json           # MCP server configuration
├── CLAUDE.md
└── .claude/
```

### Configuration Structure

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "mcpServers": {
    "server-name": {
      "command": "command-to-run",
      "args": ["arg1", "arg2"],
      "env": {
        "ENV_VAR": "value"
      }
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

## Default Configuration

A minimal configuration that enables core Claude Code tools:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "mcpServers": {
    "notes": {
      "This file configures MCP servers for Claude Code": true,
      "Add servers as needed for your project": true
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

## Adding MCP Servers

### Database Server Example

```json
{
  "mcpServers": {
    "postgres": {
      "command": "mcp-server-postgres",
      "args": ["--connection-string", "${POSTGRES_URL}"],
      "env": {
        "POSTGRES_URL": "postgresql://localhost:5432/mydb"
      }
    }
  }
}
```

### File System Server Example

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "mcp-server-filesystem",
      "args": ["--allowed-directories", "./src", "./tests"],
      "env": {}
    }
  }
}
```

### Custom Server Example

```json
{
  "mcpServers": {
    "custom-api": {
      "command": "node",
      "args": ["./tools/mcp-server.js"],
      "env": {
        "API_KEY": "${API_KEY}",
        "API_BASE_URL": "https://api.example.com"
      }
    }
  }
}
```

## Tool Permissions

### Allowed Tools List

Control which tools Claude Code can use:

```json
{
  "settings": {
    "allowedTools": [
      "Read",      // Read file contents
      "Write",     // Create/overwrite files
      "Edit",      // Modify existing files
      "Glob",      // Find files by pattern
      "Grep",      // Search file contents
      "Bash",      // Execute shell commands
      "WebFetch",  // Fetch web content
      "Task",      // Spawn sub-agents
      "BatchTool"  // Parallel operations
    ]
  }
}
```

### Restricting Tools

For more controlled environments:

```json
{
  "settings": {
    "allowedTools": [
      "Read",
      "Glob",
      "Grep"
    ]
  }
}
```

This configuration allows only read operations.

## Environment Variables

### Using Environment Variables

Environment variables can be referenced in configuration:

```json
{
  "mcpServers": {
    "api-server": {
      "command": "mcp-api-server",
      "env": {
        "API_KEY": "${API_KEY}",
        "DATABASE_URL": "${DATABASE_URL}",
        "NODE_ENV": "development"
      }
    }
  }
}
```

### Setting Environment Variables

Create a `.env` file (gitignored):

```bash
# .env
API_KEY=your-api-key-here
DATABASE_URL=postgresql://localhost:5432/mydb
```

Or set them in your shell:

```bash
export API_KEY="your-api-key-here"
```

## Security Considerations

### Secrets Management

**Never commit secrets** to `.mcp.json`. Use environment variables:

```json
{
  "mcpServers": {
    "secure-service": {
      "command": "mcp-secure",
      "env": {
        "SECRET_KEY": "${SECRET_KEY}"
      }
    }
  }
}
```

### Tool Restrictions

Limit available tools for sensitive projects:

```json
{
  "settings": {
    "allowedTools": [
      "Read",
      "Glob",
      "Grep"
    ],
    "deniedTools": [
      "Bash",
      "WebFetch"
    ]
  }
}
```

### Network Restrictions

Configure servers with network limitations:

```json
{
  "mcpServers": {
    "local-only": {
      "command": "mcp-server",
      "args": ["--bind", "127.0.0.1", "--no-external"]
    }
  }
}
```

## Project-Specific Configurations

### Development Configuration

```json
{
  "mcpServers": {
    "dev-db": {
      "command": "mcp-server-sqlite",
      "args": ["./dev.db"]
    },
    "mock-api": {
      "command": "mcp-mock-server",
      "args": ["./mocks"]
    }
  },
  "settings": {
    "allowedTools": ["Read", "Write", "Edit", "Glob", "Grep", "Bash"]
  }
}
```

### Production-Like Configuration

```json
{
  "mcpServers": {
    "prod-db": {
      "command": "mcp-server-postgres",
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    }
  },
  "settings": {
    "allowedTools": ["Read", "Glob", "Grep"]
  }
}
```

## Troubleshooting

### Server Not Starting

1. Check the command exists: `which mcp-server-name`
2. Verify environment variables are set
3. Check for error messages in Claude Code output

### Permission Issues

1. Verify `allowedTools` includes necessary tools
2. Check file system permissions
3. Ensure network access is available

### Configuration Validation

Validate your `.mcp.json` against the schema:

```bash
# Using ajv or similar validator
npx ajv validate -s mcp-schema.json -d .mcp.json
```

## Best Practices

1. **Start minimal**: Only add servers you need
2. **Use environment variables**: Never hardcode secrets
3. **Limit tools**: Only allow necessary operations
4. **Document servers**: Add comments explaining purpose
5. **Version control**: Commit `.mcp.json`, gitignore secrets
6. **Test locally**: Verify servers work before deployment
