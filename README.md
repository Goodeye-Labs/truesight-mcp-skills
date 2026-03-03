# Truesight MCP Skills

Agent skills and Cursor plugin for the [Truesight MCP](https://truesight.goodeyelabs.com/docs/mcp-integration) - step-by-step workflow playbooks for scoring inputs, building live evaluations, error analysis, and the review loop.

Works with Claude Code, Cursor, and any client that supports the [agent skills standard](https://agentskills.io/specification).

## Install as a Claude plugin

In Claude Code, run:

```bash
# Step 1: Register this repository as a marketplace
/plugin marketplace add Goodeye-Labs/truesight-mcp-skills

# Step 2: Install the plugin
/plugin install truesight@goodeye-labs-truesight
```

To upgrade:

```bash
/plugin update truesight@goodeye-labs-truesight
```

## Prerequisites

1. A [Truesight](https://truesight.goodeyelabs.com) account
2. A platform API key: go to **Settings** in Truesight, click **Create Key**, select your scopes, and copy the key

## Connect the MCP

### Cursor

Add to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "truesight": {
      "url": "https://api.truesight.goodeyelabs.com/mcp/",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY_HERE"
      }
    }
  }
}
```

Replace `YOUR_API_KEY_HERE` with your platform API key, then restart Cursor.

### VS Code (GitHub Copilot)

Requires VS Code 1.99+ with GitHub Copilot enabled. Add to `.vscode/settings.json` (or use **MCP: Add Server** from the Command Palette):

```json
{
  "mcp": {
    "servers": {
      "truesight": {
        "url": "https://api.truesight.goodeyelabs.com/mcp/",
        "headers": {
          "Authorization": "Bearer YOUR_API_KEY_HERE"
        }
      }
    }
  }
}
```

Replace `YOUR_API_KEY_HERE` with your platform API key.

### Claude Code

```bash
claude mcp add --transport http truesight \
  https://api.truesight.goodeyelabs.com/mcp/ \
  --header "Authorization: Bearer YOUR_API_KEY_HERE"
```

Add `--scope project` before `truesight` to scope it to a single project instead of your user config.

### Claude Desktop

Open Claude → **Settings** → **Developer** → **Edit Config** and add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "truesight": {
      "url": "https://api.truesight.goodeyelabs.com/mcp/",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY_HERE"
      }
    }
  }
}
```

Save and restart Claude.

### Windsurf

Open **Settings** → **Cascade** → **MCP Servers** → **View raw config** and add:

```json
{
  "mcpServers": {
    "truesight": {
      "serverUrl": "https://api.truesight.goodeyelabs.com/mcp/",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY_HERE"
      },
      "disabled": false
    }
  }
}
```

Save and click **Refresh** (or restart Windsurf).

## Skills

| Skill | What it does |
|-------|-------------|
| [`truesight-mcp`](./skills/truesight-mcp/SKILL.md) | Decision tree + playbooks for scoring inputs (A), error analysis (B), and the review-and-promote loop (C) |
| [`create-evaluation`](./skills/create-evaluation/SKILL.md) | Scope, build, and deploy a new live evaluation from scratch. Includes `judgment_configs` schema reference and mandatory `api_key` storage guidance |

### Install skills manually

#### Project-level (recommended for team workflows)

```bash
curl -fsSL https://raw.githubusercontent.com/Goodeye-Labs/truesight-mcp-skills/main/skills/truesight-mcp/SKILL.md \
  -o .claude/skills/truesight-mcp/SKILL.md --create-dirs

curl -fsSL https://raw.githubusercontent.com/Goodeye-Labs/truesight-mcp-skills/main/skills/create-evaluation/SKILL.md \
  -o .claude/skills/create-evaluation/SKILL.md --create-dirs
```

#### Global (available in all projects)

```bash
curl -fsSL https://raw.githubusercontent.com/Goodeye-Labs/truesight-mcp-skills/main/skills/truesight-mcp/SKILL.md \
  -o ~/.claude/skills/truesight-mcp/SKILL.md --create-dirs

curl -fsSL https://raw.githubusercontent.com/Goodeye-Labs/truesight-mcp-skills/main/skills/create-evaluation/SKILL.md \
  -o ~/.claude/skills/create-evaluation/SKILL.md --create-dirs
```

## Usage

Once the MCP is connected and skills are installed, your AI assistant will automatically pick up the right skill based on what you ask:

- **"Score these inputs against my quality eval"**: triggers `truesight-mcp`, Workflow A
- **"Analyze the errors in my dataset"**: triggers `truesight-mcp`, Workflow B
- **"Review and promote these flagged results"**: triggers `truesight-mcp`, Workflow C
- **"Create an evaluation for response quality"**: triggers `create-evaluation`

## License

MIT
