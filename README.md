# Truesight MCP Skills

Agent skills and Cursor plugin for the [Truesight MCP](https://truesight.goodeyelabs.com/docs/mcp-integration). Step-by-step workflow playbooks for scoring inputs, building live evaluations, error analysis, and the review loop.

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

## Install skills manually

If you installed via Claude Marketplace above, you can skip this. Use the manual commands below to install skill files directly (works with Claude Code, Cursor, and other clients).

### Project-level (recommended for team workflows)

```bash
BASE=https://raw.githubusercontent.com/Goodeye-Labs/truesight-mcp-skills/main/skills
for skill in truesight-workflows evaluate-trace error-analysis generate-synthetic-data review-and-promote-traces bootstrap-template-evaluation create-evaluation eval-audit build-review-interface; do
  curl -fsSL "$BASE/$skill/SKILL.md" -o ".claude/skills/$skill/SKILL.md" --create-dirs
done
```

### Global (available in all projects)

```bash
BASE=https://raw.githubusercontent.com/Goodeye-Labs/truesight-mcp-skills/main/skills
for skill in truesight-workflows evaluate-trace error-analysis generate-synthetic-data review-and-promote-traces bootstrap-template-evaluation create-evaluation eval-audit build-review-interface; do
  curl -fsSL "$BASE/$skill/SKILL.md" -o "$HOME/.claude/skills/$skill/SKILL.md" --create-dirs
done
```

## Skills

| Skill | What it does |
|-------|-------------|
| [`truesight-workflows`](./skills/truesight-workflows/SKILL.md) | Strict orchestrator that routes to the correct Truesight MCP skill based on user intent |
| [`generate-synthetic-data`](./skills/generate-synthetic-data/SKILL.md) | Create diverse synthetic test inputs using dimension-based variation for evaluation bootstrapping |
| [`error-analysis`](./skills/error-analysis/SKILL.md) | Analyze traces in datasets, label failure modes, consolidate categories, and prioritize fixes |
| [`bootstrap-template-evaluation`](./skills/bootstrap-template-evaluation/SKILL.md) | Provision a template dataset and deploy a live evaluation quickly |
| [`create-evaluation`](./skills/create-evaluation/SKILL.md) | Scope, build, and deploy new custom live evaluations from scratch |
| [`evaluate-trace`](./skills/evaluate-trace/SKILL.md) | Evaluate one or more inputs against an existing live evaluation, with optional handoff to review flows |
| [`review-and-promote-traces`](./skills/review-and-promote-traces/SKILL.md) | Review flagged traces, submit judgments, and promote judged items back to datasets |
| [`eval-audit`](./skills/eval-audit/SKILL.md) | Audit evaluation workflow maturity and return severity-ranked findings with next-skill actions |
| [`build-review-interface`](./skills/build-review-interface/SKILL.md) | Build a custom web annotation interface when Truesight web UI is not the preferred review surface |

## Usage

Once the MCP is connected and skills are installed, your AI assistant will automatically pick up the right skill based on what you ask:

- **"I need help choosing the right Truesight workflow"**: triggers `truesight-workflows`
- **"Generate synthetic test data for my RAG pipeline"**: triggers `generate-synthetic-data`
- **"Analyze the errors in my dataset"**: triggers `error-analysis`
- **"Bootstrap a live eval from a template"**: triggers `bootstrap-template-evaluation`
- **"Create an evaluation for response quality"**: triggers `create-evaluation`
- **"Evaluate these traces against my live eval"**: triggers `evaluate-trace`
- **"Review and promote these flagged results"**: triggers `review-and-promote-traces`
- **"Audit my eval setup and tell me what is missing"**: triggers `eval-audit`
- **"Help me build a custom annotation interface for trace review"**: triggers `build-review-interface`

## Prerequisites

Some skills (like `generate-synthetic-data` and `build-review-interface`) work without any Truesight account. For skills that use the Truesight MCP, you need a free [Truesight](https://truesight.goodeyelabs.com) account. When prompted, sign in to authorize access. All tools are available based on your account permissions.

**Want more control over permissions?** You can also connect using a [Platform API Key](https://truesight.goodeyelabs.com/docs/platform-api-keys) instead. See [Connecting with an API key](#connecting-with-an-api-key) below.

## Connect the MCP

### Claude.ai and Claude Desktop

Claude.ai and Claude Desktop share the same connectors, so you only need to set this up once.

1. Go to [**Customize > Connectors**](https://claude.ai/settings/connectors?modal=add-custom-connector)
2. Click **Add custom connector**
3. Enter:
   - **Name:** Truesight
   - **URL:** `https://api.truesight.goodeyelabs.com/mcp/`
4. Click **Add**
5. When prompted, sign in with your Truesight account to authorize access
6. Enable the connector in any conversation via the **+** button

### ChatGPT

1. Go to [**Settings > Apps > Advanced settings**](https://chatgpt.com/#settings/Connectors/Advanced)
2. Click **Create app**
3. Enter:
   - **Name:** Truesight
   - **MCP Server URL:** `https://api.truesight.goodeyelabs.com/mcp/`
   - **Authentication:** OAuth
4. Check the confirmation box and click **Create**
5. When prompted, sign in with your Truesight account to authorize access

### Cursor

Add to your project's `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "truesight": {
      "url": "https://api.truesight.goodeyelabs.com/mcp/"
    }
  }
}
```

Restart Cursor, then sign in with your Truesight account when prompted.

### Claude Code

```bash
claude mcp add --transport http truesight \
  https://api.truesight.goodeyelabs.com/mcp/
```

Add `--scope project` before `truesight` to scope it to a single project instead of your user config.

### VS Code (GitHub Copilot)

Requires VS Code 1.99+ with GitHub Copilot enabled. Add to `.vscode/settings.json` (or use **MCP: Add Server** from the Command Palette):

```json
{
  "mcp": {
    "servers": {
      "truesight": {
        "url": "https://api.truesight.goodeyelabs.com/mcp/"
      }
    }
  }
}
```

Sign in with your Truesight account when prompted.

### Windsurf

Open **Settings** > **Cascade** > **MCP Servers** > **View raw config** and add:

```json
{
  "mcpServers": {
    "truesight": {
      "serverUrl": "https://api.truesight.goodeyelabs.com/mcp/",
      "disabled": false
    }
  }
}
```

Save and click **Refresh** (or restart Windsurf). Sign in with your Truesight account when prompted.

## Connecting with an API key

If you prefer fine-grained control over which tools your AI assistant can use, connect with a [Platform API Key](https://truesight.goodeyelabs.com/docs/platform-api-keys). The scopes you assign to the key determine which tools are available.

1. Go to **Settings** in Truesight
2. Click **Create Key** in the Truesight API Keys section
3. Enter a name (e.g., "Claude", "Cursor", or "VS Code")
4. Select the scopes you need (or select all)
5. Click **Create** and copy the key immediately (it will not be shown again in full)

Then add the `headers` block to your MCP config. Examples for each client:

**Cursor** (`.cursor/mcp.json`):

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

**Claude Code:**

```bash
claude mcp add --transport http truesight \
  https://api.truesight.goodeyelabs.com/mcp/ \
  --header "Authorization: Bearer YOUR_API_KEY_HERE"
```

**Claude Desktop** (`claude_desktop_config.json`):

Claude Desktop requires [Node.js](https://nodejs.org/) and the [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) bridge (auto-installed via `npx`).

Open Claude > **Settings** > **Developer** > **Edit Config** and add:

```json
{
  "mcpServers": {
    "truesight": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://api.truesight.goodeyelabs.com/mcp/",
        "--header",
        "Authorization:${TRUESIGHT_KEY}"
      ],
      "env": {
        "TRUESIGHT_KEY": "Bearer YOUR_API_KEY_HERE"
      }
    }
  }
}
```

**VS Code** (`.vscode/settings.json`):

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

**Windsurf:**

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

Replace `YOUR_API_KEY_HERE` with your actual platform API key and restart your client.

For the full list of available tools, scopes, and troubleshooting tips, see the [MCP Integration docs](https://truesight.goodeyelabs.com/docs/mcp-integration).

## License

MIT
