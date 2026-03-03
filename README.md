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

[**Add to Cursor**](cursor://anysphere.cursor-deeplink/mcp/install?name=truesight&config=eyJ1cmwiOiJodHRwczovL2FwaS50cnVlc2lnaHQuZ29vZGV5ZWxhYnMuY29tL21jcC8iLCJoZWFkZXJzIjp7IkF1dGhvcml6YXRpb24iOiJCZWFyZXIgWU9VUl9BUElfS0VZX0hFUkUifX0=)

Click the link, then replace `YOUR_API_KEY_HERE` with your platform API key in `~/.cursor/mcp.json`.

Or add it manually to `.cursor/mcp.json`:

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

### VS Code (GitHub Copilot)

[**Install in VS Code**](vscode:mcp/install?%7B%22name%22%3A%22truesight%22%2C%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fapi.truesight.goodeyelabs.com%2Fmcp%2F%22%2C%22headers%22%3A%7B%22Authorization%22%3A%22Bearer%20YOUR_API_KEY_HERE%22%7D%7D) · [**Install in VS Code Insiders**](vscode-insiders:mcp/install?%7B%22name%22%3A%22truesight%22%2C%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fapi.truesight.goodeyelabs.com%2Fmcp%2F%22%2C%22headers%22%3A%7B%22Authorization%22%3A%22Bearer%20YOUR_API_KEY_HERE%22%7D%7D)

Click a link, then replace `YOUR_API_KEY_HERE` with your platform API key. Requires VS Code 1.99+ with GitHub Copilot enabled.

Or add it manually to `.vscode/settings.json`:

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
