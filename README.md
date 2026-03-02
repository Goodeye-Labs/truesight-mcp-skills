# Truesight MCP Skills

Agent skills and Cursor plugin for the [Truesight MCP](https://truesight.goodeyelabs.com/docs/mcp-integration) -- step-by-step workflow playbooks for scoring inputs, building live evaluations, error analysis, and the review loop.

Works with Claude Code, Cursor, and any client that supports the [agent skills standard](https://agentskills.io/specification).

## Skills

| Skill | What it does |
|-------|-------------|
| [`truesight-mcp`](./skills/truesight-mcp/SKILL.md) | Decision tree + playbooks for scoring inputs (A), error analysis (B), and the review-and-promote loop (C) |
| [`create-evaluation`](./skills/create-evaluation/SKILL.md) | Scope, build, and deploy a new live evaluation from scratch -- includes `judgment_configs` schema reference and mandatory `api_key` storage guidance |

## Prerequisites

1. A [Truesight](https://truesight.goodeyelabs.com) account
2. The [Truesight MCP configured](https://truesight.goodeyelabs.com/docs/mcp-integration) in your AI client

## Cursor Plugin

This repo is a [Cursor Marketplace](https://cursor.com/marketplace) plugin. Installing it via the marketplace adds the Truesight MCP server and both skills to Cursor in one step.

After installing, replace `YOUR_API_KEY_HERE` in `~/.cursor/mcp.json` with your [platform API key](https://truesight.goodeyelabs.com/docs/platform-api-keys).

## Install skills manually

### Project-level (recommended for team workflows)

```bash
curl -fsSL https://raw.githubusercontent.com/Goodeye-Labs/truesight-mcp-skills/main/skills/truesight-mcp/SKILL.md \
  -o .claude/skills/truesight-mcp/SKILL.md --create-dirs

curl -fsSL https://raw.githubusercontent.com/Goodeye-Labs/truesight-mcp-skills/main/skills/create-evaluation/SKILL.md \
  -o .claude/skills/create-evaluation/SKILL.md --create-dirs
```

### Global (available in all projects)

```bash
curl -fsSL https://raw.githubusercontent.com/Goodeye-Labs/truesight-mcp-skills/main/skills/truesight-mcp/SKILL.md \
  -o ~/.claude/skills/truesight-mcp/SKILL.md --create-dirs

curl -fsSL https://raw.githubusercontent.com/Goodeye-Labs/truesight-mcp-skills/main/skills/create-evaluation/SKILL.md \
  -o ~/.claude/skills/create-evaluation/SKILL.md --create-dirs
```

## Usage

Once installed, your AI assistant will automatically pick up the right skill based on what you ask:

- **"Score these inputs against my quality eval"** -- triggers `truesight-mcp`, Workflow A
- **"Analyze the errors in my dataset"** -- triggers `truesight-mcp`, Workflow B
- **"Review and promote these flagged results"** -- triggers `truesight-mcp`, Workflow C
- **"Create an evaluation for response quality"** -- triggers `create-evaluation`

## License

MIT
