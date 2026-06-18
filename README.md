# Baselight skills

[Agent Skills](https://agentskills.io/) that provides access to [Baselight](https://baselight.ai) — a data platform with thousands of queryable public datasets covering crypto/blockchain, finance, demographics, climate, healthcare, sports, and more.

## What it does

When you ask a data question, your agent will search Baselight's catalog, inspect the relevant table schema, and query it directly using DuckDB SQL — returning actual data instead of web search snippets.

Example triggers:
- "Compare GDP across countries"
- "Show me happiness score trends"
- "Query `@owid.happiness.owid_happiness_2`"

## Installation

### Claude Code plugin

This repository doubles as a [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces). Installing the plugin gives you the skill **and** a connection to the remote Baselight MCP server in one step:

```
/plugin marketplace add baselightdb/skills
/plugin install baselight@baselight
```

The plugin bundles the same `skills/baselight` skill shipped in this repo (no files are duplicated) plus an MCP entry for `https://api.baselight.app/mcp`. See [Setup](#setup) for the API key.

### npx installation

```
npx skills add baselightdb/skills --skill baselight -g
```

### Manual installation

Copy the skill into your agent's skills directory. Example for Claude Code:

```bash
cp -r skills/baselight ~/.claude/skills/baselight
```

It should pick it up automatically on the next session.

## Setup

You need a Baselight API key:

1. Sign up at [baselight.app](https://baselight.app)
2. Go to **Account Settings → Integrations → Generate New API Key**
3. Save it to `~/.baselight/credentials`:

```bash
mkdir -p ~/.baselight
echo 'BASELIGHT_API_KEY=your-key-here' >> ~/.baselight/credentials
chmod 600 ~/.baselight/credentials
```

When you install the **plugin**, Claude Code prompts for the API key and stores it in the system keychain. The MCP server uses it directly, and a `SessionStart` hook mirrors it into `~/.baselight/credentials` (replacing only the `BASELIGHT_API_KEY` line and leaving anything else intact) so the bundled **skill** — which reads that file — uses the same key. One prompt covers both.

Leave the prompt blank to manage credentials yourself: the hook becomes a no-op, and both the skill and the MCP server fall back to the `BASELIGHT_API_KEY` environment variable, then `~/.baselight/credentials`.

> **Note:** entering the key at the prompt writes it to `~/.baselight/credentials` in plaintext (in addition to the keychain), because the skill's script reads it from there.

## Structure

```
.
├── .claude-plugin/
│   ├── marketplace.json        # Plugin marketplace catalog (this repo as its own marketplace)
│   └── plugin.json             # Plugin manifest: sources skills/ + the remote MCP server
└── skills/baselight/
    ├── SKILL.md                # Skill definition and instructions for Claude
    ├── scripts/
    │   └── baselight.py        # Python MCP client (no extra deps beyond requests)
    └── references/
        └── sql-patterns.md     # DuckDB SQL reference and examples
```

The plugin uses a marketplace-root source (`"source": "./"`), so the plugin and the skill share the existing `skills/` directory — the skill files are never duplicated.

## Requirements

Python 3 with `requests`:

```bash
pip install requests
```

## License

MIT
