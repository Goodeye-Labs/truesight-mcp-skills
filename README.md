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

This installs the Truesight plugin and its MCP skills in Claude, including `truesight-workflows`, `create-evaluation`, and companion workflow skills.

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
| [`truesight-workflows`](./skills/truesight-workflows/SKILL.md) | Strict orchestrator that routes to the correct Truesight MCP skill based on user intent |
| [`evaluate-trace`](./skills/evaluate-trace/SKILL.md) | Evaluate one or more inputs against an existing live evaluation, with optional handoff to review flows |
| [`error-analysis`](./skills/error-analysis/SKILL.md) | Analyze traces in datasets, label failure modes, consolidate categories, and prioritize fixes |
| [`review-and-promote-traces`](./skills/review-and-promote-traces/SKILL.md) | Review flagged traces, submit judgments, and promote judged items back to datasets |
| [`bootstrap-template-evaluation`](./skills/bootstrap-template-evaluation/SKILL.md) | Provision a template dataset and deploy a live evaluation quickly |
| [`create-evaluation`](./skills/create-evaluation/SKILL.md) | Scope, build, and deploy new custom live evaluations from scratch |
| [`eval-audit`](./skills/eval-audit/SKILL.md) | Audit evaluation workflow maturity and return severity-ranked findings with next-skill actions |
| [`build-review-interface`](./skills/build-review-interface/SKILL.md) | Build a custom web annotation interface when Truesight web UI is not the preferred review surface |

### Install skills manually

If you installed via Claude Marketplace above, you can skip manual skill installation. Use the manual commands below only when you want to install skill files directly.

#### Project-level (recommended for team workflows)

```bash
BASE=https://raw.githubusercontent.com/Goodeye-Labs/truesight-mcp-skills/main/skills
for skill in truesight-workflows evaluate-trace error-analysis review-and-promote-traces bootstrap-template-evaluation create-evaluation eval-audit build-review-interface; do
  curl -fsSL "$BASE/$skill/SKILL.md" -o ".claude/skills/$skill/SKILL.md" --create-dirs
done
```

#### Global (available in all projects)

```bash
BASE=https://raw.githubusercontent.com/Goodeye-Labs/truesight-mcp-skills/main/skills
for skill in truesight-workflows evaluate-trace error-analysis review-and-promote-traces bootstrap-template-evaluation create-evaluation eval-audit build-review-interface; do
  curl -fsSL "$BASE/$skill/SKILL.md" -o "$HOME/.claude/skills/$skill/SKILL.md" --create-dirs
done
```

## Usage

Once the MCP is connected and skills are installed, your AI assistant will automatically pick up the right skill based on what you ask:

- **"I need help choosing the right Truesight workflow"**: triggers `truesight-workflows`
- **"Evaluate these traces against my live eval"**: triggers `evaluate-trace`
- **"Analyze the errors in my dataset"**: triggers `error-analysis`
- **"Review and promote these flagged results"**: triggers `review-and-promote-traces`
- **"Bootstrap a live eval from a template"**: triggers `bootstrap-template-evaluation`
- **"Create an evaluation for response quality"**: triggers `create-evaluation`
- **"Audit my eval setup and tell me what is missing"**: triggers `eval-audit`
- **"Help me build a custom annotation interface for trace review"**: triggers `build-review-interface`

## License

MIT
